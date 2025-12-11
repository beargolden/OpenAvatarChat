# MuseTalk相关错误及其解决方案

uv run src/demo.py --config config/chat_with_openai_compatible_bailian_cosyvoice_musetalk.yaml

## 1. ModuleNotFoundError: No module named 'mmcv._ext'

### 错误原因：mmcv版本与PyTorch/CUDA不匹配

### 解决方案：参考[MMCV安装指南](https://mmcv.readthedocs.io/en/latest/get_started/installation.html)中的"Install with pip"，执行以下命令：

```bash
uv pip uninstall mmcv mmdet mmpose mmengine

uv pip install mmcv==2.2.0 -f https://download.openmmlab.com/mmcv/dist/cu121/torch2.4/index.html
uv pip install mmdet==3.1.0 mmpose==1.3.2
```

## 2. RuntimeError: CUDA error: CUBLAS_STATUS_ALLOC_FAILED when calling `cublasCreate(handle)`

2025-12-11 10:57:49.675 | INFO     | handlers.client.rtc_client.client_handler_rtc:patched_encode_frame:104 - H.264 encoder created: h264_nvenc

ERROR:libav.h264_nvenc:Driver does not support the required nvenc API version. Required: 13.0 Found: 12.1

ERROR:libav.h264_nvenc:The minimum required Nvidia driver for nvenc is (unknown) or newer

### 错误原因：检测硬件编码器时，只是创建了CodecContext，没有调用.open()，所以检测通过了，但真正使用时才失败

### 解决方案：强制使用libx264

### 具体方法：

> (1) 备份文件
> ```bash
> cp src/handlers/client/rtc_client/client_handler_rtc.py src/handlers/client/rtc_client/client_handler_rtc.py.backup
> ```
> (2) 注释第46-59行，跳过硬件编码器检测
> ```
> vim src/handlers/client/rtc_client/client_handler_rtc.py
> 在vim中：
> 按 /hardware_encoders 回车，找到第45行
> 将第46-59行全部注释掉（在每行前加#）
> ```
> (3) 强制使用libx264（可选）
> ```
> 在第59行后添加：
> # Force use libx264 software encoder
> _selected_h264_encoder = "libx264"
> logger.info("Using libx264 software encoder (forced)")
> 按 :wq 保存退出
> ```