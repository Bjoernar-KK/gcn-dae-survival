# Bypassing Cox: Assumption-Free Cancer Survival Prediction from a Fraction of the Genome

**A replication-tested, multi-cohort benchmark of pathway-constrained graph attention (GCN-Pathway) against five internally-controlled survival-modeling baselines on three TCGA cohorts — with a dedicated audit notebook verifying that every exported embedding in the pipeline is safe (or not) to reuse downstream.**

Kit K. Ko · Independent Researcher
Correspondence: bjoernar.kk@gmail.com · ORCID: [0009-0005-5305-1287](https://orcid.org/0009-0005-5305-1287)

Preprint: *[bioRxiv link — add once posted]*

---

## TL;DR

- GCN-Pathway drops the Cox proportional-hazards assumption entirely and cuts the input from 3,000 genes to **48–56 curated pathway genes**, landing within 0.03–0.11 C-index of the best Cox-based baseline trained on the full feature set.
- Naively reporting single-run pathway-attention weights would have produced **three** plausible "discoveries." **5-seed replication shows only one survives** (TCGA-KIRC's VHL/HIF axis, 5/5 seeds). The other two (NOTCH in LGG, NRF2/KEAP1 in LUAD) never won a single seed — they were noise.
- A separate, independent audit (`NB06`) found that **none of this pipeline's exported embedding files — DAE-based or GCN-Pathway alike, in any of the three cohorts — are safe to reuse for downstream supervised benchmarking.** A model fit on any exported "held-out" embedding scores 0.08–0.31 C-index higher than that same model's own honest, properly cross-validated number.
- This repo reports both the finding and its own failure modes: two null results, one unresolved architectural bug, and one disclosed data-handling caveat, alongside the one result that replicates.

---

## Repository structure

```
├── config.yaml                          # single source of truth for every notebook (cohorts, hyperparameters, paths)
├── data/raw/                            # TCGA-{LGG,KIRC,LUAD} expression + survival files (not tracked — see Data below)
├── notebooks/
│   ├── NB00_mlp_baseline.ipynb          # Cox-MLP baseline (3,000 genes)
│   ├── NB01_dae_joint.ipynb             # DAE + Cox, isolated and joint heads
│   ├── NB02_dae_baseline.ipynb          # DAE + discrete-time hazard (no graph)
│   ├── NB03_tabular_benchmarks.ipynb    # Deep / shallow Cox-MLP baselines
│   ├── NB04_pathway_gcn.ipynb           # GCN-Pathway: the paper's central model + 5-seed replication
│   ├── NB05_representation_comparison.ipynb  # Unsupervised ablation: patient-similarity graph vs. pathway graph
│   └── NB06_representation_audit.ipynb  # Independent audit of every exported embedding, all 3 cohorts
├── checkpoints/<NB_STAGE>/              # trained model weights, one per cohort per notebook
├── embeddings/<NB_STAGE>/               # exported latent representations — see ⚠️ warning below
├── results/<NB_STAGE>/figures/          # generated figures
└── paper/                               # manuscript source (.md / .docx / .pdf)
```

Every notebook reads its hyperparameters, cohort list, and paths from `config.yaml` — nothing is hardcoded per-cohort. Restart-and-run-all should reproduce every number in the paper from `NB00` through `NB06`, in order.

## ⚠️ Before you reuse anything from `embeddings/`

**Do not fit a downstream model on any file in `embeddings/` and report its performance as if it were a fair held-out evaluation.** Every embedding export in this pipeline (DAE Isolated, DAE Joint, GCN-Pathway, DAE+GCN Fusion) is generated from each model's *final full-cohort refit*, not a genuine out-of-sample split — the "heldout" in a filename describes which scaler was used, not the training set of the model that produced the numbers in that file. `NB06` quantifies exactly how much signal this leaks (0.08–0.31 C-index, depending on cohort and embedding family — see Table 4 of the paper). The only numbers safe to cite as this pipeline's actual performance are the ones in Table 1–3 of the paper, all computed within proper cross-validation inside their source notebook.

## Results summary

**Cross-validated C-index (Table 1 of the paper):**

| Model | TCGA-LGG | TCGA-KIRC | TCGA-LUAD | Genes | Cox assumption? |
|---|---|---|---|---|---|
| MLP baseline (Cox) | 0.838 ± 0.024 | 0.708 ± 0.041 | 0.616 ± 0.017 | 3,000 | Yes |
| DAE + Cox, isolated | 0.842 | 0.709 | 0.511 | 3,000 | Yes |
| DAE + Cox, joint | 0.848 | 0.732 | 0.603 | 3,000 | Yes |
| DAE + discrete hazard | 0.804 ± 0.065 | 0.685 ± 0.052 | 0.607 ± 0.037 | 3,000 | No |
| Deep / Shallow Cox-MLP | 0.824 / 0.844 | 0.754 / 0.780 | 0.640 / 0.622 | 3,000 | Yes |
| **GCN-Pathway (this work)** | **0.790 ± 0.033** | **0.671 ± 0.034** | **0.595 ± 0.024** | **48–56** | **No** |

**Pathway-attention 5-seed replication (Table 2):** only **TCGA-KIRC / VHL_HIF** is dominant in ≥4/5 seeds (5/5, STABLE). TCGA-LGG and TCGA-LUAD show no pathway winning a majority of seeds (UNSTABLE) — including the two pathways an earlier single-run analysis would have reported as findings.

**Representation-audit leakage gap (Table 4, new in `NB06`):**

| Embedding family | LGG gap | KIRC gap | LUAD gap |
|---|---|---|---|
| DAE Isolated | +0.082 | +0.158 | +0.310 |
| DAE Joint | +0.076 | +0.135 | +0.219 |
| GCN-Pathway | +0.088 | +0.143 | +0.097 |

Full methodology, all baselines, the topology-symmetry collapse diagnostic, and the unsupervised negative-control ablation are in the paper.

## Reproducing this work

```bash
pip install -r requirements.txt
# place TCGA-{LGG,KIRC,LUAD} STAR-FPKM expression + survival TSVs in data/raw/
# run notebooks in order: NB00 → NB01 → NB02 → NB03 → NB04 → NB05 → NB06
```

Every notebook is seeded (`training.seed: 42` in `config.yaml`, CPU-only, `cudnn.deterministic=True`) for exact reproducibility on identical hardware/software; minor numerical drift is possible across different environments.

**Environment (as used for the results above):** Python 3.12.3 · PyTorch 2.12.0 (CPU) · PyTorch Geometric 2.8.0 · scikit-learn 1.8.0 · lifelines 0.30.3 · pandas 2.3.3 · see `requirements.txt` for the complete pinned list.

## Data

TCGA-LGG, TCGA-KIRC, and TCGA-LUAD expression (STAR FPKM) and survival data are publicly available from the [GDC Data Portal](https://portal.gdc.cancer.gov) and are not redistributed in this repository. Trained checkpoints are available on request.

## Known limitations (see paper §4.4 for full discussion)

- Single omic modality: gene expression only, no methylation or miRNA.
- Topological symmetry collapse in the pathway-crosstalk graph (§3.3) is diagnosed and partially, not fully, fixed.
- `NB01`'s checkpoint currently persists the joint Cox head's honest C-index but not the isolated head's — a data-handling gap identified while building `NB06`, documented in that notebook's closing cell, not yet patched.
- All evaluation is within-TCGA; no external cohort validation yet.

## Citation

If you use this code or build on this work, please cite the preprint:

```bibtex
@article{ko2026gcnpathway,
  title   = {Bypassing Cox: Assumption-Free Cancer Survival Prediction from a Fraction of the Genome},
  author  = {Ko, Kit K.},
  year    = {2026},
  journal = {bioRxiv},
  doi     = {[add once assigned]}
}
```

## License

[Add your chosen license here — e.g. MIT for code, CC-BY-4.0 for the paper text/figures, matching whatever you select at bioRxiv submission.]

## Author

**Kit K. Ko** — Independent Researcher
bjoernar.kk@gmail.com · [ORCID: 0009-0005-5305-1287](https://orcid.org/0009-0005-5305-1287)
