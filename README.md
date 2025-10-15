<!-- Repo name: thrombin-screen-cnn-docking -->
<!-- Repo description: End-to-end virtual screening for Thrombin (CHEMBL204): curate ChEMBL data, train a 1D-CNN on Morgan fingerprints, score ligands, prepare receptor/ligands, run AutoDock Vina, compute AUC/EF@k, optionally re-rank with classical ML, and visualize 3D poses. -->

# Thrombin Screening: CNN + Docking (CHEMBL204)

[![Python](https://img.shields.io/badge/python-3.9%2B-blue.svg)](#requirements)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](#license)
[![RDKit](https://img.shields.io/badge/RDKit-enabled-lightgrey)](https://www.rdkit.org/)
[![AutoDock Vina](https://img.shields.io/badge/AutoDock-Vina-informational)](http://vina.scripps.edu/)

A clean, reproducible pipeline for **ligand screening on Thrombin** (target **CHEMBL204**):
- Curate & label ligands from **ChEMBL**
- Train a **1D-CNN** on **Morgan fingerprints**
- Score ligands and select a **screening set**
- Prepare receptor & ligands; **dock with AutoDock Vina**
- Evaluate (**AUC**, **EF@k**, rank correlations)
- **Visualize 3D** receptor–ligand poses

---

## Table of Contents
- [Features](#features)
- [Data & Resources](#data--resources)
- [Requirements](#requirements)
- [Quickstart](#quickstart)
- [3D Visualization](#3d-visualization)
- [Results & Reporting](#results--reporting)
- [Repository Structure](#repository-structure)
- [Configuration](#configuration)
- [Reproducibility](#reproducibility)
- [License](#license)
- [Acknowledgements](#acknowledgements)

---

## Data & Resources
- **Primary dataset**: **ChEMBL** (curated locally)  
  - ChEMBL: https://www.ebi.ac.uk/chembl/  
  - Thrombin target page (**CHEMBL204**): https://www.ebi.ac.uk/chembl/target_report_card/CHEMBL204/
- **Protein structures** (examples): **RCSB PDB**  
  - PDB: https://www.rcsb.org/  (search “thrombin”; e.g., `1HAP`, `1PPB`)
- **Protein sequence/metadata**: **UniProt (F2)**  
  - UniProt Thrombin (P00734): https://www.uniprot.org/uniprotkb/P00734/entry
