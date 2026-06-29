gcn-dae-survival
Graph-Fused Multi-Omic Autoencoders for Assumption-Free Cancer Survival Prediction with Pathway-Level Interpretability
> Solo independent research project. All experiments run on CPU, Windows 10.  
> Manuscript submitted to *Briefings in Bioinformatics* (Oxford).
---
Overview
This repository contains the full pipeline for GCN+DAE — a graph attention network fused with a denoising autoencoder trained jointly with a discrete-time survival head — evaluated across three TCGA cancer cohorts.
Key contributions:
Survival prediction without the Cox proportional hazards assumption
Pathway-level attention weights that recover known cancer drivers without supervision
Graph fusion most benefits high-risk patients — where clinical value is greatest
End-to-end implementation in PyTorch + PyTorch Geometric
---
Results summary
Cohort	C-index	Events / N
TCGA-LGG (Brain)	0.779 ± 0.024	126 / 512
TCGA-KIRC (Kidney)	0.699 ± 0.042	175 / 531
TCGA-LUAD (Lung)	0.587 ± 0.035	182 / 504
5-fold cross-validated. No Cox assumption. Omics only (no histopathology).
---
Repository structure
```
gcn-dae-survival/
├── README.md
├── requirements.txt          # Full environment specification
├── config.yaml               # All hyperparameters (single source of truth)
│
├── NB00_mlp_baseline.ipynb   # Stage 0 — MLP baseline (Cox loss)
├── NB01_dae_joint.ipynb      # Stage 1 — DAE + joint Cox head
├── NB02_dae_baseline.ipynb   # Stage 2 — DAE + discrete hazard (no graph)
├── NB03_tabular_benchmarks.ipynb  # Stage 3 — tabular Cox-MLP benchmarks
├── NB04_pathway_gcn.ipynb    # Stage 4 — GCN+DAE (main model)
├── NB05_representation_comparison.ipynb  # Stage 5 — ablation and visualisation
│
└── data/
    └── README.md             # Download instructions (data not included)
```
---
Pipeline stages
Notebook	Description	Key output
NB00	MLP trained on raw expression + Cox loss	Discriminative upper bound
NB01	DAE encoder + Cox survival head, joint training	Isolated vs joint comparison
NB02	DAE encoder + discrete hazard head, no graph	Ablation: no graph fusion
NB03	Deep and shallow Cox-MLP tabular baselines	External benchmark reference
NB04	GCN+DAE — main model	C-index, pathway attention, latent space
NB05	DAE vs GCN+DAE ablation, radar summary	Representation quality metrics
Run notebooks in order NB00 → NB05. Each stage saves checkpoints consumed by the next.
---
Data
Data is not included in this repository due to size and access terms.
All data is publicly available from the Genomic Data Commons (GDC) portal:  
https://portal.gdc.cancer.gov
Files required per cohort
For each of TCGA-LGG, TCGA-KIRC, TCGA-LUAD download:
File type	GDC data type	Format
mRNA expression	Gene Expression Quantification	STAR FPKM `.tsv`
Survival / clinical	Clinical Supplement	`.tsv`
Place downloaded files in `data/raw/` following the naming convention in `config.yaml`:
```
data/raw/
├── TCGA-LGG.star_fpkm.tsv
├── TCGA-LGG.survival.tsv
├── TCGA-KIRC.star_fpkm.tsv
├── TCGA-KIRC.survival.tsv
├── TCGA-LUAD.star_fpkm.tsv
└── TCGA-LUAD.survival.tsv
```
---
Installation
1. Clone the repository
```bash
git clone https://github.com/[your-username]/gcn-dae-survival.git
cd gcn-dae-survival
```
2. Create a virtual environment (recommended)
```bash
python -m venv venv
venv\Scripts\activate        # Windows
# source venv/bin/activate   # macOS / Linux
```
3. Install dependencies
```bash
pip install -r requirements.txt
```
> **Note:** `torch-scatter` and `torch-sparse` are CPU builds tied to PyTorch 2.12.0.  
> If you use a different PyTorch version or GPU, reinstall these separately following  
> the [PyTorch Geometric installation guide](https://pytorch-geometric.readthedocs.io/en/latest/install/installation.html).
4. Launch JupyterLab
```bash
jupyter lab
```
---
Reproducibility
All experiments used:
Random seed: 42 (applied to Python, NumPy, PyTorch, CUDA)
Hardware: CPU only, Windows 10 (Version 10.0.26200)
Python: 3.12.3
Key package versions (full list in `requirements.txt`):
```
torch==2.12.0
torch-geometric==2.8.0
torch-scatter==2.1.2+pt212cpu
torch-sparse==0.6.18+pt212cpu
numpy==2.4.6
scikit-learn==1.8.0
lifelines==0.30.3
pandas==2.3.3
scipy==1.17.1
networkx==3.6.1
```
> Minor numerical variation may arise on different hardware due to floating-point
> non-determinism in parallel operations. Reported C-index values are five-fold
> cross-validation means and are expected to fall within the reported standard
> deviations across independent reproductions.
---
Configuration
All hyperparameters are defined in `config.yaml` and loaded by each notebook.  
No hardcoded values in notebooks — edit `config.yaml` to change any setting.
Key parameters for the main model (NB04):
```yaml
stage4_gcn:
  latent_dim: 64
  num_intervals: 10
  gcn_hidden_channels: [128, 64]
  dropout_rate: 0.4
  structural_loss_weight: 1.2
  discrete_hazard_weight: 1.0
  cosine_loss_weight: 0.5

training:
  seed: 42
  epochs: 40
  batch_size: 64
  lr: 0.0005
  weight_decay: 0.01
  patience: 8
```
---
Citation
If you use this code or build on this work, please cite:
```
[Author Name]. Graph-Fused Multi-Omic Autoencoders for Assumption-Free
Cancer Survival Prediction with Pathway-Level Interpretability.
Briefings in Bioinformatics, [year]. [DOI to be added upon publication]
```
---
Contact
For questions, code requests, or data access guidance:  
[your email address]  
Independent Researcher
---
License
This repository is released for academic and research use.  
TCGA data is subject to the NIH GDC Data Access Policy.
