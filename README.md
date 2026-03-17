# Predicting BACE-1 Inhibitor Activity from Molecular Structure

This project builds a molecular machine learning pipeline to predict whether a small molecule is **active or inactive against BACE-1** using the MoleculeNet BACE dataset. I used **Morgan fingerprints** as the primary representation and compared a simple **Logistic Regression** baseline against a stronger **Random Forest** model. The main scientific question was not just whether the models could achieve a high score on a standard random split, but whether that performance would hold under a more realistic **Murcko scaffold split**. The key result is that both models performed well on a random split, but both dropped under scaffold-based evaluation, showing that part of the apparent benchmark success likely reflects scaffold familiarity rather than full chemical generalization.

---

## Biological Problem and Motivation

BACE-1 (beta-secretase 1) is a biologically important target in small-molecule drug discovery. Predicting inhibitor activity from molecular structure is a classic cheminformatics problem and a useful benchmark for machine learning on molecules.

However, BACE-1 is also a **well-trodden benchmark**, which makes it easy to produce results that look strong on paper without demonstrating robust generalization. In molecular datasets, structurally related compounds often appear in both the training and test sets under a random split. That can inflate performance because the model may recognize familiar chemotypes rather than learn broader rules that transfer to new chemistry.

For that reason, the central motivation of this project was to compare:

- **Random-split performance**, which is often optimistic
- **Scaffold-split performance**, which is a stricter test of chemical generalization

The gap between those two evaluations is itself an important finding.

---

## Data Source(s) and What Was Used

### Primary dataset
- **Dataset:** MoleculeNet BACE dataset
- **Link:** https://deepchemdata.s3-us-west-1.amazonaws.com/datasets/bace.csv
- **File used in this repository:** `bace.csv`

### What was used
From the dataset, I used:

- **SMILES strings** from the `mol` column
- **Binary activity labels** from the `Class` column
- RDKit-generated molecule objects derived from the SMILES strings

### Basic dataset summary
- **Total molecules:** 1513
- **Inactive (Class = 0):** 822
- **Active (Class = 1):** 691
- **Missing values in attached file:** none detected
- **Invalid SMILES parses:** 0 in this file
- **Exact duplicate SMILES:** 0 in this file

This makes the dataset relatively clean and well-suited for a CHEM 169 final project.

---

## Computational Approach and Workflow Overview

The analysis followed a molecular ML workflow centered on fingerprint-based classification.

### 1. Data loading and inspection
- Loaded `bace.csv`
- Identified the SMILES and binary class columns
- Checked class balance, missing values, duplicate SMILES, and SMILES parsing success

### 2. Molecular representation
- Converted SMILES strings to RDKit molecule objects
- Generated **Morgan fingerprints**
  - radius = 2
  - fingerprint length = 1024 bits
- Also computed a small optional descriptor set:
  - molecular weight
  - logP
  - TPSA
  - H-bond donor count
  - H-bond acceptor count

### 3. Baseline modeling
- Trained **Logistic Regression** on Morgan fingerprints
- Used a **stratified random train/test split**
- Evaluated with:
  - ROC-AUC
  - PR-AUC
  - F1 score
  - confusion matrix

### 4. Stronger comparison model
- Trained **Random Forest** on the same fingerprint features and same random split
- Compared results directly against Logistic Regression

### 5. Generalization test: scaffold split
- Generated **Murcko scaffolds** using RDKit
- Split the dataset so that molecules sharing a scaffold did not appear in both train and test
- Retrained both models on the scaffold-based split
- Re-evaluated using the same metrics

### 6. Interpretation
The main interpretation focused on whether the models were truly learning transferable structure–activity patterns or benefiting from scaffold overlap under easier split conditions.

---

## Route Design and What Was Completed

This project was designed as a custom climb route centered on the idea that **evaluation strategy is the crux**, not just model choice.

### Route focus
The route was built around these stages:

1. Setup and dataset inspection  
2. Molecular feature generation  
3. Baseline model on a random split  
4. Stronger comparison model on the same split  
5. Scaffold-based split as the central generalization test  
6. Reflection on memorization vs generalization, metric choice, and benchmark limitations  

### What was completed
Completed components include:

- Dataset loading and exploration
- SMILES parsing with RDKit
- Sample molecule visualization
- Class balance analysis
- Morgan fingerprint generation
- Optional descriptor computation
- Logistic Regression baseline
- Random Forest comparison model
- Random-split evaluation
- Murcko scaffold generation
- Scaffold-based split
- Scaffold-split evaluation
- Comparison table across split strategies
- Reflection and interpretation of benchmark realism

---

