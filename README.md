# WaveDecompNet

This project reproduces **WaveDecompNet**, a neural network that separates a 3-component seismogram into an earthquake signal and a noise signal, and then extends it. The original code is by the authors of https://github.com/yinjiuxun/WaveDecompNet-paper.

## What was done

1. **Reproduction.** The authors' pipeline was rebuilt on STEAD data and run with their five bottleneck variants (None, Linear, LSTM, attention, Transformer), and compared with their released weights.
2. **New dataset.** The same architecture was trained and tested on the INSTANCE dataset, and the STEAD-trained models were tested on INSTANCE data.
3. **Architecture variants.** Four new bottlenecks (`attention_PE`, `attention_LSTM`, `hybrid`, `Transformer_fixed`) were trained on STEAD and evaluated against the original five on both test sets.
4. **Detection on real and synthetic streams.** STA/LTA event detection was run on raw data, on band-passed data and on the output of every model (the authors' pretrained models and all models trained here). The real data are 14 days of continuous recordings from two Hawaii stations (HV.HAT, IU.POHA), scored against an earthquake catalogue. The synthetic data are long streams built from the STEAD and INSTANCE test splits, with known onsets.

The analysis and conclusions are in the report.

## Folder map

```
.
├── README.md
├── RUN_GUIDE.md
├── requirements.txt
├── Author_Code/              Authors' Scripts
├── Data/
│   └── Data_Info.md          Links to the data
├── Notebooks/                
└── Results/
    ├── Evaluation/
    │   ├── Figures/
    │   └── Results/
    ├── Real Data Evaluation/
    │   ├── Figures/
    │   └── Results/
    └── Training Runs/
        ├── Stead Runs/       5 runs: the paper bottlenecks trained on STEAD
        ├── Instance Runs/    5 runs: the paper bottlenecks trained on INSTANCE
        └── Variant Runs/     4 runs: the new bottlenecks trained on STEAD
```

Each run folder holds the model weights (`.pth`), the training history (`.hdf5` and `.png`), the train/validation split (`.hdf5`) and `run_info.json`. All except the variant runs also hold a `config_*.json`.

## Where to start

- To rerun the work: `RUN_GUIDE.md`.
- To find the data: `Data/Data_Info.md`. The data files are not in this submission; they are on Kaggle.
- To check the results: `Results/Evaluation/results/` for the tables and `Results/Evaluation/figures/` for the figures.
- To check the detection results: `Results/Real Data Evaluation/Results/` for the tables (`table_real_*.csv`, `table_synth.csv`) and `Results/Real Data Evaluation/Figures/` for the figures.

## Notebooks

| Notebook | Purpose |
|---|---|
| `01_build_stead_subset` | builds the STEAD subset from the six archive chunks |
| `02_prepare_stead_data` | builds the STEAD training mixtures with the authors' scripts |
| `03_train_stead_models` | trains and evaluates the five paper bottlenecks on STEAD |
| `04_inspect_instance` | inspects the INSTANCE sample (optional) |
| `05_build_instance_mixtures` | builds the INSTANCE mixtures |
| `06_train_instance_models` | trains the five paper bottlenecks on INSTANCE |
| `07_variants_and_evaluation` | trains the four variants, scores every model, makes the figures |
| `08_real_data_all_models` | STA/LTA detection with every model on real continuous data and on synthetic streams |

## Credits and licences
- WaveDecompNet code: https://github.com/yinjiuxun/WaveDecompNet-paper, commit `afe2536cf29ead8b4d8d5c50ddf85cc1dd90ddd1` [add the repository licence and the paper citation].
- STEAD: Mousavi et al. (2019), *IEEE Access* 7, 179464-179476 [check].
- INSTANCE: Michelini et al. (2021), *Earth System Science Data* 13(12), 5509-5544, doi:10.5194/essd-13-5509-2021; licence CC BY 4.0.
