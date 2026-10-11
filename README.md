# Dataset Processing Toolkit

[简体中文](README_cn.md)

A collection of Python utilities for dataset processing, covering dataset creation, cleaning, and annotation workflows.

## Tool overview

| Tool | Main function | Dependency | Status |
| --- | --- | --- | --- |
| [dtst_proc_tool.py](#-dtst_proc_toolpy) | Convert video into an image dataset | opencv-python | Verified |
| [rearrangement.py](#-rearrangementpy) | Rename and organize images | None | Verified |
| [yolo_detect_img.py](#-yolo_detect_imgpy) | Wrapper for YOLO image detection | ultralytics | Verified |
| [yolo_ai2ai.py](#-yolo_ai2aipy) | Automatic YOLO annotation | ultralytics | Verified |

---

## 🎬 dtst_proc_tool.py

### Description
Convert video files into a preprocessed image dataset in one workflow.

### Features
- Convert video files directly into image collections.
- Preprocess images, including resizing and data augmentation.
- Organize the output directory structure automatically.
- Support the YOLO dataset format.
- **Blur-fill support added** (2025-11-28).
- **Fisheye-distortion simulation added** (2025-11-28).

### Usage
```python
# See dtst_proc_tool_use.py
from dtst_proc_tool import dpt_aio, image_resize, batch_resize_folder

# End-to-end processing (recommended)
dpt_aio(
    input_video="your_video.mp4",
    output_folder="output_dataset",
    augment=True,      # Enable data augmentation
    split=False,       # Split the dataset
    sample=1,          # Sampling rate
    blur_fill=True     # Enable blur fill
)

# Use blur fill by itself
resized_image = image_resize(
    image,
    size=(640, 640),
    cut_fill=False,    # Disable crop fill
    blur_fill=True     # Enable blur fill
)

# Process a folder of images
batch_resize_folder(
    input_folder="input_images",
    output_folder="output_images",
    size=(640, 640),
    cut_fill=False,    # Disable crop fill
    blur_fill=True     # Enable blur fill
)
```

### Requirements
```bash
pip install opencv-python
```

### How blur fill works

#### Algorithm
1. **Blur at reduced size:** downscale the original image to one eighth and apply Gaussian blur.
2. **Stretch the background:** enlarge the blurred image to the target resolution as the background.
3. **Overlay the original:** resize the original image proportionally and place it in the center.

#### Performance characteristics
- **Lower computation:** blur at one eighth size, reducing the pixel workload by about 64 times.
- **Adaptive kernel:** choose the blur-kernel size based on image dimensions.
- **Visual result:** create a natural blurred background similar to a photo-album animation.

#### Parameters
- `cut_fill=False`: disable crop-fill mode.
- `blur_fill=True`: enable blur-fill mode.
- Default fill color: YOLO's recommended gray `(114, 114, 114)`.

### How fisheye distortion works

#### Algorithm
1. **Full-image coverage:** use the image diagonal as the maximum radius so all four corners are distorted.
2. **Fisheye formula:** use `r' = r * (1 + k * r^2)` to simulate a real fisheye lens.
3. **Center symmetry:** use the image center as the origin and distort outward.

#### Characteristics
- The distortion covers the full image and reaches all four corners.
- Distortion strength is moderate, randomly selected between 0.1 and 0.3.
- Reflection borders keep image edges continuous.

#### Parameters
- Probability of application: 10%.
- Distortion strength: random value from 0.1 to 0.3.
- Use case: simulate a wide-angle lens and add variety to the dataset.

### Changelog
- **2025-11-28:** Added fisheye-distortion simulation for a wide-angle camera effect.
- **2025-11-28:** Added blur fill for more natural image padding.
- **25-11-10:** Fixed abnormal noise using a more conservative perturbation strategy.
- **25-11-10:** Made data augmentation optional.
- **25-11-10:** Added automatic YOLO-format directory organization.

---

## 📁 rearrangement.py

### Description
Clean up an image dataset by renaming image files in a directory in numeric order.

### Features
- Rename all images in a directory automatically.
- Normalize names to the `00000`–`99999` format.
- Clean up inconsistent file-name structures.

### Usage
```python
# See rearrangement_use.py
from rearrangement import rearrange_images

# Basic use
rearrange_images(
    input_dir="raw_images",       # Source image directory
    output_dir="cleaned_images"  # Organized output directory
)
```

### Requirements
- Standard Python environment.
- No additional dependencies.

### Changelog
- **25-11-18:** Initial release.

---

## 🔍 yolo_detect_img.py

### Description
A convenience wrapper for YOLO image detection that provides a simple interface for detection results.

### Features
- Wrap YOLO detection in an easy-to-use interface.
- Return structured detection results.
- Serve as a dependency for other tools in this repository.

### Usage
```python
# See the example in yolo_ai2ai.py
from yolo_detect_img import detect_image

# Detect one image
results = detect_image(
    model_path="yolo_model.pt",
    image_path="test_image.jpg"
)
```

### Requirements
```bash
# A CUDA environment is recommended
# pip3 install torch torchvision
pip install ultralytics
```

### Changelog
- **25-11-18:** Initial release.

---

## 🏷️ yolo_ai2ai.py

### Description
Use an existing YOLO model to automatically annotate images that do not yet have annotations.

### Features
- Automatically annotate with a pretrained model.
- Generate annotation files in YOLO format.
- Process an entire dataset in batches.

### Usage
```python
# See yolo_ai2ai_use.py
from yolo_ai2ai import auto_annotate_dataset

# Automatically annotate a dataset
auto_annotate_dataset(
    model_path="yolo_model.pt",        # Model path
    image_dir="unlabeled_images",      # Directory of unannotated images
    output_dir="annotated_dataset"     # Annotation output directory
)
```

### Requirements
```bash
# Depends on yolo_detect_img.py
# pip3 install torch torchvision  # A CUDA environment is recommended
pip install ultralytics
```

### Changelog
- **25-11-18:** Initial release.

---

## 🚀 Quick start

### 1. Install dependencies
```bash
# Core dependencies
pip install opencv-python ultralytics
```

### 2. Environment suggestions
- Use a Python virtual environment.
- A CUDA environment is recommended for better performance.
- Reserve enough disk space for the processed dataset.

### 3. Workflow
1. Create an image dataset from video with `dtst_proc_tool.py`.
2. Organize image file names with `rearrangement.py`.
3. Automatically annotate the dataset with `yolo_ai2ai.py`.

---

## 📝 Notes

- The repository README reports that all tool functions have been verified and run successfully.
- Back up the original data before processing.
- Reserve enough memory and disk space for the dataset size.
- Make sure you understand the parameters before using the examples.

---

## 🔄 Future plans

Continue improving performance and usability. Feedback and suggestions are welcome.

---

[License](LICENSE)
