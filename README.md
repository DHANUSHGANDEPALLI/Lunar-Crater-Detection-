# Lunar-Crater-Detection-
To Detect the sub kilometer craters on the lunar surface using YOLOv8 model , trained with custom dataset prepared by tiling LROC WAC Mosaic 100meter / pixel.
Sub–Kilometre Lunar Crater Detection Using LROC WAC and Chandrayaan-2 OHRC Data

Repository Author: Gandepalli Dhanush
Programme: M.Tech — Control & Instrumentation, Department of Electrical Engineering
Institution: Indian Institute of Technology, Bhilai
Supervisor: Dr. Nitin Khanna

I. Overview

This repository accompanies the research work conducted toward the detection of lunar impact craters with diameters below 1 km.
The presented methodology integrates:

Low–resolution global imagery (LROC WAC, ~100 m/pixel),

High–resolution local imagery (Chandrayaan–2 OHRC, ~0.25 m/pixel),

Deep learning detection models (YOLOv8, RT-DETR*, YOLOv5*),

Catalog-based supervision (LU5M812TGT global crater catalog).

The repository contains:

Model training configurations (YOLO),

Validation and training logs (metrics, curves),

Model checkpoints,

Overlay scripts,

Inference outputs on OHRC scenes,

CRS and tiling scripts.

The raw datasets are not distributed, in accordance with NASA and ISRO licensing policies.
Instructions to reproduce the datasets are provided herein.

II. Scientific Motivation

Crater size–frequency distributions are a fundamental technique for lunar geological dating.
Large crater catalogs (≥ 1 km) are readily available due to simple visibility in global mosaics.
However, sub–kilometre craters exhibit:

Higher counts per unit area,

Sensitivity to recent impact flux,

Utility in terrain hazard evaluation,

Value for high-resolution geomorphology.

The research addresses three core challenges:

Resolution discontinuity
Training data at 100 m/pixel must generalize to inference at 0.25 m/pixel.

Metadata inconsistencies
Chandrayaan-2 OHRC PDS4 data exhibits scene-wise georeferencing errors.

Catalog reliability near resolution limits
Some LU5M812TGT crater entries are ambiguous when inspected at higher resolution.

III. Repository Structure

The repository provides everything necessary to reproduce the pipeline, except raw satellite data.

.
├── config/
│   └── data.yaml            # YOLO dataset specification
├── weights/
│   ├── best.pt              # YOLOv8 trained weights
│   └── last.pt              # Final epoch checkpoint
├── scripts/
│   ├── parse_pds4.py        # OHRC PDS4 → GeoTIFF conversion
│   ├── make_tiles_wac.py    # WAC tiling (256x256)
│   ├── make_tiles_ohrc.py   # OHRC tiling (4096x4096)
│   ├── generate_labels.py   # Catalog → YOLO labels conversion
│   ├── overlay_craters.py   # Circle overlays for validation
│   └── train_yolo.py        # Full YOLOv8 training pipeline
├── runs/
│   ├── train/               # YOLOv8 training logs
│   ├── val/                 # YOLOv8 validation metrics
│   └── detect/              # Model inference on OHRC
├── metrics/                 # Confusion, PR curves, F1 plots
└── README.md

IV. Dataset Sources (Not Included)

The dataset is reproducible using publicly available lunar mission archives.

A. LROC WAC (Training)

Mission: Lunar Reconnaissance Orbiter (LRO)

Instrument: Wide Angle Camera

Resolution: ~100 m/pixel

Product Type: Global mosaic (GeoTIFF/IMG)

B. LU5M812TGT Crater Catalog (Labels)

Global machine-generated crater catalog

Provides crater centers (lat, lon) and diameters (km)

Used to construct YOLO bounding box annotations

C. Chandrayaan-2 OHRC (High-Resolution Testing)

Resolution: 0.25 m/pixel

Product Format: PDS4 (.IMG + .XML + .CSV)

Ideal for visual validation of sub-km crater morphology

Data redistribution is prohibited.
Users must obtain raw imagery from NASA/ISRO archives.

V. CRS and Coordinate Processing
A. Lunar Reference Sphere

The Moon is approximated as a sphere:

𝑅 = 1,737,400 m
R=1,737,400 m
𝐶=2𝜋𝑅≈10,921,840 m
C=2πR≈10,921,840 m

Conversion from angular degrees:1∘≈30.338 km
1∘≈30.338 km

This conversion enables diameter→pixel scaling and bounding box computation.

B. Custom CRS: GCS Moon 2000

Used for OHRC GeoTIFF projection:

GEOGCS["GCS_Moon_2000",
  DATUM["D_Moon_2000",
    SPHEROID["Moon_2000_IAU_IAG",1737400.0,0.0]],
  PRIMEM["Reference_Meridian",0.0],
  UNIT["Degree",0.0174532925199433]]


Manually applied in GDAL pipelines to correct PDS4 .IMG products.

VI. Tile Definition
A. WAC Data (Training)

Input resolution: 100 m/pixel

Tile size: 256×256

Physical coverage: 25.6 km × 25.6 km

Total tiles: 60,506

Train: 48,404

Validation: 12,102

B. OHRC Data (Testing)

Input resolution: 0.25 m/pixel

Tile size: 4096×4096

Coverage: ~1 km × 1 km

Method: Sliding window with overlap

Post-filtering usable tiles: 725 (from 2,681)

VII. Annotation Format

Bounding boxes are generated from crater centers and diameters.

YOLO format:

class_id  x_center  y_center  width  height


All values normalized to [0,1].
Class_id = 0 (single class: crater).

VIII. Model Training
YOLOv8 Training Configuration

Epochs: 400

Input: 256×256 WAC tiles

GPU: NVIDIA A100 80GB

Training time: ~20 hours

Final model performance:

Metric	Value
Precision	~0.807
Recall	~0.660
mAP@0.5	~0.766
mAP@0.5–0.95	~0.454
IX. Cross-Resolution Generalisation

A key experiment:

Model trained only at 100 m/pixel WAC detects craters at 0.25 m/pixel OHRC.

Testing tiles: 4096×4096
Examples: tile_000068, tile_000069

Observations:

Crater rims correctly localized

Low boulder-triggering

Scale invariance emerges from model features

This is a major research contribution.

X. Known Limitations

OHRC metadata inconsistencies
Some scenes remain misaligned → invalid annotations.

Catalog ambiguity
Near WAC resolution limit, “craters” are occasionally false positives.

Highly degraded craters
Shape signatures insufficient in low-resolution data.

XI. Reproducing Training

Download WAC mosaic & LU5M812TGT

Run tiling scripts

Generate YOLO labels

Train using:

yolo detect train model=yolov8n.pt data=config/data.yaml epochs=400

XII. Acknowledgements

I acknowledge:

MANAS Lab, IIT Bhilai for providing compute resources (A100 80GB),

Indian Space Research Organisation (ISRO) for OHRC and TMC products,

NASA/LROC Team for global mosaic datasets,

My supervisor Dr. Nitin Khanna for technical guidance.

XIII. Citation

If you use this work:

Gandepalli, D. “Detection of Sub–Kilometre Lunar Impact Craters using LROC WAC and Chandrayaan-2 OHRC with Deep Learning,” M.Tech Thesis, IIT Bhilai, 2025.
