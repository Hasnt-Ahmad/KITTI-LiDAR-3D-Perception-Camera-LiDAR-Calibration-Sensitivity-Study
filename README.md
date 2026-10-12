# KITTI LiDAR 3D Perception & Camera–LiDAR Calibration Sensitivity Study

A Python-based KITTI 3D perception project for visualizing LiDAR point clouds, camera images, and 3D bounding boxes. I extended the visualization pipeline with a calibration sensitivity experiment to measure how simulated camera-frame rotational and translational errors affect LiDAR-to-image projection.

- **Dataset:** KITTI 3D Object Detection
- **Focus:** 3D perception, sensor geometry, camera–LiDAR calibration, and projection sensitivity
- **Evaluation:** 30 KITTI frames
- **Tools:** Python, NumPy, OpenCV, Vedo
- **Project type:** Experimental analysis of geometric sensitivity
---

## Key Findings

The experiment introduces controlled perturbations to the camera-frame extrinsic transformation and measures the resulting displacement of projected LiDAR points.

| Configuration | Mean Pixel Displacement (px) | Mean Per-Frame 95th Percentile (px) |
|---|---:|---:|
| Baseline calibration | 0.00 | 0.00 |
| Rotation around camera Y-axis: +1° | 15.18 | 20.57 |
| Rotation around camera Y-axis: +2° | 30.39 | 41.16 |
| Rotation around camera X-axis: +1° | 12.81 | 13.39 |
| Translation along camera X-axis: +0.05 m | 3.25 | 6.41 |

**Interpretation:** In this experiment, small rotational perturbations produced noticeable changes in the image-plane positions of projected LiDAR points. A +2° rotation around the camera Y-axis resulted in an average displacement of approximately 30.39 pixels, while a +0.05 m translation along the camera X-axis resulted in approximately 3.25 pixels of average displacement.

The reported displacement values are averages of per-frame statistics across the evaluated frames. The 95th-percentile column represents the average of each frame's 95th-percentile displacement; it is not a pooled percentile across all points.

### Image Visibility Under Perturbation

This metric measures the percentage of baseline-visible points that remain inside the image boundaries after applying a perturbation.

| Configuration | Mean Visible Fraction |
|---|---:|
| Baseline calibration | 100.00% |
| Rotation around camera Y-axis: +1° | 98.80% |
| Rotation around camera Y-axis: +2° | 97.47% |
| Rotation around camera X-axis: −2° | 91.33% |

Visibility loss means that some projected points moved outside the image boundaries. It does **not** directly indicate object-detection failure or reduced sensor-fusion accuracy.

## Project Overview

Reliable autonomous-driving perception depends on correctly relating observations from different sensors. In camera–LiDAR systems, extrinsic calibration describes the geometric transformation between the LiDAR and camera coordinate frames.

Even when the underlying LiDAR point cloud remains unchanged, errors in this transformation can shift projected points relative to image features, road boundaries, and visible objects. This project investigates that geometric sensitivity using the KITTI dataset.

The original visualization pipeline supports:

- Loading KITTI LiDAR point clouds.
- Displaying camera images and LiDAR-based bird's-eye views.
- Visualizing 3D bounding boxes.
- Reading KITTI calibration parameters.
- Projecting LiDAR points into camera images.

The calibration sensitivity experiment builds on these capabilities by introducing controlled errors into the extrinsic transformation and quantifying the resulting changes in image-space projections.

## Calibration Sensitivity Experiment

### 1. Baseline Projection

LiDAR points are transformed into the camera coordinate frame using the KITTI calibration parameters. The camera projection matrix is then used to map the transformed 3D points to image coordinates.

The baseline projection provides the reference against which perturbed projections are compared.

### 2. Simulated Calibration Errors

Controlled rotational and translational perturbations are applied to the camera-frame extrinsic transformation.

The evaluated perturbations include:

- **Rotation:** ±0.5°, ±1°, and ±2° around the camera X, Y, and Z axes.
- **Translation:** ±0.02 m, ±0.05 m, and ±0.10 m along the camera X, Y, and Z axes.

These are simulated perturbations used for analysis; they are not measurements of actual calibration errors in a deployed vehicle.

### 3. Projection Displacement

