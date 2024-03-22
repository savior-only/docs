

# 深入研究比特币：（3）RGB 协议介绍

## RGB 介绍

### 一次性封印

**一次性封印（Single-Use-Seals）**：一个封印，在创建时封印，在使用时解封。

我们假设每次解封后都必须同时贴上一个新的封印，且新的封印上记录的新的所有者。那么**每个解封 - 新封印的过程就相当于一次所有权转移**，每个封印相当于一个所有权凭证。

在 BTC 中，**UTXO 恰好符合这一特征**。而在实际操作时，可以将接收方的 UTXO 的哈希值（称为承诺）添加到交易。

### 客户端验证

假设初始状态我们创建了一个所有权合约，那么每次产生对此合约（例如设置新的所有者）的修订，就将此修订的摘要写入到比特币交易中，并且将接收方的 UTXO 的哈希值也写入到交易中。

这样我们这些交易就记录了这个所有权合约的状态转移的凭证。

**检验所有权**：我们可以拉取出所有关于这个所有权合约的交易，然后让证明人出示修订历史，对这些历史进行哈希，对照链上的交易中存档的哈希，可以判断此人是否的确是修订档的对象，从而知道此人是否是新的所有者。**这过程可以完全在链下进行，也就是 RGB 所称的客户端验证**。（这些历史，包括从最新的交易开始到创世交易整条链路的与此所有权相关的所有交易。）

-   合约（Contract）是图式（Schema）的实例，图式实现了接口（Interface）
    
-   类比 OOP：Contract-Object, Schema-Class, Interface-Interface
    

### 个人解读

RGB 协议本质上是在比特币的 UTXO 里存一些交易相关的哈希，然后在客户端执行程序验证链上的这些摘要。按 RGB 的叙事，BTC 被叫做加密承诺层 (cryptographic commitment layer)

RGB 声称实现了智能合约，但其实这种合约只是客户端执行的程序，这和 EVM 完全不同。在 ETH 中，智能合约在所有的节点上执行，并且最终得到相同的结果，从而确保了合约的可靠。然而，也正是因为需要在所有节点执行，ETH 的智能合约部署成本很高，执行性能也很差。

RGB 协议认为，智能合约可以只在参与方之间执行，而不需要在所有的节点执行。数据在用户手里执行，用户也只能看到和自己有关的数据。

这样的客户端验证的模式，使得所有交易都非常私密，就像是黑暗森林中两个人只通过悄悄话通信。好处是不会因为类似 ETH 的智能合约集中式存储信息（存储在所有节点），其控制者的问题可能导致用户受损失，但是坏处，个人感觉这样会导致发展速率受到限制，因为现在的时代你不但要有说得通的叙事，还需要有好看的数据（比如 DAU、交易量、市值）而私密性过高导致这种交易对交易双方的外界而言几乎是黑盒，一定程度限制了 RGB 自身发展。

## 一次性封印的详细说明

