---
title: 'Ubuntu 22.04 配置 RoboMaster 雷达站视觉检测系统'
description: '记录在 Ubuntu 22.04 上配置华科 RoboMaster 雷达站的过程，跑通纯视觉检测'
pubDate: '2025-10-18'
heroImage: '/blog-placeholder-2.jpg'
---

## 前言

配置华中科技大学的 RoboMaster 雷达站项目，在 Ubuntu 22.04 上从零到跑通无硬件检测。

项目地址：https://github.com/tup-robomaster/Hust_Radar_2025

## 系统要求

- Ubuntu 22.04 LTS
- 内存 8GB+
- 存储 20GB+
- Python 3.9

## 第一步：安装基础环境

### 1. 安装 ROS1 Noetic

Ubuntu 22.04 官方不支持 ROS1，需要第三方源：

```bash
sudo add-apt-repository ppa:ros-for-jammy/noetic
sudo apt update
sudo apt install ros-noetic-desktop-full
echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

### 2. 安装 Miniconda

```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
conda create -n radar_env python=3.9
conda activate radar_env
```

## 第二步：安装 Python 依赖

这是关键步骤，版本必须匹配：

```bash
# 核心依赖（版本很重要！）
pip install numpy==1.24.3 scipy==1.11.4 filterpy==1.4.5

# 深度学习
pip install torch torchvision torchaudio

# 计算机视觉
pip install opencv-python open3d timm efficientnet_pytorch

# 工具库
pip install ruamel.yaml einops dill pygame shapely pyserial pandas rospkg catkin_pkg

# GPU 加速（可选）
conda install -c conda-forge cupy
```

**关键问题：NumPy 版本冲突**

必须使用 numpy 1.24.3，不要用 2.0+，否则会报错：
```
AttributeError: `np.float_` was removed in the NumPy 2.0 release
```

## 第三步：下载项目

```bash
git clone https://github.com/tup-robomaster/Hust_Radar_2025.git
cd Hust_Radar_2025
```

## 第四步：获取数据和模型

从项目 README 的百度网盘下载：
- `video.avi` (2.5GB) - 测试视频
- `stage_one.pt` - 检测模型
- `stage_two.pt` - 分类模型

```bash
mkdir -p data weights
mv video.avi data/
mv stage_*.pt weights/
```

注意：模型文件可能需要联系作者获取（jyk041029@gmail.com）

## 第五步：修复配置文件

### 1. 修复 YAML 缩进错误

```bash
sed -i 's/^ lidar_topic_name:/  lidar_topic_name:/' configs/main_config.yaml
```

### 2. 创建 detector_config.yaml

在 `configs/` 目录创建文件：

```yaml
path:
  stage_one_path: "./weights/stage_one.pt"
  stage_two_path: "./weights/stage_two.pt"
  stage_three_path: "./weights/stage_three.pt"
  tracker_path: "./configs/bytetrack.yaml"

params:
  labels: ["B1", "B2", "B3", "B4", "B5", "B7", "R1", "R2", "R3", "R4", "R5", "R7"]
  stage_one_conf: 0.1
  stage_two_conf: 0.6
  stage_three_conf: 0.5
  life_time: 90

is_record: False
record_fps: 19
```

### 3. 修正路径配置

```bash
find configs/ -name "*.yaml" -exec sed -i 's|/home/nvidia/RadarWorkspace/code/Radar_Develop/|./|g' {} \;
```

## 第六步：修复代码

### 1. PyTorch 兼容性

```bash
sed -i 's/torch.load(file, map_location="cpu")/torch.load(file, map_location="cpu", weights_only=False)/g' ultralytics/nn/tasks.py
```

### 2. 创建测试脚本

创建 `test_detection.py`：

```python
from detect.Detector import Detector
from detect.Video import Video
import cv2
import threading
from ruamel.yaml import YAML

if __name__ == '__main__':
    # 初始化
    detector = Detector("configs/detector_config.yaml")
    detector.working_flag = True  # 必须设置！
    
    capture = Video("./data/video.avi")
    
    # 启动检测线程
    threading.Thread(target=detector.detect_thread, args=(capture,), daemon=True).start()
    
    # 显示窗口
    cv2.namedWindow("Detection", cv2.WINDOW_NORMAL)
    cv2.resizeWindow("Detection", 1280, 720)
    
    # 主循环
    while True:
        result = detector.get_results()
        if result is not None and len(result) == 2:
            img, detections = result
            if img is not None:
                cv2.imshow("Detection", img)
        
        if cv2.waitKey(1) == ord('q'):
            break
    
    # 清理
    detector.working_flag = False
    capture.release()
    cv2.destroyAllWindows()
```

## 第七步：运行测试

```bash
conda activate radar_env
python3 test_detection.py
```

成功运行后会看到：
- 弹出检测窗口
- 蓝色/红色检测框标注机器人
- 显示类型（B3、R1 等）
- 显示追踪 ID

## 常见问题

### 问题 1：找不到 ROS

**现象**：`Unable to locate package ros-noetic`

**解决**：添加第三方 PPA 源（见第一步）

### 问题 2：NumPy 报错

**现象**：`np.float_ was removed`

**解决**：
```bash
pip uninstall numpy scipy filterpy -y
pip install numpy==1.24.3 scipy==1.11.4 filterpy==1.4.5
```

### 问题 3：检测线程不工作

**现象**：程序运行但无输出

**解决**：确保设置了 `detector.working_flag = True`

### 问题 4：找不到模型文件

**现象**：`FileNotFoundError: stage_one.pt`

**解决**：联系作者获取模型，或在项目 Issues 中查找

## 技术亮点

### 两阶段检测

**Stage 1**：粗检测（置信度 0.1），找出所有可能目标

**Stage 2**：精分类（置信度 0.6），识别具体类型

**多帧投票**：累积多帧结果提高准确率

### ByteTrack 追踪

- 为每个目标分配唯一 ID
- 处理遮挡和短暂消失
- 生存时间 90 帧

## 性能优化

### 1. 使用 GPU

```bash
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
```

FPS 从 15 提升到 60+

### 2. 降低分辨率

在 Video.py 中添加：
```python
frame = cv2.resize(frame, (1920, 1080))
```

FPS 从 15 提升到 30+

## 核心问题汇总

| 问题 | 原因 | 解决方案 |
|------|------|---------|
| ROS1 无法安装 | Ubuntu 22.04 不支持 | 使用第三方 PPA |
| NumPy 版本冲突 | NumPy 2.0 移除旧 API | 使用 1.24.3 |
| 检测线程不工作 | 缺少 working_flag | 手动设置为 True |
| PyTorch 加载失败 | weights_only 默认 True | 修改为 False |
| 路径错误 | 硬编码绝对路径 | 批量替换为相对路径 |

## 总结

通过这次配置学到了：
1. Ubuntu 22.04 需要第三方源安装 ROS1
2. NumPy 版本管理很关键
3. 两阶段检测提高准确率
4. ByteTrack 实现多目标追踪

配置过程虽然有坑，但跟着步骤走可以顺利运行。成功后可以进一步接入激光雷达实现 3D 定位。

## 参考资源

- 项目仓库：https://github.com/tup-robomaster/Hust_Radar_2025
- ROS1 文档：http://wiki.ros.org/noetic
- Ultralytics 文档：https://docs.ultralytics.com

感谢华中科技大学 TUP 战队开源这个项目！

如果遇到问题欢迎留言讨论。


