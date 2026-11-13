# CSI Signal Processing and Feature Engineering

## Scope

This document summarizes the DSP and feature-engineering work completed on the original WiCount archive under `analysis/dataset/WiCount/WiCount.zip`. The project did not add ML training, firmware changes, or backend/mobile deployment work. The goal was to validate preprocessing choices, compare filtering methods, test normalization options, and store windowed feature matrices with traceable metadata.

## Dataset and processing basis

- Archive used: `analysis/dataset/WiCount/WiCount.zip`
- Source data: 15 CSV recordings exported from the verified WiCount archive
- CSI interpretation: each CSV row contains a `data` payload interpreted as interleaved real/imaginary values across 52 subcarriers; amplitude is computed as `sqrt(real^2 + imag^2)`
- Validation: invalid payload rows were counted and discarded without imputation

## Filtering experiments

The notebook compared the following methods on the original amplitude matrix:

1. Raw amplitude baseline
2. Moving average: window 3, 5, 7, 11
3. Savitzky-Golay: window 5, 7, 11, polynomial order 2
4. Median filter: kernel 3, 5, 7
5. Exponential moving average: alpha 0.1, 0.3, 0.5

Metrics stored in `analysis/processed/features/filter_comparison.csv` include mean, standard deviation, variance, minimum, maximum, range, temporal standard deviation, mean absolute difference vs raw, and RMS difference vs raw.

### Quantitative comparison (from generated artifacts)

The full comparison shows the raw signal is the reference baseline. Among the smoothing methods, the lowest mean absolute error relative to raw was observed for the median filter with kernel 3, followed by Savitzky-Golay 5 and moving average 3. The summary ordering is:

- median_filter_3: MAE 0.4066, RMS difference 0.7469
- savitzky_golay_5: MAE 0.4865, RMS difference 0.6422
- median_filter_5: MAE 0.5520, RMS difference 0.8963
- savitzky_golay_7: MAE 0.5741, RMS difference 0.7568
- moving_average_3: MAE 0.5747, RMS difference 0.7625

These numbers show that a visually smoother trace is not automatically better for preserving temporal structure. Strong smoothing reduces high-frequency variation but may also remove informative temporal changes, which matters for occupancy-sensitive CSI patterns. The trade-off is therefore not only noise reduction but also signal fidelity and temporal sensitivity.

## Normalization experiments

Normalization was evaluated in descriptive mode only; the notebook explicitly keeps the scope separate from model training. Two methods were compared:

- Z-score normalization
- Min-max normalization

The summary is stored at `analysis/processed/features/normalization_summary.csv`. This experiment documents how each method alters subcarrier distributions and window-level scale, without claiming training-ready parameters because future modeling should compute normalization statistics only from training data.

## Windowing experiment

The following windows were evaluated with 50% overlap (`stride = window_size / 2`):

- 25 samples
- 50 samples
- 100 samples
- 128 samples (kept as comparison to the earlier prototype)

The processed summaries are stored at:

- `analysis/processed/windows/window_summary_25.csv`
- `analysis/processed/windows/window_summary_50.csv`
- `analysis/processed/windows/window_summary_100.csv`
- `analysis/processed/windows/window_summary_128.csv`
- `analysis/processed/windows/window_summary_all_sizes.csv`

Total number of windows generated from the 15 recordings:

- 25-sample windows: 6529
- 50-sample windows: 3122
- 100-sample windows: 1550
- 128-sample windows: 1206

The corresponding recorded counts by occupancy are:

- occupancy 0: 433 + 207 + 103 + 80 = 823 windows
- occupancy 1: 3037 + 1452 + 720 + 561 = 5770 windows
- occupancy 2: 2652 + 1268 + 630 + 490 = 5040 windows
- occupancy 3: 407 + 195 + 97 + 75 = 774 windows

This confirms class imbalance remains significant and that a session-aware split is required before any modeling stage to avoid leakage.

## Feature extraction

For each window, the notebook computed the following data for each subcarrier:

- mean, median, standard deviation, variance, minimum, maximum, range, Q1, Q3, IQR
- skewness, kurtosis
- mean absolute deviation
- temporal standard deviation
- RMS
- energy
- zero-crossing rate
- first-difference mean and standard deviation

Cross-subcarrier features include:

- mean amplitude across subcarriers
- standard deviation across subcarriers
- variance across subcarriers
- median pairwise subcarrier correlation

The feature matrices were written to:

- `analysis/processed/features/feature_matrix_25.csv`
- `analysis/processed/features/feature_matrix_50.csv`
- `analysis/processed/features/feature_matrix_100.csv`
- `analysis/processed/features/feature_matrix_128.csv`

The quality summary is stored in `analysis/processed/features/feature_quality_summary.csv`.

## Visualization outputs

The notebook saved the following figures in the organized output folders:

- `analysis/outputs/signal_filtering/filter_methods_comparison.png`
- `analysis/outputs/feature_analysis/feature_correlation_heatmap.png`
- `analysis/outputs/feature_analysis/feature_distribution_by_class.png`
- `analysis/outputs/feature_analysis/normalization_summary.png`
- `analysis/outputs/feature_analysis/pca_exploration.png`
- `analysis/outputs/windowing/window_counts.png`
- `analysis/outputs/windowing/window_recording_variation.png`

## Key findings and limitations

- The parallel analysis confirms the WiCount archive is usable for exploratory preprocessing and feature engineering.
- Filter choice affects temporal fidelity: methods that smooth too aggressively can bias the amplitude dynamics even when plots look cleaner.
- The occupancy classes are imbalanced across recordings; the observed class distribution is not uniformly balanced across sessions.
- Repeated timestamps and session-level grouping remain data-quality constraints. Random row slicing would leak near-duplicate information from the same recording into train and validation sets.
- The pipeline stops at preprocessing and feature engineering; all findings are descriptive and should not be interpreted as proof of final occupancy estimation performance.

## Recommended next stage

The best follow-up before any ML work is to define a session-aware train/validation/test split on the raw recordings and then apply the same preprocessing pipeline to the training set alone. The current work is a strong preprocessing baseline for that next step, but it is not a model-training result.
