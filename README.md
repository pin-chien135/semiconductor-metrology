# semiconductor-metrology
## Semiconductor Metrology Data Quality & Process Monitoring
A Python-based project focused on semiconductor metrology data analysis, measurement system evaluation, and process monitoring.

## Project Overview

This project simulates wafer Critical Dimension (CD) measurement data and applies statistical methods to evaluate measurement reliability and identify potential sources of variation.

The goal is to develop a practical data analysis workflow for semiconductor manufacturing, covering measurement system analysis (MSA), tool matching, statistical process control (SPC), and anomaly detection.

### semiconductor-metrology/

├── content/

│   ├── msa_data.csv

│   └── tool_matching_data.csv

├── reports/

│   ├── figures/             # .png

│   ├── results/             # .csv

│   └── msa_report.md

│

├── scripts/   # .py

│   ├── 01_generate_data.py

│   ├── 02_eda.py

│   ├── 02_tool_matching_analysis.py

│   └── 04_msa_validation.py

└── README.md

## Objects
Evaluate measurement system repeatability and reproducibility.

Identify operator-related and equipment-related measurement variations.

Assess measurement consistency across different tools.

Monitor process behavior and detect abnormal measurement patterns.

Build a reproducible data analysis workflow using Python.

## Project Modules

Data Simulation & EDA (Completed)

-Generate wafer CD measurement data and perform exploratory analysis.

Measurement System Analysis (MSA) (Completed)

-Crossed ANOVA Gauge R&R, variance components, operator comparison, and residual diagnostics.

Tool Matching (In Progress)

-Evaluate measurement bias and consistency across different tools.

Statistical Process Control (SPC)

-Apply control charts and monitor process shifts and drift.

Anomaly Detection

-Explore statistical and machine learning methods for abnormal pattern detection.

Data Visualization

-Develop an interactive dashboard for measurement data analysis.


## Technical Stack

Programming: Python

Data Processing: Pandas, NumPy

Statistical Analysis: SciPy, Statsmodels

Data Visualization: Matplotlib

Methods: ANOVA, Gauge R&R, Paired t-test, Bland–Altman analysis, SPC


## Data Disclaimer
All datasets in this repository are simulated for educational and analytical purposes. They do not represent actual semiconductor manufacturing data or proprietary information.

## Project Status
Currently developing the measurement system analysis and tool matching modules, with a focus on translating statistical results into actionable engineering insights.
