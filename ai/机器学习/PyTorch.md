PyTorch  是用于机器学习计算的包

有以下几个模块

+ torch ： 基础模块（张量计算）
+ torch.nn：神经网络层、损失函数等。
+ torch.autograd：自动微分（反向传播）。
+ torch.optim ： 优化器
+ torch.utils ： 工具函数
+ torch.utils.data   ： 数据加载



# torch

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



```python
# 张量相加
e = torch.randn(2, 3)
f = torch.randn(2, 3)
print(e + f)

# 逐元素乘法
print(e * f)

# 张量的转置
g = torch.randn(3, 2)
print(g.t())  # 或者 g.transpose(0, 1)

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


## zeros

```
# 创建8行12列 全部都是0的张量
torch.zeros(8, 12)
```

## arange

```
# 
torch.arange(0, 8)
```

