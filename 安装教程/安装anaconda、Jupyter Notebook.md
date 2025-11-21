一、下载 anaconda 

在https://www.anaconda.com/download/success 选择对应版本的应用程序

anaconda 是python 的项目包依赖管理

Anaconda 是一个免费的开源Python和R语言发行版，专为数据科学和机器学习而设计。它包含了一个强大的包管理器 **Conda**，以及许多用于科学计算、数据分析和机器学习的常用库和工具，如 NumPy、Pandas、Matplotlib 等。Anaconda 易于安装和管理，可以通过 Conda 创建和管理独立的虚拟环境，解决了版本冲突问题，并且支持 Windows、macOS 和 Linux 等多种操作系统

Anaconda则是一个打包的集合，里面预装好了conda、某个版本的python、众多packages、科学计算工具等等，所以也称为Python的一种发行版。

下载完成之后就安装即可

二、下载jupyter notebook

```
conda install jupyter notebook
```

三、启动jupyter notebook

```sh
jupyter notebook
```

四、关联jupyter notebook 与anaconda环境

```
conda install nb_conda
```

出现下面问题：

```
(base) ➜  ~ conda install nb_conda

2 channel Terms of Service accepted
Channels:
 - defaults
Platform: osx-64
Collecting package metadata (repodata.json): done
Solving environment: failed

LibMambaUnsatisfiableError: Encountered problems while solving:
  - package nb_conda-2.2.1-py27_0 requires python >=2.7,<2.8.0a0, but none of the providers can be installed

Could not solve for environment specs
The following packages are incompatible
├─ nb_conda =* * is installable with the potential options
│  ├─ nb_conda 2.2.1 would require
│  │  └─ python >=2.7,<2.8.0a0 *, which can be installed;
│  ├─ nb_conda 2.2.1 would require
│  │  └─ python >=3.10,<3.11.0a0 *, which can be installed;
│  ├─ nb_conda 2.2.1 would require
│  │  └─ python >=3.11,<3.12.0a0 *, which can be installed;
│  ├─ nb_conda 2.2.1 would require
│  │  └─ python >=3.5,<3.6.0a0 *, which can be installed;
│  ├─ nb_conda 2.2.1 would require
│  │  └─ python >=3.6,<3.7.0a0 *, which can be installed;
│  ├─ nb_conda 2.2.1 would require
│  │  └─ python >=3.7,<3.8.0a0 *, which can be installed;
│  ├─ nb_conda 2.2.1 would require
│  │  └─ python >=3.8,<3.9.0a0 *, which can be installed;
│  └─ nb_conda 2.2.1 would require
│     └─ python >=3.9,<3.10.0a0 *, which can be installed;
└─ pin on python 3.13.* =* * is not installable because it requires
   └─ python =3.13 *, which conflicts with any installable versions previously reported.

Pins seem to be involved in the conflict. Currently pinned specs:
 - python=3.13
```



解决方式为： 创建一个 Python 3.10/3.11 的新环境（最干净、最推荐）

```
conda create -n py311 python=3.11
conda activate py311
conda install nb_conda
```





五、下载对应的依赖包

```
# 之间在notebook 里面执行下面的代码 就可以安装依赖包，并且同一内核下不需要重写下载
!pip install duckduckgo-search fastcore
```





六、创建对应的环境

在对应的环境下需要 安装Jupyter

python 3.11

fastai=2.8.5   与PyTorch 2.2.2 版本有冲突，需要降级到2.1 

PyTorch=2.1

numpy=1.24 

scipy=1.10

```sh
conda create -n py311 python=3.11 -y && \
conda activate py311 && \
pip install  fastcore fastai ddgs fastdownload && \
conda install ipykernel -y && \
python -m ipykernel install --user --name py311 --display-name "Python 3.11 (py311)"
```

```
conda activate py311
python --version  # 3.11

conda install notebook -y

# 注册kernel
python -m ipykernel install --user --name py311 --display-name "Python 3.11 (py311)"

# 启动
jupyter notebook

每次新开终端，需要先 conda activate py311，然后才能直接用 jupyter notebook
```



在pycharm 控制台中操作

```
# 1. 创建虚拟环境（推荐 conda） 虚拟环境名称:fastai_env  指定python版本为3.10
conda create -n fastai_env python=3.10 -y
# 2. 激活虚拟环境，需要在pycharm中为项目选择这个虚拟环境
conda activate fastai_env

# 3. 安装 PyTorch CPU 版本
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cpu

# 4. 安装 fastai
pip install fastai==2.8.5

# 5. 通过pip 来下载对应的包
pip install ddgs

# 6. 报错提示 numpy 的版本过高，不要使用2.x的   --force-reinstall 就会删除之前的版本
pip install "numpy<2" --force-reinstall

# 7. 报错提示 ImportError: cannot import name 'GradScaler' from 'torch.amp' (/opt/anaconda3/envs/fastai_env/lib/python3.10/site-packages/torch/amp/__init__.py)
# 这个问题困扰了很久，本质问题还是 fastai== 2.8.5 与 torch ==2.2.2 版本冲突，解决方式就是找到对应的版本，但是未找到
# 最后还是根据报错堆栈找到报错的 16.py文件，修改文件里面的.保存之后就可以运行啦

from torch.cuda.amp.autocast_mode import autocast
from torch.cuda.amp.grad_scaler import OptState,GradScaler
```



创建jupyter notebook

```
1. 选择对应的虚拟环境
conda activate fastai_env

2. 下载 jupyter
conda install notebook -y


# 注册kernel
python -m ipykernel install --user --name fastai_env --display-name "fastai_env"

# 启动
jupyter notebook


```

