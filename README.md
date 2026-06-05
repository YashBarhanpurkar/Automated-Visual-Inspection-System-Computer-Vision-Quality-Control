# Automated Visual Inspection System — Computer Vision Quality Control

> A modular computer vision pipeline that automates product quality control on a simulated production line — classifying product variants by texture signature and enforcing ingredient distribution thresholds via automated Accept/Reject logic.

---

## The Problem

In food and consumer goods manufacturing, manual visual inspection is a production bottleneck: it is slow, inconsistent, and operator-dependent. A single shift change can introduce variance in what gets accepted or rejected. This project replaces manual auditing with an **automated, quantitative inspection pipeline** — applying computer vision to enforce consistent, data-driven quality thresholds across every unit on the line.

This directly simulates the core engineering challenge in **Industrie 4.0 automated QC systems**: translating physical product properties into digital measurement signals, then executing automated pass/fail decisions.

---

## System Architecture

The pipeline is built as a modular, sequential processing chain — each stage feeds clean output to the next.

```
[ Raw Production Image ]
        │
        ▼
[ 1. Pre-processing ]
  - RGB → Grayscale + HSV conversion
  - Gaussian blur (7×7 kernel) on both channels
  - Dual-channel noise suppression
        │
        ▼
[ 2. Segmentation ]
  - Canny edge detection on Gray + Saturation channels (bitwise OR fusion)
  - Morphological closing to seal contour boundaries
  - Largest-contour extraction → binary product mask
  - Morphological opening to remove edge artefacts
        │
        ▼
[ 3. Feature Engineering ]
  - GLCM texture features (Contrast, Homogeneity) — product type classification
  - Entropy calculation (information complexity / surface detail density)
  - HSV channel statistics (Avg Hue, Avg Saturation, Avg Intensity)
        │
        ▼
[ 4. Quality Control — Ingredient Ratio Analysis ]
  - Otsu's threshold on Saturation channel of isolated product
  - Pixel-density quantification of ingredient distribution (chip_percentage)
  - Automated Accept / Reject decision against predefined density bounds
        │
        ▼
[ Output: Visual Audit Report — masks, histograms, feature metrics per unit ]
```

---

## Technical Implementation

### Dual-Channel Edge Detection (Key Design Decision)

Standard CV pipelines run Canny edge detection on grayscale alone. This pipeline runs it on **both** the grayscale channel and the HSV Saturation channel simultaneously, then fuses the results with a bitwise OR:

```python
edges_gray = cv2.Canny(blurred_gray, 30, 100)
edges_sat  = cv2.Canny(blurred_sat, 20, 80)
edges      = cv2.bitwise_or(edges_gray, edges_sat)
```

This captures edges defined by brightness contrast (grayscale) AND edges defined by colour contrast (saturation) — producing a significantly more complete product boundary on items with low luminance contrast but high colour variation.

### Texture Classification via GLCM

Product type is classified using Gray-Level Co-occurrence Matrix (GLCM) features — a spatial texture descriptor computed at distance=5, angle=0°:

```python
glcm        = graycomatrix(gray, distances=[5], angles=[0], levels=256, symmetric=True, normed=True)
contrast    = graycoprops(glcm, 'contrast')[0, 0]    # Surface roughness / edge intensity
homogeneity = graycoprops(glcm, 'homogeneity')[0, 0] # Texture uniformity
```

**Contrast** measures local pixel intensity variation — higher for rough/complex surfaces. **Homogeneity** measures how close pixel pairs are to the diagonal — higher for smooth, uniform textures. Together these two features form a 2D feature space for product variant classification.

### Ingredient Distribution Quantification

The ingredient ratio (e.g., chocolate chip coverage) is computed by Otsu-thresholding the Saturation channel of the isolated product region:

```python
chip_thresh      = threshold_otsu(cookie_s_pixels)
chip_mask        = (s_channel > chip_thresh) & cookie_pixels_mask
chip_percentage  = (np.sum(chip_mask) / np.sum(cookie_pixels_mask)) * 100
```

This outputs a precise, reproducible percentage — replacing subjective human judgment with a mathematical threshold enforceable at line speed.

---

## Output Per Unit

For each image processed, the system outputs:

| Output | Description |
|---|---|
| Original image | Source reference |
| Grayscale | Luminance channel |
| HSV | Colour space transformation |
| Binary mask | Segmented product boundary |
| Isolated product | Background-removed unit |
| HSV isolated | Colour analysis of product only |
| Intensity histogram | Pixel distribution profile |
| Chip mask | Ingredient coverage overlay |
| Feature metrics | Hue, Saturation, Intensity, Contrast, Homogeneity, Entropy, Chip% |

---

## Technical Stack

| Component | Library / Tool |
|---|---|
| Computer Vision | OpenCV (`cv2`) |
| Texture Analysis | scikit-image (`graycomatrix`, `graycoprops`, `threshold_otsu`) |
| Numerical Processing | NumPy, SciPy (`entropy`) |
| Visualisation | Matplotlib |
| Language | Python 3 |

---

## Repository Structure

```
Computer-Vision-Cookie-Inspection-System/
│
├── notebooks/
│   └── main.ipynb                    # Full pipeline execution and visual audit output
│
├── src/
│   └── inspection_pipeline.py        # Modular pipeline (production-ready structure)
│
├── data/
│   └── cookies_exampleclass/         # Input image batch (110 production line images)
│
└── requirements.txt
```

---

## Manufacturing Relevance (Industrie 4.0)

This project demonstrates the engineering stack behind **Automated Optical Inspection (AOI)** systems used in real production environments:

| Pipeline Stage | Industrial Equivalent |
|---|---|
| Dual-channel segmentation | Background subtraction in machine vision cameras |
| GLCM texture classification | Surface defect detection (PCB, metal parts, textiles) |
| Ingredient ratio thresholding | Fill-level / content distribution QC (pharma, food, packaging) |
| Accept/Reject automation | PLC-triggered reject actuator on conveyor line |
| Batch processing loop | Frame-by-frame processing in line-scan camera systems |

The modular function architecture (`identify_cookie()`) mirrors the design of production vision modules where each processing stage is independently testable, replaceable, and scalable to multi-camera setups.

---

## Future Roadmap

- [ ] Quantify and report classification accuracy across the full 110-image batch
- [ ] Add a labelled dataset and train a CNN classifier for multi-class product variant detection
- [ ] Implement processing speed benchmark (ms/image) to establish line-speed viability
- [ ] Extend Accept/Reject logic to multi-parameter composite scoring (texture + ingredient ratio + shape)
- [ ] Package `inspection_pipeline.py` as a deployable module with a config-driven threshold file

---

## Setup

```bash
git clone https://github.com/YashBarhanpurkar/cookie-inspection.git
cd cookie-inspection
pip install -r requirements.txt
# Open notebooks/main.ipynb in Jupyter
# Place production images in data/cookies_exampleclass/
```

---

## Author

**Yash Barhanpurkar**  
M.Sc. Global Production Engineering — Technische Universität Berlin  
[LinkedIn](https://www.linkedin.com/in/yash-barhanpurkar/) · [GitHub](https://github.com/YashBarhanpurkar)

---

*Topics: `computer-vision` `opencv` `quality-control` `automated-inspection` `industrie-4-0` `python` `glcm` `production-line` `manufacturing` `image-processing`*