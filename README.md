# VentiMorph-RelNet

VentiMorph-RelNet v2.8.1 is an anatomically guided multimodal 2.5D segmentation framework for MS3SEG brain MRI. The repository contains the proposed-model training notebooks, matched baseline implementations, ablation experiments, five-fold out-of-fold (OOF) evaluation, frozen locked-test evaluation, statistical analysis, reproducibility audit, and publication-figure generation.

## Repository layout

```text
VentiMorph-RelNet/
├── VentiMorph-RelNet_proposed_model/   # Fold 0–4 proposed-model training
├── baselines/                    # FCN-8s, SegNet, U-Net, U-Net++
├── ablations/                    # A1–A4 component ablations
├── oof_validation/               # 80-patient five-fold OOF evaluation
├── locked_test/                  # frozen 20-patient final test
├── final_evaluation/             # statistics, audit, journal/XAI 

## Experimental design

- Development set: **80 patients**
- Locked test set: **20 patients**
- OOF: **5 folds**, each with 64 training and 16 validation patients
- Frozen split SHA-256:



The locked test set was not used for training, model selection, threshold tuning, calibration fitting, or post-processing optimization.

## MRI input and preprocessing

The study uses the released preprocessed/co-registered MS3SEG FLAIR, T1-weighted, and T2-weighted volumes.

For each modality independently:
1. finite nonzero voxels are identified;
2. intensities are clipped to the 0.5th–99.5th percentiles;
3. z-score normalization is applied using the nonzero-voxel mean and standard deviation;
4. nonfinite values are replaced by zero.

The 2.5D input contains three adjacent slices per modality:


FLAIR: z-1, z, z+1
T1:    z-1, z, z+1
T2:    z-1, z, z+1
```

for a total of **9 channels**. Edge slices are replicated at volume boundaries.

## Proposed model

VentiMorph-RelNet includes:
- modality-specific feature stems;
- learned pixel-wise multimodal fusion;
- encoder-decoder segmentation backbone;
- anisotropic context aggregation;
- ventricle-guided anatomical relation refinement;
- stop-gradient ventricle guidance;
- mid-decoder ventricle-only guidance;
- final directional near/mid/far relation guidance;
- bounded zero-initialized relation residuals;
- deep supervision.

Approximate parameter count: **5.97 M**.

## Common training protocol

```text
epochs          60
initial LR      5e-5
optimizer       AdamW
gradient clip   1.0
batch size      4
seed            42
input channels  9
classes         4
```

The notebooks include lesion-aware slice sampling and the training augmentations used in the experiments.

## Baselines

Reimplemented under the matched protocol:
- FCN-8s
- SegNet
- U-Net
- U-Net++

## Ablations

Development/OOF only:
- A1: no relation refinement
- A2: no spatial modality gate
- A3: no deep supervision
- A4: no stop-gradient guidance

Ablations were not evaluated on the locked test set.

## Main results

### 80-patient OOF

| Model | Mean foreground Dice |
|---|---:|
| FCN-8s | 0.5950 |
| SegNet | 0.6640 |
| U-Net | 0.7866 |
| U-Net++ | 0.7971 |
| A1 No Relation | 0.7997 |
| A2 No Spatial Gate | 0.7983 |
| A3 No Deep Supervision | 0.8013 |
| A4 No Stop-Gradient | 0.7982 |
| **VentiMorph v2.8.1** | **0.8063** |

### Locked 20-patient test

| Model | Mean foreground Dice |
|---|---:|
| FCN-8s | 0.6197 |
| SegNet | 0.7049 |
| U-Net | 0.8119 |
| U-Net++ | 0.8208 |
| **VentiMorph v2.8.1** | **0.8317** |

VentiMorph class-wise final-test Dice:
- ventricle: **0.9241**
- Normal-WMH: **0.7535**
- Abnormal-WMH: **0.8174**

## Final inference protocol

For VentiMorph and each baseline final test:

```text
five Fold 0–4 best_model.pt checkpoints
→ equal-weight softmax averaging
→ argmax
→ no threshold tuning
→ no test-set calibration
→ no extra test-set post-processing
```

## Statistical analysis

The final analysis includes:
- paired Wilcoxon signed-rank tests;
- paired bootstrap 95% confidence intervals;
- matched-pairs rank-biserial effect size;
- paired Cohen's dz;
- Holm correction for multiple comparisons.

See `final_evaluation/` and `results/statistics/`.

## Reproducibility

The repository includes code/notebooks and lightweight result summaries. Large artifacts are intentionally excluded from GitHub.

Not included:
- MS3SEG source data;
- `.pt` model checkpoints;
- transfer archives;
- prediction NIfTI volumes;
- large raw output ZIPs.

These should be archived separately (recommended: **Zenodo**) if a complete reproducibility release is desired.

## Dataset

The MS3SEG dataset is not redistributed here. Obtain it from the original dataset source and follow its data-use terms.

## Installation

```bash
pip install -r requirements.txt
```

The experiments were run in Kaggle notebook environments with NVIDIA T4 GPUs for model training/inference. Final statistical and figure-generation notebooks can run on CPU.

## Recommended execution order

```text
1. VentiMorph-RelNet_proposed_model/
2. baselines/
3. ablations/
4. oof_validation/
5. freeze inference protocol
6. locked_test/
7. final_evaluation/
```


## Manuscript

Working title:

**VentiMorph-RelNet: Anatomically guided multimodal 2.5D segmentation of ventricles and normal-versus-abnormal white matter hyperintensities in multiple sclerosis MRI**

