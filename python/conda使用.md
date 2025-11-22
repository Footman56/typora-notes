conda 是一款管理python环境的工具

### 1. **创建环境**

创建一个新的环境，指定 Python 版本（例如 3.9）：

```
conda create --name myenv python=3.9
```

### 2. **激活/切换环境**

激活一个已经存在的环境：

```
conda activate myenv
```

### 3. **停用环境**

退出当前环境，回到基础环境：

```
conda deactivate
```

### 4. **列出所有环境**

列出所有已经创建的环境：

```
conda env list
```

或者

```
conda info --envs
```

### 5. **删除环境**

删除指定的环境及其中所有的包和依赖：

```
conda env remove --name myenv
```

### 6. **安装包**

在当前环境中安装一个包（例如安装 `numpy`）：

```
conda install numpy
```

安装指定版本的包：

```
conda install numpy=1.21
```

### 7. **更新包**

更新已安装的包：

```
conda update numpy
```

更新所有包：

```
conda update --all
```

### 8. **卸载包**

卸载已安装的包：

```
conda remove numpy
```

### 9. **搜索包**

搜索可以安装的包：

```
conda search numpy
```

### 10. **查看已安装包**

查看当前环境中已安装的所有包：

```
conda list
```

### 11. **导出环境**

导出当前环境的配置到 `environment.yml` 文件，方便其他人重建环境：

```
conda env export > environment.yml
```

### 12. **从环境文件创建环境**

根据 `environment.yml` 文件创建环境：

```
conda env create -f environment.yml
```

### 13. **更新 Conda 本身**

更新 Conda 版本：

```
conda update conda
```

### 14. **清理不再使用的包**

清理缓存和未使用的包：

```
conda clean --all
```