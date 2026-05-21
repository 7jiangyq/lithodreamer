# LithoDreamer Dataset Specification

This document provides a detailed specification of the LithoDreamer dataset,
including its multi-stage structure, process parameters, metadata schema,
dataset splits, and intended usage.

---

## Data Availability

Due to data size and double-blind review policy, only a subset of the dataset
is released at submission time for verification and inspection.
The complete dataset will be publicly released upon acceptance of the paper.

---

## 1. Multi-Stage Lithography Pipeline

Each sample corresponds to a complete lithography pipeline:

**Layout - Mask - Resist Image - After Development Image (ADI)**

All stages are spatially aligned and generated under the same lithography
process configuration, enabling consistent modeling of stage-to-stage
physical evolution and process interventions.

---

## 2. Process Parameters

Each sample is associated with a lithography process configuration defined by:

- **Source type:** `Annular`, `Circular`, `BullsEye`
- **Resist threshold:** `0.09231251`, `0.1236402`, `0.1436665`
- **Focus (nm):** `0`, `50`
- **Exposure dose:** `1.0`, `1.2`

These parameters form **36 distinct process configurations**.
All configurations are included in the training and in-domain test sets.

---

## 3. Dataset Directory Structure

```text
LithoDreamer-Dataset/
├── train/
│   ├── layout/
│   ├── mask/
│   ├── resist/
│   ├── adi/
│   ├── source/
│   ├── metadata/
│   │   ├── 000001.json
│   │   ├── 000002.json
│   │   └── ...
│   └── index.csv
├── test_in_domain/
├── test_ood_55nm/
├── test_ood_28nm_lithosim/
├── DATASET.md
├── README.md
└── LICENSE
```

---

## 4. Dataset Splits

- **train:** 280,000 samples covering all 36 process configurations
- **test_in_domain:** 20,000 samples from the same configuration set
- **test_ood_55nm:** unseen process configuration at 55 nm
- **test_ood_28nm_lithosim:** public LithoSim dataset (cross-node OOD)

All out-of-domain evaluations are performed using models trained solely 
on the training set.

---

## 5. Metadata Design Overview

LithoDreamer adopts a hybrid metadata design:

- Per-sample JSON files for complete, self-contained sample descriptions
- A lightweight index.csv for fast dataset loading, filtering, and statistics

This design supports large-scale datasets while maintaining flexibility and
reproducibility.

---

## 6. Metadata Schema (Per-Sample JSON)

Each metadata file is located at:

<split>/metadata/XXXXXX.json

where `XXXXXX` is a zero-padded sample identifier.

### 6.1 JSON Schema

Each per-sample metadata file follows the schema below:

```json
{
  "sample_id": "000001",
  "split": "train",
  "files": {
    "layout": "train/layout/000001.png",
    "mask": "train/mask/000001.png",
    "resist": "train/resist/000001.png",
    "adi": "train/adi/000001.png",
    "source": "train/source/Annular.png"
  },
  "process_parameters": {
    "source_type": "Annular",
    "resist_threshold": 0.09231251,
    "focus_nm": 0,
    "exposure_dose": 1.0
  }
}
```

### 6.2 Field Descriptions:
- sample_id: Zero-padded unique identifier for each sample
- split: 
  train, 
  test_in_domain, 
  test_ood_55nm, 
  test_ood_28nm_lithosim
- files:
  layout: Path to layout image
  mask: Path to mask image
  resist: Path to resist image
  adi: Path to after-development image (ADI)
  source: Path to illumination source pattern image
- process_parameters:
  source_type: Illumination source category
  resist_threshold: Resist development threshold
  focus_nm: Defocus amount in nanometers
  exposure_dose: Relative exposure dose multiplier

---

## 7. Index File Format (index.csv)

Each dataset split provides a lightweight global index file:

<split>/index.csv

### 7.1 Column Definitions

Each row in `index.csv` corresponds to one sample and contains the following columns:

- `sample_id`: Sample identifier (zero-padded)
- `split`: Dataset split
- `source_type`: Illumination source type
- `resist_threshold`: Resist threshold
- `focus_nm`: Focus value in nanometers
- `exposure_dose`: Exposure dose multiplier

### 7.2 Example

```csv
sample_id,split,source_type,resist_threshold,focus_nm,exposure_dose
000001,train,Annular,0.09231251,0,1.0
000002,train,Annular,0.09231251,0,1.0
000003,train,Circular,0.1236402,50,1.2
```

---

## 8. Design Rationale

This metadata design balances scalability, clarity, and flexibility:

- Per-sample JSON files ensure full reproducibility
- index.csv enables efficient large-scale loading and filtering
- Process parameters are decoupled from directory structure, supporting 
  systematic out-of-domain evaluation
