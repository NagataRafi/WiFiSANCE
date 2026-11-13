# Changelog

## [0.1.0] - 2026-11-12

### Added

- Initial project structure.
- Initial project documentation.
- PRD, roadmap, and agent instructions.

## [Unreleased]

### Added

- WiCount dataset clone and verified archive at `analysis/dataset/WiCount/`.
- Dataset schema, class counts, CSI representation, and limitations audit.
- CSI visualization notebook and 3D comparison output based on original dataset rows.
- Analysis requirements and a project `.venv` reusing available scientific packages, with only missing Jupyter packages installed.
- README and roadmap updated to reflect the external dataset analysis and its scope.
- Deep descriptive CSI analysis of all 15 WiCount recordings, including session/class statistics, quality checks, six figures, and grouped-split leakage guidance; no ML training.
- DSP and feature-engineering notebook covering filter comparison, normalization review, windowing experiments, and feature matrix export on the full WiCount archive.
- Generated artifacts for signal filtering, feature analysis, and windowing under `analysis/outputs` and `analysis/processed` without moving into ML training.
- CSI quality-assurance notebook and report covering feature-definition validation, duplicate timestamp analysis, redundancy review, PCA standardization, and leakage-safe split guidance; no ML training.
- Final WiCount dataset validation notebook and report, including original-row-order timestamp/sequence audit, non-destructive feature reduction recommendations, group-split feasibility, preprocessing leakage review, and generated validation plots; no ML training.
- Limited WiCount ML baseline experiment with 13 recording-level holdout folds for classes 1 and 2, comparing full and frozen compact features across Logistic Regression, Random Forest, and RBF-SVM; OOF metrics and error artifacts generated.
- Added pinned scikit-learn 1.9.1 to analysis requirements for the requested baseline classifiers, plus experiment documentation and roadmap status updates; no ESP32, backend, or mobile implementation.

### Planned

- CSI acquisition proof of concept.
- WiFiSENSE-specific ESP32 data collection and ground-truth labeling workflow.
- ML baseline and evaluation.
- Backend/API and mobile presentation layer.
