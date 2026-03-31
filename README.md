# Stereo Vision Disparity Map & 3D Reconstruction

A Python script that computes a **disparity map** from a stereo image pair and reprojects the result into **3D point cloud coordinates** using OpenCV.

---

## Overview

This project takes a left and right stereo camera image, computes pixel-level disparity using block matching, and reconstructs 3D spatial coordinates via reprojection. It's a foundational building block for depth estimation, autonomous navigation, and 3D scene reconstruction.

---

## Requirements

- Python 3.7+
- OpenCV (`opencv-python`)
- NumPy
- Matplotlib

Install dependencies:

```bash
pip install opencv-python numpy matplotlib
```

---

## Usage

1. Place your stereo image pair in the project directory and name them:
   - `left_image.jpg`
   - `right_image.jpg`

2. Run the script:

```bash
python disparity_map.py
```

3. A disparity map will be displayed, and extracted 3D point coordinates will be printed to the console.

---

## How It Works

### 1. Stereo Matching
The script uses **StereoBM** (Block Matching) by default to compute the disparity map — a per-pixel measure of how much a feature shifts between the left and right images. Greater disparity = closer to the camera.

```python
stereo = cv2.StereoBM_create(numDisparities=16*5, blockSize=15)
disparity = stereo.compute(imgL, imgR)
```

**StereoSGBM** (Semi-Global Block Matching) is also available (commented out) and generally produces smoother, more accurate results at the cost of higher computation.

### 2. Disparity Normalization
The raw disparity values are normalized to the `[0, 255]` range for visualization:

```python
disparity = cv2.normalize(disparity, disparity, alpha=255, beta=0, norm_type=cv2.NORM_MINMAX)
```

### 3. 3D Reprojection
Using a **Q matrix** (disparity-to-depth mapping matrix) and `cv2.reprojectImageTo3D`, each disparity pixel is mapped to a 3D `(X, Y, Z)` coordinate. The script uses example/dummy calibration parameters — replace these with real calibration data for accurate metric 3D output.

```python
points_3D = cv2.reprojectImageTo3D(disparity, Q)
```

### 4. Outlier Filtering
Points with near-zero disparity (background/sky or unmatched regions) are masked out:

```python
mask = disparity > disparity.min()
output_points = points_3D[mask]
```

---

## Configuration

| Parameter | Default | Description |
|---|---|---|
| `numDisparities` | `80` (16×5) | Search range for disparity; must be divisible by 16 |
| `blockSize` | `15` | Size of the matching block; larger = smoother but less detail |
| `focal_length` | `0.8 × image width` | Estimated focal length for 3D reprojection |

---

## Switching to StereoSGBM

Uncomment the `StereoSGBM_create` block in the script for better quality results, especially on textured scenes:

```python
stereo = cv2.StereoSGBM_create(
    minDisparity=0,
    numDisparities=80,
    blockSize=15,
    ...
)
```

---

## Using Real Calibration Data

For accurate metric 3D reconstruction, replace the dummy `Q` matrix with the one obtained from `cv2.stereoCalibrate()` or `cv2.stereoRectify()` using a calibration checkerboard:

```python
# From stereoRectify:
_, _, _, _, Q, _, _ = cv2.stereoRectify(...)
```

---

## Output

- **Disparity Map** — Grayscale visualization (brighter = closer)
- **3D Point Array** — NumPy array of shape `(N, 3)` with `(X, Y, Z)` coordinates for all valid pixels

```
Extracted 3D points:  (284736, 3)
```

---

## Limitations

- Dummy calibration parameters produce relative, not metric, 3D coordinates
- StereoBM can struggle in low-texture or heavily occluded regions
- Images must be rectified (epipolar-aligned) before computing disparity
