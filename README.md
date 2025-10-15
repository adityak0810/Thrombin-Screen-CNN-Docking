<!-- Repo name: thrombin-screen-cnn-docking -->
<!-- Repo description: End-to-end virtual screening for Thrombin (CHEMBL204): curate ChEMBL data, train a 1D-CNN on Morgan fingerprints, score ligands, prepare receptor/ligands, run AutoDock Vina docking, compute AUC/EF@k metrics, (optionally) rerank with classical ML, and visualize poses. -->

# Thrombin Screening: CNN + Docking (CHEMBL204)

**TL;DR**: A reproducible ligand-screening pipeline for Thrombin. It pulls labeled data from a local ChEMBL SQLite DB, trains a 1D-CNN on Morgan fingerprints, scores all ligands, auto-prepares receptor/ligands, docks with AutoDock Vina, evaluates (AUC, EF@k, correlations), and optionally trains a classical ML re-ranker. Py3Dmol is used for quick pose inspection.

---

## Workflow

1. **Data curation (ChEMBL, SQLite)**
   - Target: `CHEMBL204` (Thrombin).
   - Labels from activity thresholds (nM): actives `< 100`, inactives `> 5000`.
   - Query canonical SMILES; deduplicate; shuffle with fixed seed.

2. **Featurization**
   - Morgan fingerprints: `nBits=2048`, `radius=2` (RDKit).
   - Converts SMILES → bit vectors for modeling.

3. **Modeling (1D-CNN)**
   - Architecture: 3×Conv1d(+BN+ReLU)+MaxPool → AdaptiveMaxPool → Dropout → MLP → logit.
   - Loss: `BCEWithLogitsLoss` with `pos_weight` for class imbalance.
   - Split: stratified Train/Val/Test (`test_size=0.2`, `val=0.1` of train).
   - Optimizer: Adam (`lr=1e-3`), scheduler: ReduceLROnPlateau.
   - Metrics: Accuracy, Precision, Recall, F1, ROC-AUC; confusion matrix; report.
   - Saves: `CNN_thrombin.pt`; test predictions CSV.

4. **Scoring whole set**
   - Runs the trained CNN over all curated ligands to get `cnn_prob ∈ [0,1]`.
   - Exports `ligand_binding_data.csv` with: `name/molecule_chembl_id, smiles, label, standard_value, cnn_prob`.

5. **Screening set selection**
   - Take top-probability actives plus a matched sample of inactives (configurable `TOP_K`, `PROB_THRESH`).
   - Write `runs/vs1/screened_ligands.csv`.

6. **Preparation (OpenBabel + RDKit)**
   - **Receptor**: convert `receptor.pdb` → `receptor.pdbqt` (add H, Gasteiger charges).
   - **Pocket center**: compute `(center_x,y,z)` from `pocket.pdb` atoms.
   - **Grid box**: default `(20,20,20) Å` (tunable).
   - **Ligands**: SMILES → RDKit ETKDG 3D → UFF minimize → SDF → PDBQT (OpenBabel).
   - Logs `prep_status.csv`.

7. **Docking (AutoDock Vina)**
   - Vina command uses receptor `.pdbqt`, ligand `.pdbqt`, computed center/box, `exhaustiveness=16`.
   - Parses best affinity (kcal/mol), saves pose paths.
   - Output: `docking_results.csv` with `name, smiles, label, cnn_prob, dock_affinity_kcalmol, pose_path`.

8. **Evaluation & ranking**
   - **Prospective AUC** of `cnn_prob` on docked subset.
   - **EF@k** (10/25/50): enrichment of actives at top by `cnn_prob`.
   - **Spearman correlation** between `cnn_prob` and affinity (more negative = better).
   - Final ranking: sort by `dock_affinity_kcalmol` (asc) then `cnn_prob` (desc).
   - Write `ranked_results.csv`.
   - Plots: distribution of docking scores; scatter of `-affinity` vs `cnn_prob`.

9. **(Optional) Classical ML reranker**
   - Feature sets: Morgan bits (optionally + physchem descriptors).
   - Models: Ridge / RandomForest / GradientBoosting (with optional PCA).
   - Scaffold-aware CV (GroupKFold by Bemis–Murcko scaffold).
   - Target: docking affinity (kcal/mol) **or** computed pActivity (if available).
   - Saves `reranked_results.csv`; includes correlation plots.

10. **Visualization (Py3Dmol)**
    - Quick viewer for receptor + top ligand pose; style toggles.

---

## Requirements

- **Python** ≥ 3.9  
- **Libraries**: `torch`, `scikit-learn`, `pandas`, `numpy`, `matplotlib`, `rdkit`, `py3Dmol` (optional), `scipy` (optional)  
- **Tools** (in `PATH`):
  - AutoDock **Vina** (`vina`)
  - **OpenBabel** (`obabel`)
- **Data**: local **ChEMBL SQLite** DB (e.g., `chembl_35.db`)

**Install (example):**
```bash
conda install -c conda-forge rdkit openbabel
pip install torch scikit-learn pandas numpy matplotlib scipy py3Dmol
