# semiconductor-metrology
Semiconductor Metrology Data Quality & Process Monitoring

A Python-based project focused on semiconductor metrology data analysis, measurement system evaluation, and process monitoring.

Project Overview

This project simulates wafer Critical Dimension (CD) measurement data and applies statistical methods to evaluate measurement reliability and identify potential sources of variation.

The goal is to develop a practical data analysis workflow for semiconductor manufacturing, covering measurement system analysis (MSA), tool matching, statistical process control (SPC), and anomaly detection.

Project Objectives

Evaluate measurement system repeatability and reproducibility.

Identify operator-related and equipment-related measurement variations.

Assess measurement consistency across different tools.

Monitor process behavior and detect abnormal measurement patterns.

Build a reproducible data analysis workflow using Python.

Project Modules

Module

Description

Status

Data Simulation & EDA

Generate wafer CD measurement data and perform exploratory analysis.

Completed

Measurement System Analysis (MSA)

Crossed ANOVA Gauge R&R, variance components, operator comparison, and residual diagnostics.

Completed

Tool Matching

Evaluate measurement bias and consistency across different tools.

In Progress

Statistical Process Control (SPC)

Apply control charts and monitor process shifts and drift.

Planned

Anomaly Detection

Explore statistical and machine learning methods for abnormal pattern detection.

Planned

Data Visualization

Develop an interactive dashboard for measurement data analysis.

Planned

Technical Stack

Programming: Python

Data Processing: Pandas, NumPy

Statistical Analysis: SciPy, Statsmodels

Data Visualization: Matplotlib

Methods: ANOVA, Gauge R&R, Paired t-test, Bland–Altman analysis, SPC

Repository Structure

semiconductor-metrology/
│
├── content/                 # Simulated measurement datasets
├── scripts/                 # Data generation and analysis scripts
├── notebooks/               # Exploratory analysis
├── reports/
│   ├── figures/             # Analysis plots
│   └── results/             # Statistical analysis outputs
│
└── README.md

Data Disclaimer

All datasets in this repository are simulated for educational and analytical purposes. They do not represent actual semiconductor manufacturing data or proprietary information.

Project Status

Currently developing the measurement system analysis and tool matching modules, with a focus on translating statistical results into actionable engineering insights.
