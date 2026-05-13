# TP53 Mutation Prediction from Gene-Expression Profiles

This repository contains a machine-learning and bioinformatics project for predicting **TP53 mutation status from RNA gene-expression profiles**.

The assignment goal was to use gene expression measurements as features and TP53 mutation annotations as labels. The project addresses both required tasks:

1. **Task 1:** predict TP53 mutant versus non-mutant / wild type.
2. **Task 2:** predict TP53 mutation type. In this project, mutation annotations are grouped into biologically meaningful mutation-type / consequence classes rather than modelling every rare raw mutation mechanism separately.

The analysis uses both:

- **CCLE / DepMap cell-line data**
- **TCGA Pan-Cancer human tumour data**

The main scientific question is whether transcriptomic profiles contain predictive signal for TP53 mutation status, while accounting for leakage risks and cancer lineage / cancer-type confounding.

---

## Repository contents

| File | Purpose |
|---|---|
| `ccle_eda.ipynb` | CCLE exploratory data analysis, sample matching, TP53 label construction, RNA filtering, train/validation/test split creation, and processed-output generation. |
| `ccle_modelling.ipynb` | CCLE modelling notebook for binary TP53 mutation prediction, metadata baselines, RNA models, joint models, interpretation, sensitivity analyses, and CCLE Task 2 mutation-type classification. |
| `DATA.md` | Dataset source notes for CCLE / DepMap, TCGA / UCSC Xena, and the TP53 target-gene prior list. |
| `requirements.txt` | Python dependencies needed to run the notebooks. |
| `README.md` | Project overview and running instructions. |

The final submitted notebook was produced by combining the project components into one executed Jupyter Notebook so that the full workflow could be reviewed in a single file with code and outputs.

---

## Data sources

The raw datasets are not committed to this repository because they are large. They should be downloaded separately or attached through Kaggle / local input datasets.

### CCLE / DepMap

Download from the DepMap portal:

<https://depmap.org/portal/data_page/?tab=currentRelease>

Required files:

| File | Description |
|---|---|
| `Model.csv` | Cell-line metadata, including model IDs and cancer lineage information. |
| `OmicsSomaticMutations.csv` | Somatic mutation calls. TP53 mutation labels are built from rows where `HugoSymbol == "TP53"`. |
| `OmicsExpressionProteinCodingGenesTPMLogp1.csv` | RNA expression matrix, with protein-coding genes as expression features. |

### TCGA / UCSC Xena

Download from UCSC Xena TCGA Pan-Cancer resources:

<https://xenabrowser.net/datapages/>

Required files:

| File | Description |
|---|---|
| `tcga_RSEM_gene_tpm.gz` | TCGA RNA expression matrix. |
| `mc3.v0.2.8.PUBLIC.xena.gz` | MC3 somatic mutation file. TP53 labels are built from rows where `gene == "TP53"`. |
| `Survival_SupplementalTable_S1_20171025_xena_sp` | Curated phenotype table used for cancer-type metadata. |

### TP53 biological prior knowledge

The project also uses a TP53 target-gene list from the TP53 database / Fischer 2017 target-gene resource as prior biological knowledge for interpretation and selected model components.

See `DATA.md` for more detailed data-source notes.

---

## Kaggle data setup used for this project

For the submitted run, the large datasets were attached as Kaggle input datasets rather than uploaded directly to GitHub.

CCLE Kaggle dataset:

<https://kaggle.com/datasets/0caa0d93ad3c5539a0e438c7d5912de8f74db23c35005e980565a063040797a1>

TCGA Kaggle dataset:

<https://kaggle.com/datasets/1afc69a244bb09cbd7cb88dfc472a2d173433b6cc7f1d56d93d7da12bab0fdb9>

The notebooks include file-discovery logic for common Kaggle, Colab, local, and sandbox paths. If running locally, place the files under a `data/` directory or update the path variables at the top of the notebooks.

Expected local layout:

```text
data/
  Model.csv
  OmicsSomaticMutations.csv
  OmicsExpressionProteinCodingGenesTPMLogp1.csv
  mc3.v0.2.8.PUBLIC.xena.gz
  Survival_SupplementalTable_S1_20171025_xena_sp
  tcga_RSEM_gene_tpm.gz
  tp53_target_genes_fischer2017.csv
```

The CCLE EDA notebook writes processed intermediate files to:

```text
processed/
```

