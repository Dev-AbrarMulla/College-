# SFM Hackathon – Structure from Motion Project

## 🧠 Overview
This project implements a simplified **Structure from Motion (SfM)** pipeline using **SIFT feature detection**, **feature matching**, **fundamental & essential matrix estimation**, and **camera pose recovery**.  
We used a stereo pair (`aloeL.jpg` and `aloeR.jpg`) to estimate the 3D relationship between two camera views.

---

## ⚙️ Pipeline Steps & Results

### 1. Feature Detection (SIFT)
- **SIFT** was applied to both images after converting them to grayscale.
- Around **100 strongest keypoints** were visualized to show feature stability.
- Saved output:  
  - `img1_keypoints_gray.jpg`  
  - `img2_keypoints_gray.jpg`

### 2. Feature Matching
- Two matchers were tested:
  - **Brute-Force (BFMatcher)** with L2 norm.
  - **FLANN-based Matcher** (KDTree algorithm).
- After applying **Lowe’s ratio test (0.75)**, around **73 good matches** were found before RANSAC.
- Saved output:  
  - `output_bf_match_color.jpg`  
  - `output_flann_match.jpg`

### 3. Fundamental Matrix Estimation
- Computed using **RANSAC** for robust outlier rejection.  
- **Total good matches before RANSAC:** 73  
- **Inliers after RANSAC:** 39  
- **Fundamental Matrix (F):**
[[-6.46e-07 -4.96e-05 8.69e-03]
[ 4.37e-05 8.97e-06 -1.23e-01]
[-6.10e-03 1.16e-01 1.00e+00]]

- RANSAC threshold effects:
- 0.5 px → 34 inliers  
- 1.0 px → 39 inliers  
- 5.0 px → 54 inliers  

### 4. Essential Matrix & Pose Recovery
- Camera intrinsics (assumed):
  K = [[800, 0, 268.5],
      [ 0, 800, 232.5],
      [ 0, 0, 1 ]]

**Essential Matrix (E):**
  - [[ -0.0628 -22.2175 0.7512]
    [ 24.5382 1.3259 43.8469]
    [-0.9516 -45.6665 0.0727]]

**Recovered Rotation (R):**
    [[ 0.9984 0.0015 -0.0567]
    [-0.0022 0.9999 -0.0128]
    [ 0.0567 0.0129 0.9983]]
  
**Recovered Translation (t):**
  [[ 0.899],
  [-0.015],
  [-0.438]]

### 5. 3-D Builder (Triangulation)
- Using the recovered **R** and **t**, projection matrices **P1** and **P2** were formed:
  P1 = K [I | 0]
  P2 = K [R | t]

**Triangulation** produced 42 total 3D points.  
  Foreground points filtered based on depth (**22 points kept**).  
  Example reconstructed points:
  [-2.99, -0.05, 13.90]
  [-1.74, 3.12, 11.00]
  [-1.72, 0.39, 13.99]
  [-1.30, 2.89, 11.80]

Saved visualization: `foreground_reconstruction.png`

### 1. Why do SIFT features remain stable across multiple images?
SIFT features are **scale-, rotation-, and partially illumination-invariant**.  
They describe local image patterns using gradients instead of pixel intensities, allowing consistent detection even when the object’s view, scale, or lighting changes.

---

### 2. What parameters or unknowns increase when cameras are uncalibrated?
When cameras are **uncalibrated**, intrinsic parameters like:
- focal length (fx, fy)  
- principal point (cx, cy)  
- skew  
are **unknown**.  
Thus, the system must estimate both extrinsics (R, t) and intrinsics, increasing the number of unknowns and reducing reconstruction accuracy — you get a **projective** reconstruction instead of a metric one.

---

### 3. What does bundle adjustment refine after triangulation?
Bundle adjustment refines:
- 3D point coordinates  
- Camera rotation and translation  
- (Optionally) intrinsics  
It minimizes **reprojection error** — how far projected points deviate from observed keypoints — for optimal accuracy and consistency.

---

### 4. What is the real-world trade-off between accuracy and runtime in SfM?
- **Higher accuracy:** more features, tighter RANSAC, dense matching → slower.  
- **Faster runtime:** fewer features, looser thresholds → less precise structure.  
In real-world systems:
- Real-time SLAM → prioritize speed.  
- 3D scanning or mapping → prioritize accuracy.
