---
title: GCN 基础：GraphConv、GATConv、SAGEConv 的实现（PyG+DGL）
url: https://www.less-bug.com/posts/gcn-basis-graphconv-gatconv-sageconv-implementation-pyg-dgl/
clipped_at: 2024-04-01 16:24:52
category: default
tags: 
 - www.less-bug.com
---


# GCN 基础：GraphConv、GATConv、SAGEConv 的实现（PyG+DGL）

## 简介

GraphConv、GATConv 和 SAGEConv 是三种常用的图卷积层，功能都是类似的，用来学习图结构数据中的节点表示，以便于后续的图分析任务，比如说节点分类、图分类或链接预测等等。

三者的核心区别在于怎么聚合邻接节点的信息：GraphConv 采用**平均池化**，GATConv 通过**注意力机制**赋予不同邻居不同的重要性，而 SAGEConv 则提供了多种聚合函数选择。这些差异影响了导致训练出来的模型有不同的表现。

## 使用示例

在用法上都是类似的。一般来说使用 GATConv 我们会比较关注注意力头数，使用 SAGEConv 我们会比较关注聚合方式。

```python
 1
import dgl.nn as dglnn
 2

 3
# GraphConv
 4
conv1 = dglnn.GraphConv(in_feats, out_feats)
 5
x = conv1(g, x)
 6

 7
# GATConv
 8
conv2 = dglnn.GATConv(in_feats, out_feats, num_heads)
 9
x = conv2(g, x)
10

11
# SAGEConv
12
conv3 = dglnn.SAGEConv(in_feats, out_feats, 'mean')
13
x = conv3(g, x)
```

## 消息传递范式

设 `` $x_{v} \in R^{d_{1}}$ `` 为节点 `` $v$ `` 的特征，`` $w_{e} \in R^{d_{2}}$ `` 为边 `` $(u,v)$ `` 的特征。**消息传递范式**定义了如下的节点和边的计算过程：

```

边计算LATEX_BLOCK___\text{边计算: }m_{e}^{(t + 1)} = \phi \left(x_{v}^{(t)},x_{u}^{(t)},w_{e}^{(t)}\right),(u,v,e) \in E___LATEX_BLOCK
```

```

节点计算LATEX_BLOCK___\text{节点计算: }x_{v}^{(t + 1)} = \psi \left(x_{v}^{(t)},ho\left(\left\{m_{e}^{(t + 1)}:(u,v,e) \in E\right\}\right)\right)___LATEX_BLOCK
```

其中，`` $\phi $ `` 是一个**消息函数** , 它根据边的特征和相邻节点的特征生成消息；`` $\psi $ `` 是一个**更新函数** , 它根据节点的当前特征和来自邻居的消息来更新节点的特征，其中 `` $ho$ `` 是一个**聚合函数**。

## 实现 GraphConv

### 原理

GraphConv 通过聚合邻居节点的特征表示，来更新每个节点的特征表示，依赖于加权邻居特征的求和。

### 解释