## Results Summary

### Main finding
The most important result of this project is that both models achieved strong performance on a **random split**, but both showed a measurable drop in **ROC-AUC** under a **Murcko scaffold split**. This suggests that some of the apparent success on the benchmark comes from scaffold-related overlap between training and test molecules rather than full generalization to novel chemistry.

### Model performance summary

| Model | Split Type | ROC-AUC | PR-AUC | F1 Score |
|---|---|---:|---:|---:|
| Logistic Regression | Random | 0.873 | 0.842 | 0.755 |
| Random Forest | Random | 0.882 | 0.860 | 0.775 |
| Logistic Regression | Scaffold | 0.828 | 0.871 | 0.800 |
| Random Forest | Scaffold | 0.829 | 0.875 | 0.814 |

### Explicit ROC-AUC drop
- **Logistic Regression:** 0.873 → 0.828 (**drop = 0.045**)
- **Random Forest:** 0.882 → 0.829 (**drop = 0.054**)

This ROC-AUC drop is the clearest evidence that random-split performance was optimistic.

### Important note on PR-AUC and F1
PR-AUC and F1 did not decrease under the scaffold split in this particular analysis. This does **not** mean scaffold generalization is easier. In this split, the scaffold-based test set had a higher fraction of active compounds than the random-split test set, which can make PR-AUC and F1 look better. For comparing split strategies here, **ROC-AUC is the most reliable headline metric**.

---

## Key Figures / Tables

The repository includes figures and tables summarizing model performance. Key outputs include:

- ROC curves comparing Logistic Regression and Random Forest on the random split
- PR curves for both models
- Confusion matrices
- ROC-AUC bar chart comparing random vs scaffold split
- Final model comparison table

### Figure links
- [ROC curve figure](figures/roc_curves_random_vs_scaffold.png)
- [PR curve figure](figures/pr_curves_random_vs_scaffold.png)
- [Random split comparison chart](figures/model_comparison_random_split.png)
- [Scaffold split comparison chart](figures/model_comparison_scaffold_split.png)
- [Confusion matrix examples](figures/)

### Results notebook
- [Main analysis notebook](notebooks/bace1_project.ipynb)

If your filenames differ, update the links above to match your actual repository structure.

---

## Interpretation

The random-split results initially suggest that both models perform quite well, with ROC-AUC values around 0.87–0.88. On paper, that looks like a strong benchmark outcome.

But the scaffold-based evaluation tells a more careful story. Once structurally related compounds are separated by Murcko scaffold, both models lose ROC-AUC, and the advantage of Random Forest over Logistic Regression almost disappears. That means some of the gain seen under the random split likely came from exposure to related chemotypes during training rather than a genuine ability to generalize to new scaffold space.

In that sense, the key conclusion is not simply “Random Forest is best.” The stronger conclusion is:

> **Random-split performance overestimates generalization, and scaffold-aware evaluation provides a more realistic picture of model behavior for molecular discovery tasks.**

---

## Limitations

This project has several important limitations:

1. **Binary labels compress biological information**  
   Activity is reduced to active/inactive, which loses information about potency magnitude and assay nuance.

2. **Benchmark size is still limited**  
   Although the dataset is clean and useful, 1513 molecules represent only a tiny portion of relevant chemical space.

3. **Potential label noise / assay variability**  
   Biological activity measurements can vary across experiments, which affects model training and evaluation.

4. **Scaffold split is stricter, but still not the real world**  
   It is more realistic than a random split, but it is still an offline benchmark rather than a prospective screen on newly synthesized compounds.

5. **Mild class imbalance still affects metric interpretation**  
   The full dataset is reasonably balanced, but split-dependent class proportions can still change how PR-AUC and F1 should be interpreted.

---

## Next Steps

There are several natural extensions to this project:

- Add descriptor-only and descriptor+fingerprint comparisons
- Perform more systematic error analysis on false positives and false negatives
- Test additional scaffold-splitting strategies or repeated scaffold-based resampling
- Try another nonlinear model such as XGBoost or a simple neural network
- Move beyond binary classification toward potency regression, if suitable labels are available
- Evaluate prospective or external generalization on a different BACE-like dataset

A particularly important next step would be to test whether the conclusions are stable across **multiple scaffold-based splits**, rather than relying on only one partition.

---

## Reproducibility Instructions

### Environment
This analysis was run in Python using standard scientific Python libraries and RDKit.

### Recommended dependencies
- pandas
- numpy
- scikit-learn
- matplotlib
- rdkit

### Install dependencies
Example:

```bash
pip install pandas numpy scikit-learn matplotlib rdkit
