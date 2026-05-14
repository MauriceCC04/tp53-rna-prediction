# TP53 Mutation Prediction from Gene Expression Profiles

This repository contains a machine-learning and bioinformatics project for predicting **TP53 mutation status from RNA gene-expression profiles**.

The assignment objective was:

> Use gene expression profiles to predict TP53 mutation status.

The project addresses both required tasks:

1. **Task 1:** predict TP53 mutant versus non-mutant / wild type.
2. **Task 2:** predict TP53 mutation type. In this project, mutation annotations are grouped into biologically meaningful TP53 mutation-type / consequence classes rather than modelling every rare raw mutation mechanism separately.

The analysis uses:

- **CCLE / DepMap cell-line data**
- **TCGA Pan-Cancer human tumour data**

A major focus of the project is methodological correctness: labels are derived from mutation annotations, features are RNA expression measurements, and the modelling workflow explicitly checks leakage, class imbalance, and cancer lineage / cancer-type confounding.

---

## Repository contents

```text
.
├── README.md
├── DATA.md
├── requirements.txt
├── submission.ipynb
└── notebooks/
    ├── ccle_eda.ipynb
    ├── ccle_modelling.ipynb
    ├── tcga_eda.ipynb
    └── tcga_modelling.ipynb
```

| Path | Purpose |
|---|---|
| `submission.ipynb` | Final combined submission notebook. This is the notebook intended for grading, with the full analysis combined into one executed report. |
| `notebooks/ccle_eda.ipynb` | CCLE exploratory analysis and dataset preparation. Loads raw CCLE/DepMap files, constructs TP53 labels, performs sample matching, filters RNA features, creates train/validation/test membership, and writes processed artefacts. |
| `notebooks/ccle_modelling.ipynb` | CCLE modelling notebook. Loads processed CCLE artefacts and evaluates RNA-only, metadata-only, and RNA + metadata models for Task 1 and Task 2. |
| `notebooks/tcga_modelling.ipynb` | TCGA Pan-Cancer modelling notebook. Builds the TCGA tumour-only cohort, constructs TP53 labels from mutation annotations, loads RNA expression, evaluates RNA-only and RNA + cancer-type models, and performs cancer-type-aware analyses. |
| `notebooks/tcga_eda.ipynb` | TCGA EDA notebook file. The current TCGA modelling workflow is primarily implemented in `notebooks/tcga_modelling.ipynb`. |
| `DATA.md` | Data-source notes for CCLE/DepMap, TCGA/UCSC Xena, and the TP53 target-gene prior list. |
| `requirements.txt` | Core Python dependencies. Optional RAPIDS/cuML GPU packages are listed as comments. |
| `.gitignore` | Excludes local environments, raw data under `data/`, and processed outputs under `processed/`. |

The raw datasets and processed intermediate files are intentionally not committed to GitHub because they are large.

---

## Data setup

### Kaggle datasets used for the modelling notebooks

For the submitted modelling runs, I created Kaggle datasets so that the large CCLE and TCGA files did not need to be uploaded directly to GitHub.

Attach both datasets when running the modelling notebooks on Kaggle:

**CCLE Kaggle dataset**

<https://kaggle.com/datasets/0caa0d93ad3c5539a0e438c7d5912de8f74db23c35005e980565a063040797a1>

This dataset is used by the CCLE modelling notebook and should contain the processed CCLE files generated from `notebooks/ccle_eda.ipynb`, including:

```text
processed/
  ccle_sample_metadata.parquet
  ccle_split_membership.parquet
  ccle_X_rna_filtered_all.parquet
  ccle_X_meta.parquet
  ccle_modeling_schema.json
  ccle_sure_outliers.txt
  ccle_possible_outliers.txt
```

The CCLE modelling notebook searches for a `processed/` directory under common Kaggle, Colab, and local paths, then copies the processed files into a writable working directory before modelling.

**TCGA Kaggle dataset**

<https://kaggle.com/datasets/1afc69a244bb09cbd7cb88dfc472a2d173433b6cc7f1d56d93d7da12bab0fdb9>

This dataset is used by the TCGA modelling notebook and should contain the raw TCGA/Xena inputs:

```text
mc3.v0.2.8.PUBLIC.xena
Survival_SupplementalTable_S1_20171025_xena_sp
tcga_RSEM_gene_tpm
tp53_target_genes_fischer2017.csv
```

The notebook can also search for `.gz` versions or similarly named files where applicable.

### Original public data sources

The data can also be downloaded manually from the original sources.

#### CCLE / DepMap

Source: <https://depmap.org/portal/data_page/?tab=currentRelease>

Required files:

| File | Description |
|---|---|
| `Model.csv` | Cell-line metadata, including model ID and cancer lineage / disease fields. |
| `OmicsSomaticMutations.csv` | Somatic mutation calls. TP53 mutation labels are constructed from rows where `HugoSymbol == "TP53"`. |
| `OmicsExpressionProteinCodingGenesTPMLogp1.csv` | Protein-coding RNA expression matrix, with values in log2(TPM + 1). |

