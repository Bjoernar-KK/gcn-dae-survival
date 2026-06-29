# gcn-dae-survival
scipy==1.17.1
networkx==3.6.1
```

> Minor numerical variation may arise on different hardware due to floating-point
> non-determinism in parallel operations. Reported C-index values are five-fold
> cross-validation means and are expected to fall within the reported standard
> deviations across independent reproductions.

---

## Configuration

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

## Citation

If you use this code or build on this work, please cite:

```
[Author Name]. Graph-Fused Multi-Omic Autoencoders for Assumption-Free
Cancer Survival Prediction with Pathway-Level Interpretability.
Briefings in Bioinformatics, [year]. [DOI to be added upon publication]
```

---

## Contact

For questions, code requests, or data access guidance:  
**[your email address]**  
Independent Researcher

---

## License

This repository is released for academic and research use.  
TCGA data is subject to the [NIH GDC Data Access Policy](https://gdc.cancer.gov/access-data/data-access-policies).
