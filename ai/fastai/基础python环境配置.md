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

# conda install pytorch torchvision torchaudio cudatoolkit=11.8 -c pytorch -c nvidia
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu128	
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

    ① cudaGetDeviceCount() error 2: out of memory

   这个错误一般不是显存真的不足，而是：

   在 fork 出 DataLoader worker 前，GPU 已经被主进程初始化，导致子进程无法重新初始化 CUDA。

   👉 Windows DataLoader 使用 `spawn`，不能在 worker 里重新创建 CUDA 上下文。

   ② DataLoader worker exited unexpectedly

   num_workers > 0 会让每个子进程尝试使用 GPU，从而崩溃。很多 Windows + fastai + PyTorch 用户都会遇到这个情况

​		解决方式为：

```python
dls = DataBlock(
        blocks=(ImageBlock, CategoryBlock),
        get_items=get_image_files,
        splitter=RandomSplitter(valid_pct=0.2, seed=42),
        get_y=parent_label,
        item_tfms=[Resize(192, method='squish')]
    ).dataloaders(
        path,
        bs=16,  # ↓ 显存不够就改 8 或 4
        num_workers=0  # ★★★ 关键：解决你所有 worker GPU 报错
    )

    # dls.show_batch(max_n=6)

    # ====================================
    # 2. 创建 learner（不用传 device）
    #    + 使用半精度训练减少显存
    # ====================================
    learn = vision_learner(
        dls,
        resnet18,
        metrics=error_rate,
        pretrained=True,
        normalize=True
    ).to_fp16()  # ★★★ 强烈推荐：减少显存使用一半

    # ====================================
    # 3. 清空 GPU 显存（防止 CUDA 已初始化）
    # ====================================
    import torch

    torch.cuda.empty_cache()

    # ====================================
    # 4. 开始训练（不会再报错）
    # ====================================
    learn.fine_tune(3)

    is_bird, _, probs = learn.predict(PILImage.create('img.png'))
    print(f"This is a: {is_bird}.")
    print(f"Probability it's a bird: {probs[0]:.4f}")
```









