# VentiMorph-RelNet

Code and experiment notebooks for **VentiMorph-RelNet**, a ventricle-guided 2.5D network for segmenting ventricles, normal white-matter hyperintensities (WMH), and abnormal WMH in multiple sclerosis MRI.

This repository accompanies the manuscript *VentiMorph-RelNet: Ventricle-guided multimodal 2.5D segmentation of normal and abnormal white-matter hyperintensities in multiple sclerosis MRI*. It includes the model implementation, four baseline models, component ablations, and the evaluation notebooks used in the study. Additional notebooks for the revised analyses are collected in `revision_analyses/`.

## Dataset

Experiments were conducted on **MS3SEG**, using co-registered FLAIR, T1-weighted, and T2-weighted MRI. The study includes 100 patients: 80 for model development and 20 held out for final testing. The development set was divided into five patient-level folds, with 64 training patients and 16 validation patients per fold.

Each input consists of three adjacent slices from each MRI modality (nine channels in total). The model predicts four classes: background, ventricles, normal WMH, and abnormal WMH.

The dataset is not included here. Access must be obtained from the original data provider. The notebooks expect the data to be arranged in their configured input directories, so these paths may need to be changed before running them locally or on Kaggle.

## Model

VentiMorph-RelNet uses modality-specific feature extraction followed by learned fusion and an encoder–decoder segmentation network. Ventricle-guided relation refinement introduces anatomical context into the segmentation process. The implementation also uses spatial modality gating, stop-gradient guidance, and deep supervision.

The proposed network has approximately **5.97 million parameters**. Model definitions and training settings are available in `VentiMorph-RelNet_proposed_model/`.

## Repository layout

```text
VentiMorph-RelNet/
├── VentiMorph-RelNet_proposed_model/   Original model training (five folds)
├── Baselines/                         FCN-8s, SegNet, U-Net, U-Net++
├── Ablations/                         A1–A4 component ablations
├── oof_validation/                    Out-of-fold evaluation notebooks
├── locked_test/                       Final 20-patient test evaluation
├── final_evaluation/                  Statistics, figures, and split checks
└── revision_analyses/
    ├── three_seed_validation/         Additional runs with seeds 123 and 2026
    ├── calibration/                   Foreground and classwise ECE
    └── distance_lesion_analysis/      Distance-band, lesion-IoU, and paired analysis
```

All experiments are provided as Jupyter notebooks (`.ipynb`). The notebooks were developed for Kaggle GPU environments and include experiment-specific paths and dependencies.

## Training and evaluation

The original model and baseline experiments use five-fold training with seed 42. The proposed-model notebooks use AdamW, a starting learning rate of `5e-5`, batch size 4, and a maximum of 60 epochs. Checkpoint selection and other training details are specified in the notebooks.

For the main evaluation, out-of-fold predictions were collected across the 80 development patients. The 20-patient test set was kept separate and evaluated after model selection. For the VentiMorph-RelNet and U-Net++ comparison on this test set, predictions from the five selected fold checkpoints were averaged at the softmax-probability level before assigning the final class.

The patient split is recorded by the following SHA-256 checksum:

```text
d5b26261c93305d0554dae92454cee9363c8df5d946c944d47ab1b8a25898459
```

The split-checking notebook is `final_evaluation/05-reproducibility-and-split-audit.ipynb`.

## Results

### Five-fold out-of-fold evaluation

Mean foreground Dice on the 80-patient development set (seed 42):

| Model | Dice |
|---|---:|
| FCN-8s | 0.5950 |
| SegNet | 0.6640 |
| U-Net | 0.7866 |
| U-Net++ | 0.7971 |
| VentiMorph-RelNet | **0.8063** |

### Locked test set

Mean foreground Dice on the 20 held-out patients:

| Model | Dice |
|---|---:|
| FCN-8s | 0.6197 |
| SegNet | 0.7049 |
| U-Net | 0.8119 |
| U-Net++ | 0.8208 |
| VentiMorph-RelNet | **0.8317** |

The classwise Dice scores for VentiMorph-RelNet were 0.9241 for ventricles, 0.7535 for normal WMH, and 0.8174 for abnormal WMH.

### Ablation study

The four ablations examine relation refinement (A1), spatial modality gating (A2), deep supervision (A3), and stop-gradient guidance (A4). All four use the original seed-42, five-fold development-set protocol.

| Configuration | Mean foreground Dice |
|---|---:|
| A1 — without relation refinement | 0.7997 |
| A2 — without spatial modality gate | 0.7983 |
| A3 — without deep supervision | 0.8013 |
| A4 — without stop-gradient guidance | 0.7982 |
| Full VentiMorph-RelNet | **0.8063** |

## Additional analyses

The notebooks in `revision_analyses/` were added during the manuscript revision.

### Training across three seeds

`revision_analyses/three_seed_validation/` contains five-fold training notebooks for VentiMorph-RelNet and U-Net++ using seeds 123 and 2026. Together with the original seed-42 experiments, these provide three-seed validation summaries:

| Model | Mean validation Dice ± SD |
|---|---:|
| VentiMorph-RelNet | **0.8186 ± 0.0011** |
| U-Net++ | 0.8073 ± 0.0015 |

These are averages of the five validation-fold scores for each seed, using the selected best checkpoint in each fold. They are separate from the out-of-fold and locked-test results reported above.

### Calibration

`revision_analyses/calibration/` contains the VentiMorph-RelNet and U-Net++ notebooks for foreground and classwise expected calibration error (ECE). The locked-test comparison uses 20 confidence bins.

| ECE measure | VentiMorph-RelNet | U-Net++ |
|---|---:|---:|
| All voxels | 0.000867 | 0.000870 |
| Ground-truth foreground | 0.056802 | 0.058225 |
| Predicted foreground | 0.092651 | 0.092223 |
| Foreground union | 0.146311 | 0.149322 |

These measures use different voxel subsets, so they should be interpreted separately.

### Ventricle distance and lesion analysis

`revision_analyses/distance_lesion_analysis/` contains the notebook used for the paired patient comparison, analysis by distance from the reference ventricle, and lesion-level matching at different intersection-over-union (IoU) thresholds. The distance bands are 0–3 mm, greater than 3–10 mm, and greater than 10 mm. Lesion-level results are examined at IoU thresholds of 0.10, 0.25, and 0.50.

Some parts of this notebook require the original MRI volumes and saved model predictions; the paired comparison figure can be produced from the relevant saved results when available.

## Running the notebooks

1. Obtain access to MS3SEG and prepare the input files according to the paths in the training notebooks.
2. Run the five proposed-model training notebooks in `VentiMorph-RelNet_proposed_model/`. Baseline and ablation runs are organized in their respective directories.
3. Use `oof_validation/` to evaluate the development-set predictions.
4. Run the split audit in `final_evaluation/` before comparing results.
5. Use `locked_test/` with the selected fold checkpoints for the final test evaluation.
6. Run the notebooks in `revision_analyses/` for the additional validation, calibration, and lesion analyses.

A complete environment file is not currently included. Before running a notebook, check its package imports, data paths, and checkpoint references. The repository also does not redistribute the MRI data or the large trained-model and prediction archives.

## Citation

If you use this code, please cite the associated VentiMorph-RelNet manuscript once its final publication details are available.
