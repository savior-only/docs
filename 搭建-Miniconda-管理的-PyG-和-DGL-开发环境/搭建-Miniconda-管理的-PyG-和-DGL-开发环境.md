---
title: 搭建 Miniconda 管理的 PyG 和 DGL 开发环境
url: https://www.less-bug.com/posts/build-pyg-and-dgl-development-environments-managed-by-miniconda/
clipped_at: 2024-03-24 08:34:30
category: default
tags: 
 - www.less-bug.com
---


# 搭建 Miniconda 管理的 PyG 和 DGL 开发环境

## 环境

-   **OS**: Ubuntu 22.04.4 LTS on Windows 10 x86
    
-   **Kernel**: 5.15.146.1-microsoft-standard-WS
    

## 安装 PyTorch Geometric

步骤如下:

1.  **打开终端(Terminal)或命令提示符(Command Prompt)。**
    
2.  **创建一个新的 Conda 环境:**
    

```plain
conda create -n graph_env python=3.11
```

这将创建一个名为 `graph_env` 的新环境,使用 Python 3.11 版本。你可以根据需要选择其他 Python 版本。但是目前暂时别装 3.12，很多库还不支持。

3.  **激活新创建的环境:**

```plain
conda activate graph_env
```

4.  **安装 PyTorch:**

```plain
conda install pytorch==2.0.1 torchvision torchaudio pytorch-cuda=11.8 -c pytorch -c nvidia
```

这里我们安装的是 PyTorch 2.0.1 版本,并启用了 CUDA 11.8 支持。

5.  **安装 PyG:**

```plain
pip install pyg_lib torch_scatter torch_sparse torch_cluster torch_spline_conv torch_geometric -f https://data.pyg.org/whl/torch-2.0.1+cpu.html
```

6.  **验证安装:**

进入 Python 交互式环境,并导入 PyTorch 和 PyG:

```python
1
import torch
2
import torch_geometric
3
print(torch.__version__)
4
print(torch_geometric.__version__)
```

如果没有错误,则表示安装成功。

7.  **开始使用 PyG 进行开发。**

就是这些步骤!现在你已经在 Miniconda 下创建了一个干净的 PyG 开发环境。根据你的具体需求,你可以继续安装其他必需的库和依赖项。

## 安装 DGL

在上述基础上

```plain
conda install -c dglteam/label/cu118 dgl
pip install torchdata==0.6.1 PyYAML pydantic
conda install pandas
```

## 实验：实现简单的卷积

### 原理

图神经网络(GNN)的消息传递公式：

```

LATEX_BLOCK___h_{i}^{(l + 1)} = \sigma (b^{(l)} + \sum \limits_{j \in N(i)}e_{ij}h_{j}^{(l)}W^{(l)})___LATEX_BLOCK
```

用它计算每个节点的新特征表示。其中：

-   `` $h_{i}^{(l + 1)}$ `` 表示节点 `` $i$ `` 在第 `` $(l + 1)$ `` 层的新特征向量。
    
-   `` $\sigma $ `` 是激活函数,通常使用 ReLU 或其他非线性函数。
    
-   `` $N(i)$ `` 表示与节点`` $i$ ``相邻的所有节点的集合。
    
-   `` $e_{ij}$ `` 是连接节点 `` $i$ `` 和 `` $j$ `` 的边的权重, 可以是二值(0或1)或者其他实数。
    
-   `` $h_{j}^{(l)}$ `` 是节点 `` $j$ `` 在第 `` $l$ `` 层的特征向量。
    
-   `` $W^{(l)}$ `` 是第 `` $l$ `` 层的可学习参数矩阵,用于线性变换邻居节点的特征。
    
-   `` $b^{(l)}$ `` 是第 `` $l$ `` 层的偏置项。
    

这个公式的计算过程如下:

1.  遍历节点 `` $i$ `` 的所有邻居节点 `` $j$ ``。
    
2.  将邻居节点 `` $j$ `` 的特征 `` $h_{j}^{(l)}$ `` 与边权重 `` $e_{ij}$ `` 相乘, 得到加权特征。
    
3.  将所有加权邻居特征相加。
    
4.  将相加结果与可学习参数 `` $W^{(l)}$ `` 进行线性变换。
    
5.  加上偏置项 `` $b^{(l)}$ ``。
    
6.  通过激活函数 `` $\sigma $ `` 得到节点`` $i$ ``在第 `` $(l + 1)$ `` 层的新特征 `` $h_{i}^{(l + 1)}$ ``。
    

### PyG 实现

