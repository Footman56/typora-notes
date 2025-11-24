# 一、明确系统参数

win11 ，naviada  CUDA version 12.9 

conda 来管理虚拟环境

# 二、命令操作

1. 创建并且激活虚拟环境

```
conda create -n fastai-env python=3.10 -y
conda activate fastai-env
```

2. 安装 PyTorch（根据你的 NVIDIA 显卡自动安装 CUDA）

```
conda install pytorch torchvision torchaudio cudatoolkit=11.8 -c pytorch -c nvidia
```

3. 安装 fastai

```
pip install fastai
```





# 三、遇到的问题

1. 最初python=3.9 在install fastai 时 需要下载 spacy ， 但是spaCy / Thinc / Blis 无法在 Windows + Python 3.9 上编译

   解决措施： 升级Python版本到 3.10 

2. 报错为：

   ```
   RuntimeError: An attempt has been made to start a new process before the current process has finished its bootstrapping phase. This probably means that you are not using fork to start your child processes and you have forgotten to use the proper idiom in the main module: if __name__ == '__main__': freeze_support() ... The "freeze_support()" line can be omitted if the program is not going to be frozen to produce an executable.
   ```

   **Windows 下运行 fastai 的并行下载功能时，会触发 Python multiprocessing 的限制**。

   这不是 fastai 的问题，而是 **Windows 必须使用 `if __name__ == '__main__':` 来保护多进程代码**。

3. 报错如下：

   ```
   NVIDIA GeForce RTX 5060 Ti with CUDA capability sm_120 is not compatible with the current PyTorch installation. The current PyTorch install supports CUDA capabilities sm_50 sm_60 sm_61 sm_70 sm_75 sm_80 sm_86 sm_90. If you want to use the NVIDIA GeForce RTX 5060 Ti GPU with PyTorch, please check the instructions at https://pytorch.org/get-started/locally/
   ```

   5060Ti 显卡是比较新的,PyTorch 版本与显卡有冲突

   