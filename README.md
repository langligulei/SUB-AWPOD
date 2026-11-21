# SUB-AWPOD: Sparse Full-Appendage Surface-Pressure Reconstruction for a Submarine

Reproduction materials for the manuscript:

> **Sparse reconstruction of the full-appendage surface pressure of a submarine based on Area-Weighted Proper Orthogonal Decomposition and optimized sensor placement** (revised version, under review)

This repository hosts the surface-pressure dataset, the optimized 126-tap sensor layout, the unseen test library, and all numerical results reported in the manuscript. Large binary files are distributed through Baidu Netdisk (links below); everything else is stored directly in the repository.

---

## 1. Method at a glance

- **Geometry**: DARPA SUBOFF AFF-8 (fully appended configuration).
- **Snapshot library**: steady RANS (STAR-CCM+, SST *k*–ω). The three-component maneuvering library contains **232 training conditions**, all computed as **independent steady RANS solutions** (no data-level mirroring or symmetry augmentation); **52 unseen conditions** form the strictly separated test library. A straight-ahead library (10 training / 6 test) is used for the grid-convergence and straight-sailing validations.
- **Reduced basis**: Area-Weighted POD (AWPOD) on a topology-consistent surface mesh with ≈ 1.22 × 10⁵ shared faces; cell-area weights *W* = diag(*Aᵢ*).
- **Sensor placement**: Q-DEIM/DEIM selection combined with a zoned hull–sail–fin layout, giving **126 pressure taps** (106 hull + 4 sail + 4 × 4 fins).
- **Online reconstruction**: prior-regularized gappy POD solve with a velocity prior *ĉ₀* = *f*(*u, v, w*) from ridge/RBF regression over the training library; *r* = 30 retained modes.

## 2. Repository structure

```
SUB-AWPOD/
├── LICENSE                              # MIT License
├── README.md                            # this file
├── Grid_Convergence_Summary.csv         # grid-convergence data (Fig. 2 of the paper)
├── modal_surface_dataset_mix/           # surface-pressure modal dataset (training library)
│   ├── topology.h5                      # surface topology: shared faces, cell areas (W), normals
│   ├── cases.csv                        # list of the 232 training conditions (Appendix A2)
│   ├── cases/
│   │   └── cases_link.md                # Baidu Netdisk link for the 232 case snapshots (*.h5, ~3.2 GB)
│   └── sensor_check/                    # sensor observations and extraction audit
│       ├── extraction_report.csv        # per-condition sensor extraction audit
│       ├── sensor_measurements_long.csv # sensor measurements, long format
│       ├── sensor_measurements_matrix.csv
│       └── star_sensor_csv/             # per-condition tap readings exported from STAR-CCM+
│                                          #   PracticalSensorFromStar_<case>.csv (52 test conditions)
├── results/
│   ├── tables/                          # all quantitative results (CSV)
│   └── figures/                         # all result figures (PNG)
└── unseen_cases/
    └── unseen_cases_link.md             # Baidu Netdisk link for the 52 unseen test conditions (*.h5, ~141 MB)
```

## 3. Large-file download (Baidu Netdisk)

Two directories are too large for GitHub and are distributed via Baidu Netdisk. After downloading, place the files in the corresponding directories so that the layout above is restored.

| Content | Files | Size | Destination | Link | Access code |
|---|---|---|---|---|---|
| 232 training-condition snapshots (`suboff_0000_*.h5`) | 232 HDF5 | ~3.2 GB | `modal_surface_dataset_mix/cases/` | <https://pan.baidu.com/s/1M60HAeROmZ-8KVumn64fjw?pwd=v2xp> | `v2xp` |
| 52 unseen test conditions (`suboff_0000_*.h5`) | 52 HDF5 | ~141 MB | `unseen_cases/` | <https://pan.baidu.com/s/1jie46y8DDYTzXFX56BjnfQ?pwd=r7u1> | `r7u1` |

