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
| 9 | `09_build_exploration_mixtures` | Kaggle | downloads the exploration SEG-Y and numpy files and builds the VG2D, F3 and STR3D mixtures |
| 10 | `10_train_exploration_lstm` | Kaggle | trains the final LSTM on each exploration dataset |
| 11 | `11_evaluate_exploration` | Kaggle | scores every model on the STEAD, INSTANCE and three exploration test splits |

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
| `SEED`, `EPOCHS` | random seed and number of epochs | 99, 100 |
| `RUN_PART_C` | evaluation on the STEAD test split, with baselines | `True` once, after all runs exist |
| `RUN_PART_A`, `RUN_PART_D`, `RUN_DATA_SCAN`, `RUN_PART_E` | pretrained models on the Hawaii example, real continuous data, full-file scan, scorecard | |

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

### Step 9: `09_build_exploration_mixtures`

| Switch | Meaning | Value used |
|---|---|---|
| `RUN_INSPECT` | header table and a quick-look image per file | `True`; check the header bytes in this output before building |
| `RUN_BUILD` | builds and checks one mixture file per dataset | `True` |
| `DATASETS_TO_BUILD` | `"VG2D"`, `"F3"`, `"STR3D"` | all three |
| `N_MIXTURES` | mixtures per dataset | 30,000 |


### Step 10: `10_train_exploration_lstm`
Attach `wdn-exploration-mixtures`. The notebook writes the same step3_training_1D_presplit.py and config.py as the INSTANCE run. Settings:

| Switch | Meaning | Value used |
|---|---|---|
| `RUN_SMOKE` | 2-epoch run on 5,000 rows, projects the full time | `True` before the full run |
| `RUN_FULL` | full training, one dataset after the other | `True` |
| `RUN_QUICK_EVAL` | each new model on its own test split, with training curves | `True` |
| `TRAIN_DATASETS` | `"VG2D"`, `"F3"`, `"STR3D"` | all three |
| `BOTTLENECK`, `SEED`, `EPOCHS` | the final model, seed, epochs | `LSTM`, 99, 100 |
| `SESSION_LIMIT_H`, `SAFETY_FACTOR` | time guard: a dataset is not started if it will not fit in the session | 12.0, 0.9 |
| `FORCE_FULL_RUN` | train without a smoke projection | `False` |

### Step 11: `11_evaluate_exploration`
Attach `wdn-exploration-mixtures`, `wdn-runs-exploration`, `wdn-runs-all`, `wdn-training-data` and `instance-mixtures-snr40`.  Settings:

| Switch | Meaning | Value used |
|---|---|---|
| `RUN_EVAL` | scores every model on every test split | `True` |
| `RUN_TABLES` | transfer matrix, ranking, paired differences, EV against SNR, check against step 7 | `True` |
| `RUN_FIGURES` | report figures | `True` |
| `TEST_SETS` | `"STEAD"`, `"INSTANCE"`, `"VG2D"`, `"F3"`, `"STR3D"` | all five |
| `EVAL_BATCH` | evaluation batch size | 128 |
| `REUSE_CACHE` | reuses the results cached per model and split | `True` |
| `MAX_TEST_ROWS` | limit for a quick trial | `None` (full test splits) |

## 3. Kaggle datasets used as inputs

All the datasets listed below are listed in my kaggle dataset link : https://www.kaggle.com/prabhanshu22/datasets  
| Kaggle dataset | Holds | 
|---|---|
| `DAT_Repo_Data` | the STEAD subset (`merged.csv`, `merged.hdf5`) |
| `wdn-code` | the authors' code with our patches (`Author_Code/` in this submission) | 
| `wdn-training-data` | the STEAD mixtures |
| `instance-mixtures-snr40` | the INSTANCE mixtures |
|  `wdn-runs-all` | the run folders in `Results/Training Runs/` (Stead, Instance and Variant runs) |
| `wdn-exploration-mixtures` | the VG2D, F3 and STR3D mixtures |
| `wdn-runs-exploration` | the three exploration run folders |

## 4. Running from the terminal

