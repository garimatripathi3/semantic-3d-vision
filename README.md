# Semantic 3D Vision

> End-to-end pipeline: 2D images → 3D reconstruction → 
> semantic understanding using COLMAP + Open3D + SAM

![Pipeline](outputs/github_banner.png)

---

## Overview

Semantic 3D Vision reconstructs a complete labeled 3D scene 
from a collection of 2D images. No depth sensors. No LiDAR. 
Just images.

The pipeline runs Structure from Motion to recover 3D structure, 
builds a solid mesh, then uses Meta's Segment Anything Model to 
semantically label every 3D point — creating an intelligent 3D 
representation that understands what each part of the scene is.

This mirrors the core technology behind construction monitoring 
platforms like Track3D, where daily drone images are converted 
into semantically rich 3D models for progress tracking.

---

## Results

![SAM Segmentation](outputs/temple_sam.png)

| Metric | Value |
|--------|-------|
| Dataset | Middlebury Temple — 251 images |
| Sparse 3D Points | 30,267 |
| Mesh Triangles | 170,095 |
| Mesh Vertices | 85,170 |
| SAM Segments | 11 |
| Labeled 3D Points | 17,297 |
| Reprojection Error | ~0.4px |
| Camera Focal Length | 1529px |

---

## Pipeline

```
251 Input Images
        ↓
COLMAP Feature Extraction
SIFT keypoints (~10K per image)
        ↓
Sequential Feature Matching
RANSAC filters wrong matches
        ↓
Sparse Reconstruction (SfM)
Triangulation + Bundle Adjustment
→ 30,267 3D points + 251 camera poses
        ↓
Statistical Outlier Removal
20 neighbors, 2σ threshold
→ 109 noise points removed
        ↓
Poisson Surface Reconstruction
Normal estimation + depth=9 octree
→ 170,095 triangle mesh
        ↓
SAM Automatic Segmentation
Zero-shot, no training needed
→ 11 segments on 2D image
        ↓
2D → 3D Label Projection
Camera intrinsics + extrinsics
u = fx·x/z + cx
→ 17,297 semantically labeled 3D points
```

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| COLMAP | SfM + sparse 3D reconstruction |
| Open3D | Point cloud processing + mesh |
| SAM (ViT-B) | Zero-shot 2D segmentation |
| Python | numpy, struct, cv2, matplotlib |

---

## Key Concepts

**Structure from Motion**
Estimates camera poses and 3D structure from 2D image 
correspondences. Uses SIFT for features, RANSAC for outlier 
rejection, triangulation for 3D point recovery, and bundle 
adjustment to minimize reprojection error globally.

**Statistical Outlier Removal**
Each point's distance to its 20 nearest neighbors is computed. 
Points beyond 2 standard deviations of the mean distance are 
classified as noise and removed.

**Poisson Surface Reconstruction**
Converts point cloud to watertight mesh by estimating per-point 
surface normals and solving a Poisson equation at octree 
resolution depth=9.

**Zero-Shot SAM Segmentation**
SAM's automatic mask generator places a uniform grid of points 
across the image, runs the lightweight mask decoder for each, 
and merges overlapping predictions. No domain-specific training 
required.

**2D to 3D Label Projection**
Each 3D world point P is transformed into camera space:
```
P_cam = R @ P + t
u = fx * P_cam.x / P_cam.z + cx
v = fy * P_cam.y / P_cam.z + cy
```
The SAM segment label at pixel (u, v) is assigned to that 
3D point, producing a semantically labeled point cloud.

---

## How to Run

**Open in Google Colab — no GPU needed**

```bash
git clone https://github.com/yourusername/semantic-3d-vision
```

1. Upload `semantic_3d_pipeline.ipynb` to Google Colab
2. Run all cells — dataset downloads automatically
3. Download output `.ply` files
4. View at viewer.open3d.org

---

## Project Structure

```
semantic-3d-vision/
├── semantic_3d_pipeline.ipynb   ← complete pipeline notebook
├── README.md
└── outputs/
    ├── github_banner.png        ← pipeline visualization
    ├── temple_sam.png           ← SAM segmentation result
    ├── temple_clean.ply         ← cleaned point cloud
    ├── temple_mesh.ply          ← 3D mesh
    └── temple_semantic.ply      ← semantic point cloud
```

---

## Real World Applications

**Construction monitoring**
Drone images → daily 3D reconstruction → semantic labeling 
of walls, beams, floors, equipment → automated progress 
tracking without manual inspection.

**Archaeological preservation**
Photograph artifacts → permanent labeled 3D record → 
identify and measure individual structural components.

**Infrastructure inspection**
Camera survey → 3D model → automatic detection and labeling 
of structural elements for maintenance planning.

---

## Limitations and Future Work

- Currently uses one image for 2D→3D projection — 
  multi-view label fusion would improve coverage
  (17,297 / 30,267 points labeled = 57%)
- SAM segments are unlabeled — adding Grounded SAM 
  with text prompts would give named categories
  ("column", "wall", "opening")
- Dense reconstruction with MVS would produce 
  millions of points for higher detail
- Change detection between two reconstructions 
  would enable progress monitoring

---

## Author

Garima Tripathi
Computer Vision Engineer
[https://www.linkedin.com/in/garimatripathi35/?skipRedirect=true]

---

## References

- COLMAP: Schönberger & Frahm, CVPR 2016
- Segment Anything: Kirillov et al., Meta AI 2023
- Middlebury Temple Dataset: Seitz et al., CVPR 2006
- Open3D: Zhou et al., 2018
