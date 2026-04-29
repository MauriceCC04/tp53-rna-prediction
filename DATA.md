# Data

## DepMap / CCLE (Cell Line Data)

**Download:** https://depmap.org/portal/data_page/?tab=currentRelease — Release 26Q1, no login required.

| File | Description |
|---|---|
| `Model.csv` | 2,116 cell lines with metadata: `ModelID`, `OncotreeLineage`, `OncotreePrimaryDisease` (49 columns) |
| `OmicsSomaticMutations.csv` | ~751k somatic mutation calls (long format). Key columns: `HugoSymbol`, `VariantType`, `VariantInfo`, `DNAChange`, `ProteinChange`, `LikelyLoF`, `Hotspot`. Filter to `HugoSymbol == "TP53"` for 1,319 mutations across 1,174 cell lines. |
| `OmicsExpressionProteinCodingGenesTPMLogp1.csv` | RNA expression matrix: cell lines × ~19,000 genes, values in log₂(TPM+1) |

---

## TCGA (Patient Tumour Data)

**Download:** https://xenabrowser.net/datapages/ → TCGA Pan-Cancer (PANCAN) cohort, no login required.

| File | Where to find it on Xena | Description |
|---|---|---|
| `tcga_RSEM_gene_tpm.gz` | Gene Expression RNAseq → *TOIL RSEM tpm (n=10,535)* | RNA expression matrix: ~10,535 samples × ~19,000 genes, log₂(TPM+0.001). TPM chosen to match DepMap units. |
| `mc3.v0.2.8.PUBLIC.xena.gz` | Somatic Mutation → *MC3 (public)* | ~2.9M mutation rows. Key columns: `sample`, `gene`, `effect`, `Amino_Acid_Change`, `SIFT`, `PolyPhen`. Filter to `gene == "TP53"`. |
| `Survival_SupplementalTable_S1_20171025_xena_sp` | Phenotype → *Curated clinical data* | 12,591 samples across 33 cancer types. Used for the `cancer type abbreviation` column (equivalent of `OncotreeLineage` for TCGA). |

---

## TP53 Target Gene List

**File:** `tp53_target_genes_fischer2017.csv`  
**Source:** https://tp53.cancer.gov/target_genes — table manually parsed, no download button provided.  
343 confirmed direct p53 transcriptional targets from Fischer (2017), *Oncogene*. DOI: 10.1038/onc.2016.502. Used as prior biological knowledge in the feature selection step.