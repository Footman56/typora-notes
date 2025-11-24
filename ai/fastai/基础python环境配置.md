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

   5060Ti 显卡是比较新的,PyTorch 版本与显卡有冲突。 需要安装兼容 CUDA version 12.9  的pyTorch

   ```python
   pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu128	
   ```

   用于验证安装成功的脚本为：

   ```python
   import torch
   
   print("PyTorch 版本：", torch.__version__)
   print("CUDA 可用：", torch.cuda.is_available())
   print("支持的 CUDA 版本：", torch.version.cuda)
   print("当前GPU设备：", torch.cuda.current_device())
   print("设备名称：", torch.cuda.get_device_name())
   
   # 进行一个简单的GPU计算测试
   if torch.cuda.is_available():
       x = torch.tensor([1.0, 2.0, 3.0]).cuda()
       y = x * 2
       print("测试张量计算：", y)
       print("✅ 恭喜！PyTorch 已成功配置并运行在你的 RTX 5060 Ti 上。")
   else:
       print("❌ CUDA 仍不可用，请检查安装。")
   ```

   4. 想要在python 中访问外网的接口，首先需要开启代理之后再添加下面代码

   ```python
   import os
   
   # 需要修改代理的端口 一般clash 的端口为7890
   os.environ['HTTP_PROXY'] = 'http://127.0.0.1:7890'
   os.environ['HTTPS_PROXY'] = 'http://127.0.0.1:7890'
   ```

   5. 报错为：

   ```python
   RuntimeError: Unexpected error from cudaGetDeviceCount(). Did you run some cuda functions before calling NumCudaDevices() that might have already set an error? Error 2: out of memory
   
   RuntimeError: DataLoader worker (pid(s) 16636) exited unexpectedly [W1124 14:11:39.000000000 CudaIPCTypes.cpp:16] Producer process has been terminated before all shared CUDA tensors released. See Note [Sharing CUDA tensors]
   ```

   

   



