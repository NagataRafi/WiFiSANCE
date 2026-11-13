# WiCount Deep CSI Dataset Analysis

## Scope and method

- Source: original `analysis/dataset/WiCount/WiCount.zip` from Hugging Face repository `RS2002/WiCount`.
- Analysis notebook: `analysis/notebooks/csi_deep_analysis.ipynb`.
- Unit of recording/session: one CSV file, 15 recordings total.
- The ZIP was read in place. No data was extracted, rewritten, synthesized, or used to train a model.
- Every payload was parsed according to the verified WiCount convention: 104 interleaved numeric values, with each adjacent pair representing real and imaginary parts for one of 52 CSI subcarriers.
- Amplitude was calculated for each sample/subcarrier as `sqrt(real**2 + imaginary**2)`.
- Reported standard deviation and variance are sample statistics (`ddof=1`). Class amplitude statistics pool all valid CSI rows and subcarriers; class values are therefore weighted by their number of rows. Recording-level statistics are also reported so session variation remains visible.
- Timestamp intervals were computed between adjacent rows in source order. The source timestamps have no explicit timezone suffix; elapsed durations do not require assigning one.

## Dataset coverage and quality

| Occupancy folder | Recordings | CSI records | Share of recordings | Share of CSI records |
|---:|---:|---:|---:|---:|
| 0 | 1 | 5,218 | 6.67% | 6.64% |
| 1 | 7 | 36,559 | 46.67% | 46.50% |
| 2 | 6 | 31,935 | 40.00% | 40.62% |
| 3 | 1 | 4,906 | 6.67% | 6.24% |
| **Total** | **15** | **78,618** | **100%** | **100%** |

Across 78,618 rows, 4,088,136 per-subcarrier amplitude values were analyzed. Payload parsing found no malformed/non-finite CSI values, wrong component counts, or `len` mismatches. Timestamp parsing and RSSI parsing had no invalid rows; the source `first_word_invalid` flag was never set. No sequence-number discontinuities or backwards timestamp intervals were observed.

All recordings span approximately one minute: observed timestamp durations range from 59.801 to 60.004 seconds, with a median of 59.988 seconds. The median of each recording's positive adjacent timestamp intervals ranges from 11 to 16 ms; the corresponding 95th percentiles range from 17 to 30 ms. There are 8,900 equal-timestamp adjacent intervals among 78,603 total intervals (11.32%). These equal timestamps are not invalid CSI payloads, but they mean timestamps do not uniquely identify every row. The reason for repeated timestamps is not specified by the dataset and should not be guessed.

## Pooled amplitude statistics by class

Each class statistic below pools all valid amplitudes across its recordings, rows, and subcarriers. `N amplitude values` is the number of row/subcarrier observations contributing to the statistic.

| Class | Recordings | CSI records | N amplitude values | Mean | Median | Std. dev. | Variance | Min | Max | Q1 | Q3 | IQR |
|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| 0 | 1 | 5,218 | 271,336 | 14.312 | 13.342 | 5.459 | 29.799 | 2.236 | 30.265 | 10.198 | 17.493 | 7.295 |
| 1 | 7 | 36,559 | 1,901,068 | 14.043 | 13.038 | 5.799 | 33.632 | 0.000 | 36.878 | 9.849 | 17.692 | 7.843 |
| 2 | 6 | 31,935 | 1,660,620 | 13.595 | 12.530 | 5.959 | 35.504 | 0.000 | 36.770 | 9.220 | 17.493 | 8.273 |
| 3 | 1 | 4,906 | 255,112 | 13.549 | 12.806 | 5.797 | 33.603 | 0.000 | 36.346 | 9.220 | 17.493 | 8.273 |

The pooled means decrease across the folder labels in this dataset, but the amplitude distributions overlap substantially and the capture order is confounded with class. This descriptive pattern does not establish a people-count effect.

## Amplitude statistics by recording

The amplitude columns use all valid CSI rows and 52 subcarriers within each CSV.

| Class | Recording (timestamp from filename) | Rows | Duration (s) | Mean | Median | Std. dev. | Variance | Min | Max | IQR |
|---:|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| 0 | 2023-12-21 19:23:53.985 | 5,218 | 59.985 | 14.312 | 13.342 | 5.459 | 29.799 | 2.236 | 30.265 | 7.295 |
| 1 | 2023-12-21 19:27:21.930 | 4,780 | 59.993 | 14.157 | 13.153 | 5.875 | 34.518 | 0.000 | 36.401 | 7.905 |
| 1 | 2023-12-21 19:31:17.706 | 5,189 | 60.004 | 14.176 | 13.153 | 5.595 | 31.305 | 0.000 | 36.056 | 7.213 |
| 1 | 2023-12-21 19:34:42.783 | 5,234 | 59.997 | 14.100 | 13.038 | 5.955 | 35.462 | 0.000 | 36.770 | 8.151 |
| 1 | 2023-12-21 19:37:36.861 | 5,163 | 59.985 | 14.126 | 13.038 | 5.826 | 33.942 | 0.000 | 36.359 | 7.905 |
| 1 | 2023-12-21 19:40:12.110 | 5,421 | 59.980 | 13.955 | 13.000 | 5.726 | 32.784 | 0.000 | 33.242 | 7.615 |
| 1 | 2023-12-21 19:42:28.267 | 5,644 | 59.975 | 13.932 | 12.806 | 5.776 | 33.367 | 0.000 | 34.670 | 8.318 |
| 1 | 2023-12-21 19:45:07.501 | 5,128 | 59.994 | 13.875 | 12.806 | 5.838 | 34.081 | 0.000 | 36.878 | 8.205 |
| 2 | 2023-12-21 19:47:14.669 | 5,376 | 59.988 | 13.463 | 12.728 | 5.847 | 34.185 | 0.000 | 35.609 | 7.868 |
| 2 | 2023-12-21 19:49:37.940 | 5,307 | 59.986 | 13.620 | 12.649 | 6.435 | 41.407 | 0.000 | 36.401 | 8.804 |
| 2 | 2023-12-21 19:54:01.269 | 5,141 | 59.994 | 13.570 | 12.207 | 5.984 | 35.808 | 0.000 | 36.056 | 8.501 |
| 2 | 2023-12-21 19:56:22.903 | 5,467 | 59.984 | 13.660 | 12.530 | 5.805 | 33.692 | 0.000 | 35.171 | 8.258 |
| 2 | 2023-12-21 20:02:00.329 | 5,221 | 59.988 | 13.631 | 12.530 | 5.764 | 33.226 | 0.000 | 35.805 | 8.030 |
| 2 | 2023-12-21 20:04:39.406 | 5,423 | 59.997 | 13.627 | 12.530 | 5.894 | 34.741 | 0.000 | 36.770 | 8.500 |
| 3 | 2023-12-21 20:07:25.709 | 4,906 | 59.801 | 13.549 | 12.806 | 5.797 | 33.603 | 0.000 | 36.346 | 8.273 |

