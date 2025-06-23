# TensoRF-reconstruction


    # TensoRF 物体重建系统

> 基于 TensoRF 的物体重建与新视图合成实现 | 深度学习与神经网络期末项目

## 项目概述

本仓库实现了基于 **TensoRF**（Tensor Radiance Fields）的物体重建系统，通过多角度采集物体图像，使用 COLMAP 估计相机参数，训练 TensoRF 模型并生成新视角渲染视频。

## 核心特点

- 🚀 **高效重建**：将隐式 MLP 替换为显式张量，大幅提升训练速度
- 💾 **低显存占用**：相比原始 NeRF 减少资源消耗
- 🎨 **高质量渲染**：通过张量分解实现高质量新视图合成
- 📊 **量化评估**：提供 PSNR、SSIM 等量化指标评估重建质量

## 目录结构

```
├── .gitignore
├── README.md
├── mouse
│   ├── train
│   ├── test
│   ├── transforms_train.json
│   └── transforms_test.json
└── TensoRF-main
    ├── LICENSE
    ├── README.md
    ├── requirements.txt
    ├── opt.py
    ├── renderer.py
    ├── train.py
    ├── utils.py
    ├── configs
    │   └── your_own_data.txt
    ├── dataLoader
    │   ├── blender.py
    │   ├── colmap2nerf.py
    │   ├── llff.py
    │   └── ...
    ├── models
    │   ├── tensorBase.py
    │   ├── tensoRF.py
    └── extra
```

## 快速开始

### 环境配置

```bash
# 创建 Python 环境
conda create -n TensoRF python=3.8
conda activate TensoRF
pip install torch torchvision
pip install tqdm scikit-image opencv-python configargparse lpips imageio-ffmpeg kornia lpips tensorboard
```

### 数据准备

1. 将原始图片放入 `mouse/train/` 和 `mouse/test/` 目录
2. 在TensoRF-main目录下运行 COLMAP 预处理脚本：
```bash
mkdir -p ../mouse/colmap_project/sparsetrain
mkdir -p ../mouse/colmap_project/databasetrain
colmap feature_extractor --database_path ../mouse/colmap_project/databasetrain.db --image_path ../mouse/train
colmap exhaustive_matcher --database_path ../mouse/colmap_project/databasetrain.db
colmap mapper --database_path ../mouse/colmap_project/databasetrain.db --image_path ../mouse/train --output_path ../mouse/colmap_project/sparsetrain
mkdir -p ../mouse/custom_object/sparsetrain_text
colmap model_converter --input_path ../mouse/colmap_project/sparsetrain/0 --output_path ../mouse/custom_object/sparsetrain_text --output_type TXT
python dataLoader/colmap2nerf.py --text ../mouse/custom_object/sparsetrain_text --images ../mouse/train --out ../mouse/transforms_train.json
mkdir -p ../mouse/colmap_project/sparsetest
mkdir -p ../mouse/colmap_project/databasetest
colmap feature_extractor --database_path ../mouse/colmap_project/databasetest.db --image_path ../mouse/test
colmap exhaustive_matcher --database_path ../mouse/colmap_project/databasetest.db
colmap mapper --database_path ../mouse/colmap_project/databasetest.db --image_path ../mouse/test --output_path ../mouse/colmap_project/sparsetest
mkdir -p ../mouse/custom_object/sparsetest_text
colmap model_converter --input_path ../mouse/colmap_project/sparsetest/0 --output_path ../mouse/custom_object/sparsetest_text --output_type TXT
python dataLoader/colmap2nerf.py --text ../mouse/custom_object/sparsetest_text --images ../mouse/test --out ../mouse/transforms_test.json
```
3. 生成 `transforms_train.json` 和 `transforms_test.json` 配置文件

### 训练模型

```bash
python train.py --config configs/your_own_data.txt
```

### 关键训练参数

| 参数 | 值 | 说明 |
|------|----|------|
| 优化器 | Adam | β₁=0.9, β₂=0.99 |
| 学习率 | 5e-3 → 1e-4 | 余弦衰减 |
| Batch Size | 1024 | 每迭代射线数 |
| 总迭代次数 | 30,000 | |
| 损失函数 | MSE + TV 正则 | λ_density=0.1, λ_app=0.01 |
| 初始分辨率 | 80³ | 512,000 体素 |
| 最终分辨率 | 150³ | 3,375,000 体素 |
| 密度分量 | [10,10,10] | |
| 外观分量 | [30,30,30] | |


## 实验结果

| 指标 | 训练值 |
|------|--------|
| PSNR | 34.76 dB |
| MSE | 0.0012 |

## 模型输出

训练完成后，可在 `log/tensorf_mouse_VM` 目录找到：
- 重建的 3D 模型 (`tensorf_mouse_VM.ply`)
- 渲染视频 (`video.mp4`)
- 深度图 (`depthvideo.mp4`)
- 评估指标 (`mean.txt`)

