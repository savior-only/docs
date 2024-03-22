

# 深入理解比特币：（2）Lightning 网络

## 概述

-   一种支付渠道系统：能够实现点对点的交易。
    
-   实质上是多重签名钱包。
    

## 支付通道

两方将 UTXO 锁到一个多签地址，然后链下协商最终 UTXO 分配方案，链下可以多次更改分配方案，这个操作称为支付通道内的支付。多重签名地址要求双方共同签名才能花费其中的资金。

一旦支付渠道被创建，Peer A 和 Peer B 可以在支付渠道内进行无限次的交易。每笔交易都会更新双方之间的余额状态，但只有当支付渠道关闭时，最终的余额状态才会被记录到区块链上。

支付通道的建立需要 P2P 的协商过程，两方中的任何一方都需要既作为 Server 又作为 Client，目前的事实标准是 [lightning/bolts: BOLT: Basis of Lightning Technology (Lightning Network Specifications)](https://github.com/lightning/bolts)

### 建立通道

根据 A、B 的公钥生成一个多签地址，钱会打到这个地址（这笔交易称为 funding transaction，入金交易）

#### CSV

入金交易可以设定一个 CSV 延迟。CSV 是 BIP68 引入的，允许用户一个序列号 `nSequence`，在该序列号之前，资金不能被花费。

#### Upfront shutdown

“Upfront shutdown”是指双方在创建通道时就要协商好关闭通道的规则。包括但不限于：

1.  **CSV 延迟**：双方同意在关闭通道时使用的 CSV 延迟时间。CSV 延迟确保了如果一方试图单方面关闭通道，另一方将有时间来响应并确保交易的正确执行。
    
2.  **惩罚机制**：双方同意在对方不合作时可以实施的惩罚措施。例如，如果一方在关闭通道时消失，另一方可以使用惩罚交易来拿回自己的资金，并对不合作方进行罚款。
    
3.  **关闭交易的格式**：双方同意在关闭通道时使用的交易格式，例如，是否使用简化的支付验证（SPV）证明或者部分签名比特币交易（PSBT）。
    

#### ZeroConf

ZeroConf 交易在广播到比特币网络后几乎立即就可以被闪电网络节点使用，而不需要等待常规的区块确认。

#### HTLC

HTLC（Hashed Time-Locked Contract，哈希时间锁定合约）允许用户在两个不同的区块链之间或者在不需要信任的双方之间安全地进行原子交换。

从原来上简单来说就是**用哈希验证的、带过期时间的**交易。

HTLC 的工作原理如下：

1.  **哈希锁定**：发起方创建一个哈希值，这个哈希值是对秘密信息的加密哈希。这个哈希值被用作锁定资金的条件。发起方将资金发送到一个特殊的地址，这个地址包含了一个条件：只有知道正确的秘密信息（可以生成上述哈希值的信息）的人才能解锁这些资金。
    
2.  **时间锁定**：资金被锁定了一段时间，如果在这段时间内没有提供正确的秘密信息，资金将自动退还给发起方。这个时间锁确保了如果交换失败，资金不会被永久锁定。
    
3.  **秘密信息的传输**：在哈希锁定的资金被发送之后，发起方通过一个安全的渠道将秘密信息传递给接收方。接收方使用这个秘密信息来解锁资金并取出。
    

#### 协议

在进行身份验证并初始化连接之后，可以开始建立通道。

建立通道有两种路径，这里介绍了一个传统版本，还有一个第二个版本（太复杂，不写了）。在 `init` 消息中协商了可以用于建立通道的协议。

这包括资金节点（出资者）发送一个 `open_channel` 消息，随后响应节点（受益者）发送 `accept_channel`。随着通道参数确定，出资者能够创建资金交易和两个版本的承诺交易，所述。然后，出资者通过 `funding_created` 消息发送资金输出的交易指针，以及受益者承诺交易版本的签名。一旦受益者学习了资金输出的交易指针，他就能够生成出资者承诺交易版本的签名并使用 `funding_signed` 消息发送过来。

一旦通道出资者收到 `funding_signed` 消息，他必须**将资金交易广播到比特币网络**。在发送 / 接收 `funding_signed` 消息后，双方都应等待为了资金交易能够进入区块链并达到指定的确认深度（确认次数），在双方都发送了 `channel_ready` 消息之后，通道就建立完成并可以开始正常操作。`channel_ready` 消息中包含将用于构建通道认证证明的信息。

```bash
        +-------+                              +-------+
        |       |--(1)---  open_channel  ----->|       |
        |       |<-(2)--  accept_channel  -----|       |
        |       |                              |       |
        |   A   |--(3)--  funding_created  --->|   B   |
        |       |<-(4)--  funding_signed  -----|       |
        |       |                              |       |
        |       |--(5)---  channel_ready  ---->|       |
        |       |<-(6)---  channel_ready  -----|       |
        +-------+                              +-------+

        - 这里节点 A 是 '出资方'，节点 B 是 '受益方'
```

如果在任何阶段失败，或者一方认为另一方提供的通道条款不适合，通道建立将失败。

我比较关心创建的交易是什么样子，让我们看看：

`lnd/funding/manager.go`:

```go
1
func makeFundingScript(channel *channeldb.OpenChannel) ([]byte, error) {
2
 // ...
3
	multiSigScript, err := input.GenMultiSigScript(
4
		localKey.SerializeCompressed(),
5
		remoteKey.SerializeCompressed(),
6
	)
```

输出脚本是一个 P2WSH，包括：

```bash
2 <pubkey1> <pubkey2> 2 OP_CHECKMULTISIG
```

-   `pubkey` 按照字典序升序排列。

这个余额状态的更新是通过一系列的“撤销交易”（revocation transactions）和“承诺交易”（commitment transactions）来实现的。在每笔交易中，一方会创建一个新的承诺交易，该交易将资金按新的余额分配给双方，然后将自己的撤销交易私钥交给另一方。如果对方试图广播旧的承诺交易，那么未签名的撤销交易可以用来取回资金。这样，双方就都有了一种机制来确保对方不会广播过时的交易。

那么，这个合约究竟是怎么设计，从而实现上述功能的呢？

假设 A、B 约定了，时间 0 A 给 B 0.1BTC，时间 1 B 给 A 0.11 BTC，结果 B 在时间 0 后立即平仓。

```bash
lncli openchannel <node_id> <amount>	
```

参考代码：

`lnd/server.go`:

```go
1
func (s *server) OpenChannel(
2
	req *funding.InitFundingMsg) (chan *lnrpc.OpenStatusUpdate, chan error)
```

`lnd/funding/manager.go`:

```go
1
func (f *Manager) handleInitFundingMsg(msg *InitFundingMsg)
```

### 承诺交易

A 和 B 可以在支付通道内部进行任意数量的交易，而这些交易不会立即广播到比特币区块链上。但是怎么确保这些交易不被耍赖？这里引入承诺交易（commitment transaction）。

在一定条件下，我们可以用比特币交易来表达一种可信的承诺 —— 即使这笔交易没有得到比特币区块链的确认，但因为它是有效的（随时可以得到确认），所以，该交易（它所指明的结果）也是有意义的。

-   **更新承诺交易**：Alice 和 Bob 通过更新承诺交易来相互转移余额。这些更新可以频繁进行，且不需要在网络中宣布。而且这些承诺交易是可撤销的。
    
-   **交易有效性**：每次支付时，双方都会使对方的旧承诺交易失效，确保只有最新交易能在非合作关闭通道时用于恢复余额。
    

#### 承诺交易过程

交易可以反复发生多次，怎么做到呢？

我们可以生成一个新的承诺交易，用于更新通道状态并反映最新的支付。这个新的承诺交易会覆盖之前的承诺交易，并将之前的交易视为过时。

安全性：这个时候考虑一个情况，A 和 B 如果先后达成两次交易，例如 B 转 A 1 元，A 转 B 两元。结果第一次交易后，A 偷偷广播交易上链。怎么处理？

为了确保旧的承诺交易不会被提交到比特币区块链上，双方会交换撤销密钥（revocation key），可以在对方偷偷广播旧的交易时撤销这个交易。所以，一旦一方收到对方的撤销密钥，他们就可以确信对方不会提交旧的承诺交易。

撤销密钥的原理：[闪电网络：技术与用户体验（二）：通道与支付](https://www.btcstudy.org/2024/02/07/lightning-network-technology-improvement-and-users-experience-part-2/#%E6%89%BF%E8%AF%BA%E4%BA%A4%E6%98%93)

#### 承诺交易结构

下面从 BOLT 摘录的内容是承诺交易的结构：

-   version: 2
    
-   locktime: upper 8 bits are 0x20, lower 24 bits are the lower 24 bits of the obscured commitment number
    
-   txin count: 1
    
    -   `txin[0]` outpoint: `txid` and `output_index` from `funding_created` message
        
    -   `txin[0]` sequence: upper 8 bits are 0x80, lower 24 bits are upper 24 bits of the obscured commitment number
        
    -   `txin[0]` script bytes: 0
        
    -   `txin[0]` witness: `0 <signature_for_pubkey1> <signature_for_pubkey2>`
        

这个 48 位的承诺数字生成过程如下：

```bash
SHA256(open_channel 中的 payment_basepoint || accept_channel 中的 payment_basepoint)
```

这个操作的目的是在单方面关闭时模糊了通道中的承诺数量，但仍然为双方提供了有用的索引。节点（知晓 `payment_basepoint` 的节点）能够快速找到一个被撤销的承诺交易。

#### commitment\_signed

双方通信过程中会产生诸多消息，这里代表性地选取 commitment\_signed 简单介绍。

`commitment_signed` 是指在双方之间的支付通道中，一方已经生成了他们的最新承诺交易，并对这个交易进行了签名，然后发送给对方的消息。

具体来说，当一方生成了新的承诺交易，并在交易中包含了双方的最新资金分配，同时对该交易进行签名后，他们会发送一个包含该签名的 `commitment_signed` 消息给对方。这个消息通知对方，表示当前承诺交易已经得到对方认可，并准备好了以便在需要时广播到比特币网络上。

结构如下：

1.  type: 132 (`commitment_signed`)
    
2.  data:
    
    -   \[`channel_id`:`channel_id`\]
        
    -   \[`signature`:`signature`\]
        
    -   \[`u16`:`num_htlcs`\]
        
    -   \[`num_htlcs*signature`:`htlc_signature`\]
        

#### 承诺交易输出

为了给予惩罚交易的机会，以防出现撤销的承诺交易，所有返还资金给承诺交易所有者（又称“本地节点”）的输出必须延迟 `to_self_delay` 个区块。对于 HTLC，这种延迟在第二阶段的 HTLC 交易中实现（对于被本地节点接受的 HTLC，是 HTLC-success；对于本地节点提供的 HTLC，是 HTLC-timeout）。

HTLC 输出需要进行单独的交易阶段，以便即使在 `to_self_delay` 延迟期内，HTLC 也能够超时或被执行。否则，HTLC 所需的最短超时时间会因此延长，导致通过网络的 HTLC 需要更长的超时时间。

每个输出的金额必须舍去到整数聪。如果扣除 HTLC 交易费用后的金额小于承诺交易所有者设置的 `dust_limit_satoshis`，那么不应产生该输出（因此资金增加至费用）。

### 关闭通道

分两种情况讨论。

#### 合作关闭

-   正常情况下，双方都希望通过合作来关闭支付通道并提取他们的资金。他们会签署最后一个承诺交易并将其广播到区块链上，完成结算过程。

#### 不合作关闭

-   如果一方（称为恶意方）在支付通道到期时拒绝合作关闭通道，另一方（称为受害方）可以使用以下机制来提取资金：
    
    -   **撤销交易**：受害方持有恶意方之前提供的撤销交易密钥，可以用来广播撤销交易，取回自己的资金（扣除时间锁等待期）。
        
    -   **罚没交易**：如果恶意方试图广播一个旧的承诺交易，受害方可以使用恶意方提供的罚没交易来取回所有锁定的资金（包括恶意方的资金）作为惩罚。 **时间锁**：
        
-   每个承诺交易都包含一个时间锁，这意味着即使恶意方不合作，受害方在经过一段时间后也可以单方面关闭支付通道。这个时间锁通常是几周或几个月，具体取决于网络参数。 **Watchtowers**：
    
-   为了增强安全性，用户可以选择使用 watchtower 服务。Watchtower 会监控支付通道，并在检测到恶意行为时帮助受害方执行罚没交易，从而保护用户的资金。这种服务可以自己部署也可以用第三方的。但是专门部署这种服务即便目击违规也没有奖励。
    

## HTLC 的进一步说明

哈希时间锁定合约（HTLC）是闪电网络中用于实现跨节点支付的一种机制。

们通过一个 A-B-C 交易链条的例子来解释 HTLC 如何在闪电网络中工作：

1.  **初始状态**：
    
    -   A 想向 C 支付 1 BTC。
        
    -   但是 A 和 C 之间没有直接的支付通道，他们之间有一个中间节点 B。
        
2.  **创建 HTLC**：
    
    -   A 创建一个 HTLC，包含一个只有 C 知道的“秘密”的哈希值（`H(R)`），并将这个 HTLC 发送给 B。
3.  **转发 HTLC**：
    
    -   B 收到 HTLC 后，检查哈希值，然后创建一个新的 HTLC（使用相同的哈希值）发送给 C。
        
    -   这样，A-B-C 链条上就形成了一个 HTLC 链。
        
4.  **C 解锁 HTLC**：
    
    -   C 收到 HTLC 后，因为他知道“秘密”，所以可以提供正确的信息来解锁 HTLC。
        
    -   一旦 C 解锁，他就获得了 1 BTC。
        
5.  **反向确认**：
    
    -   C 解锁 HTLC 后，他会将“秘密”告诉 B。
        
    -   B 使用这个“秘密”来解锁他从 A 那里收到的 HTLC。
        
    -   最后，B 将“秘密”告诉 A，A 确认支付完成。
        
6.  **更新通道余额**：
    
    -   整个过程中，A-B 和 B-C 的支付通道余额会相应更新，以反映这次交易。

## 守望塔

在闪电网络中，Watchtowers（守望塔）是一种安全机制，用于防止通道被恶意利用。以下是 Watchtowers 的工作原理和如何运行自己的 Watchtower 的详细解释：

1.  **通道违约风险**：
    
    -   闪电网络中的每个通道更新都会生成一个新的承诺交易。一方可以随时使用最新的承诺交易单方面关闭通道，这称为强制关闭。
        
    -   为了防止一方利用旧的、过期的或被撤销的承诺交易来获取不正当利益，闪电网络引入了惩罚机制。任何发布旧承诺交易的一方都需要等待一定时间才能取回资金。
        
2.  **Watchtowers 的作用**：
    
    -   Watchtower 是运行在独立机器和网络上的闪电网络节点，用于监视其他节点（称为“被守护节点”）的通道。
        
    -   Watchtower 需要几乎始终在线，监视每个比特币区块，寻找潜在的通道违约。
        
3.  **如何运作**：
    
    -   被守护节点为每个远程节点的每个承诺交易创建一个签名交易，用于撤销通道违约。这个交易被加密，并与交易 ID 的前半部分一起发送给 Watchtower。
        
    -   当 Watchtower 发现通道违约时，它可以解密签名撤销交易并将其发布到比特币网络，从而防止违约并惩罚攻击者。
        
4.  **时间锁定（CSV）的作用**：
    
    -   时间锁定是一种脚本，定义了在哪些条件下可以花费被锁定的资金。CSV 延迟通常根据通道大小动态调整，也可以手动设置。例如，144 个区块的 CSV 延迟大约允许 24 小时的时间来撤销通道违约。
5.  **Watchtower 的运行**：
    
    -   目前的 Watchtower 主要是作为利他主义守望塔运行，即 Watchtower 在成功干预通道违约时不会获得补偿。
        
    -   节点操作者通常会运行自己的 Watchtower，以保护自己的节点，理想情况下，Watchtower 应运行在不同的机器、网络和地理位置上。一个节点可以使用多个 Watchtower，而一个 Watchtower 也可以守护多个节点。通过使用 Watchtower，闪电网络节点可以降低通道被恶意利用的风险，即使节点不总是在线。这种机制增强了整个网络的安全性。
        

## Gossip 网络

### 概要

在闪电网络中，通过点对点的 Gossip 网络，节点宣布自己和公共通道的存在，并传播到整个网络中。Gossip 网络还传递了关于如何到达特定节点以及它们期望转发资金所需的费用的信息。

### 详细说明

-   **Gossip 网络结构**：
    
    -   闪电网络通过维护自己的 Gossip 网络来了解其他节点和它们的通道的存在。
        
    -   Gossip 网络传递的信息包括节点的别名、支持的功能以及如何联系它们。
        
    -   该网络还包含有关每个通道的信息，以及如何在区块链上验证通道的信息，以及其同行收取的费用。
        
-   **图形组装**：
    
    -   节点使用 Gossip 网络来组装网络图。
        
    -   这个图是计算支付路由所必需的。
        
    -   例如，一个纯粹的路由节点可能不需要这些信息。
        
-   **图形更新和维护**：
    
    -   节点可能不会接收到老节点更新的信息，这些信息可能已经从网络中消失，而其他节点仍然有这些信息。
        
    -   节点还可以根据自己的判断，移除图中的通道，如果它认为对等节点不再可用，或者已经很久没有收到过它们的消息。
        
    -   对等节点经常更新它们的费用，每次更新都必须通过 Gossip 网络宣布。这可能使 Gossip 网络看起来非常嘈杂和资源消耗大。
        
-   **图形分析**：
    
    -   使用图形收集的信息，我们可以计算基本统计信息，如公共节点的总数、它们的通道和容量。
        
    -   我们还可以对图进行计算，如每个节点的中心性。
        
    -   中心性衡量的是随机路由在网络中经过一个给定节点的频率。中心性较高的节点意味着更多的假设路由经过它。
        
    -   虽然优秀的路由节点通常在图中处于中心位置，但将节点优化为中心性通常不被认为是理想策略，因为它并不是路由费用的完美代理。
        
-   **防范垃圾攻击**：
    
    -   为了防范垃圾攻击，闪电节点只会中继来自至少有一个公共通道的节点的 Gossip 消息。
        
    -   这意味着你需要拥有比特币并支付链上的交易费用。
        

## Lightning 发票

### Lightning 发票示例

Lightning 发票示例：

```bash
lnbc20m1pvjluezpp5qqqsyqcyq5rqwzqfqqqsyqcyq5rqwzqfqqqsyqcyq5rqwzqfqypqhp58yjmdan79s6qqdhdzgynm4zwqd5d7xmw5fk98klysy043l2ahrqsfpp3qjmp7lwpagxun9pygexvgpjdc4jdj85fr9yq20q82gphp2nflc7jtzrcazrra7wwgzxqc8u7754cdlpfrmccae92qgzqvzq2ps8pqqqqqqpqqqqq9qqqvpeuqafqxu92d8lr6fvg0r5gv0heeeqgcrqlnm6jhphu9y00rrhy4grqszsvpcgpy9qqqqqqgqqqqq7qqzqj9n4evl6mr5aj9f58zp6fyjzup6ywn3x6sk8akg5v4tgn2q8g4fhx05wf6juaxu9760yp46454gpg5mtzgerlzezqcqvjnhjh8z3g2qqdhhwkj
```

### Lightning 发票

Lightning 发票由人类可读部分和数据部分组成。下面逐个介绍。

#### 区分大小写

闪电网络发票与其他 bech32 编码字符串一样，通常完全是小写的。但是在二维码里用大写可以省空间，所以这类情况大写为主。

#### 前缀

闪电网络发票以 `ln` 开头，后跟的是 BIP173 为本地 Segwit 地址定义的相同双字母代码，例如 `bc` 比特币、 `tb` 测试网比特币、 `bs` 比特币图章和 `bcrt` 比特币注册测试。由于发票是 Bech32 编码的，因此还需要在末尾包含适当的校验和。

#### 量

前缀后跟金额。虽然典型的 Lightning 发票会包含金额，但可以开具没有金额的发票。Lightning Invoices 引用的是比特币，而不是 satoshi。为了节省四舍五入发票的空间，金额后面可以跟着乘数。例如，一张 satoshi Lightning 发票将显示为 `10n`，一百个 satoshi 显示 `1u`，一个 milli-satoshi 显示为 `10p`。

#### 时间戳

数据部分的第一部分是 unix 时间戳。

#### 标记

有许多标签可用于指示其他数据。其中一些数据是必需的，而其他数据可以选择由收款人提供。

### 解码 Lightning 发票

可以使用命令 `lncli decodepayreq` 解码发票。

对于上面的示例，结果如下：

```json
 1
{
 2
    "destination": "03e7156ae33b0a208d0744199163177e909e80176e55d97a2f221ede0f934dd9ad",
 3
    "payment_hash": "0001020304050607080900010203040506070809000102030405060708090102",
 4
    "num_satoshis": "2000000",
 5
    "timestamp": "1496314658",
 6
    "expiry": "3600",
 7
    "description": "",
 8
    "description_hash": "3925b6f67e2c340036ed12093dd44e0368df1b6ea26c53dbe4811f58fd5db8c1",
 9
    "fallback_addr": "1RustyRX2oai4EYYDpQGWvEL62BBGqN9T",
10
    "cltv_expiry": "9",
11
    "route_hints": [
12
        {
13
            "hop_hints": [
14
                {
15
                    "node_id": "029e03a901b85534ff1e92c43c74431f7ce72046060fcf7a95c37e148f78c77255",
16
                    "chan_id": "72623859790382856",
17
                    "fee_base_msat": 1,
18
                    "fee_proportional_millionths": 20,
19
                    "cltv_expiry_delta": 3
20
                },
21
                {
22
                    "node_id": "039e03a901b85534ff1e92c43c74431f7ce72046060fcf7a95c37e148f78c77255",
23
                    "chan_id": "217304205466536202",
24
                    "fee_base_msat": 2,
25
                    "fee_proportional_millionths": 30,
26
                    "cltv_expiry_delta": 4
27
                }
28
            ]
29
        }
30
    ],
31
    "payment_addr": null,
32
    "num_msat": "2000000000",
33
    "features": {
34
    }
35
}
```

以下是各个字段的解释：

-   **destination**: 收款人的公钥，这是用于接收付款的地址。
    
-   **payment\_hash**: 256 位的 SHA256 散列，用于支付的哈希值。在支付过程中，此哈希值将被用作付款的一部分，并且可以作为支付的证明。
    
-   **num\_satoshis**: 以 satoshi 为单位的付款金额。
    
-   **timestamp**: 闪电发票的时间戳，以 UNIX 时间戳表示。
    
-   **expiry**: 发票的过期时间，以秒为单位。在此时间之后，发票将不再有效。
    
-   **description**: 付款目的的简短描述，使用 UTF-8 编码。在此示例中为空。
    
-   **description\_hash**: 如果字段 `description` 不提供空间，则此处包含较长描述的哈希值。
    
-   **fallback\_addr**: 闪电支付失败时的回退地址。如果出现问题，可以使用该地址进行备用支付。
    
-   **cltv\_expiry**: 最后一个 HTLC 在路由中的最终 CLTV（CheckLockTimeVerify）到期时间。
    
-   **route\_hints**: 提供了额外的路由信息，用于指导付款路由的路径。
    
    -   **hop\_hints**: 每个跳跃的信息，包括节点 ID、通道 ID、费用基数、比例费率和 CLTV 到期增量。
-   **payment\_addr**: 付款地址，如果存在的话。
    
-   **num\_msat**: 以毫 satoshi 为单位的付款金额。
    
-   **features**: 付款过程中所需的功能。
    

## LND 的使用

作为目前比较流行的 LN 节点，下面简要说明常用的命令。

| 命令  | 示例  | 说明  |
| --- | --- | --- |
| `create` | `lncli create` | 创建钱包，需设定密码和记住助记词。 |
| `unlock` | `lncli unlock` | 解锁 lnd 钱包。 |
| `getinfo` | `lncli getinfo` | 查看本地 lnd 进程相关信息。 |
| `newaddress` | `lncli newaddress` | 生成用于注入资金的 lnd 地址。 |
| `walletbalance` | `lncli walletbalance` | 查看钱包余额。 |
| `connect` | `lncli connect <node_id>@<ip>:<port>` | 连接到远程节点。 |
| `openchannel` | `lncli openchannel <node_id> <amount>` | 开启与指定节点的通道。 |
| `listchannels` | `lncli listchannels` | 列出所有打开的通道。 |
| `getchaninfo` | `lncli getchaninfo <chan_id>` | 获取通道状态。 |
| `exportchanbackup` | `lncli exportchanbackup --all --output_file <file>` | 备份所有通道或指定通道的静态备份。 |
| `verifychanbackup` | `lncli verifychanbackup --multi_file <file>` | 验证通道备份是否可用。 |
| `restorechanbackup` | `restorechanbackup` | 从通道静态备份进行恢复。 |
| `closechannel` | `lncli closechannel <chan_id>` | 关闭指定通道。 |
| `closeallchannels` | `lncli closeallchannels` | 关闭所有通道。 |
| `stop` | `lncli stop` | 停止并关闭 lnd 进程。 |

这些命令涵盖了 lnd 的主要功能，包括通道管理、钱包操作、网络连接和支付等。

## 参考

-   [lightning/bolts: BOLT: Basis of Lightning Technology (Lightning Network Specifications)](https://github.com/lightning/bolts)
    
-   [LN Things Part 1: Creating a channel | Elle Mouton](https://ellemouton.com/posts/creating-a-channel/)
    
-   [A Deep Dive into LND: Overview and Channel Funding Process](https://blog.muun.com/a-deep-dive-into-lnd-overview-and-channel-funding-process/)
    
-   [DoS: Fake Lightning Channels – Matt Morehouse](https://morehouse.github.io/lightning/fake-channel-dos/)