#### TCGA / UCSC Xena

Source: <https://xenabrowser.net/datapages/>

Required files:

| File | Description |
|---|---|
| `mc3.v0.2.8.PUBLIC.xena` | MC3 somatic mutation annotations. TP53 labels are constructed from rows where `gene == "TP53"`. |
| `Survival_SupplementalTable_S1_20171025_xena_sp` | Curated TCGA phenotype table, used for tumour type / cancer-type metadata. |
| `tcga_RSEM_gene_tpm` | TCGA RNA expression matrix. |

#### TP53 prior knowledge

The project uses a TP53 target-gene prior list:

```text
tp53_target_genes_fischer2017.csv
```

This list is based on the TP53 target-gene resource described in `DATA.md` and is used only for interpretation and selected model components, not as a mutation label source.

---

## Local directory layout

For local execution, place raw data in a `data/` directory at the repository root:

```text
data/
  Model.csv
  OmicsSomaticMutations.csv
  OmicsExpressionProteinCodingGenesTPMLogp1.csv
  mc3.v0.2.8.PUBLIC.xena
  Survival_SupplementalTable_S1_20171025_xena_sp
  tcga_RSEM_gene_tpm
  tp53_target_genes_fischer2017.csv
```

The CCLE EDA notebook writes intermediate files to:

```text
processed/
```

Both `data/` and `processed/` are ignored by Git because they contain large local or generated files.

---

## Environment setup

Install the core Python dependencies:

```bash
pip install -r requirements.txt
```

Core dependencies include:

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

Optional GPU packages are listed as comments in `requirements.txt`:

```text
cupy-cuda12x
cuml-cu12
```

These are only needed for optional GPU acceleration in the TCGA model-search section.

---

## Recommended runtime

The project was designed so that:

- `notebooks/ccle_eda.ipynb` can run locally on CPU if the raw CCLE files are available.
- `notebooks/ccle_modelling.ipynb` can run locally or on Kaggle, but using the Kaggle CCLE dataset is the easiest path because it already contains the processed CCLE artefacts.
- `notebooks/tcga_modelling.ipynb` is computationally heavier and is best run on **Kaggle with the 2xT4 GPU accelerator** and the TCGA Kaggle dataset attached.

The TCGA notebook can use RAPIDS/cuML for repeated elastic-net logistic-regression searches when available. Final reported fits and some subgroup analyses use scikit-learn for reproducibility and clearer convergence behaviour. If RAPIDS/cuML is unavailable, the notebook can fall back to CPU scikit-learn logic, but this will be slower.

---

## How to run

From the repository root:

1. Place the raw CCLE files in `data/`.
2. Run:

```text
notebooks/ccle_eda.ipynb
```

This creates `processed/` outputs.

3. Run:

```text
notebooks/ccle_modelling.ipynb
```

This loads the processed CCLE artefacts and evaluates the CCLE models.

On Kaggle, attach the CCLE Kaggle dataset and run `notebooks/ccle_modelling.ipynb` directly. The notebook is written to discover the `processed/` directory from the attached dataset.

### Option B: rerun the modelling notebooks on Kaggle

1. Create or open a Kaggle notebook.
2. Enable the **2xT4 GPU** accelerator.
3. Attach the TCGA Kaggle dataset.
4. Upload or open:

```text
notebooks/tcga_modelling.ipynb
```

5. Run all cells.

The notebook searches for the TCGA mutation, phenotype, expression, and TP53 target-gene files under the Kaggle input directory.

---

## Prediction tasks

### Task 1: TP53 mutant versus wild type

Task 1 is binary classification:

```text
RNA expression profile -> TP53 mutant or TP53 wild type
```

The main binary label is based on nonsynonymous / protein-altering TP53 mutation annotations. Synonymous and ambiguous variants are treated cautiously because they may not have the same functional interpretation as protein-altering TP53 mutations.

### Task 2: TP53 mutation type

Task 2 is implemented as coarse mutation-type / consequence classification.

Rather than modelling every rare mutation mechanism separately, related annotations are grouped into biologically meaningful classes with enough sample support for supervised learning.

In CCLE, the main Task 2 classes are:

```text
wild_type
missense_or_inframe
lof_or_splice
```

In TCGA, the corresponding classes are:

```text
WT
missense
truncating_or_splice
```

This grouping separates:

- wild-type TP53;
- missense or in-frame alterations that preserve the reading frame but alter amino-acid sequence or local protein structure;
- nonsense, frameshift, splice-site, or truncating events that are more likely to disrupt TP53 protein function.

This is not a claim that every possible raw DNA-level mutation mechanism is modelled separately. It is a statistically stable mutation-type / consequence version of Task 2.