Q1/Q3 and the full per-recording time/quality table are available in the executed notebook.

## Visualizations

All images are generated by the executed notebook from original WiCount rows:

- [Amplitude distributions across all recordings](../analysis/outputs/amplitude_all_recordings.png) — one box per CSV; individual fliers are hidden for legibility.
- [Amplitude distributions by class](../analysis/outputs/amplitude_distribution_by_class.png) — normalized histogram shapes; class sample counts are annotated.
- [Amplitude boxplot by class](../analysis/outputs/amplitude_boxplot_by_class.png) — pooled class distributions on a common scale.
- [Subcarrier correlation heatmap](../analysis/outputs/subcarrier_correlation_heatmap.png) — Pearson correlation across 52 subcarrier amplitude series pooled over all records.
- [Temporal variability by recording](../analysis/outputs/temporal_variability_by_recording.png) — per-session/per-subcarrier temporal standard deviation and recording-level summaries.
- [3D CSI comparison using all sessions](../analysis/outputs/csi_3d_all_sessions.png) — session-equally-weighted amplitude surface per class. Every available recording contributes; class 0 and class 3 necessarily each have one recording. The x-axis is normalized within-session progress, not absolute clock time.

## Interpretation and limitations

- CSI amplitude varies across time and subcarriers within recordings. The pooled class means are 13.549–14.312 while within-class standard deviations are 5.459–5.959. The broad, overlapping distributions do not support a simple class-separating threshold from these summaries.
- The median temporal standard deviation across subcarriers for each session-level summary is 0.975 for class 0, 2.446 for class 1, 3.163 for class 2, and 3.579 for class 3. Classes 0 and 3 each have only one session, so these values cannot represent within-class session variability or establish a causal count relationship.
- The 15 recordings were captured on the same date within about 45 minutes, and labels appear in chronological blocks: class 0, then class 1, class 2, and class 3. Room/device drift, setup changes, activity, or time can therefore be confounded with the folder label. Available metadata does not resolve these factors.
- The dataset README describes an ESP32-S3 meeting-room collection, but WiFiSENSE hardware, firmware, placement, and acquisition settings remain unverified. Results from this external dataset do not establish compatibility or performance for WiFiSENSE.
- The dataset has no independent-session replication for classes 0 and 3 and is imbalanced both by recording count (maximum/minimum 7:1) and CSI rows (7.45:1).
- Repeated timestamps occur in 8,900 adjacent intervals; the reason is not documented. Do not equate row count with unique timestamp count or assume a fixed sampling rate from the timestamp column.
- A pooled correlation heatmap can include between-session and between-class effects; correlated subcarriers are not independent experimental replicates.
- Results are descriptive only. No causal inference or Machine Learning training was performed.

## Data leakage risk and split strategy

Randomly splitting rows or windows would place highly related observations from the same roughly 60-second recording in both training and evaluation. This risks leakage and overstates generalization to unseen sessions. Keep every window derived from a CSV/session in exactly one split, and carry the source recording/session ID through preprocessing.

With one recording each for classes 0 and 3, a recording-grouped split cannot both train on and independently evaluate those labels. Grouped cross-validation on the current set is not a substitute for collecting additional independent sessions. Before model work, collect repeated sessions for every class, randomize/interleave class order, and record the acquisition day, room, device placement, activity, and configuration. Then use recording/session-grouped validation and reserve a later day or room for external testing where feasible.

## Recommended next experiments

1. Reproduce the acquisition with multiple independent sessions for each occupancy label, especially 0 and 3.
2. Randomize or interleave occupancy conditions in time to reduce class/time confounding.
3. Keep hardware, TX/RX placement, channel, firmware, and room state fixed for a controlled comparison, while logging them; design separate tests for placement/activity variation.
4. Investigate timestamp duplication and its relationship to `seq`, local timestamp, and logger behavior without imputing a sampling rate.
5. Confirm dataset reuse/licensing terms and verify WiCount assumptions against the actual WiFiSENSE ESP32 CSI output before adapting a parser.
6. Only after sufficient independent sessions exist, define a recording-grouped evaluation plan. Model training is outside the scope of this analysis.
