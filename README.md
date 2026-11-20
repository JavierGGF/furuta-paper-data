# Short-Horizon Prediction of a Belt-Driven Furuta Pendulum

Data and frozen-model companion for *Short-Horizon Prediction of a Belt-Driven Furuta Pendulum: Model Complexity and Input Protocols*, Javier G. Fontanet.

## Contents
- `data/R1` through `data/R4`: unchanged recorded telemetry CSV files.
- `records.json`: record aliases and acquisition identifiers.
- `data/assessment_windows.csv`: exact origins and targets for all four nominal horizons.
- `models/primary`: historical OLS8, matched OLS4-A/OLS8-A and frozen Ridge4-A.
- `models/exploratory`: saved linear coefficients, neural weights and normalizations.
- `results`: numeric tables, all-horizon exploratory results and paired per-window errors used for the cumulative distributions.
- `documentation`: signal definitions, assessment rules and model conventions.
- `SHA256SUMS`: checksums of supplied files.

This is a data and model package, not executable software. Reimplement the equations using the conventions in the manuscript and documentation. The physical apparatus is not required to evaluate saved models on these records. Reproducing physical acquisition requires the apparatus and is outside this package. Firmware, controller source, unpublished development history and training implementations are not included. The saved models support reevaluation, not reproduction of their fitting from this package alone.

## Experimental scope
The primary matched models were fitted on R1 and assessed on R2-R4. Exploratory models were fitted on R1+R2 and assessed on R3/R4. All records were inspected during development. The comparisons are retrospective and conditional on the stated command protocols; they do not establish physical parameter identification, selection-independent generalization or improved control. Windows are dependent observations within physical records.
