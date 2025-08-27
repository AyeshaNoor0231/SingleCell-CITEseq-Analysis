# Predicting Protein Expression from RNA using CITE-seq Data

## Introduction
The human body is composed of trillions of cells, each performing specialized functions. Understanding the molecular profile of individual cells is a central goal of modern biology. Traditional techniques average signals across thousands or millions of cells, obscuring the heterogeneity that exists between them.  

Recent advances in single-cell technologies now allow researchers to measure molecules at the level of individual cells, providing insights into development, disease progression, and immune responses.  

Among the molecules within a cell, **RNA transcripts** and **proteins** are of particular interest:
- **RNA** reflects the instructions being read from the genome.
- **Proteins** carry out most cellular functions.  

Studying RNA is crucial to understand protein abundance and interpret cell states, but measuring proteins is often technically challenging and costly compared to RNA.  

This project focuses on **CITE-seq** (Cellular Indexing of Transcriptomes and Epitopes by sequencing), which measures both RNA and proteins in the same cell. The goal is to predict protein expression levels from RNA transcript data using machine learning.

---

## Background: CITE-seq Technology
CITE-seq combines single-cell RNA sequencing (scRNA-seq) with antibody-derived protein measurements:

- **scRNA-seq:** Captures RNA molecules from each cell, producing a gene expression matrix (cells × genes).
- **CITE-seq:** Uses antibodies tagged with DNA barcodes to measure cell surface proteins. Sequencing these barcodes reveals protein abundance.

**Output:** A paired dataset where each cell has:
- High-dimensional RNA profile (tens of thousands of genes)
- Lower-dimensional protein profile (dozens to hundreds of proteins)

This dual measurement is valuable because proteins are the true effectors of cellular function.

---

## Problem Statement
**Question:** Can we computationally predict protein expression profiles from RNA data?  

**Task:**
- **Input:** Single-cell RNA expression data (22,050 genes per cell)  
- **Output:** Single-cell protein abundance data (~140 proteins per cell)  

This is a **supervised regression problem** where both inputs and outputs are continuous values.

---

## Data Description
The dataset is from the Kaggle competition: **Open Problems in Single-Cell Analysis**

| File | Shape | Description |
|------|-------|-------------|
| `train_cite_inputs.h5` | ~70,000 × 22,050 | RNA expression values per cell |
| `train_cite_targets.h5` | ~70,000 × 140 | Protein abundances per cell |
| `test_cite_inputs.h5` | 48,663 × 22,050 | Test set RNA data |

**Notes:**
- RNA data is highly sparse (~78% zeros).  
- Protein data has no zeros and is more stable.  

---

## Methodology
Pipeline:  
`Data Exploration → Input Check → Sparsity Analysis → Data Cleanup → Variance Analysis → Feature Selection → Standardization → Train/Validation Split → Model Building`

### Input Data Check
- RNA: 70,988 × 22,050  
- Protein: 70,988 × 140  
- Missing values: 0  

### Sparsity
- RNA: 78% zeros  
- Protein: no zeros  

### Data Cleanup
- Removed 449 genes with zero values across all cells → 21,601 genes remaining  
- Non-zero RNA expression: 3–12, mean ~4.4  

### Variance & Feature Selection
- Gene variance median: 0.83, max > 9  
- Top 3,000 variable genes chosen → ~14% of features, sparsity reduced to 44%, memory <1GB  

### Standardization
- Both RNA and protein values standardized (mean=0, variance=1)  
- Train-validation split: 80–20 (~56k train, ~14k validation)

---

## Model Building
**Ridge Regression** was used to:
- Capture patterns between RNA and proteins  
- Avoid overfitting to spurious correlations  

Other methods (Random Forest, Gradient Boosting) were too slow for this large dataset.

---

## Results

**Overall Metrics**
- Mean Squared Error (MSE): 0.8227  
- Overall R² score: 0.1732 (≈17% variance explained)  
- Average per-protein R²: 0.1737  

**Per-protein R² range:** –0.05 → 0.73  

**Protein Predictability Distribution**
| R² range | Count | Interpretation |
|-----------|-------|----------------|
| >0.5      | 15    | RNA is a strong predictor |
| 0.3–0.5   | 37    | Moderate predictability |
| 0.1–0.3   | 65    | Weak but real signals |
| <0        | 31    | RNA alone cannot predict |

**Top 10 Most Predictable Proteins:** CD41 (0.73), CD71 (0.66), CD88 (0.64), CD48 (0.62)  

**Bottom 10 Least Predictable Proteins:** IgD (–0.05), CD194 (–0.048), CD268 (–0.042), CD16 (–0.041)

---

## Discussion
- Predicting protein levels from RNA is challenging, but feasible for certain proteins.  
- Overall, the model explains ~17% of protein variation.  
- Some proteins (e.g., CD41) are highly predictable, while others (e.g., IgD) are not.  
- Biological insight:  
  - Structural/housekeeping proteins → RNA levels mirror protein abundance.  
  - Immune receptors/signaling proteins → Controlled post-transcriptionally, less predictable from RNA alone.  

**Conclusion:** RNA can serve as a useful signal for certain proteins, but additional regulatory information is necessary for many proteins.

---

## References
- Burkhardt, D.B., Luecken, M.D., Benz, A., Holderrieth, P., Bloom, J., Lance, C., Chow, A., Holbrook, R., et al.  
  *Open Problems in Single-Cell Analysis.* Kaggle, 2022.  
  [https://kaggle.com/competitions/open-problems-multimodal](https://kaggle.com/competitions/open-problems-multimodal)
- Stoeckius, M., Hafemeister, C., Stephenson, W., et al.  
  *Simultaneous epitope and transcriptome measurement in single cells.*  
  Nature Methods, 14, 865–868 (2017). https://doi.org/10.1038/nmeth.4380  