下面的 GraphConv 公式选自 [GraphConv — DGL 2.1.0 documentation](https://docs.dgl.ai/generated/dgl.nn.pytorch.conv.GraphConv.html#dgl.nn.pytorch.conv.GraphConv)

```

LATEX_BLOCK___h_{i}^{(l + 1)} = \sigma (b^{(l)} + \sum \limits_{j \in N(i)}\frac{e_{ji}}{c_{ji}}h_{j}^{(l)}W^{(l)})___LATEX_BLOCK
```

其中：

-   `` $h_{i}^{(l + 1)}$ `` 是第 `` $l + 1$ `` 层的节点 `` $i$ `` 的特征向量
    
-   `` $\sigma $ `` 是非线性激活函数
    
-   `` $b^{(l)}$ `` 是偏置项
    
-   `` $W^{(l)}$ `` 是层的权重矩阵
    
-   `` $e_{ji}$ `` 是从节点 `` $j$ `` 到 `` $i$ `` 的边的权重
    
-   `` $c_{ji}$ `` 是归一化常数（例如度的倒数或其他归一化策略）
    
-   `` $N(i)$ `` 表示节点 `` $i$ `` 的邻居节点集合。
    
-   公式的核心是邻居节点特征的加权平均，其中权重取决于边的权重和归一化常数。
    

### 消息传递对应

GraphConv 公式可以看作是消息传递范式的一种具体实现，其中消息函数 `` $\phi $ `` 是线性变换，聚合函数 `` $ho$ `` 是求和，更新函数 `` $\psi $ `` 是非线性激活。

1.  **消息计算**:
    
    -   消息 `` $m_{ji} = \frac{e_{ji}}{c_{ji}}h_{j}^{(l)}W^{(l)}$ ``
        
    -   这里 `` $\frac{e_{ji}}{c_{ji}}$ `` 是归一化系数，`` $h_{j}^{(l)}W^{(l)}$ `` 是节点 `` $j$ `` 经过线性变换得到的消息。
        
2.  **消息聚合**:
    
    -   节点 `` $i$ `` 聚合来自其邻居节点 `` $j$ `` 的消息：`` $\sum \limits_{j \in N(i)}m_{ji}$ ``
3.  **节点更新**:
    
    -   节点 `` $i$ `` 的新特征 `` $h_{i}^{(l + 1)} = \sigma (b^{(l)} + \sum \limits_{j \in N(i)}m_{ji})$ ``
        
    -   这里 `` $\sigma $ `` 是非线性激活函数，`` $b^{(l)}$ `` 是偏置项。
        

### 度数不平衡问题

度数不平衡问题是指在一个图数据集中，某些节点的邻居数量 (即节点的度数) 远大于其他节点。这种不平衡现象可能会导致 GNN 模型的性能下降，因为模型在聚合邻居节点的信息时，高度节点所承载的信息量过多，而低度节点所承载的信息量过少。

举个简单的例子，假设我们有一个社交网络图，其中有一个名人节点，拥有数百万个粉丝 (邻居节点)。在 GNN 模型中，当聚合该名人节点的邻居信息时，需要处理大量的邻居节点输入，这可能使得该节点的表示向量过于平滑，丢失了一些重要的特征信息。相反，如果一个普通用户只有几个朋友节点，那么在聚合邻居信息时，输入数据就过于稀疏，可能无法充分利用 GNN 的邻居聚合机制来提取有用的特征。

为了缓解度数不平衡问题，研究人员提出了一些解决方案，例如：

1.  采样技术：对高度节点的邻居进行采样，只选择部分邻居参与聚合，从而降低计算负担。
    
2.  度数归一化：通过一些归一化技术，缩小不同节点度数之间的差距。
    
3.  虚拟节点：为低度节点人工添加一些虚拟邻居节点，以增加其邻居数量。
    
4.  注意力机制：使用注意力机制对邻居节点进行加权，降低高度节点邻居的权重。
    

### 对称归一化

GraphConv 采用对称归一化，来处理邻接矩阵中可能存在的度数不平衡问题。

对称归一化的公式可以通过以下步骤推导：

1.  首先定义无向图 `` $G = (V,E)$ `` , 其中 V 为节点集合，E 为边集合。
    
2.  令 A 为图 G 的邻接矩阵，其中 `` $A_{ij} \geq 1$ `` 表示节点 `` $i$ `` 和节点 `` $j$ `` 之间存在边，否则 `` $A_{ij} = 0$ ``。
    
3.  定义度矩阵 D, 它是一个对角矩阵，对角线元素为每个节点的度数，即 `` $D_{ii} = \sum _{j}A_{ij}$ ``。
    
4.  我们希望将邻接矩阵 A 归一化，使得每个节点的特征向量在聚合邻居节点特征时，受到邻居节点数量的影响较小。
    
5.  对于每个节点 i, 我们可以计算其特征向量在聚合时的缩放系数：
    
    ```
    
    LATEX_BLOCK___\alpha _{i} = \frac{1}{\sqrt{D_{ii}}}___LATEX_BLOCK
    ```
    
6.  将缩放系数应用于邻接矩阵 A, 得到归一化的邻接矩阵：
    
    ```
    
    LATEX_BLOCK___A\limits^{ \sim } = D^{−\frac{1}{2}}AD^{−\frac{1}{2}}___LATEX_BLOCK
    ```
    
    其中 `` $D^{−\frac{1}{2}}$ `` 表示度矩阵 D 的 - 1/2 次方数。这种归一化方式被称为对称归一化。
    
7.  在实际应用中，为了避免**除以零的问题** , 通常会在度矩阵 D 中加入一个自环，即 `` $D\limits^{ \sim } = D + I$ ``, 其中 I 是单位矩阵。对应的归一化公式为：
    
    ```
    
    LATEX_BLOCK___A\limits^{ \sim } = D\limits^{ \sim }^{−\frac{1}{2}}AD\limits^{ \sim }^{−\frac{1}{2}}___LATEX_BLOCK
    ```
    
    - - -
    

对于节点 `` $i$ ``, 原始的特征向量为 `` $Ax_{i}$ ``, 其范数为：

```

LATEX_BLOCK___\|Ax_{i}\| = \sqrt{\sum \limits_{j \in N(i)}a_{ij}^{2}}___LATEX_BLOCK
```

其中 `` $N(i)$ `` 表示节点 `` $i$ `` 的邻居集合。

而对称归一化后的特征向量为 `` $A\limits^{ \sim }x_{i} = D^{−\frac{1}{2}}AD^{−\frac{1}{2}}x_{i}$ ``, 其范数为：

```

LATEX_BLOCK___\|A\limits^{ \sim }x_{i}\| = \sqrt{\sum \limits_{j \in N(i)}\frac{a_{ij}^{2}}{d_{i}d_{j}}}___LATEX_BLOCK
```

其中 `` $d_{i}$ `` 和 `` $d_{j}$ `` 分别为节点 `` $i$ `` 和 `` $j$ `` 的度数。可以看出，对称归一化后，**高度数节点的特征向量范数会被放大，而低度数节点的特征向量范数会被缩小** , 从而使得不同节点的特征向量范数保持在相同的量级。

这种归一化操作能够减小节度数对特征传播的影响，提高图卷积神经网络的性能。

### 实现

#### PyG

首先给出一个简单版的实现：

```python
 1
import torch
 2
from torch import Tensor
 3
from torch_geometric.nn import MessagePassing
 4
from torch_geometric.typing import OptTensor
 5

 6
class GraphConv(MessagePassing):
 7
    def __init__(self, in_channels: int, out_channels: int):
 8
        super().__init__(aggr='add')
 9
        self.lin = torch.nn.Linear(in_channels, out_channels)
10

11
    def forward(self, x: Tensor, edge_index: Tensor) -> Tensor:
12
        return self.propagate(edge_index, x=x)
13

14
    def message(self, x_j: Tensor) -> Tensor:
15
        return self.lin(x_j)
16

17
    def update(self, aggr_out: Tensor) -> Tensor:
18
        return aggr_out
```

这就是一个最简单的 GraphConv，和上一节 [搭建 Miniconda 管理的 PyG 和 DGL 开发环境](https://www.less-bug.com/posts/build-pyg-and-dgl-development-environments-managed-by-miniconda/) 几乎没有什么不同。我们加点东西：

```python
 1
import torch
 2
from torch import Tensor
 3
from torch_geometric.nn import MessagePassing
 4
from torch_geometric.utils import add_self_loops, degree
 5
torch.manual_seed(42)
 6

 7
class GraphConv(MessagePassing):
 8
    def __init__(self, in_channels: int, out_channels: int):
 9
        super(GraphConv, self).__init__(aggr='add')  # "Add" aggregation.
10
        self.lin = torch.nn.Linear(in_channels, out_channels)
11
        self.bias = torch.nn.Parameter(torch.Tensor(out_channels))
12
        self.reset_parameters()
13

14
    def reset_parameters(self):
15
        torch.nn.init.xavier_uniform_(self.lin.weight)
16
        torch.nn.init.zeros_(self.bias)
17

18
    def forward(self, x: Tensor, edge_index: Tensor):
19
        # Add self-loops to the adjacency matrix.
20
        edge_index, _ = add_self_loops(edge_index, num_nodes=x.size(0))
21

22
        # Compute normalization.
23
        row, col = edge_index
24
        deg = degree(col, x.size(0), dtype=x.dtype)
25
        deg_inv_sqrt = deg.pow(-0.5)
26
        norm = deg_inv_sqrt[row] * deg_inv_sqrt[col]
27

28
        # Start propagating messages.
29
        return self.propagate(edge_index, size=(x.size(0), x.size(0)), x=x, norm=norm)
30

31
    def message(self, x_j: Tensor, norm: Tensor) -> Tensor:
32
        # Normalize node features.
33
        return norm.view(-1, 1) * x_j
34

35
    def update(self, aggr_out: Tensor):
36
        # Add bias after aggregation.
37
        biased = self.lin(aggr_out) + self.bias
38
        return biased
39

40
from torch_geometric.data import Data
41

42
def test_basic():
43
    # init test
44
    in_channels = 16
45
    out_channels = 32
46
    conv = GraphConv(in_channels, out_channels)
47

48
    # prepare input data
49
    x = torch.randn(4, in_channels) # 4x16
50
    edge_index = torch.tensor([[0, 1, 1, 2],
51
                               [1, 0, 2, 1]], dtype=torch.long)
52
    data = Data(x=x, edge_index=edge_index)
53

54
    # foiward test
55
    out = conv(data.x, data.edge_index)
56
    assert out.size() == (4, out_channels)
57

58
    # backward test
59
    if torch.cuda.is_available():
60
        conv = conv.cuda()
61
        data = data.cuda()
62
    out.mean().backward()
63
    assert conv.lin.weight.grad is not None
64
    assert conv.bias.grad is not None
65

66
def test_reset_parameters():
67
    # init test
68
    in_channels = 16
69
    out_channels = 32
70
    conv = GraphConv(in_channels, out_channels)
71

72
    # test parameter initialization
73
    conv.reset_parameters()
74
    assert torch.allclose(conv.bias, torch.zeros(out_channels))
75
    lin_weight = conv.lin.weight
76
    assert torch.allclose(lin_weight, lin_weight.data)
77

78
test_basic()
79
test_reset_parameters()
```

这里，以及后面 DGL 的实现中，我们用到一个 `` $W$ `` 参数初始化方法 xavier\_uniform。

xavier\_uniform 可以将神经网络的权重初始化为适当的值，据说可以促进模型的学习和收敛速度。它会根据输入和输出神经元的数量自动计算一个范围，在这个范围内均匀地随机初始化权重。

这种方法基于这样一种假设：如果权重初始化得太小，信号将在每一层中逐渐消失；如果权重初始化得太大，信号将在每一层中逐渐爆炸。

那么，可以根据参数张量的输入神经元和输出神经元的数量，计算标准差，接着根据标准差计算初始化的范围，将权重均匀地随机初始化在这个范围内。这个范围相对合理，有助于避免梯度消失或梯度爆炸问题。

伪代码实现：

```python
1
import math
2
import torch
3

4
def xavier_uniform_(tensor, gain=1.0):
5
    fan_in, fan_out = torch.nn.init._calculate_fan_in_and_fan_out(tensor)
6
    std = gain * math.sqrt(2.0 / (fan_in + fan_out))
7
    bound = math.sqrt(3.0) * std  # Calculate the range of the uniform distribution
8
    with torch.no_grad():
9
        return tensor.uniform_(-bound, bound)  # Uniformly initialize the tensor within the calculated range
```

- - -

forward 中我们用了 add\_self\_loops 这个函数，它的作用是 **添加自环边**。函数原型如下所示：

```python
1
def add_self_loops(edge_index, num_nodes=None, edge_weight=None, fill_value=1.0):
2
    # return the edge index with self loop added
```

在这个函数中，参数 edge\_index 代表图的边索引，num\_nodes 代表节点的数量，edge\_weight 代表边的权重，fill\_value 则是自环边的值。

自环边：是指起点和终点为同一个节点的边。作用：添加自环边可以帮助节点捕获自身特征，有助于提高模型的稳定性和泛化能力，还能避免对称归一化除以 0 的问题。

- - -

接下来我们进行对称归一化（symmetric normalization）：用 degree 求每个节点的度。degree 返回的是一个包含每个节点度数的向量。然后对度求倒数的平方根。最后用 deg\_inv\_sqrt 向量对边的权重进行归一化。

```python
1
        # Compute normalization.
2
        row, col = edge_index
3
        deg = degree(col, x.size(0), dtype=x.dtype)
4
        deg_inv_sqrt = deg.pow(-0.5)
5
        norm = deg_inv_sqrt[row] * deg_inv_sqrt[col]
```

- - -

最后我们用 `self.propagate` 传播信息，它会从邻居节点聚合信息到当前节点，执行更新等。

#### DGL

```python
 1
import dgl
 2
import torch
 3
import torch.nn as nn
 4
import torch.nn.functional as F
 5

 6
class GraphConv(nn.Module):
 7
    def __init__(self, in_channels: int, out_channels: int):
 8
        super(GraphConv, self).__init__()
 9
        self.weight = nn.Parameter(torch.Tensor(in_channels, out_channels))
10
        self.bias = nn.Parameter(torch.Tensor(out_channels))
11
        self.reset_parameters()
12

13
    def reset_parameters(self):
14
        torch.nn.init.xavier_uniform_(self.weight)
15
        torch.nn.init.zeros_(self.bias)
16

17
    def forward(self, g, h):
18
        with g.local_scope():
19
            h = torch.matmul(h, self.weight)
20
            g.ndata['h'] = h
21
            g.update_all(dgl.function.copy_u('h', 'm'), dgl.function.sum('m', 'h_neigh'))
22
            h_neigh = g.ndata['h_neigh']
23
            return h_neigh + self.bias
```

## 实现 GATConv

### 原理

GATConv 使用注意力机制来聚合邻居节点的信息。它的核心思想是学习每个邻居节点的重要性（而不是简单地将它们的信息平均聚合）。

```

LATEX_BLOCK___h_{i}^{(l + 1)} = \sum \limits_{j \in N(i)}\alpha _{i,j}W^{(l)}h_{j}^{(l)}___LATEX_BLOCK
```

其中 `` $\alpha _{ij}$ `` 是节点 `` $i$ `` 和节点 `` $j$ `` 之间的注意力分数：

```

LATEX_BLOCK___\begin{matrix} \alpha _{ij}^{l} &  = soft\max _{i}(e_{ij}^{l}) \\ e_{ij}^{l} &  = LeakyReLU\left(a\limits^{\rightarrow }^{T}[Wh_{i}\|Wh_{j}\right.\right.\right. \\  \end{matrix}___LATEX_BLOCK
```

详解：

-   首先计算一个中间变量 `` $e_{ij}$ ``：将节点 `` $i$ `` 和节点 `` $j$ `` 的隐藏表示 `` $Wh_{i}$ `` 和 `` $Wh_{j}$ `` **连接** 起来，然后使用一个单层前馈神经网络 (包含一个 LeakyReLU 激活函数) 来计算。
    
-   然后对 `` $e_{ij}$ `` 应用 softmax 函数，得到最终的注意力分数 `` $\alpha _{ij}$ ``。这个分数表示的是节点 `` $j$ `` 在 计算 节点 `` $i$ `` 的下一层表示 时的重要性。
    

### 实现

#### PyG

```python
 1
import torch
 2
from torch import Tensor
 3
from torch_geometric.nn.conv import MessagePassing
 4

 5
from torch_geometric.utils import softmax
 6

 7
class GATConv(MessagePassing):
 8
    def __init__(self, in_channels: int, out_channels: int):
 9
        super().__init__(aggr='add')  # "Addition" aggregation.
10
        self.in_channels = in_channels
11
        self.out_channels = out_channels
12

13
        self.lin = torch.nn.Linear(in_channels, out_channels, bias=False)
14
        self.att = torch.nn.Linear(2 * out_channels, 1, bias=False)
15
        self.act = torch.nn.LeakyReLU() # not a real layer, just for activation
16

17
    def forward(self, x: Tensor, edge_index: Tensor) -> Tensor:
18
        x = self.lin(x)
19

20
        # compute attention coefficients based on edge features e_ij
21
        edge_attr = torch.cat([x[edge_index[0]], x[edge_index[1]]], dim=-1)
22
        edge_attr = self.act(self.att(edge_attr))
23

24
        # alpha_ij is the normalized attention scores
25
        alpha = softmax(edge_attr, edge_index[1])
26

27
        # calc message passing with attention scores
28
        out = self.propagate(edge_index, x=x, alpha=alpha)
29

30
        return out
31

32
    def message(self, x_j: Tensor, alpha: Tensor) -> Tensor:
33
        # x_j is the input node features, alpha is the attention scores as weights
34
        return alpha * x_j
35

36

37
def test_gatconv():
38
    edge_index = torch.tensor([[0,1,1,2,2,4],[2,0,2,3,4,3]])
39
    x = torch.ones((5, 8))
40
    conv = GATConv(8, 4)
41
    output = conv(x, edge_index)
42
    print(output)
43

44
test_gatconv()
```

我们定义了

-   一个全连接层，用来线性变换输入输出 -、
    
-   一个注意力曾，将两个节点的输出特征拼接后的维度（`2 * out_channels`）映射到 1 维，生成一个标量注意力系数。
    
-   用 LeakyReLU 做激活
    

前向传播的时候：

-   首先通过 `self.lin(x)` 对输入特征 `x` 进行线性变换。
    
-   计算注意力系数：拼接边的两端节点的特征，然后通过 `self.att` 线性层和 `self.act` 激活函数。
    
-   用 `softmax` 函数归一化系数，确保从一个节点流出的所有注意力系数之和为 1。
    
-   最后，调用 `self.propagate` 方法进行消息传递。`self.propagate` 方法内部会调用 `message` 方法来计算每条边上传递的消息
    

消息传递：

消息定义为是边的源节点特征 `x_j` 与其对应的注意力系数 `alpha` 的乘积。即是说每个节点接收到的来自邻居的信息，是根据邻居的重要性加权。

#### DGL

```python
 1
import torch
 2
import torch.nn as nn
 3
import dgl
 4

 5
class GATConv(nn.Module):
 6
    def __init__(self, in_feats, out_feats):
 7
        super(GATConv, self).__init__()
 8
        self.fc = nn.Linear(in_feats, out_feats, bias=False)
 9
        self.attn_fc = nn.Linear(2 * out_feats, 1, bias=False)
10
        self.reset_parameters()
11

12
    def reset_parameters(self):
13
        gain = nn.init.calculate_gain('relu')
14
        nn.init.xavier_uniform_(self.fc.weight, gain=gain)
15
        nn.init.xavier_uniform_(self.attn_fc.weight, gain=gain)
16

17
    def edge_attention(self, edges):
18
        z2 = torch.cat([edges.src['z'], edges.dst['z']], dim=1)
19
        a = self.attn_fc(z2)
20
        return {'e': a}
21

22
    def message_func(self, edges):
23
        return {'z': edges.src['z'], 'e': edges.data['e']}
24

25
    def reduce_func(self, nodes):
26
        alpha = torch.softmax(nodes.mailbox['e'], dim=1)
27
        h = torch.sum(alpha * nodes.mailbox['z'], dim=1)
28
        return {'h': h}
29

30
    def forward(self, g, x):
31
        z = self.fc(x)
32
        g.ndata['z'] = z
33
        g.apply_edges(self.edge_attention)
34
        g.update_all(self.message_func, self.reduce_func)
35
        return g.ndata.pop('h')
36

37
def test_gatconv():
38
    edge_index = torch.tensor([[0,1,1,2,2,4],[2,0,2,3,4,3]])
39
    h = torch.ones((5, 8))
40
    g = dgl.graph((edge_index[0], edge_index[1]))
41
    conv = GATConv(8, 4)
42
    output= conv(g, h)
43
    print(output)
44

45
test_gatconv()
```

初始化时类似。这里 DGL 版本我们试了一下不用激活函数，直接套 softmax。

前向传播：

-   也是线性变换后算注意力系数。使用 `apply_edges` 方法应用 `edge_attention` 函数计算所有边的注意力系数。最后，通过 `update_all` 方法同时执行消息传递和聚合操作，更新所有节点的特征。

消息传递：

-   `message_func` 方法定义每条边要传递的信息：源节点的变换后特征 `z` 和计算得到的注意力系数 `e`。
    
-   `reduce_func` 可以处理来自不同邻居的消息。先对注意力系数进行 softmax 归一化，然后使用归一化后的注意力系数加权邻居节点的特征，并将加权特征求和以更新每个节点的特征。
    

这里我们第一次用到了 mailbox。`nodes.mailbox` 是一个临时向量，用来存储收到的信息。`nodes.mailbox['msg']` 可以同时访问所有节点收到的 `msg` 所合成的向量。

## 多头 GATConv

### 原理

多头图注意力卷积 (Multi-head Graph Attention Convolution, GATConv) 是图神经网络中常用的一种卷积操作，它将注意力机制引入到图卷积中，可以为不同的邻居节点分配不同的权重，增强图神经网络的表达能力。下面我来介绍多头 GATConv 的原理并给出代码实现。

多头 GATConv 的公式可以表示为：

```

LATEX_BLOCK___x_{i}^{(l + 1)} = \bigoplus \limits_{k = 1}^{K}\sigma \left(\sum \limits_{j \in N(i)}\alpha _{ij}^{k}W^{k}x_{j}^{(l)}\right)___LATEX_BLOCK
```

其中，

-   `` $x_{i}^{(l)}$ `` 表示第 `` $l$ `` 层第 `` $i$ `` 个节点的特征向量
    
-   `` $x_{i}^{(l + 1)}$ `` 表示第 `` $l + 1$ `` 层第 `` $i$ `` 个节点的特征向量
    
-   `` $N(i)$ `` 表示节点 `` $i$ `` 的邻居节点集合
    
-   `` $K$ `` 表示注意力头的数量
    
-   `` $W^{k}$ `` 是第 `` $k$ `` 个注意力头的权重矩阵
    
-   `` $\alpha _{ij}^{k}$ `` 是第 `` $k$ `` 个注意力头计算出的节点 `` $i$ `` 到节点 `` $j$ `` 的注意力权重
    
-   `` $\sigma $ `` 是激活函数（通常为 ReLU）
    
-   `` $ \oplus $ `` 表示拼接操作。
    

多头 GATConv 的计算过程：

1.  对于每个注意力头 `` $k$ ``，使用权重矩阵 `` $W^{k}$ `` 对节点特征进行线性变换：`` $x_{j}^{k} = W^{k}x_{j}^{(l)}$ ``。
    
2.  计算节点 `` $i$ `` 到其邻居节点 `` $j$ `` 的注意力权重 `` $e_{ij}^{k}$ ``：`` $e_{ij}^{k} = \text{LeakyReLU}(a^{k^{T}}[x_{i}^{k}\|x_{j}^{k}\right.\right.\right.$ ``，其中 `` $a^{k}$ `` 是第 `` $k$ `` 个注意力头的注意力向量，`` $\|\right.$ `` 表示拼接操作。
    
3.  使用 softmax 函数对注意力权重进行归一化：`` $\alpha _{ij}^{k} = \text{softmax}_{j}(e_{ij}^{k}) = \frac{\exp ⁡(e_{ij}^{k})}{\sum \limits_{j^{′} \in N(i)}\exp ⁡(e_{ij^{′}}^{k})}$ ``。
    
4.  对于每个注意力头 `` $k$ ``，使用注意力权重对邻居节点的特征进行加权求和：`` $x_{i}^{k} = \sigma (\sum \limits_{j \in N(i)}\alpha _{ij}^{k}x_{j}^{k})$ ``。
    
5.  将所有注意力头的结果拼接起来得到最终的节点表示：`` $x_{i}^{(l + 1)} = \bigoplus \limits_{k = 1}^{K}x_{i}^{k}$ ``。
    

### 实现（PyG）

```python
 1
import torch
 2
from torch import Tensor
 3
from torch_geometric.nn.conv import MessagePassing
 4
from torch_geometric.utils import softmax
 5

 6
class GATConv(MessagePassing):
 7
    def __init__(self, in_channels: int, out_channels: int, heads: int, concat: bool):
 8
        super().__init__(aggr='add')
 9
        self.in_channels = in_channels
10
        self.out_channels = out_channels
11
        self.heads = heads
12
        self.concat = concat
13

14
        self.lin = torch.nn.Linear(in_channels, heads * out_channels, bias=False)
15
        self.att = torch.nn.Parameter(torch.Tensor(1, heads, 2 * out_channels))
16
        self.reset_parameters()
17

18
    def reset_parameters(self):
19
        torch.nn.init.xavier_uniform_(self.lin.weight)
20
        torch.nn.init.xavier_uniform_(self.att)
21

22
    def forward(self, x: Tensor, edge_index: Tensor) -> Tensor:
23
        # x.shape = (num_nodes, in_channels)
24
        x = self.lin(x)
25
        # x.shape = (num_nodes, heads * out_channels)
26
        out = self.propagate(edge_index, x=x, size=None)
27
        # out.shape = (num_nodes, heads * out_channels)
28

29
        if self.concat:
30
            out = out
31
            # out.shape = (num_nodes, heads * out_channels)
32
        else:
33
            out = out.view(-1, self.heads, self.out_channels).mean(dim=1)
34
            # out.shape = (num_nodes, out_channels)
35

36
        return out
37

38
    def message(self, x_i: Tensor, x_j: Tensor, index: Tensor, size_i: int) -> Tensor:
39
        # x_i.shape = (num_edges, heads * out_channels)
40
        # x_j.shape = (num_edges, heads * out_channels)
41
        x_i = x_i.view(-1, self.heads, self.out_channels)
42
        x_j = x_j.view(-1, self.heads, self.out_channels)
43
        x = torch.cat([x_i, x_j], dim=-1)
44
        # x.shape = (num_edges, heads, 2 * out_channels)
45
        alpha = (x * self.att).sum(dim=-1)
46
        # alpha.shape = (num_edges, heads)
47
        alpha = softmax(alpha, index, size_i)
48
        # alpha.shape = (num_edges, heads)
49
        return (x_j * alpha.unsqueeze(-1)).view(-1, self.heads * self.out_channels)
50
        # return.shape = (num_edges, heads * out_channels)
51

52

53

54
def test_gatconv():
55
    edge_index = torch.tensor([[0, 1, 1, 2, 2, 4], [2, 0, 2, 3, 4, 3]])
56
    x = torch.ones((5, 8))  # 5 nodes with 8-dimensional features
57
    heads = 2  # Number of attention heads
58
    concat = True  # Whether to concatenate or average the attention heads
59
    conv = GATConv(8, 4, heads=heads, concat=concat)
60
    output = conv(x, edge_index)
61
    print(output.shape)
62
    print(output)
63

64
test_gatconv()
```

这里我发现一个坑点，`self.propagate` 的输入一定要是二维的 Shape，否则遇到类似这样的报错：

```plain
IndexError: Found indices in 'edge_index' that are larger than 1 (got 4). Please ensure that all indices in 'edge_index' point to valid indices in the interval [0, 2) in your node feature matrix and try again.
```

## SAGEConv

### 原理

GraphSAGE（SAmple and aggreGatE）层来自 [Inductive Representation Learning on Large Graphs](https://arxiv.org/pdf/1706.02216.pdf) 论文。

SAGEConv 通过聚合邻居节点的信息来更新当前节点的表示

```

LATEX_BLOCK___\begin{matrix} \begin{matrix}  \\ h_{N(i)}^{(l + 1)} &  = aggregate\left(\{e_{ji}h_{j}^{l},∀j \in N(i)\}\right) \\ h_{i}^{(l + 1)} &  = \sigma \left(W \cdot concat(h_{i}^{l},h_{N(i)}^{l + 1})\right) \\ h_{i}^{(l + 1)} &  = norm(h_{i}^{(l + 1)}) \\  \end{matrix} \\  \end{matrix}___LATEX_BLOCK
```

其中 `` $e_{ji}$ `` 是从节点 `` $j$ `` 到节点 `` $i$ `` 的边的标量权重。需要确保 `` $e_{ji}$ `` 可以与 `` $h_{j}^{l}$ `` 进行广播。

1.  公式 1 表示聚合邻居节点的特征，其中 `` $N(i)$ `` 表示节点 `` $i$ `` 的邻居节点集合，`` $e_{ji}$ `` 是从节点 `` $j$ `` 到节点 `` $i$ `` 的边的标量权重，用于加权邻居节点的特征 `` $h_{j}^{l}$ ``。
    
2.  公式 2 更新当前节点的表示。使用当前节点的特征 `` $h_{i}^{l}$ `` 和聚合的邻居节点信息 `` $h_{N(i)}^{l + 1}$ `` 来更新当前节点的表示 `` $h_{i}^{(l + 1)}$ ``。其中 `` $W$ `` 是可学习的权重矩阵，`` $\sigma $ `` 是激活函数。
    
3.  公式 3 是特征归一化，确保训练过程的稳定。
    

### 实现

#### PyG

```python
 1
import torch
 2
from torch_geometric.nn import MessagePassing
 3
from torch_geometric.utils import add_self_loops, degree
 4

 5
class SAGEConv(MessagePassing):
 6
    def __init__(self, in_channels, out_channels):
 7
        super(SAGEConv, self).__init__(aggr='mean')  # "mean"
 8
        self.lin = torch.nn.Linear(in_channels, out_channels)
 9
        self.act = torch.nn.ReLU()
10

11
    def forward(self, x, edge_index):
12
        edge_index, _ = add_self_loops(edge_index, num_nodes=x.size(0))
13

14
        row, col = edge_index
15
        deg = degree(col, x.size(0), dtype=x.dtype)
16
        deg_inv_sqrt = deg.pow(-0.5)
17
        norm = deg_inv_sqrt[row] * deg_inv_sqrt[col]
18

19
        return self.propagate(edge_index, x=x, norm=norm)
20

21
    def message(self, x_j, norm):
22
        return norm.view(-1, 1) * x_j
23

24
    def update(self, aggr_out):
25
        return self.act(self.lin(aggr_out))
26

27
def test_sageconv():
28
    edge_index = torch.tensor([[0,1,1,2,2,4],[2,0,2,3,4,3]])
29
    x = torch.ones((5, 8))
30
    conv = SAGEConv(8, 4)
31
    output = conv(x, edge_index)
32
    print(output)
33

34
test_sageconv()
```

#### DGL

```python
 1
import torch
 2
from torch import nn
 3
import dgl
 4
import dgl.function as fn
 5

 6
class SAGEConv(nn.Module):
 7
    def __init__(self, in_channels: int, out_channels: int):
 8
        super().__init__()
 9
        self.fc = nn.Linear(in_channels * 2, out_channels)
10
        self.act = nn.ReLU()
11

12
    def forward(self, g: dgl.DGLGraph, h: torch.Tensor) -> torch.Tensor:
13
        with g.local_scope():
14
            g.ndata['h'] = h
15
            g.update_all(fn.copy_u('h', 'm'), fn.mean('m', 'neigh'))
16
            neigh = g.ndata['neigh']
17
            return self.act(self.fc(torch.cat([h, neigh], dim=1)))
18

19
def test_sageconv():
20
    edge_index = torch.tensor([[0,1,1,2,2,4],[2,0,2,3,4,3]])
21
    h = torch.ones((5, 8))
22
    g = dgl.graph((edge_index[0], edge_index[1]))
23
    conv = SAGEConv(8, 4)
24
    output= conv(g, h)
25
    print(output)
26

27
test_sageconv()
```

这里我们都只是用了最简单的聚合器，参考中的《GraphSAGE 源码解析 - 知乎》一文中有更多聚合器的实现参考（GCN+mean+LSTM+pool）。

### 对比 GraphConv

和 GraphConv 不同，SAGEConv 使用的聚合函数是灵活可配置的，可以是平均、最大值、LSTM 等，一般用平均。另外，GraphConv 只使用邻居节点的特征进行聚合（一般是加法），SAGEConv 将当前节点的特征 `` $h_{i}^{l}$ `` 与聚合的邻居节点特征 `` $h_{N(i)}^{l + 1}$ `` 进行**拼接** , 然后通过全连接层进行**特征融合**，使得模型拥有更强的表达能力。

## 参考

-   GAMMA Lab Onoboard 手册
    
-   [图神经网络中的注意力机制\_gatv2conv-CSDN 博客](https://blog.csdn.net/morgan777/article/details/121960955)
    
-   Dr. Claude 3 Opus（狗头）
    
-   [【Code】GraphSAGE 源码解析 - 知乎](https://zhuanlan.zhihu.com/p/142205899)
