# Synthetic Data Generation for Smart-Meter Household Load Profiles in the UK Energy Sector

**Module:** CIS4006-N Advanced Practice
**Programme:** MSc Artificial Intelligence with Advanced Practice
**Institution:** Teesside University

A practical artefact for the reflective ICA. The notebook builds and compares four generative methods for UK domestic half-hourly load profiles, then writes every metric, table, figure and trained model to `outputs/` so the report can quote them directly.

## Why synthetic data

Real UK domestic smart-meter readings are personal data under UK GDPR and the Data Protection Act 2018 because they reveal occupancy, lifestyle and routine. Synthetic profiles that preserve the statistical structure of real consumption can train downstream AI for forecasting, demand response and anomaly detection without the privacy risk. The notebook simulates a UK-style dataset to the published characteristics of UK domestic load (DECC, 2014; UK Power Networks, 2014) so the work is fully reproducible without a registered data-sharing agreement.

## Methods compared

| Method                        | Type                                        | Purpose                                         |
| ----------------------------- | ------------------------------------------- | ----------------------------------------------- |
| Gaussian Mixture Model (GMM)  | Classical density estimator                 | Strong tabular baseline                         |
| Variational Autoencoder (VAE) | Deep latent-variable model                  | Smooth latent space, smooth samples             |
| Vanilla GAN                   | Fully-connected adversarial model           | Visible mode-collapse and instability behaviour |
| LSTM-GAN                      | Recurrent adversarial model (TimeGAN-style) | Temporal structure across 48 half-hours         |

Evaluation is identical across all four:

- per-dimension Kolmogorov–Smirnov statistic (mean over 48 half-hours)
- per-dimension Wasserstein distance (mean over 48 half-hours)
- mean profile MAE
- PCA and t-SNE overlays against held-out real test data
- TSTR (Train on Synthetic, Test on Real) downstream classifier
- nearest-neighbour distance to the training set as a first-pass privacy check

## Repository layout

```
Synthetic Data Generation In Energy Sector/
├── README.md                              this file
├── SETUP.md                               step-by-step environment install
├── requirements.txt                       Python dependencies (PyTorch installed separately)
├── verify_env.py                          environment sanity check
├── build_notebook.py                      regenerates the notebook from spec
├── insert_observations.py                 adds the post-run observation cells
├── inspect_outputs.py                     prints text outputs from the executed notebook
├── notebooks/
│   └── synthetic_smart_meter.ipynb        the practical artefact (47 + 24 cells)
├── data/
│   └── uk_smart_meter_simulated.csv       generated dataset (7000 rows × 48 half-hours + metadata)
├── outputs/
│   ├── figures/                           16 PNG plots
│   ├── models/                            VAE, GAN, LSTM-GAN weights (.pt)
│   └── synthetic_data/                    .npy synthetic arrays + comparison/TSTR/privacy CSVs + results_summary.json
├── reports/                               reflective ICA goes here later
└── synthetic-energy-venv/                 isolated Python 3.10 environment
```

## Environment

Built and tested on:

- Windows 11
- Python 3.10
- NVIDIA GeForce RTX 5060 Laptop GPU (Blackwell, sm_120, 8 GB VRAM)
- PyTorch 2.11.0 + CUDA 12.8

For first-time install follow [SETUP.md](SETUP.md) end-to-end. After install, every new terminal session needs:

```powershell
.\synthetic-energy-venv\Scripts\Activate.ps1
```

## Running the notebook

1. Open the project folder in VS Code.
2. Activate the venv (line above).
3. Open `notebooks/synthetic_smart_meter.ipynb`.
4. Click the kernel picker (top-right of the notebook) and select **Python (synthetic-energy)**.
5. Run cells top to bottom. Total wall-clock on an RTX 5060 is roughly 5 to 7 minutes.

Reproducibility: `SEED = 42` is set across `random`, `numpy`, `torch` and `torch.cuda` in section 2.

## What the notebook produces

After a full run the notebook writes:

- `data/uk_smart_meter_simulated.csv` — 7000 daily profiles (500 households × 14 days) with archetype and weekend metadata
- 16 PNG figures under `outputs/figures/` (EDA, training losses, sample overviews, PCA and t-SNE overlays, privacy histogram)
- 5 PyTorch weight files under `outputs/models/`
- 4 synthetic `.npy` arrays under `outputs/synthetic_data/`
- 3 CSV tables (`comparison_table.csv`, `tstr_table.csv`, `privacy_table.csv`)
- `outputs/synthetic_data/results_summary.json` — every headline metric and configuration value in one machine-readable place

The final code cell prints a single `=== FINAL SUMMARY ===` block with the configuration, dataset stats, model parameter counts, training times, distributional metrics, TSTR accuracy and privacy distances. The reflective ICA's *Activities* and *Contribution* sections quote from this block.

## Headline results from the executed run

| Method      | KS (mean) | Wasserstein (mean) | Mean profile MAE |
| ----------- | --------- | ------------------ | ---------------- |
| GMM         | 0.0288    | 0.0063             | 0.0029           |
| Vanilla GAN | 0.1364    | 0.0363             | 0.0336           |
| LSTM-GAN    | 0.4145    | 0.1075             | 0.0723           |
| VAE         | 0.4560    | 0.0958             | 0.0090           |

Lower is better in all three columns. The classical GMM beat all three deep models on this dataset and at this scale, which is the finding the report leads with.

Training times on the RTX 5060: VAE 43.7 s, vanilla GAN 67.6 s, LSTM-GAN 99.8 s.

TSTR weekday-vs-weekend baseline: Real to Real 0.9564; all synthetic-to-real settings collapsed to 0.7107 (the majority class). The collapse is an evaluation-design weakness called out explicitly in the notebook observations and the *Reflection* section of the report.

## ICA report mapping

The notebook's printed numbers and figures feed all five marking criteria:

- **Activities (1):** the simulator, the four trained models, the comparison block.
- **Contribution (2):** comparing four generative methods on the same data with the same metrics.
- **Reflection (3):** the honest "what worked / what did not" hooks in the observation cells (oversmoothed VAE, flat-equilibrium GAN, TSTR collapse).
- **Personal benefits (4):** first end-to-end generative-modelling project on a personal GPU.
- **Professional values (5):** mapped to BCS RITTech P1 Autonomy, P2 Influence, P3 Complexity and P4 Business Skills in the observation cells and in section 14 (artefact persistence).



## References

- BCS, The Chartered Institute for IT (2024) *Registered IT Technician (RITTech) Criteria*. Swindon: BCS.
- DECC (2014) *Energy Consumption in the UK*. London: Department of Energy and Climate Change.
- Goodfellow, I. *et al.* (2014) 'Generative adversarial nets', *Advances in Neural Information Processing Systems*, 27.
- Information Commissioner's Office (2018) *Guide to the General Data Protection Regulation*. Wilmslow: ICO.
- Kingma, D. P. and Welling, M. (2014) 'Auto-encoding variational Bayes', *ICLR*.
- UK Power Networks (2014) *SmartMeter Energy Consumption Data in London Households* (Low Carbon London project). London Datastore.
- Yoon, J., Jarrett, D. and van der Schaar, M. (2019) 'Time-series generative adversarial networks', *Advances in Neural Information Processing Systems*, 32.