[这篇文章](https://petertodd.org/2017/scalable-single-use-seal-asset-transfer) 由 Peter Todd 撰写，主要探讨了如何利用一次性封印（Single-Use-Seals）和发布证明（Proof-of-Publication）来实现资产的安全转移，特别是在区块链和加密货币的背景下。以下是文章的主要观点和结构概述：

### 定义

-   一次性封印是一种抽象机制，用于防止双花（double-spends）。
    
-   它支持两个基本操作：关闭和验证。
    
-   关闭封印的操作（`Close(l, m) → w_l`）是将封印 `l` 关闭在消息 `m` 上，生成一个见证 `w_l`。
    
-   验证封印的操作（`Verify(l, w_l, m) → bool`）是验证封印 `l` 是否已经关闭在消息 `m` 上。
    
-   安全的封印实现要求不可能有两个不同的消息 `m1` 和 `m2` 对同一个封印进行验证并返回真。
    

在比特币的环境中，一个 UTXO 只能被消费一次。因此，比特币交易的输出可以被看作是一次性封印，而当这个输出被用作另一个交易的输入时，该封印就被“打破”或“使用”了。所以在 RGB 的语境，只要查询一个 UTXO 是否被花费就知道财产在这一环节是否被转移。

### 不可分割资产转移

-   使用安全的一次性封印，可以构建一个不可分割资产转移系统。
    
-   每个资产由其创世封印 `l0` 标识。
    
-   转移资产时，最近的封印 `ln` 被关闭在一个新的封印 `l(n+1)` 上，生成一个见证 `w_ln`，证明该转移。
    
-   通过验证从创世封印到最新封印的一系列见证和封印，接收者可以安全地验证他们收到了所需的代币。
    

### 可分割资产转移

-   对于可分割资产，转移的是资产的数量而非单个独特的代币。
    
-   使用“输出”概念，每个输出 `x` 都关联一个封印 `l` 和值 `v`。
    
-   转移可分割资产定义了“花费”和“分割”的概念。
    
-   花费 `D` 是对输出集 `xi .. xj` 的承诺；分割 `V` 是对零个或多个封印 / 值对的承诺。
    
-   接收者可以通过生成新的封印，请求发送者创建新的分割输出，并验证新创建的分割输出的有效性来验证资产的安全转移。
    

### 总结

在实际中就是利用 UTXO 本身的机制实现防止双花。

## 客户端验证的详细说明

客户端验证范式的核心思想是减少在分布式网络中需要验证状态转换的参与者的数量，从而提高效率。说人话，就是把合约放到用户电脑上算，数据可用性也交给客户端自己处理，分布式网络只负责提供所有权信息可用性（也即“出版证明”）。

假设有一个分布式网络，其中多个节点需要进行数据交换和验证。在传统的区块链系统中，每一个新区块的生成和状态的改变都需要网络中的所有节点进行验证。这确保了整个网络的一致性，但同时也消耗了大量的计算资源。

在客户端验证的范式中，这个过程有所不同。设想这样一个场景：

1.  节点 A 想要将一笔交易发送给节点 B。
    
2.  在传统的区块链系统中，这笔交易会被广播到整个网络，所有的节点都会验证交易的合法性，包括签名是否正确，发送者是否有足够的余额等。
    
3.  在客户端验证的范式中，节点 A 和节点 B 可以直接进行交易，并仅由这两个节点来验证交易的合法性。
    
4.  节点 A 使用一个加密哈希函数来创建一个代表这个交易的加密承诺（例如，一个哈希值）。
    
5.  这个加密承诺随后被发布到一个公开的「出版证明（Proof-of-Publication）」媒介中，比如一个区块链。
    
6.  通过这个加密承诺，任何想要验证交易有效性的第三方节点都可以在不需要了解交易所有细节的情况下，验证交易是否已经被节点 A 和节点 B 同意，并且已经被公开地记录在案。
    
7.  这个「出版证明」媒介同时还提供了收据证明、非发布证明、成员资格证明等功能，确保了交易的可验证性和不可否认性。
    

通过这种方式，只有交易的相关方需要进行详细的验证工作，而网络中的其他节点可以通过验证加密承诺来确认交易的状态，从而大大减少了整个网络需要执行的工作量。这种方法特别适合于那些不需要每个节点都存储和处理所有数据的去中心化应用。

## 交易过程

假设 Bob 想要给 Alice 转移资产。步骤如下：

1.  Alice 需要构建一个 invoice，大约包含如下信息：合约 ID、状态转移参数、UTXO.
    
2.  Bob 需要给出自己的资产的所有来龙去脉证明，也就是说从此资产从诞生到最后流转到 Bob 手上的整条证据，这个东西称为 consignment。
    
3.  Alice 收到证明，验证之后，生成一个确认签名给 Bob。
    
4.  Bob 将此交易的摘要信息发布到 BTC 网络，区块入链，交易完成。
    

这个过程中，资产转移到谁手上，谁就要负责数据可用性，否则资产相当于永久遗失。

### 盲化

在密码学和隐私保护领域，盲化（Blinding）是一种技术，用于在进行计算或交互时隐藏敏感信息。它通过对数据进行变换或添加随机性来实现。

盲化的目的是防止参与方在处理数据时获取有关数据的具体信息，同时仍然能够进行必要的计算或交互。通过盲化，可以保护用户的隐私和数据的机密性。

具体而言，在 RGB 中盲化是为了保护受益人的隐私。

### 储藏室（Stash）

RGB 将其所有操作所需的所有数据存储在称为 **Stash** 的虚拟容器中。也就是说丢失 Stash == 丢失资产。

## 合约

官方文档给出了一个 Rust 示例合约：

```rust
 1
use rgbstd::interface::{rgb20, ContractBuilder};
 2

 3
use std::convert::Infallible;
 4
use std::fs;
 5

 6
use amplify::hex::FromHex;
 7
use bp::{Chain, Outpoint, Tx, Txid};
 8
use rgb_schemata::{nia_rgb20, nia_schema};
 9
use rgbstd::containers::BindleContent;
10
use rgbstd::contract::WitnessOrd;
11
use rgbstd::resolvers::ResolveHeight;
12
use rgbstd::stl::{
13
    Amount, ContractData, DivisibleAssetSpec, Precision, RicardianContract, Timestamp,
14
};
15
use rgbstd::validation::{ResolveTx, TxResolverError};
16
use strict_encoding::StrictDumb;
17

18
struct DumbResolver;
19

20
impl ResolveTx for DumbResolver {
21
    fn resolve_tx(&self, _txid: Txid) -> Result<Tx, TxResolverError> {
22
        Ok(Tx::strict_dumb())
23
    }
24
}
25

26
impl ResolveHeight for DumbResolver {
27
    type Error = Infallible;
28
    fn resolve_height(&mut self, _txid: Txid) -> Result<WitnessOrd, Self::Error> {
29
        Ok(WitnessOrd::OffChain)
30
    }
31
}
32

33
#[rustfmt::skip]
34
fn main() {
35
    let name = "Token";  // name of token
36
    let decimal = Precision::CentiMicro; // Decimal: 8
37
    let desc = "RGB Token";  // token description
38
    let spec = DivisibleAssetSpec::with("RGB20", name, decimal, Some(desc)).unwrap();
39
    let terms = RicardianContract::default();
40
    let contract_data = ContractData { terms, media: None };
41
    let created = Timestamp::now();
42
    let txid = "9fef7f1c9cd1dd6b9ac5c0a050103ad874346d4fdb7bcf954de2dfe64dd2ce05";
43
    // token owner address in utxo
44
    let beneficiary = Outpoint::new(Txid::from_hex(txid).unwrap(), 0);
45

46
    const ISSUE: u64 = 1_000_000_000;
47

48
    let contract = ContractBuilder::with(rgb20(), nia_schema(), nia_rgb20())
49
        .expect("schema fails to implement RGB20 interface")
50
        .set_chain(Chain::Testnet3)
51
        .add_global_state("spec", spec)
52
        .expect("invalid nominal")
53
        .add_global_state("created", created)
54
        .expect("invalid creation date")
55
        .add_global_state("issuedSupply", Amount::from(ISSUE))
56
        .expect("invalid issued supply")
57
        .add_global_state("data", contract_data)
58
        .expect("invalid contract text")
59
        .add_fungible_state("assetOwner", beneficiary, ISSUE)
60
        .expect("invalid asset amount")
61
        .issue_contract()
62
        .expect("contract doesn't fit schema requirements");
63

64
    let contract_id = contract.contract_id();
65
    debug_assert_eq!(contract_id, contract.contract_id());
66

67
    let bindle = contract.bindle();
68
    eprintln!("{bindle}");
69
    bindle.save("contracts/rgb20-token.contract.rgb")
70
        .expect("unable to save contract");
71
    fs::write("contracts/rgb20-token.contract.rgba", bindle.to_string())
72
        .expect("unable to save contract");
73
}
```

关键点：

1.  在 `main` 函数中：
    
    -   定义了智能合约的属性，如名称、小数位数、描述、发行总量等。
        
    -   使用 `ContractBuilder` 来构建智能合约。
        
    -   导出智能合约文件，并保存到指定的目录中。
        
2.  主要使用了 `rgbstd` 提供的模块和方法来构建智能合约，并使用 `bp` 库来操作交易相关的数据结构。
    

## 源码浅析

实际上 RGB 从代码而言，其主要工作和区块链的关系并不大，更像是一个单机小工具，因为它也不会限制你如何将信息传给你的对等交易节点。从下面的使用例子就可以看出：

### 命令行基本使用

导入一个合约：

```bash
rgb import contract.rgb
```

列出 Stash 中的合约：

```bash
rgb
```

输出：

```bash
Schemata:
---------
marina-karl-basic-4h3xkAmiRdQs2HwQmnFVqoj5DG3NhGkJRpNZXjNrJT2N: RGB20

Interfaces:
---------
RGB20 complex-field-union-DbwzvSu4BZU81jEpE9FVZ3xjcyuTKWWy2gmdnaxtACrS

Contracts:
---------
rgb:BasilYellowGrand0BgHH3yYaCUvaw5Tc8KfrVB1aWN4vBvCXCuMfK98N3o6Z
```

查询合约状态：

```shell
1
rgb state BasilYellowGrand0BgHH3yYaCUvaw5Tc8KfrVB1aWN4vBvCXCuMfK98N3o6Z RGB20
```

```bash
Global state:
Nominal:=(ticker=("TEST"), name=("Test asset"), details=~, precision=8)

Owned state:
  (amount=100000000000000, owner=913d9a55efd7748ee89b577328a8f22189432c04275a106c4915cd1dac543562:0, witness=~)
```

发行合约：

```bash
rgb issue 4h3xkAmiRdQs2HwQmnFVqoj5DG3NhGkJRpNZXjNrJT2N RGB20 examples/rgb20-demo.yaml
```

合约示例：

```yaml
 1
interface: RGB20
 2

 3
globals:
 4
  Nominal:
 5
    ticker: TEST
 6
    name: Test asset
 7
    details: ~
 8
    precision: 8
 9
  ContractText: >
10
    **skipped**
11
assignments:
12
  Assets:
13
    seal: tapret1st:01d46e52c4bdb51931a0eae83e958c78bdef9cac2057b36d55370410edafdd42:0
14
    amount: 100000000000000
```

-   interface：表示合同遵循的接口规范
    
-   globals：全局状态
    
-   assignments：将所拥有的状态（发行的资产数量）分配给单次使用封印，这些封印是以比特币 UTXO 为前缀的
    

### rgb-std

RGB-STD 标准库是一个用于实现 RGB 协议的核心库。从注释我们可以得知其大体功能。

核心库（CORE LIB）提供了以下功能：

-   `issue`：根据给定的模式（Schema）、元数据（Metadata）、全局状态（GlobalState）和分配（Assignments），生成初始状态（Genesis）。

标准库（STD LIB）提供了以下功能：

-   `import`：将资产（Stash）和模式（Schema）或接口（Interface）导入到资产库（Stash）中。
    
-   `state`：根据资产清单（Inventory）和合约 ID（ContractId），获取合约的当前状态（ContractState）。
    
-   `interpret`：根据合约状态（ContractState）和接口（Interface），解释合约的状态，生成解释后的状态（InterpretedState）。
    

标准库还提供了以下用于资产转移的功能：

-   `issue`：根据给定的模式（Schema）、状态（State）和接口（Interface），调用 `core::issue` 函数生成资产转移的证明（Consignment）。
    
-   `extract`：根据资产清单（Inventory）、合约 ID（ContractId）和接口（Interface），生成资产转移的证明（Consignment），用于合约转移。
    
-   `compose`：根据资产清单（Inventory）、合约 ID（ContractId）、接口（Interface）和输出点（Outpoint）列表，生成资产转移的证明（Consignment），用于描述已存在的状态。
    
-   `transfer`：根据资产转移的证明（Consignment）和其他参数，准备资产转移的状态转换（StateTransition）。
    
-   `preserve`：根据资产库（Stash）、输出点（Outpoint）列表和状态转换（StateTransition），创建空白的状态转换（StateTransition）。
    
-   `consign`：根据资产库（Stash）和状态转换（StateTransition），提取历史数据，生成资产转移的证明（Consignment）。
    

其他功能包括 `reveal`、`validate`、`enclose` 和 `consume`，用于处理资产转移和合约的验证、封装和消耗。

钱包库（WALLET LIB）提供了以下功能：

-   `embed`：将合约信息嵌入到部分签名交易（PSBT）中，以便在交易中添加合约信息。
    
-   `commit`：将转换信息添加到部分签名交易（PSBT）中，以便在交易中添加转换信息。
    
-   `bundle`：将多个单独的转换合并为一个部分签名交易（PSBT）。
    
-   `finalize`：由 BP（Blockchain Provider）执行的操作，将单独的承诺（commitments）转换为最终的交易。
    

### Command::Issue

issue 用来发布合约。我们看看怎么实现的。

```bash
rgb issue 4h3xkAmiRdQs2HwQmnFVqoj5DG3NhGkJRpNZXjNrJT2N RGB20 examples/rgb20-demo.yaml
```

-   代码关键词：`Command::Issue { schema, contract }`
    
-   4h3xkAmiRdQs2HwQmnFVqoj5DG3NhGkJRpNZXjNrJT2N 应该就是 SchemaId，contract 对应上面的 yaml 路径。
    
-   interface: RGB20 就是 iface\_name
    

Hoard 类存放了所有的运行时数据，在 runtime 中：

-   通过 iface\_name 查找对应的 iface
    
-   通过 schema id 查找对应的 schema 和 iimpls，后者是一系列的 IfaceImpl
    
-   IfaceImpl 看起来像是一个类型容器
    
-   在 iimpls 中通过 iface id 提取对应的 iface\_impl
    
-   进一步构造 ContractBuilder 的实例 builder
    
    -   builder 通过 add\_global\_state 将全局状态字段序列化
        
    -   builder 通过 add\_fungible\_state 添加可分割资产状态
        
    -   最后调用 issue\_contract 完成合约的构造
        

### Command::Export

合约发行之后可以导出。一般就是序列化操作。

通过 `runtime.export_contract` 导出，得到一个 `Bindle<Contract>`

Bindle 是一个包装器，用于封装不同类型的 RGB 容器。简要梳理：

-   `BindleContent` trait：
    
    -   定义了处理 RGB 容器的通用行为，包括序列化、反序列化等。
        
    -   为不同类型的容器实现了 `BindleContent` trait，包括 `Schema`、`Contract`、`Transfer`、`Iface` 和 `IfaceImpl`。
        
-   `Bindle` 结构体：
    
    -   包含了容器的 ID、数据和签名。
        
    -   实现了 `Deref` trait，以便通过 `Deref` 操作符访问内部的数据。
        
    -   提供了 `new` 方法用于创建新的 `Bindle` 实例。
        
    -   提供了一系列方法，用于获取容器的 ID、拆分容器以及解绑容器。
        
-   `BindleParseError` 枚举：
    
    -   定义了从字符串解析 `Bindle` 实例时可能出现的错误类型，包括结构错误、ID 无效、ID 不匹配等。
-   `FromStr` 和 `Display` trait 的实现：
    
    -   实现了从字符串解析 `Bindle` 实例以及将 `Bindle` 实例格式化为字符串的功能。
-   `load` 方法：
    
    -   提供了从 `Read` 对象加载 `Bindle` 实例的能力。
-   `UniversalBindle` 枚举：
    
    -   定义了一个通用的 `Bindle` 类型，可以表示不同类型的 `Bindle` 实例，包括 `Iface`、`SubSchema`、`IfaceImpl`、`Contract` 和 `Transfer`。
-   `LoadError` 枚举：
    
    -   定义了加载 `Bindle` 实例时可能出现的错误类型，包括文件数据无效、解码错误等。
-   `_fs` 模块（如果特性为 `fs`）：
    
    -   提供了从文件加载和保存 `Bindle` 实例的功能。

总结而言就是一个方便序列化和反序列化的 RGB 容器。通过 save 方法，调用 strict\_encode 完成序列化。

### Command::Invoice

根据不同条件选择不同收款方，并创建了一个 RGB 发票，打印到控制台。

创建发票主要用 RgbInvoiceBuilder

### src/invoice.rs

-   `RgbTransport`：表示 RGB 传输方式的枚举类型，包括 JSON-RPC、REST HTTP、WebSockets 等。
    
-   `InvoiceState`：表示发票的状态，可以是空（Void）、金额（Amount）、非同质化资产（Data）或附加标识符（Attach）。
    
-   `ChainNet`：表示链网络的枚举类型，包括比特币主网（BitcoinMainnet）、比特币测试网（BitcoinTestnet）、Liquid 主网（LiquidMainnet）等。
    
-   `XChainNet`：表示带有链网络信息的泛型类型，用于将数据与特定的链网络关联起来。
    
-   `Beneficiary`：表示受益人的枚举类型，可以是盲签名（BlindedSeal）或见证输出（WitnessVout）。
    
-   `RgbInvoice`：表示 RGB 发票的结构体，包含了传输方式、合约、接口、操作、受益人、发票状态、过期时间等信息。
    

盲签名（BlindedSeal）和见证输出（WitnessVout）是与 RGB 发票中的受益人（Beneficiary）相关的概念。

1.  盲签名（BlindedSeal）：盲签名是一种密码学技术，用于在不暴露原始数据的情况下对数据进行签名。在 RGB 发票中，盲签名作为受益人的一种类型，表示受益人使用盲签名作为验证方式。盲签名可以保护受益人的隐私，因为在签名过程中，受益人的身份和数据是被隐藏的。
    
2.  见证输出（WitnessVout）：见证输出是比特币中的一种特殊输出类型，用于存储与交易验证相关的数据。在 RGB 发票中，见证输出作为受益人的一种类型，表示受益人使用见证输出作为验证方式。通过使用见证输出，受益人可以提供与交易验证相关的证据，以证明其在交易中的权益。
    

这两种受益人类型（盲签名和见证输出）提供了不同的验证方式，用于确保 RGB 发票的安全性和可信度。具体使用哪种类型取决于发票的设计和需求。

关键函数：

-   `ChainNet::layer1()`：根据链网络返回相应的 Layer1 类型（Bitcoin 或 Liquid）。
    
-   `ChainNet::is_prod()`：判断链网络是否为生产环境。
    
-   `ChainNet::address_network()`：根据链网络返回相应的地址网络类型。
    
-   `XChainNet::with()`：根据给定的链网络和数据创建 XChainNet 类型的实例。
    
-   `XChainNet::bitcoin()`：根据给定的网络类型和数据创建 XChainNet 类型的实例。
    
-   `XChainNet::chain_network()`：获取 XChainNet 实例的链网络类型。
    
-   `XChainNet::into_inner()`：获取 XChainNet 实例中的数据。
    
-   `RgbInvoice::chain_network()`：获取 RGB 发票的链网络类型。
    
-   `RgbInvoice::address_network()`：获取 RGB 发票的地址网络类型。
    
-   `RgbInvoice::layer1()`：获取 RGB 发票的 Layer1 类型。
    
-   `RgbInvoice::is_prod()`：判断 RGB 发票是否为生产环境。
    

### Command::Prepare

主要功能是构建 PSBT。

简要梳理：

-   包含了六个命令参数：
    
    -   `v2`：一个布尔值，表示是否使用 PSBT 版本 2。
        
    -   `method`：一个支付方法。
        
    -   `invoice`：一个 RGB 发票。
        
    -   `fee`：手续费。
        
    -   `sats`：以 Satoshi 为单位的金额。
        
    -   `psbt_file`：一个可选的 PSBT 文件名。
        
-   创建了一个 RGB 运行时环境。
    
-   使用给定的手续费和金额创建了一个传输参数（TransferParams）。
    
-   调用 `construct_psbt` 方法构建了一个 PSBT（部分签名交易）。
    
-   根据 PSBT 版本和是否提供了 PSBT 文件名，选择了不同的输出方式：
    
    -   如果提供了 PSBT 文件名，则将 PSBT 编码并写入文件。
        
    -   如果未提供 PSBT 文件名：
        
        -   如果 PSBT 版本为 V0，则将 PSBT 打印到控制台。
            
        -   如果 PSBT 版本为 V2，则以更详细的格式打印 PSBT 到控制台。
            
-   返回修改后的运行时环境。
    

PSBT 详细构建过程：

这段代码是一个方法，负责构建一个 PSBT（部分签名交易）以进行 RGB 资产的转移。以下是对代码的简要梳理：

-   首先，从发票中提取所需的信息，包括合约 ID、接口名称、操作类型和分配名称。
    
-   通过合约 ID 和接口名称获取合约，并根据操作类型确定要执行的操作。
    
-   根据发票中的状态信息（拥有的资产数量）和合约规则，确定需要使用的输出。如果是金额状态，则根据发票中指定的金额选择合适的输出。如果是其他状态，则抛出错误。
    
-   根据发票中的受益人信息，确定受益人地址。如果是已知地址，则为其创建一个脚本；如果是盲签密封，则不创建。
    
-   根据上述信息，构建 PSBT 所需的输出点（Outpoint）和受益人（Beneficiary）列表。
    
-   使用 RGB 钱包构建 PSBT。在这一步中，PSBT 将包含输出、找零和其他元数据。
    
-   对于 PSBT 中的找零输出，设置为 TAPROOT 脚本，并将其添加到 TAPROOT 主机数据中。
    
-   对于 PSBT 中的输出，检查是否需要添加 OP\_RETURN 输出，并根据需要进行添加。
    
-   完成 PSBT 的构建，并将 RGB 批次数据嵌入到 PSBT 中。
    
-   最后返回构建好的 PSBT 和相关元数据。
    

该方法实现了根据 RGB 发票的信息构建 PSBT 的功能，并处理了 PSBT 中的输出、找零以及其他元数据的设置。

### Runtime » construct\_psbt

-   从发票中提取所需的信息，包括合约 ID、接口名称、操作类型和分配名称。
    
-   通过合约 ID 和接口名称获取合约，并根据操作类型确定要执行的操作。
    
-   根据发票中的状态信息（拥有的资产数量）和合约规则，确定需要使用的输出。如果是金额状态，则根据发票中指定的金额选择合适的输出。如果是其他状态，则抛出错误。
    
-   根据发票中的受益人信息，确定受益人地址。如果是已知地址，则为其创建一个脚本；如果是盲签密封，则不创建。
    
-   根据上述信息，构建 PSBT 所需的输出点（Outpoint）和受益人（Beneficiary）列表。
    
-   使用 RGB 钱包构建 PSBT。在这一步中，PSBT 将包含输出、找零和其他元数据。
    
-   对于 PSBT 中的找零输出，设置为 TAPROOT 脚本，并将其添加到 TAPROOT 主机数据中。
    
-   对于 PSBT 中的输出，检查是否需要添加 OP\_RETURN 输出，并根据需要进行添加。
    
-   完成 PSBT 的构建，并将 RGB 批次数据嵌入到 PSBT 中。
    
-   最后返回构建好的 PSBT 和相关元数据。
    

在上述代码中，TAPROOT 用于构建支付交易的输出脚本。TAPROOT 可以将多重签名（Multisig）和智能合约隐藏在单个公钥散列（P2TR）中，使得所有类型的交易看起来都像是单个用户的常规支付。这样可以提高用户的隐私保护，降低交易链上的可追踪性。

#### Outpoint

在比特币网络中，Outpoint 是指用于唯一标识交易输出（Transaction Output）的数据结构。每个交易输出都有一个对应的 Outpoint，它包含两个主要部分：

1.  **交易 ID（Transaction ID）：** 交易 ID 是一个唯一标识符，用于识别交易在区块链中的位置。它是由交易的全部内容进行哈希运算而得到的，因此可以确保在网络中的唯一性。
    
2.  **输出索引（Output Index）：** 输出索引是指交易中特定输出的编号，从零开始逐个递增。它用于区分同一笔交易中的不同输出。
    

### Command::Consign

主要目的是将给定的发票和 PSBT 文件组合起来，生成最终的转移信息，并将转移信息保存到指定的输出文件中。这个过程涉及关键函数 transfer

### Runtime » transfer

概括，在执行 RGB 智能合约的转移操作时要进行这些比较操作：

1.  **数据提取**：PSBT 中包含了交易的部分签名和相关的输入输出信息。通过提取 PSBT 中的 RGB 提交数据（fascia），可以获取到关于 RGB 合约的状态转移所需的信息，包括资产、终端等。
    
2.  **Taproot 支持**：在 PSBT 中存在 Taproot 相关的输出时，需要提取其中的终端派生信息和 Taproot 承诺，并将其添加到钱包中以支持 Taproot 相关的功能
    
3.  **确定受益人**：根据发票中指定的受益人信息，确定将资金发送给哪些地址，并构建对应的受益人列表。
    
4.  **数据消耗**：在执行状态转移之前，需要从库存中消耗 RGB 提交数据（fascia），以确保转移的合法性。
    
5.  **生成转移对象**：根据提取的数据和确定的受益人信息，生成一个转移对象，包含了转移操作所需的所有信息，例如合同 ID、受益人信息、终端等。
    

具体步骤如下：

1.  首先，从发票中获取合同 ID，确保发票中包含了合同信息。
    
2.  调用 PSBT 对象的 `rgb_commit` 方法，该方法用于从 PSBT 中提取 RGB 提交数据（fascia）。
    
    1.  这里会转换为可以 mpc（多方计算）的 bundles
3.  如果 PSBT 中存在与 Taproot 相关的输出，即包含了 Taproot 证明，那么从中获取终端派生信息和 Taproot 承诺，并将其添加到钱包中。
    
4.  获取 PSBT 的事务 ID。
    
5.  根据发票中的受益人信息，确定将资金发送给哪些地址，构建受益人列表。
    
    -   如果受益人是一个见证地址（`WitnessVout`），则找到 PSBT 中对应的输出，并构建一个对应的 `XChain::Bitcoin` 封闭密封（Seal）。
        
    -   如果受益人是一个盲封（`BlindedSeal`），则直接构建一个对应的 `XChain::Bitcoin` 密封。
        
6.  从库存中消耗 RGB 提交数据（fascia）。
    
    1.  涉及 Inventory 的 consume 函数，关键词 `fn consume(&mut self, fascia: Fascia)`。函数首先确保锚点的一致性，并将锚点导入到 stash 中。然后，它遍历 fascia 中的每个 bundle，将其转换为一个或多个状态转换，并将它们导入到 stash 中。
7.  调用库存的 `transfer` 方法，将合同 ID 和受益人列表传递给它，并生成一个转移对象。
    
8.  对于每个终端，检查是否有与 PSBT 相关的锚定束（anchored bundle），如果有，则将 PSBT 转换为未签名的交易，并将其分配给终端的见证事务字段（`witness_tx`）。
    
9.  返回生成的转移对象。
    

这个方法主要用于处理从 PSBT 中提取的数据，并根据发票中的信息执行转移操作。

锚点：在区块链中，锚点是一种确定特定数据或状态在区块链上位置的机制。它们充当了在区块链中定位数据或状态的标记或参考点。一般来说就是一个或一系列的哈希值。

## 参考

-   [Scalable Semi-Trustless Asset Transfer via Single-Use-Seals and Proof-of-Publication](https://petertodd.org/2017/scalable-single-use-seal-asset-transfer)
    
-   [RGB smart contracts](https://rgb.tech/power-user/)
