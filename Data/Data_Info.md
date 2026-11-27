# Data

The data files are **not included** in this submission because of their size. They are available as Kaggle datasets (links below) and can be rebuilt from the original sources with the notebooks in `Notebooks/`.

All training and evaluation data are **mixtures**: a 60 s, 3-component earthquake waveform plus a 60 s, 3-component noise waveform, both at 10 Hz. Each mixture has two targets: the earthquake alone and the noise alone. The exploration mixtures (section 5) follow the same file format, but each window is 600 samples at the native sampling interval of the survey, and the three channels are three adjacent traces.

---

## 1. Kaggle datasets

| Dataset | Used for | Kaggle link | Built by |
|---|---|---|---|
| STEAD data | STEAD-trained models | https://www.kaggle.com/datasets/prabhanshu22/wdn-training-data | `01_build_stead_subset.ipynb`, `02_prepare_stead_data.ipynb` |
| INSTANCE mixtures | INSTANCE-trained models; cross-dataset testing | https://www.kaggle.com/datasets/prabhanshu22/instance-mixtures-snr40 | `05_build_instance_mixtures.ipynb` |
| Exploration mixtures | VG2D, F3 and STR3D models; cross-dataset testing | https://www.kaggle.com/datasets/prabhanshu22/wdn-exploration-mixtures | `09_build_exploration_mixtures.ipynb` |

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
| Mixtures | 187,000 [100,000 with STEAD noise and 87,000 with POHA noise] |
| Variant | SNR 40, downsampled to 10 Hz |
| Split | 60 / 20 / 20 after mixing: 112,200 train, 37,400 validation, 37,400 test |

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

## 4. Real continuous data (HV.HAT and IU.POHA)

Two Hawaii stations, downloaded from IRIS (FDSN) and resampled to 10 Hz.

| | `IU.POHA` | `HV.HAT` |
|---|---|---|
| Channels | `BH1`, `BH2`, `BHZ` (location `00`, 40 Hz) | `HHE`, `HHN`, `HHZ` |
| Noise for training | 2021-06-01 to 2021-07-31: 87,191 noise windows of 60 s, 87,000 used in the STEAD mixtures (`02_prepare_stead_data.ipynb`) | NA  |
| Detection test | 2021-08-02, 14 days (`08_real_data_all_models.ipynb`) | same period |

Catalogue for the detection test: `catalog_local.20210731-20210901.xml` from the WaveDecompNet repository, 483 events, M 1.5 to 4.0.

---

## 5. Exploration datasets

Three exploration seismic surveys, used to train and test the final LSTM model. Built by `09_build_exploration_mixtures.ipynb` with the recipe of the INSTANCE build. The raw files are downloaded by the notebook (Internet on) and are not stored in this submission.

| | `VG2D` | `F3` | `STR3D` |
|---|---|---|---|
| Category | 2D marine | 3D marine | 3D land |
| Survey | Mobil AVO Viking Graben line 12 | F3 Netherlands (facies-classification benchmark, Zenodo 3755060) | Stratton 3D (Bureau of Economic Geology, UT Austin) |
| File used | `seismic.segy`, raw prestack shot gathers (about 750 MB; 1,001 shots x 120 channels, 1,500 samples, 4 ms) | `seismic_entire_volume.npy` inside `raw.zip` (1.08 GB; 601 inlines x 901 crosslines x 255 samples, float64) | `Stratton3D_32bit.sgy`, processed volume (444 MB; 1,501 samples, 2 ms) |
| Signal | brute stack built in the notebook from the raw gathers (no stack is published) | post-stack volume, up-sampled x3 to reach 600 samples | processed volume |
| Noise | real: the part of the raw traces before the first break (automatic pick); segments shorter than 600 samples are mirror-padded | synthetic: band-limited Gaussian noise, low-frequency swell bursts and spikes | synthetic (no unfiltered volume is published) |
| Sampling interval | 4 ms | 4 ms, set with `--dt-override` (not stored in the file) | 2 ms |
| Spatial block / guard (split) | 100 CDPs / 2 | 20 inlines / 1 | 20 lines / 1 |
| File | `training_datasets_VG2D.hdf5` | `training_datasets_F3.hdf5` | `training_datasets_STR3D.hdf5` |
| Build record | `build_report_VG2D.json` | `build_report_F3.json` | `build_report_STR3D.json` |


---

## 6. Sources and licences

**INSTANCE:** https://www.pi.ingv.it/instance/ (sample file Instance_sample_dataset_v3.tar.bz2), licence CC BY 4.0, dataset DOI 10.13127/instance.
Michelini, A., Cianetti, S., Gaviano, S., Giunchi, C., Jozinović, D., Lauciani, V. (2021). INSTANCE, the Italian seismic dataset for machine learning. *Earth System Science Data*, 13(12), 5509-5544. doi:10.5194/essd-13-5509-2021

**STEAD:** Mousavi, S. M., Sheng, Y., Zhu, W., Beroza, G. C. (2019). STanford EArthquake Dataset (STEAD): A global data set of seismic signals for AI. *IEEE Access*, 7, 179464-179476.

**WaveDecompNet:** https://github.com/yinjiuxun/WaveDecompNet-paper

**Mobil AVO Viking Graben line 12 (`VG2D`):** http://s3.amazonaws.com/open.source.geoscience/open_data/Mobil_Avo_Viking_Graben_Line_12/seismic.segy 

**F3 Netherlands (`F3`):** https://zenodo.org/records/3755060 (file `raw.zip`), benchmark licence MIT; F3 data CC BY-SA 3.0, dGB Earth Sciences / NAM. Because of the share-alike term, the F3 mixtures are shared under the same licence.

**Stratton 3D (`STR3D`):** http://s3.amazonaws.com/open.source.geoscience/open_data/stratton/segy/processed/Stratton3D_32bit.sgy, Bureau of Economic Geology, University of Texas at Austin

**Real continuous data (section 4):** waveforms from the HV and IU networks via FDSN (EarthScope, IRIS); catalogue `catalog_local.20210731-20210901.xml` from the WaveDecompNet repository.