The CCLE modelling notebook expects these processed files to be available.

---

## Environment setup

Install the required Python packages with:

```bash
pip install -r requirements.txt
```

Main dependencies:

- `numpy`
- `pandas`
- `scipy`
- `scikit-learn`
- `statsmodels`
- `matplotlib`
- `seaborn`
- `pyarrow`
- `jupyter`
- `nbconvert`

Optional GPU packages are listed as comments in `requirements.txt`. They are only needed for optional GPU-accelerated TCGA modelling sections in a compatible CUDA environment.

The project was run successfully on Kaggle using a 2xT4 GPU environment. The CCLE EDA notebooks can run locally on CPU. The larger TCGA modelling sections are computationally heavier and are more practical on Kaggle, Colab, or another high-memory/GPU environment.

---

## How to run

### Option 1: run the final submitted notebook

If you are reviewing the submitted project, open the final executed notebook supplied with the submission package. It contains the combined CCLE and TCGA analysis, including code and outputs.

To rerun it:

1. Install dependencies with `pip install -r requirements.txt`.
2. Place the CCLE and TCGA files in the expected `data/` location, or attach the Kaggle datasets.
3. Update path variables if necessary.
4. Run all cells from top to bottom.
5. Export to HTML if needed:

```bash
jupyter nbconvert --to html submission.ipynb
```

### Option 2: run the modular CCLE notebooks

Run the notebooks in this order:

1. `ccle_eda.ipynb`
   - loads raw CCLE expression, mutation, and metadata files;
   - constructs TP53 mutation labels;
   - filters low-information RNA features;
   - creates train/validation/test membership;
   - saves processed artefacts under `processed/`.

2. `ccle_modelling.ipynb`
   - loads the processed CCLE artefacts;
   - trains binary TP53 mutation-status models;
   - compares RNA-only, metadata-only, and RNA plus metadata models;
   - performs robustness checks, calibration, and interpretation;
   - implements CCLE Task 2 mutation-type / consequence classification.

---

## Prediction tasks

### Task 1: TP53 mutant versus wild type

Task 1 is a binary classification problem:

```text
RNA expression profile -> TP53 mutant or TP53 wild type
```

The main binary label is based on non-synonymous / protein-altering TP53 mutation annotations. Synonymous and ambiguous variants are treated cautiously because they may not produce the same biological effect as protein-altering TP53 mutations.

### Task 2: TP53 mutation type

Task 2 is implemented as a multiclass mutation-type / consequence classification problem.

Rather than modelling every rare raw mutation mechanism separately, related mutation annotations are grouped into broader biologically interpretable classes with enough sample support for supervised learning.

For CCLE, the main classes are:

```text
wild_type
missense_or_inframe
lof_or_splice
```

For TCGA, the corresponding classes are:

```text
WT
missense
truncating_or_splice
```

This grouping reflects the distinction between:

- wild-type TP53;
- missense or in-frame alterations that preserve the reading frame but alter sequence or local protein structure;
- nonsense, frameshift, splice-site, or truncating events that are more likely to disrupt TP53 protein function.

This is a coarse mutation-type / consequence task, not a claim that every possible DNA-level mutation mechanism is modelled separately.

---

## Method summary

The workflow is designed to avoid common leakage and confounding problems in transcriptomic prediction tasks.

Key steps:

1. Match RNA expression samples to mutation annotations and metadata.
2. Construct TP53 labels from mutation files, not from RNA expression.
3. Exclude mutation annotation columns from model features.
4. Filter low-information RNA features using training data where appropriate.
5. Use leakage-safe pipelines for scaling, supervised feature selection, and model fitting.
6. Compare RNA-only models with metadata-only baselines.
7. Evaluate cancer lineage / cancer-type confounding using metadata baselines, grouped analyses, and within-cancer checks.
8. Use appropriate metrics for imbalanced binary and multiclass classification.

Model families include:

- dummy baselines;
- regularized logistic regression;
- linear SVM;
- random forest;
- k-nearest neighbours;
- PCA-based logistic regression;
- soft-voting ensembles;
- metadata-only and RNA plus metadata models.

---

## Main results from the submitted notebook

These results are from the executed final notebook submitted for assessment.

### CCLE Task 1: binary TP53 mutation status

Primary RNA-only elastic-net logistic regression on the held-out test set:

