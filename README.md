# WaveDecompNet

This project reproduces **WaveDecompNet**, a neural network that separates a 3-component seismogram into an earthquake signal and a noise signal, and then extends it. The original code is by the authors of https://github.com/yinjiuxun/WaveDecompNet-paper.

## What was done

1. **Reproduction.** The authors' pipeline was rebuilt on STEAD data and run with their five bottleneck variants (None, Linear, LSTM, attention, Transformer), and compared with their released weights.
2. **New dataset.** The same architecture was trained and tested on the INSTANCE dataset, and the STEAD-trained models were tested on INSTANCE data.
3. **Architecture variants.** Four new bottlenecks (`attention_PE`, `attention_LSTM`, `hybrid`, `Transformer_fixed`) were trained on STEAD and evaluated against the original five on both test sets.

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
    └── Training Runs/
        ├── Stead Runs/       5 runs: the paper bottlenecks trained on STEAD
        ├── Instance Runs/    5 runs: the paper bottlenecks trained on INSTANCE
        ├── Variant Runs/     4 runs: the new bottlenecks trained on STEAD
        └── Exploration Runs/ 3 runs: the LSTM trained on VG2D, F3 and STR3D
```

## Where to start

- To rerun the work: `RUN_GUIDE.md`.
- To find the data: `Data/Data_Info.md`. The data files are not in this submission; they are on Kaggle.

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
| `09_build_exploration_mixtures` | builds the VG2D, F3 and STR3D mixture files from SEG-Y and numpy data |
| `10_train_exploration_lstm` | trains the final LSTM on each exploration dataset |
| `11_evaluate_exploration` | scores every model on the five test splits: transfer matrix, ranking, paired differences, figures |


## Credits and licences
- WaveDecompNet code: https://github.com/yinjiuxun/WaveDecompNet-paper, commit `afe2536cf29ead8b4d8d5c50ddf85cc1dd90ddd1` [add the repository licence and the paper citation].
- STEAD: Mousavi et al. (2019), *IEEE Access* 7, 179464-179476 [check].
- INSTANCE: Michelini et al. (2021), *Earth System Science Data* 13(12), 5509-5544, doi:10.5194/essd-13-5509-2021; licence CC BY 4.0.
- Mobil AVO Viking Graben line 12 (VG2D): raw shot gathers from the open.source.geoscience data repository [add the source citation and licence].
- F3 Netherlands (F3): full volume from the F3 facies-classification benchmark, Zenodo 3755060 (MIT) [add the benchmark paper citation]. The F3 data are CC BY-SA 3.0 (dGB Earth Sciences / NAM), so the F3 mixtures are shared under the same licence.
- Stratton 3D (STR3D): Bureau of Economic Geology, University of Texas at Austin [add the source citation and licence].
