# WiCount Dataset Audit

## Scope and method

- Source: [Hugging Face RS2002/WiCount](https://huggingface.co/datasets/RS2002/WiCount)
- Local clone: `analysis/dataset/WiCount/`
- Repository revision cloned: `d2b848ab26bf971a0915a9b8ab64a052d3088988`
- Original data archive: `analysis/dataset/WiCount/WiCount.zip`
- Inspection method: repository card and metadata review, archive member listing, and a full row-by-row read of all CSV files from the ZIP. The archive was not modified or extracted.

The clone contains an LFS-managed `WiCount.zip`. Git LFS reported the object as downloaded; the file is 9,389,189 bytes and has a ZIP signature, rather than being an LFS pointer. Its archive lists 15 CSV files under `data/0/` through `data/3/`.

## Facts found

### Directory and file layout

The archive has one folder per occupancy label: `0`, `1`, `2`, and `3`. It contains 15 non-empty CSV recordings:

| Occupancy label from folder | CSV files | Rows (CSI records) |
|---:|---:|---:|
| 0 | 1 | 5,218 |
| 1 | 7 | 36,559 |
| 2 | 6 | 31,935 |
| 3 | 1 | 4,906 |
| **Total** | **15** | **78,618** |

The row counts were read from the CSV records, not inferred from filenames or the archive README. The classes are imbalanced; classes 0 and 3 each have one recording, while classes 1 and 2 have more.

### CSV schema

All inspected CSVs use the same 28-column schema:

`type`, `seq`, `timestamp`, `taget_seq`, `taget`, `mac`, `rssi`, `rate`, `sig_mode`, `mcs`, `cwb`, `smoothing`, `not_sounding`, `aggregation`, `stbc`, `fec_coding`, `sgi`, `noise_floor`, `ampdu_cnt`, `channel_primary`, `channel_secondary`, `local_timestamp`, `ant`, `sig_len`, `rx_state`, `len`, `first_word_invalid`, `data`.

The spellings `taget_seq` and `taget` are present in the source and are intentionally recorded as-is.

### CSI representation and parsing checks

- The `data` CSV field is a JSON-compatible list of 104 numeric values.
- The dataset README describes each adjacent pair as the real and imaginary components of one complex CSI value: `real + imaginary*j`.
- The 104 values therefore represent 52 complex subcarrier values, consistent with the dataset README.
- The amplitude representation is derived as `sqrt(real**2 + imaginary**2)` for each adjacent pair. Amplitude is not a separate source field.
- All 78,618 rows were checked: each `data` list parsed, contained 104 finite numeric values, and matched the row's `len` value of 104.
- No empty CSV cells or invalid `timestamp`, `rssi`, or `local_timestamp` values were found in the row-by-row check.

### Time, RSSI, and experiment metadata

- `timestamp` is present and parses for every row. The dataset README describes it as UTC+8. Sample values use a date-time string without an explicit timezone suffix.
- `local_timestamp` is present and integer-valued in every row; its units/epoch are not specified in the CSV schema.
- `rssi` is present and integer-valued in every row; observed values range from -90 to -23. The dataset card does not define a unit in the field description.
- Other per-row device fields include packet/PHY metadata, channel fields, `mac`, and a `type` value. Every inspected row has `type=CSI_DATA`.
- The dataset README describes collection using ESP32-S3 devices in an indoor meeting-room setting; it reports 2.4 GHz, 20 MHz bandwidth, 802.11n/OFDM, 52 subcarriers, one antenna per device, and an approximate 100 Hz sampling rate. These are statements from the dataset documentation, not measurements independently confirmed by this repository audit.
- The README says occupancy labels are represented by the parent folders and describes the recordings as 60-second samples. The row counts above are directly inspected; no independent guarantee of the README's experimental protocol is inferred from them.

### License and provenance

- The cloned dataset repository does not contain a `LICENSE` file.
- The Hugging Face dataset metadata/card does not expose a license value.
- The dataset README identifies the associated CSI-BERT2 paper and its authors. No license terms are inferred from the paper citation.
- Usage or redistribution rights therefore remain unverified and should be clarified before redistribution or adoption beyond this analysis.

## Assumptions not verified

- The folder names are used as occupancy labels because the repository README and task description state that they encode the number of people. The original ground-truth collection procedure was not independently audited.
- The data list is parsed according to the README's interleaved real/imaginary convention. A successful parse and consistent length do not independently validate the physical calibration or subcarrier ordering.
- The notebook's 3D view uses one deterministically selected recording per occupancy class for a comparable visual demonstration. It is not a complete statistical comparison of all sessions.

## Limitations

- The recordings are class-imbalanced, and classes 0 and 3 are represented by only one CSV each.
- The data is from the dataset's documented ESP32-S3 setup and indoor meeting-room experiment. It does not establish performance in other rooms, placements, devices, firmware, or populations.
- The CSV does not provide explicit units for `local_timestamp` or `rssi`, and each `timestamp` string lacks a timezone suffix even though the README states UTC+8.
- The dataset's label folders do not by themselves document session-level activity, subject identity, or all collection conditions.
- Visual differences in CSI amplitude are descriptive only; occupancy cannot be attributed as the cause without controlling for environmental and device factors.
- License information is unavailable in the cloned repository metadata.

## Relevance to WiFiSENSE and ESP32 compatibility

WiCount is relevant as an external people-counting CSI reference and provides real CSI samples with labels 0–3, timestamps, RSSI, and device metadata. Its documented ESP32-S3 source may help inform the CSI parsing and visualization workflow.

WiFiSENSE's actual ESP32 variants, firmware/toolchain, radio configuration, and CSI output format remain `TBD`. Compatibility with WiCount or direct reuse of its processing assumptions is therefore **not verified**. This external dataset does not replace the required WiFiSENSE two-device TX/RX proof of concept or a project-specific dataset.