Each notebook can be run from a terminal with [papermill](https://papermill.readthedocs.io). It runs the notebook from top to bottom, prints each cell's output live, and saves an executed copy, so the original notebook is left unchanged.

The switches are not passed on the command line. Open the notebook, set the switches in its configuration cell as given in **section 2** for that step, save it, and then run the command. The configuration cell is the first code cell after the imports (cell 2 in `01_build_stead_subset`).

Steps 2 to 11 read their inputs from `/kaggle/input` (see **section 3**). Run them on Kaggle, from the console or from a code cell with `!` in front of each line. On a machine without `/kaggle/input`, the notebooks write to the working folder given with `--cwd`.

### 4.1 One-time setup

```bash
git clone https://github.com/Prabhunotfound/WaveDecompNet.git
cd WaveDecompNet
python -m pip install -r requirements.txt papermill ipykernel
mkdir -p work runs logs
```

All the commands below are run from the `WaveDecompNet` folder.

### 4.2 Step 1: STEAD subset (local PC)

Set `WORK` and `OUT` as in **Step 1**, then:

```bash
papermill Notebooks/01_build_stead_subset.ipynb runs/01_build_stead_subset.ipynb --cwd work --log-output
```

The run resumes from `progress.json` if it is stopped, so the same command can be run again.

### 4.3 Step 2: STEAD mixtures (Kaggle, Internet on)

Attach the inputs listed in **Step 2**. There are no switches.

```bash
papermill Notebooks/02_prepare_stead_data.ipynb runs/02_prepare_stead_data.ipynb --log-output
```

### 4.4 Step 3: train the five paper bottlenecks on STEAD (Kaggle, GPU)

Set the switches as in **Step 3**, with `BOTTLENECK` set to one bottleneck. Run once per bottleneck, changing `BOTTLENECK` each time:

```bash
B=LSTM   # the value of BOTTLENECK in the notebook: None, Linear, LSTM, attention or Transformer
papermill Notebooks/03_train_stead_models.ipynb runs/03_train_stead_${B}.ipynb --cwd work --log-output
```

When all five runs exist, set `RUN_PART_C = True` (and the training switches to `False`) and run it once more:

```bash
papermill Notebooks/03_train_stead_models.ipynb runs/03_evaluate_stead.ipynb --cwd work --log-output
```

### 4.5 Steps 4 and 5: INSTANCE (Kaggle, Internet on)

No switches to change (`N_MIXTURES = 30000` in step 5):

```bash
papermill Notebooks/04_inspect_instance.ipynb runs/04_inspect_instance.ipynb --cwd work --log-output
papermill Notebooks/05_build_instance_mixtures.ipynb runs/05_build_instance_mixtures.ipynb --cwd work --log-output
```

### 4.6 Step 6: train the five paper bottlenecks on INSTANCE (Kaggle, GPU)

Set the switches as in **Step 6** (`RUN_E3_SMOKE`, `RUN_E3_FULL`, `E3_BOTTLENECK`). Run once per bottleneck:

```bash
B=LSTM   # the value of E3_BOTTLENECK in the notebook
papermill Notebooks/06_train_instance_models.ipynb runs/06_train_instance_${B}.ipynb --cwd work --log-output
```

### 4.7 Step 7: variants and evaluation (Kaggle, GPU)

Set the switches as in **Step 7**. First the self-test, then one session per variant (`RUN_SMOKE`, `RUN_TRAIN`, `VARIANT`), and finally `RUN_EVAL` and `RUN_FIGURES` once all runs are attached:

```bash
V=hybrid   # the value of VARIANT in the notebook: attention_PE, attention_LSTM, hybrid or Transformer_fixed
papermill Notebooks/07_variants_and_evaluation.ipynb runs/07_train_${V}.ipynb --cwd work --log-output
```

```bash
papermill Notebooks/07_variants_and_evaluation.ipynb runs/07_evaluate.ipynb --cwd work --log-output
```

### 4.8 Step 8: real continuous data (Kaggle, Internet on for `RUN_DOWNLOAD`)

Set the switches as in **Step 8**. With `RESUME = True`, the same command can be run again after a stop:

```bash
papermill Notebooks/08_real_data_all_models.ipynb runs/08_real_data_all_models.ipynb --cwd work --log-output
```

### 4.9 Step 9: exploration mixtures (Kaggle, Internet on)

Set the switches as in **Step 9**. Check the header table from `RUN_INSPECT` before building:

```bash
papermill Notebooks/09_build_exploration_mixtures.ipynb runs/09_build_exploration_mixtures.ipynb --cwd work --log-output
```

### 4.10 Step 10: train the LSTM on the exploration data (Kaggle, GPU)

Set the switches as in **Step 10**. Finished datasets are skipped, so the same command continues in a new session:

```bash
papermill Notebooks/10_train_exploration_lstm.ipynb runs/10_train_exploration_lstm.ipynb --cwd work --log-output
```

### 4.11 Step 11: evaluation on all test splits (Kaggle, GPU)

Set the switches as in **Step 11**:

```bash
papermill Notebooks/11_evaluate_exploration.ipynb runs/11_evaluate_exploration.ipynb --cwd work --log-output
```

### 4.12 Long runs

To keep a run going after the terminal is closed, start it in the background and follow its log. For example, for step 3:

```bash
B=LSTM
nohup papermill Notebooks/03_train_stead_models.ipynb runs/03_train_stead_${B}.ipynb --cwd work --log-output > logs/03_train_stead_${B}.log 2>&1 &
tail -f logs/03_train_stead_${B}.log
```