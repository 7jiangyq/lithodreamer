# LithoDreamer Dataset

**LithoDreamer: A Physics-Informed World Model Dataset for Multi-Stage Computational Lithography**

LithoDreamer is a large-scale, physics-driven lithography dataset designed for
**multi-stage world modeling**, covering the complete lithography pipeline:

**Layout - Mask - Resist Image - After Development Image (ADI)**

All stages are spatially aligned and generated under controlled process
configurations, enabling joint modeling of **process interventions** and
**state evolution**.

---

## Why LithoDreamer?

Most existing lithography datasets (e.g., LithoSim, Unitho) provide only
*partial stage pairs*, such as:

- Layout - Mask
- Layout - Mask - Resist Image

LithoDreamer differs in that it provides **stage-aligned multi-stage tuples**
generated under explicit lithography process conditions, enabling:

- Physics-informed world models
- Process-aware generative modeling
- Cross-stage prediction and intervention analysis
- Robustness and generalization evaluation under process variations

---

## Dataset Overview

- **Process node:** 55 nm
- **Stages:** Layout, Mask, Resist Image, ADI
- **Total samples:** 300k+
- **Training set:** 280k
- **In-domain test set:** 20k

> **Data availability note.**
> Due to data size and review policy, only a subset of the dataset is released at submission time.
> The complete dataset will be publicly released upon acceptance of the paper.
 

### Out-of-Domain (OOD) Test Sets
- **55 nm unseen process configuration**
  - Source: Annular
  - Resist threshold: 0.119340
  - Focus: 0 nm
  - Exposure dose: 1.0×
- **Public 28 nm LithoSim dataset** (cross-node OOD)

---

## Process Parameters

Each sample is generated under a lithography process configuration defined by:

- Illumination source type  
- Resist development threshold  
- Defocus  
- Exposure dose  

These parameters form multiple distinct process configurations.
All configurations used for training and in-domain testing are fully covered.

**Detailed parameter definitions and value ranges are provided in
[`DATASET.md`](DATASET.md).**

---

## Data Format

Each sample consists of a stage-aligned tuple:

> **(Layout, Mask, Resist Image, ADI)**

All images are spatially aligned and stored as PNG files with consistent
resolution. Sample-level process parameters and file mappings are provided
via per-sample metadata files.

Process parameters are stored **at the sample level**, rather than encoded in
directory names, enabling flexible conditioning and out-of-domain evaluation.

---

## Directory Structure

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
│   └── ...
├── test_ood_55nm/
│   └── ...
├── test_ood_28nm_lithosim/
│   └── ...
├── DATASET.md
├── README.md
└── LICENSE
```

---

## Quick Start: Loading the Dataset

This example is provided for illustration purposes only.
Users may adapt it to their preferred data loading framework (e.g., PyTorch, TensorFlow).

Below is a Python-style pseudo-code example demonstrating how to load
the LithoDreamer dataset using `index.csv` and per-sample metadata files.

```python
import csv
import json
from PIL import Image

class LithoDreamerDataset:
    def __init__(self, root, split="train"):
        self.root = root
        self.split = split
        self.index_file = f"{root}/{split}/index.csv"
        self.metadata_dir = f"{root}/{split}/metadata"

        with open(self.index_file, "r") as f:
            reader = csv.DictReader(f)
            self.samples = list(reader)

    def __len__(self):
        return len(self.samples)

    def __getitem__(self, idx):
        sample = self.samples[idx]
        sample_id = sample["sample_id"]

        meta_path = f"{self.metadata_dir}/{sample_id}.json"
        with open(meta_path, "r") as f:
            meta = json.load(f)

        layout = Image.open(f"{self.root}/{meta['files']['layout']}")
        mask = Image.open(f"{self.root}/{meta['files']['mask']}")
        resist = Image.open(f"{self.root}/{meta['files']['resist']}")
        adi = Image.open(f"{self.root}/{meta['files']['adi']}")

        process_params = meta["process_parameters"]

        return {
            "layout": layout,
            "mask": mask,
            "resist": resist,
            "adi": adi,
            "process": process_params
        }
```

Each returned sample contains both multi-stage images and the corresponding
process parameters, enabling joint modeling of state evolution and process interventions.