**Naming convention**: `suboff_0000_u<U>v<V>w<W>` encodes the inflow velocity components (m/s) with `m` for the minus sign and `p` for the decimal point; e.g. `suboff_0000_u-0p5_v0p5_w0p5` denotes (*u*, *v*, *w*) = (−0.5, +0.5, +0.5) m/s. Files under `sensor_check/star_sensor_csv/` use the equivalent `um0p5` prefix style.

## 4. Data description

### 4.1 `modal_surface_dataset_mix/` — training library

- **`topology.h5`**: topology-consistent surface discretization shared by all conditions — face connectivity (≈ 1.22 × 10⁵ shared faces), cell areas *Aᵢ* (the area weights *W*), and unit normals *nᵢ*. All POD, sensor-placement, and load-integration operations in the paper are defined on this topology.
- **`cases.csv`**: index of the 232 training conditions with their (*u*, *v*, *w*) parameters; this is the source of Table A2 in the appendix.
- **`cases/*.h5`** (via Netdisk): surface-pressure snapshots, one HDF5 per condition, all from independent steady RANS solutions.
- **`sensor_check/`**: the 126-tap measurement data — per-tap readings exported from STAR-CCM+ (`star_sensor_csv/`), merged long/matrix tables, and the extraction audit report.

### 4.2 `unseen_cases/` — test library

52 conditions never used in the POD basis, the sensor placement, or the velocity prior (strict separation in velocity-parameter space, no data leakage); this is the source of Table A3 in the appendix.

### 4.3 `results/` — numerical results

`results/tables/` contains the CSV source data of every quantitative table and figure in Section 4:

| File | Content |
|---|---|
| `pod_mode_sensitivity.csv`, `sensor_count_sensitivity.csv` | Modal-order and tap-count scans (Table 1, Fig. 8) |
| `multi_condition_generalization.csv`, `inversion_algorithm_results.csv` | Reconstruction errors over the unseen library (Table 3) |
| `baseline_comparison.csv`, `random_sensor_raw.csv`, `random_sensor_statistics.csv` | Baselines: mean field, RBF interpolation, equally weighted gappy POD, random layouts (Table 4, Fig. 11) |
| `parameter_rbf_prediction.csv` | Velocity-prior regression accuracy |
| `pod_energy_by_test_case.csv` | Per-condition POD energy / truncation analysis |
| `noise_robustness_raw.csv`, `noise_robustness_statistics.csv` | Noise-robustness experiments |
| `pressure_error_summary.csv`, `pressure_integral_errors.csv` | Field errors and integrated 6-DOF load errors (Figs. 13–14) |

`results/figures/` contains the corresponding rendered figures (`figure_*.png`), the full-set pressure-distribution plates (`pressure_distribution_all_cases_page_*.png`), and the per-condition pressure/error maps for all 52 test conditions (`*_pressure_error_distribution.png`).

### 4.4 `Grid_Convergence_Summary.csv`

Drag-coefficient convergence over the grid-refinement sequence, including the Richardson-extrapolated continuum estimate — the source data of Fig. 2.

## 5. Reproduction code

The reproduction pipeline (Python: dataset construction, AWPOD basis, Q-DEIM/QR sensor placement, prior-regularized gappy POD solve, baseline comparisons, and figure generation) is being organized and will be uploaded to this same repository. Requirements: Python ≥ 3.11 with `numpy`, `scipy`, `h5py`, `pandas`, `matplotlib`.

## 6. Version note

The dataset in this repository (uploaded November 2026) is the corrected, fully recomputed CFD library used in the revised manuscript; it supersedes any earlier copies of the three-component maneuvering data.

## 7. License and citation

- Released under the **MIT License** (see `LICENSE`).
- If you use these materials, please cite the manuscript; full citation details will be added upon publication.

## 8. Contact

Questions and issues: please open an issue at <https://github.com/langligulei/SUB-AWPOD/issues>.
