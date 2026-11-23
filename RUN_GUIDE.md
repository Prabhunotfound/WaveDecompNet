# Run guide

How to reproduce the results, in order. Notebook names refer to the `Notebooks/` folder. Data links are in `Data/Data_Info.md`.

All training and evaluation ran on Kaggle, except step 1, which ran on a local PC. All runs use seed 99. The authors' code is cloned from https://github.com/yinjiuxun/WaveDecompNet-paper .

## 1. Order

| # | Notebook | Where | Purpose |
|---|---|---|---|
| 1 | `01_build_stead_subset` | Local PC | downloads the six STEAD chunks and keeps the needed traces |
| 2 | `02_prepare_stead_data` | Kaggle | authors' `step2a` to `step2d`: POHA noise download, mixtures, final file |
| 3 | `03_train_stead_models` | Kaggle | trains the five paper bottlenecks on STEAD, evaluates them |
| 4 | `04_inspect_instance`  | Kaggle | reads the INSTANCE sample, changes nothing |
| 5 | `05_build_instance_mixtures` | Kaggle | builds the INSTANCE mixtures |
| 6 | `06_train_instance_models` | Kaggle | trains the five paper bottlenecks on INSTANCE |
| 7 | `07_variants_and_evaluation` | Kaggle | self-test, batch-size test, trains the four variants, scores every model, figures |
| 8 | `08_real_data_all_models` | Kaggle | STA/LTA detection with every model on real continuous data and on synthetic streams |

## 2. Step by step

### Step 1: `01_build_stead_subset`
Set `WORK` and `OUT` in the configuration cell, then run `run()`.

### Step 2: `02_prepare_stead_data`
Internet on. Attach the STEAD subset dataset (`DAT_Repo_Data`) and the code dataset (`wdn-code`). Run the cells in order.

### Step 3: `03_train_stead_models`
Attach the STEAD mixtures dataset. Settings are at the top of the notebook:

| Switch | Meaning | Value used |
|---|---|---|
| `RUN_PART_B_SMOKE` | 2-epoch check on 5,000 samples | `True` before every full run |
| `RUN_PART_B_FULL` | full training of **one** bottleneck and **one** seed | `True` |
| `BOTTLENECK` | `None`, `Linear`, `LSTM`, `attention` or `Transformer` | one per session |
| `SEED`, `EPOCHS` | 99, 100 |
| `RUN_PART_C` | evaluation on the STEAD test split, with baselines | `True` once, after all runs exist |
| `RUN_PART_A`, `RUN_PART_D`, `RUN_DATA_SCAN`, `RUN_PART_E` | pretrained models on the Hawaii example, real continuous data, full-file scan, scorecard |

Run the notebook once for each of the five bottlenecks.

### Steps 4 and 5: INSTANCE
Both need Internet on, because they download the INSTANCE sample. Step 4 only inspects; its summary explains the choices made in step 5.

### Step 6: `06_train_instance_models`
Same as step 3, but it reads the INSTANCE mixtures and uses the split stored in the file (`presplit: true` in each `run_info.json`). One bottleneck per session.

### Step 7: `07_variants_and_evaluation`
Attach the STEAD data, the INSTANCE data and every training run. Settings:

| Switch | Meaning | Used for |
|---|---|---|
| `RUN_SELFTEST` | parameter counts, shapes, batch independence | run before any training |
| `RUN_BATCHDEP` | paper `Transformer` score against evaluation batch size | the batch-size finding |
| `RUN_SMOKE`, then `RUN_TRAIN` | 2-epoch check, then full training of one variant | one variant per session |
| `VARIANT` | `attention_PE`, `attention_LSTM`, `hybrid` or `Transformer_fixed` | |
| `TRAIN_ON` | `"STEAD"` or `"INSTANCE"` | `"STEAD"` for all four variants |
| `RUN_EVAL` | scores every run it finds on the STEAD and INSTANCE test splits | |
| `RUN_FIGURES` | builds the figures from `results/` | |
| `EVAL_BATCH` | evaluation batch size | 128 |

Evaluate only after all runs are attached.

### Step 8: `08_real_data_all_models`
Attach the STEAD data, the INSTANCE data and every training run.  Settings:

| Switch | Meaning |
|---|---|
| `RUN_SELFTEST` | loads every model, checks shapes, STA/LTA against ObsPy and exact stitching |
| `RUN_DOWNLOAD` | continuous data, station metadata, catalogue, travel times |
| `RUN_REAL` | separates the real data with every model, STA/LTA sweep, catalogue matching |
| `RUN_SYNTH` | long synthetic streams with known onsets, same pipeline |
| `RUN_FIGURES` | tables and figures |
| `RUN_PAPER_FIGS` | STA/LTA figures in the layout of the paper |
| `RESUME` | skips models that already have results |

## 3. Kaggle datasets used as inputs

| Kaggle dataset | Holds | 
|---|---|
| `DAT_Repo_Data` | the STEAD subset (`merged.csv`, `merged.hdf5`) |
| `wdn-code` | the authors' code with our patches (`Author_Code/` in this submission) | 
| `wdn-training-data` | the STEAD mixtures |
| `instance-mixtures-snr40` | the INSTANCE mixtures |
|  `wdn-runs-all` | the run folders in `results/training_runs/` |