For each evaluated point, the baseline and perturbed image coordinates are compared. Pixel displacement is calculated as the Euclidean distance between the two projected positions:

$$
d_{\text{pixel}} =
\sqrt{
(u_{\text{perturbed}}-u_{\text{baseline}})^2+
(v_{\text{perturbed}}-v_{\text{baseline}})^2
}
$$

Here, $u$ and $v$ denote horizontal and vertical image coordinates.

The analysis summarizes displacement using:

- Mean pixel displacement.
- Median pixel displacement.
- 95th-percentile pixel displacement.
- Mean absolute horizontal displacement.
- Mean absolute vertical displacement.
- Fraction of points displaced by more than 3 pixels.
- Fraction of points remaining inside the image boundaries.
- Fraction of points retaining positive camera projection depth.

The 3-pixel threshold is an analysis choice, not a validated safety or perception threshold.

### 4. Evaluation Across KITTI Frames

The experiment was evaluated across 30 KITTI frames. Per-frame statistics were calculated and then summarized across the evaluated frames to compare different perturbation configurations.

The distance-based analysis also examined the effect of a +2° rotation around the camera Y-axis at different camera-depth intervals.

| Camera Depth | Mean Pixel Displacement (px) | Median Pixel Displacement (px) | Evaluated Points |
|---|---:|---:|---:|
| 0–10 m | 31.22 | 29.64 | 214,326 |
| 10–20 m | 29.95 | 28.57 | 223,019 |
| 20–30 m | 29.12 | 28.28 | 64,424 |
| 30–40 m | 28.59 | 28.05 | 33,354 |
| 40–50 m | 28.12 | 27.65 | 18,643 |
| 50–80 m | 27.21 | 26.67 | 17,783 |

These values show a modest decrease in average displacement across the evaluated depth bins for this particular perturbation and dataset sample. They should not be interpreted as evidence that rotational calibration errors are inherently less significant at greater distances.

The 80 m and beyond bin contained no evaluated points and is therefore excluded from the table.

## Methodology

The experiment follows this workflow:

1. Load LiDAR points, camera images, and calibration parameters from KITTI.
2. Transform LiDAR points into the camera coordinate frame.
3. Project baseline-visible points into the camera image.
4. Apply a controlled rotation or translation to the extrinsic transformation.
5. Reproject the same points using the perturbed transformation.
6. Calculate pixel displacement, image visibility, and related statistics.
7. Aggregate per-frame measurements and compare perturbation configurations.
8. Analyze displacement across camera-depth intervals.

The evaluated point set is based on the existing pipeline's baseline visibility filtering. Consequently, this experiment measures how initially visible points move under perturbation rather than evaluating every raw LiDAR point.

## Technology Stack

- **Python** — experiment implementation and data processing.
- **NumPy** — numerical operations and coordinate transformations.
- **OpenCV** — camera image handling and projection-related operations.
- **Vedo** — 3D point-cloud visualization.
- **KITTI 3D Object Detection Dataset** — LiDAR, camera, labels, and calibration data.
---

# Camera Visualization

Several visualization functions are implemented for examining the relationship between the camera and LiDAR data.

### Point projection

```python
visualize_points(index)
```
<img width="1204" height="358" alt="image" src="https://github.com/user-attachments/assets/7fa2c4d0-e34f-4379-b233-286d0cdb79d2" />


Projects LiDAR points onto a blank camera-sized image.

### BEV

```python
visualize_bev(index)
```
<img width="1028" height="365" alt="image" src="https://github.com/user-attachments/assets/cd205bc3-0450-4cd0-a2d2-7db79afcf627" />


Creates a Bird's-Eye View representation of the LiDAR scene.

### Camera image with 3D boxes

```python
visualize_image_box(index)
```
<img width="1189" height="361" alt="image" src="https://github.com/user-attachments/assets/c1e0d9ff-128e-40b0-af8c-1ef6db2dd478" />


Projects 3D object bounding boxes onto the camera image.

### Combined camera visualization

```python
visualize(index)
```
<img width="1208" height="364" alt="image" src="https://github.com/user-attachments/assets/c4038b48-343a-41ef-9713-6762e01dbff7" />


Combines:

* LiDAR points
* 3D bounding boxes
* Camera projection

into a single visualization.

---

