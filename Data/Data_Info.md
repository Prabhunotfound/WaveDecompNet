# Data

The data files are **not included** in this submission because of their size. They are available as Kaggle datasets (links below) and can be rebuilt from the original sources with the notebooks in `notebooks/`.

All training and evaluation data are **mixtures**: a 60 s, 3-component earthquake waveform plus a 60 s, 3-component noise waveform, both at 10 Hz. Each mixture has two targets: the earthquake alone and the noise alone.

---

## 1. Kaggle datasets

| Dataset | Used for | Kaggle link | Built by |
|---|---|---|---|
| STEAD data | STEAD-trained models | https://www.kaggle.com/datasets/prabhanshu22/wdn-training-data | `01_build_stead_subset.ipynb`, `02_prepare_stead_data.ipynb` |
| INSTANCE mixtures | INSTANCE-trained models; cross-dataset testing | https://www.kaggle.com/datasets/prabhanshu22/instance-mixtures-snr40 | `05_build_instance_mixtures.ipynb` |

**Access:** [public, or shared with which Kaggle account]

**Contents of the STEAD Kaggle dataset:** [subset (`merged.csv`, `merged.hdf5`), mixture file, or both]

The exact paths used for each run are saved in that run's `run_info.json`.

---

## 2. STEAD

**Source.** The six STEAD archive chunks, downloaded from Google Drive by `01_build_stead_subset.ipynb` (file IDs in its `DRIVE_IDS` cell):

| Chunk | Link |
|---|---|
| 1 | https://drive.google.com/uc?id=1sgLd3eVIrTMcuWtOQRN1fGmiNQ7vc227 |
| 2 | https://drive.google.com/uc?id=1ONNNtiOwFodAAeV-ZWp_-gtWhRcYgU_E |
| 3 | https://drive.google.com/uc?id=15bt3WS-mQJmkVY4VOb5LYk45xQBYtkoM |
| 4 | https://drive.google.com/uc?id=12Rw-WI9T21g0hrqtF1KiETYzV4rz177k |
| 5 | https://drive.google.com/uc?id=1rR6bbqu1yE8N_EVIb3kH3wzPPVLFerFX |
| 6 | https://drive.google.com/uc?id=1QfONZf8TBuQlkvKjwJgVMgZub4RDUkH9 |

Project page: https://github.com/smousavi05/STEAD

**Step 1 : Subset (01_build_stead_subset.ipynb)** - It keeps 100,000 noise traces (chunk 1) and the earthquake_local traces with mean snr_db >= 40 (chunks 2 to 6).

| File | Content |
|---|---|
| `merged.csv` | metadata of all six chunks, 1,265,657 rows x 36 columns |
| `merged.hdf5` | 235,464 traces, each (6000, 3) float32 |

**Step 2, Mixtures (02_prepare_stead_data.ipynb)**

| Item | Value |
|---|---|
| Mixtures | 187,000 [100,000 with STEAD noise and 87,000 with POHA noise; confirm] |
| Variant | SNR 40, downsampled to 10 Hz |
| Split | 60 / 20 / 20 after mixing: 112,200 train, 37,400 validation, 37,400 test |
| File | [file name] |

---

## 3. INSTANCE

| Item | Value |
|---|---|
| Source | INSTANCE sample dataset v3: 10,000 events from 300 earthquakes, 1,000 noise traces, raw counts |
| Build | `05_build_instance_mixtures.ipynb` |
| Earthquake pool | mean E/N/Z SNR >= 40 dB, P pick present, 60 s window inside the trace |
| Window | starts 10 s before the P pick |
| Noise | two non-overlapping 60 s windows per 120 s trace |
| Excluded | `source_id` 1116011 (candidate overlap with STEAD) |
| Mixtures | 30,000 |
| Split | 60 / 20 / 20 by earthquake and by noise trace before mixing: 18,000 train, 6,000 validation, 6,000 test |
| File | `training_datasets_INSTANCE_snr40.hdf5` (about 0.9 GB) |
| Build record | `instance_build_report.json` |

| Key | Meaning | Shape / type |
|---|---|---|
| `X_train` | mixture, all splits | (N, 600, 3), float64 |
| `Y_train` | earthquake only | (N, 600, 3), float64 |
| `split_id` | 0 = train, 1 = validation, 2 = test | (N,) |
| `source_id` | earthquake id | (N,) |
| `noise_name` | noise trace name | (N,) |
| `time` | time axis | |

---

## 4. Sources and licences

**INSTANCE:** https://www.pi.ingv.it/instance/ (sample file Instance_sample_dataset_v3.tar.bz2), licence CC BY 4.0, dataset DOI 10.13127/instance.
Michelini, A., Cianetti, S., Gaviano, S., Giunchi, C., Jozinović, D., Lauciani, V. (2021). INSTANCE, the Italian seismic dataset for machine learning. *Earth System Science Data*, 13(12), 5509-5544. doi:10.5194/essd-13-5509-2021

**STEAD:** Mousavi, S. M., Sheng, Y., Zhu, W., Beroza, G. C. (2019). STanford EArthquake Dataset (STEAD): A global data set of seismic signals for AI. *IEEE Access*, 7, 179464-179476.

**WaveDecompNet:** https://github.com/yinjiuxun/WaveDecompNet-paper

**Real continuous data (section 5):** waveforms from the HV and IU networks via FDSN (EarthScope, IRIS); catalogue `catalog_local.20210731-20210901.xml` from the WaveDecompNet repository.

---

## 5. Real continuous data 

Used only for the detection test. Nothing here is a mixture and nothing is used for training. The data are downloaded by the notebook and are not stored in this submission.

| Item | Value |
|---|---|
| Stations | HV.HAT , IU.POHA |
| Period | 2021-08-02, 14 days |
| Source | FDSN clients EarthScope and IRIS |
| Catalogue | `catalog_local.20210731-20210901.xml` from the authors' repository, events within 5 degrees of the station |

**Synthetic streams.** Long streams with known onsets, built by the same notebook from the STEAD and INSTANCE test splits. They use no new data. The streams and their event tables are in `Results/Real Data Evaluation/Results/synth/`.
