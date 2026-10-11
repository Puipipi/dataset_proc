# 数据集处理工具集

[English](README.md)

这是一个用于数据集处理的 Python 工具集合，包含多个实用工具，可简化数据集创建、清洗和标注流程。

## 工具概览

| 工具名称 | 主要功能 | 依赖 | 状态 |
| --- | --- | --- | --- |
| [dtst_proc_tool.py](#-dtst_proc_toolpy) | 视频转图像数据集 | opencv-python | 已验证 |
| [rearrangement.py](#-rearrangementpy) | 图像重命名整理 | 无 | 已验证 |
| [yolo_detect_img.py](#-yolo_detect_imgpy) | YOLO 图像检测封装 | ultralytics | 已验证 |
| [yolo_ai2ai.py](#-yolo_ai2aipy) | YOLO 自动标注工具 | ultralytics | 已验证 |

---

## 🎬 dtst_proc_tool.py

### 功能描述
将视频文件转换为经过预处理的图像数据集，提供一站式视频到图像转换服务。

### 主要特性
- 视频文件直接转换为图像集合。
- 图像预处理（包括尺寸调整、数据增强等）。
- 自动组织输出文件夹结构。
- 支持 YOLO 数据集格式。
- **新增模糊填充功能**（2025-11-28）。
- **新增鱼眼畸变模拟**（2025-11-28）。

### 使用方法
```python
# 参考 dtst_proc_tool_use.py
from dtst_proc_tool import dpt_aio, image_resize, batch_resize_folder

# 一条龙处理（推荐）
dpt_aio(
    input_video="your_video.mp4",
    output_folder="output_dataset",
    augment=True,      # 是否启用数据增强
    split=False,       # 是否分割数据集
    sample=1,          # 采样率
    blur_fill=True     # 启用模糊填充
)

# 单独使用模糊填充
resized_image = image_resize(
    image,
    size=(640, 640),
    cut_fill=False,    # 禁用裁剪填充
    blur_fill=True     # 启用模糊填充
)

# 批量处理文件夹
batch_resize_folder(
    input_folder="input_images",
    output_folder="output_images",
    size=(640, 640),
    cut_fill=False,    # 禁用裁剪填充
    blur_fill=True     # 启用模糊填充
)
```

### 环境要求
```bash
pip install opencv-python
```

### 模糊填充功能详解

#### 技术原理
1. **缩小模糊：** 将原图缩小到 1/8 尺寸后进行高斯模糊处理。
2. **拉伸背景：** 将模糊后的小图拉伸到目标分辨率，作为背景底板。
3. **叠加原图：** 将原图按比例缩小后放置在模糊背景的中心。

#### 性能优势
- **计算效率：** 在 1/8 尺寸上进行模糊，像素处理量约减少 64 倍。
- **自适应核：** 根据图像尺寸自动调整模糊核大小。
- **视觉效果：** 产生自然的模糊背景，类似相册动画效果。

#### 参数说明
- `cut_fill=False`：禁用裁剪填充模式。
- `blur_fill=True`：启用模糊填充模式。
- 默认填充颜色：YOLO 推荐的灰色 `(114, 114, 114)`。

### 鱼眼畸变功能详解

#### 技术原理
1. **全图覆盖：** 使用图像对角线作为最大半径，确保四个角都产生畸变。
2. **鱼眼公式：** 使用 `r' = r * (1 + k * r^2)` 模拟真实鱼眼镜头效果。
3. **中心对称：** 以图像中心为原点，向外辐射畸变。

#### 效果特点
- 畸变覆盖全图，四个角都会产生效果。
- 畸变强度适中，随机取值范围为 0.1–0.3。
- 使用反射边界模式，保持图像边缘连续。

#### 参数说明
- 触发概率：10%。
- 畸变强度：随机取值 0.1–0.3。
- 应用场景：模拟广角镜头拍摄效果，增加数据多样性。

### 更新日志
- **2025-11-28：** 新增鱼眼畸变模拟，用于模拟广角镜头拍摄效果。
- **2025-11-28：** 新增模糊填充功能，提供更自然的图像填充效果。
- **25-11-10：** 修复噪点异常，采用更保守的干扰策略。
- **25-11-10：** 增加是否启用增强的可选项。
- **25-11-10：** 支持自动生成 YOLO 格式的文件组织结构。

---

## 📁 rearrangement.py

### 功能描述
清洗和整理图像数据集，将目录内的图片按数字顺序重新命名。

### 主要特性
- 自动重命名文件夹内所有图片。
- 按 `00000`–`99999` 格式标准化命名。
- 清理混乱的文件名结构。

### 使用方法
```python
# 参考 rearrangement_use.py
from rearrangement import rearrange_images

# 基本使用
rearrange_images(
    input_dir="raw_images",       # 原始图像文件夹
    output_dir="cleaned_images"  # 整理后的输出文件夹
)
```

### 环境要求
- 标准 Python 环境。
- 无需额外依赖。

### 更新日志
- **25-11-18：** 首次发布。

---

## 🔍 yolo_detect_img.py

### 功能描述
YOLO 图像检测便捷封装，提供简单接口以获取检测结果。

### 主要特性
- 将 YOLO 检测封装为易用接口。
- 返回结构化检测结果。
- 作为仓库内其他工具的依赖组件。

### 使用方法
```python
# 示例见 yolo_ai2ai.py
from yolo_detect_img import detect_image

# 检测单张图像
results = detect_image(
    model_path="yolo_model.pt",
    image_path="test_image.jpg"
)
```

### 环境要求
```bash
# 建议使用 CUDA 环境
# pip3 install torch torchvision
pip install ultralytics
```

### 更新日志
- **25-11-18：** 首次发布。

---

## 🏷️ yolo_ai2ai.py

### 功能描述
使用现有 YOLO 模型自动标注尚无标注的图像数据集。

### 主要特性
- 使用预训练模型自动标注。
- 生成 YOLO 格式的标注文件。
- 批量处理整个数据集。

### 使用方法
```python
# 参考 yolo_ai2ai_use.py
from yolo_ai2ai import auto_annotate_dataset

# 自动标注数据集
auto_annotate_dataset(
    model_path="yolo_model.pt",        # 模型路径
    image_dir="unlabeled_images",      # 未标注图像目录
    output_dir="annotated_dataset"     # 标注输出目录
)
```

### 环境要求
```bash
# 依赖 yolo_detect_img.py
# pip3 install torch torchvision  # 建议使用 CUDA 环境
pip install ultralytics
```

### 更新日志
- **25-11-18：** 首次发布。

---

## 🚀 快速开始

### 1. 安装依赖
```bash
# 基础依赖
pip install opencv-python ultralytics
```

### 2. 环境建议
- 使用 Python 虚拟环境。
- 建议使用 CUDA 环境以获得更好的性能。
- 根据数据集大小预留足够的磁盘空间。

### 3. 使用流程
1. 使用 `dtst_proc_tool.py` 从视频创建图像数据集。
2. 使用 `rearrangement.py` 整理图像文件名。
3. 使用 `yolo_ai2ai.py` 自动标注数据集。

---

## 📝 注意事项

- 仓库原 README 说明所有工具功能均已验证并正常运行。
- 建议处理前备份原始数据。
- 根据数据集大小预留足够的内存和存储空间。
- 使用示例代码前请先理解参数含义。

---

## 🔄 更新计划

持续优化工具性能和用户体验，欢迎反馈和建议。

---

[许可证](LICENSE)