| Metric | Value |
|---|---:|
| AUROC | 0.930 |
| AUPRC | 0.951 |
| Balanced accuracy | 0.850 |
| F1 | 0.865 |

A lineage-only metadata model also performs above baseline, showing that lineage is a real confounder and must be considered. The RNA model still substantially outperforms the metadata-only baseline on the main random holdout.

### CCLE Task 2: mutation-type / consequence classification

Best CCLE Task 2 model: RNA multinomial logistic regression.

| Metric | Value |
|---|---:|
| Accuracy | 0.785 |
| Balanced accuracy | 0.778 |
| Macro-F1 | 0.775 |
| Weighted F1 | 0.787 |
| One-vs-rest macro AUROC | 0.898 |
| One-vs-rest macro AUPRC | 0.806 |

Per-class F1:

| Class | F1 |
|---|---:|
| `wild_type` | 0.825 |
| `missense_or_inframe` | 0.792 |
| `lof_or_splice` | 0.709 |

### TCGA Task 1: binary TP53 mutation status

RNA-only elastic-net logistic regression on the TCGA holdout test set:

| Metric | Value |
|---|---:|
| AUROC | 0.906 |
| AUPRC | 0.781 |
| Balanced accuracy | 0.846 |
| F1 | 0.773 |

The cancer-type-only baseline also performs above dummy baseline, confirming that cancer type is a major confounder. Grouped and within-cancer analyses are used to interpret this cautiously.

### TCGA Task 2: mutation-type / consequence classification

Best TCGA Task 2 model: RNA plus cancer-type multinomial logistic regression.

| Metric | Value |
|---|---:|
| Accuracy | 0.721 |
| Balanced accuracy | 0.582 |
| Macro-F1 | 0.563 |
| Weighted F1 | 0.739 |
| One-vs-rest macro AUROC | 0.838 |
| One-vs-rest macro AUPRC | 0.559 |

Per-class F1:

| Class | F1 |
|---|---:|
| `WT` | 0.880 |
| `missense` | 0.465 |
| `truncating_or_splice` | 0.342 |

Task 2 is harder than Task 1 because mutation-type classes are smaller, more imbalanced, and may have overlapping downstream expression consequences.

---

## Interpretation

The results support the conclusion that RNA expression profiles contain predictive information about TP53 mutation status. However, the signal should not be interpreted as purely causal TP53 biology.

Important interpretation points:

- TP53 is a transcriptional regulator, so mutation status can plausibly affect downstream expression programs.
- Cancer lineage and cancer type strongly influence RNA expression and TP53 mutation prevalence.
- Metadata-only models are included to measure this confounding risk.
- Feature importance and selected genes should be treated as predictive associations, not direct mechanistic proof.
- TP53 target-gene information is used to support interpretation, but the model results are not claimed to prove causality.

---

## Limitations

1. **Task 2 uses grouped mutation classes.**
   The mutation-type task groups rare annotations into broader consequence classes. It does not separately model every raw mutation mechanism such as each insertion, deletion, frameshift, or substitution subtype.

2. **Lineage and cancer-type confounding remain important.**
   Stratification and grouped evaluation reduce and quantify this issue, but they do not remove all confounding.

3. **CCLE and TCGA differ biologically and technically.**
   CCLE cell lines and TCGA primary tumours are not interchangeable. Tumour purity, microenvironment, batch effects, and cohort composition may affect TCGA results.

4. **Predictive association is not causation.**
   A gene can be predictive because it is downstream of TP53, correlated with lineage, associated with proliferation, or affected by dataset structure.

---

## Reproducibility notes

- Random seeds are fixed where applicable.
- Model fitting, scaling, and supervised feature selection are performed after splitting and inside modelling workflows where appropriate.
- The final notebook was executed end-to-end before submission.
- The submitted report should include both the executed `.ipynb` and exported `.html` version.

---

## Project conclusion

Gene-expression profiles can predict TP53 mutation status with meaningful performance, especially for binary mutant versus wild-type classification. Mutation-type prediction is feasible when mutation annotations are grouped into biologically meaningful consequence classes, but it is more difficult and should be interpreted cautiously.

The main scientific conclusion is that TP53 mutation status leaves a detectable transcriptomic signal, but this signal is mixed with cancer lineage, tumour type, and dataset-specific structure. Therefore, careful leakage control, appropriate metrics, metadata baselines, and conservative biological interpretation are essential.