```python
 1
import torch
 2
import torch.nn as nn
 3
from torch import Tensor
 4
from torch_geometric.nn.conv import MessagePassing
 5

 6
class PyG_conv(MessagePassing):
 7
    def __init__(self, in_channel: int, out_channel: int):
 8
        super().__init__()
 9
        self.in_channel = in_channel
10
        self.out_channel = out_channel
11
        self.W = nn.Parameter(torch.ones((in_channel, out_channel)))
12
        self.b = nn.Parameter(torch.ones(out_channel))
13

14
    def forward(self, x: Tensor, edge_index: Tensor, edge_weight: Tensor):
15
        out = self._propagate_impl(edge_index, x, edge_weight)
16
        return out
17

18
    def _propagate_impl(self, edge_index: Tensor, x: Tensor, edge_weight: Tensor):
19
        src, dst = edge_index
20
        num_nodes = x.size(0)
21
        num_edges = edge_index.size(1)
22
        out = torch.zeros(num_nodes, self.in_channel, device=x.device)
23

24
        for i in range(num_edges):
25
            msg = self.message(x[src[i]], edge_weight[i])
26
            out[dst[i]] = out[dst[i]] + msg
27

28
        out = self._update_impl(out)
29
        return out
30

31
    def _update_impl(self, out: Tensor) -> Tensor:
32
        return out @ self.W + self.b
33

34
    def message(self, x_j: Tensor , edge_weight: Tensor) -> Tensor:
35
        return edge_weight.view(-1, 1) * x_j
36

37

38
if __name__ == '__main__':
39
    import numpy as np
40
    # 2x6 tensor, represents the connectivity of the points,
41
    # e.g. the first edge connects node 0 and node 2
42
    edge_index = torch.tensor([[0,1,1,2,2,4],[2,0,2,3,4,3]])
43
    # 5 nodes and 8 features per node
44
    x = torch.ones((5, 8))
45
    # 6 edges and uniform edge weight of 2
46
    edge_weight = 2 * torch.ones(6)
47
    conv = PyG_conv(8, 4)
48
    output = conv(x, edge_index, edge_weight)
49

50
    assert np.allclose(output.detach().numpy(), [
51
        [17., 17., 17., 17.],
52
        [ 1.,  1.,  1.,  1.],
53
        [33., 33., 33., 33.],
54
        [33., 33., 33., 33.],
55
        [17., 17., 17., 17.]
56
    ])
```

上面的写法是为了体现原理，下面给出更工程的写法：

```python
 1
class PyG_conv(MessagePassing):
 2
    def __init__(
 3
        self,
 4
        in_channel: int,
 5
        out_channel: int,
 6
    ):
 7
        ...
 8

 9
    def forward(self, x: Tensor, edge_index: Tensor, edge_weight: Optional[Tensor] = None) -> Tensor:
10
        return self.propagate(edge_index, x=x, edge_weight=edge_weight)
11

12
    def message(self, x_j: Tensor, edge_weight: Optional[Tensor]) -> Tensor:
13
        return edge_weight.view(-1, 1) * x_j
14

15
    def update(self, aggr_out: Tensor) -> Tensor:
16
        return torch.matmul(aggr_out, self.W) + self.b
```

### DGL 实现

```python
 1
import torch
 2
import torch.nn as nn
 3
from torch import Tensor
 4
from torch_geometric.nn.conv import MessagePassing
 5

 6
import dgl
 7
import dgl.function as fn
 8

 9
class DGL_conv(nn.Module):
10
    def __init__(self, in_channel: int, out_channel: int):
11
        super().__init__()
12
        self.in_channel = in_channel
13
        self.out_channel = out_channel
14
        self.W = nn.Parameter(torch.ones(in_channel, out_channel))
15
        self.b = nn.Parameter(torch.ones(out_channel))
16

17
    def forward(self, g: Tensor, h: Tensor, edge_weight: Tensor) -> Tensor:
18
        with g.local_scope():
19
            g.ndata['h'] = h
20
            g.edata['w'] = edge_weight
21
            # fn.u_mul_e('h', 'w', 'm'):
22
            #   h x w -> m
23
            #   u means node features, e means edge features, m means message.
24
            #   multiplies the node features h with the edge weights w and stores the result in the message m.
25
            #   returns a EdgeFlow object
26
            # fn.sum('m', 'h'):
27
            #   m -> h
28
            #   aggregates the messages m by summing them up and stores the result in the node features h.
29
            g.update_all(fn.u_mul_e('h', 'w', 'm'), fn.sum('m', 'h'))
30
            h = g.ndata['h']
31
            h = torch.matmul(h, self.W) + self.b
32

33
        return h
34

35

36
if __name__ == '__main__':
37
    import numpy as np
38
    src = torch.tensor([0, 1, 1, 2, 2, 4])
39
    dst = torch.tensor([2, 0, 2, 3, 4, 3])
40
    h = torch.ones((5, 8))
41
    g = dgl.graph((src, dst))
42
    edge_weight = 2 * torch.ones(6)
43
    conv = DGL_conv(8, 4)
44
    output = conv(g, h, edge_weight)
45
    assert np.allclose(output.detach().numpy(), [
46
       [17., 17., 17., 17.],
47
       [ 1.,  1.,  1.,  1.],
48
       [33., 33., 33., 33.],
49
       [33., 33., 33., 33.],
50
       [17., 17., 17., 17.]
51
    ])
```

`g.update_all()` 函数是 DGL 中的一个重要函数,它负责在图上执行消息传递和节点特征更新的过程。在 DGL 中,`fn.u_mul_e('h', 'w', 'm')` 和 `fn.sum('m', 'h')` 这样的函数并不会直接执行，而是需要等到调用 `g.update_all()` 函数的时候才会真正执行。

这是因为 DGL 采用了延迟执行的机制。当你调用这些函数时,它们只是创建了一些计算图节点,并将它们存储在图对象中,等待最终的 `g.update_all()` 函数被调用。

当调用 `g.update_all()` 函数时,DGL 会遍历图中的所有节点和边,并按照之前定义的计算图节点,依次执行消息传递和节点特征更新的操作。这种方式可以大大提高计算效率,因为它可以将多个操作进行合并和优化,减少不必要的计算。