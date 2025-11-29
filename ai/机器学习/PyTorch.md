PyTorch  是用于机器学习计算的包

有以下几个模块

+ torch ： 基础模块（张量计算）
+ torch.nn：神经网络层、损失函数等。
+ torch.autograd：自动微分（反向传播）。
+ torch.optim ： 优化器
+ torch.utils ： 工具函数
+ torch.utils.data   ： 数据加载



## 张量（Tensor）

是 PyTorch 中的核心数据结构，用于存储和操作多维数组。 可以使用GPU进行计算加速

+ 维度  数字是0维 ，向量是1维，矩阵是2维
+ 形状： 张量的形状是指每个维度上的大小。例如，一个形状为`(3, 4)`的张量意味着它有3行4列
+ 数据类型  int8,int32

```python
import torch

# 创建一个 2x3 的全 0 张量
a = torch.zeros(2, 3)
print(a)

# 创建一个 2x3 的全 1 张量
b = torch.ones(2, 3)
print(b)

# 创建一个 2x3 的随机数张量
c = torch.randn(2, 3)
print(c)

# arange :创建等间隔数值序列,返回的结构是向量 [0,2,4,6]
torch.arange(0, 8,2)

# 从 NumPy 数组创建张量
import numpy as np
numpy_array = np.array([[1, 2], [3, 4]])
tensor_from_numpy = torch.from_numpy(numpy_array)
print(tensor_from_numpy)

# 在指定设备（CPU/GPU）上创建张量
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
d = torch.randn(2, 3, device=device)
print(d)
```

## 形状

```python
# 张量的形状
print(g.shape)  # 返回形状
print(g.dim()) # 返回维度
```

张量在运算时存在广播规则：就是对张量进行拓展保证与运算的维度相同。

广播规则有下面几个要求

+ 右对齐，缺失的维度补 1

+ 维度相同 可以直接运算

+ 其中一个维度是1 的，可以拓展到另一个维度

  ```
  (2, 3) + (1, 3) → OK  (第二个张量第一维是 1)
  (5, 1, 4) + (1, 7, 4) → OK
  ```


## 维度变化

技巧： 先看原始向量的size， size的下标从0开始

```python
# 新增维度
x = torch.tensor([1, 2, 3, 4])
print(f"原始: {x.shape}")  # torch.Size([4])

# 增加行维度（变成单行矩阵）
x_row = x.unsqueeze(0)     # 或 x[None, :],在下标为0的地方 添加维度1 torch.Size([4]) =》torch.Size([1, 4]) 
print(f"行向量: {x_row.shape}")  #  


# 增加列维度（变成单列矩阵）
x_col = x.unsqueeze(1)     # 或 x[:, None]
print(f"列向量: {x_col.shape}")  # torch.Size([4, 1])
```

```python
# 移除维度
x = torch.randn(1, 3, 1, 5)
print(f"原始形状: {x.shape}")  # torch.Size([1, 3, 1, 5])

y = x.squeeze()  # 移除所有长度为1的维度
print(f"squeeze后: {y.shape}")  # torch.Size([3, 5])

z = x.squeeze(0)  # 只移除第0维
print(f"squeeze(0)后: {z.shape}")  # torch.Size([3, 1, 5])
```

```python
x = torch.arange(12)  # tensor([0, 1, 2, ..., 11])
print(f"原始: {x.shape}")  # torch.Size([12])

# 重塑为2x6矩阵
x_2d = x.view(2, 6)
print(f"2x6矩阵:\n{x_2d}")

# 重塑为3x2x2的3维张量
x_3d = x.reshape(3, 2, 2)
print(f"3x2x2张量:\n{x_3d}")
```

```python
x = torch.tensor([[1, 2, 3],
                  [4, 5, 6]])
print(f"原始:\n{x}, 形状: {x.shape}")

# 转置
x_t = x.T  # 或 x.transpose(0, 1)
print(f"转置:\n{x_t}, 形状: {x_t.shape}")

# 维度交换
x_3d = torch.randn(2, 3, 4)
x_permuted = x_3d.permute(2, 0, 1)  # 维度顺序从[0,1,2]变为[2,0,1] 重新排列所有维度
print(f"维度交换后形状: {x_permuted.shape}")  # torch.Size([4, 2, 3])


# 对于4维张量：
# 正索引：   0,    1,   2,  3
# 负索引：  -4,   -3,  -2, -1
torch.transpose(input, dim0, dim1) # input:输入张量 ; dim0: 要交换的第一个维度; dim1: 要交换的第二个维度
# 转置：交换第0维和第1维
x_t = torch.transpose(x, 0, 1)
print("\n转置后:")
print(x_t)
print(f"形状: {x_t.shape}")  # torch.Size([3, 2])
```



## 张量运算

```python
# 矩阵乘法要求： (m×n) @ (n×p) = (m×p)
A = torch.randn(2, 3)    # 2x3
B = torch.randn(3, 4)    # 3x4
C = A @ B     
```

```python
# 张量相加
e = torch.randn(2, 3)
f = torch.randn(2, 3)
print(e + f)

# 逐元素乘法
print(e * f)
```



## softmax

```python
torch.softmax 是一个用于将数值转换为概率分布的函数

import torch

scores = torch.tensor([
    [1.0, 2.0, 3.0, 4.0],
    [2.0, 2.0, 2.0, 2.0], 
    [0.1, 0.1, 0.1, 0.1]
])

# dim=0: 沿着行方向（列方向计算）,softmax在哪个维度计算，那个维度上的和就为1；0维是行，那么就是多个行的和为1
prob_dim0 = torch.softmax(scores, dim=0)
# 计算的结果为：
tensor([[0.2424, 0.4652, 0.7028, 0.8654],
        [0.6590, 0.4652, 0.2585, 0.1171],
        [0.0986, 0.0696, 0.0387, 0.0175]])
```



```python
import math

# 两种形式：
math.log(x)        # 计算x的自然对数（以e为底）
math.log(x, base)  # 计算以base为底的x的对数
```

