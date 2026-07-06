# Chapter 7: Bulk Deconvolution

## Core Idea
Infer cell-type proportions in bulk RNA-seq samples by solving `y = bX` — mixture `y` equals cellular proportions `b` times a single-cell-derived signature matrix `X`. The single-cell era transformed this from a biased sorted-cell exercise into an unbiased reference-driven computation.

## Frameworks Introduced

- **Linear deconvolution model `y = bX`**
  - When to use: bulk RNA-seq or microarray from heterogeneous tissue, with a matching scRNA-seq reference
  - How: build signature matrix `X` from scRNA-seq clusters; solve for `b` with regularized regression (CIBERSORTx, MuSiC, DWLS, SCDC, FARDEEP)

- **Cobos et al. five recommendations** (community-standard guidance from benchmarks)
  1. Keep input data in **linear scale** (library-size normalize, do not log-transform before deconvolution)
  2. **Avoid** row-scaling, column min-max, column z-score, and quantile normalization
  3. Use regression methods (CIBERSORTx, FARDEEP); if scRNA-seq reference available, also run DWLS, MuSiC, SCDC and compare
  4. **Stringent marker selection**: rank by difference between the top- and second-highest expressing cell types
  5. **Comprehensive reference**: include every cell type present in the bulk mixture — missing cell types are the single largest error source

- **AutoGeneS multi-objective feature selection**
  - When to use: no prior marker knowledge, want to minimize collinearity in `X`
  - How: simultaneously minimize inter-cell-type correlation and maximize inter-cell-type distance; outputs a signature gene list fed into any downstream deconvolution method

- **CPM (Cell Population Mapping)**
  - When to use: compositional changes *within* a cell type (sub-states, trajectories) rather than across cell types
  - How: focuses variation within cell types; captures continuous transitions along trajectories

## Key Concepts
- **Signature matrix (`X`)**: cell-type × gene profile matrix; today built from scRNA-seq clusters, historically from FACS/CyTOF sorted cells (biased by antibody panel)
- **Mixture (`y`)**: bulk gene expression profile, microarray or RNA-seq
- **Proportions (`b`)**: inferred cell-type fractions; non-negative, sum to 1 per sample
- **Missing cell types**: reference omits a cell type actually present in bulk — largest accuracy hit
- **Rare cell types**: deconvolution consistently fails to estimate proportions for cell types with few reference cells (Cobos cut-off is user-defined, e.g. 50 cells)
- **Collinearity**: highly correlated cell types in `X` make `b` unstable; mitigated by feature selection

## Mental Models
- **Use scRNA-seq references, not sorted-cell panels** — sorting requires antibody pre-selection that bakes in cell-type assumptions and misses unknown populations
- **Try ≥2 methods, judge biologically** — deconvolution is hard enough that no single tool should be trusted; compare outputs against known biology (e.g. neutrophil counts in blood) before picking one
- **Treat the output as fractions, not absolute counts** — `b` sums to 1; condition comparisons need compositional-aware statistics (see ch05)
- **Rare cell types are a structural limit, not a tuning problem** — if a cell type is rare in the reference, no method will recover it accurately; either drop it or enrich the reference

## Anti-patterns
- **Quantile / z-score / row-scaling normalization on inputs**: breaks the linear-scale assumption that benchmarks rely on
- **Log-transforming before deconvolution**: destroys the linear relationship; log only for visualization
- **Reference built from one sample**: MuSiC and similar multi-sample methods fail; need ≥2 scRNA-seq samples with overlapping cell types
- **Including proliferating / mixed cells in reference**: their proportions are not cell-type-specific and contaminate the signature
- **Trusting a single method's output without biological validation**: benchmark spread between methods is large; always cross-check

## Code Examples

Library-size normalize and prepare scRNA-seq reference (linear scale, no log):

```python
import scanpy as sc
import numpy as np

adata.X = adata.layers["counts"]  # use raw counts
adata = adata[adata.obs["cells"] == "Whole_blood"].copy()
# drop non-cell-type-specific clusters
adata = adata[~adata.obs["cluster_labels_res.0.8"].isin(
    ["None", "Mixed_cells", "Prol. cells"])].copy()
# drop rare cell types (user-defined cutoff)
ct_counts = adata.obs["cluster_labels_res.0.8"].value_counts()
ct_to_keep = ct_counts[ct_counts > 50].index
adata = adata[adata.obs["cluster_labels_res.0.8"].isin(ct_to_keep)].copy()
# intersect genes with bulk
bulk_sc_genes = np.intersect1d(bulk.index, adata.var_names)
bulk = bulk.loc[bulk_sc_genes, :].copy()
adata = adata[:, bulk_sc_genes].copy()
```

- **What it demonstrates**: the canonical pre-MuSiC pipeline — raw counts, drop mixed/proliferating clusters, drop rare cell types, gene intersection. Library-size normalization happens next via `sc.pp.normalize_per_cell(..., counts_per_cell_after=1e4)`; log1p is applied only to a copy used for visualization.

Running MuSiC via rpy2 (R package accessed from Python):

```python
import anndata2ri
from rpy2.robjects import pandas2ri
pandas2ri.activate()
anndata2ri.activate()
%load_ext rpy2.ipython
```

```R
%%R -i adata_r,cell_subsets_r,bulk,sc_genes -o musicRes
library(MuSiC); library(Biobase)
df = new("AnnotatedDataFrame", data = data.frame(
    cellNames = cell_subsets_r,
    Sample = factor(rep(1, nrow(adata_r@colData))),
    row.names = row.names(adata_r@colData)))
SCDataES <- ExpressionSet(
    assayData = Matrix::as.matrix(adata_r@assays@data@listData[[1]]),
    phenoData = df, protocolData = df)
bulkDataES <- ExpressionSet(assayData = as.matrix(bulk))
musicRes = MuSiC::music_prop(bulk.eset = bulkDataES, sc.eset = SCDataES,
                             clusters = "cellNames")
```

- **What it demonstrates**: MuSiC requires an `ExpressionSet` and runs one bulk sample at a time. Output is a list; `musicRes[0]` is the N×M proportion matrix (samples × cell types).

## Reference Tables

### Deconvolution method families

| Family | Example tools | Strengths | Weaknesses |
|---|---|---|---|
| Linear regression | CIBERSORTx, MuSiC, dtangle, DWLS, SCDC, FARDEEP | Best-benchmarked; interpretable; works with scRNA-seq reference | Sensitive to collinearity; needs comprehensive reference |
| Enrichment-based | xCell | Single cell type at a time; simple | Less accurate with many cell types |
| Non-linear / deep learning | Scaden | Promising accuracy | Early; interpretability unclear; not yet benchmark-dominant |

### Normalization do's and don'ts (Cobos et al.)

| Operation | Verdict |
|---|---|
| Library-size normalization | ✓ Recommended |
| Linear scale input | ✓ Required |
| Log-transform before deconvolution | ✗ Avoid (log only for visualization) |
| Row scaling | ✗ Avoid |
| Column min-max | ✗ Avoid |
| Column z-score | ✗ Avoid |
| Quantile normalization | ✗ Avoid |

## Worked Example

**Bulk deconvolution of COVID-19 whole blood** — 49 bulk RNA-seq samples (39 COVID patients + 10 healthy controls) deconvolved against a single-cell whole blood reference (Schulte-Schrepping et al.).

Pipeline:
1. Load raw counts, subset to `Whole_blood` cells, drop `None`/`Mixed_cells`/`Prol. cells` clusters
2. Drop rare cell types (<50 cells)
3. Intersect bulk and scRNA-seq genes
4. Library-size normalize to 1e4; select 5,000 HVGs (cell_ranger flavor); log1p **a copy** for UMAP visualization only
5. Sub-sample 80 cells per type (memory limit for R conversion); convert to `ExpressionSet`
6. Run `MuSiC::music_prop`; receive N×M proportion matrix
7. Validate: correlate inferred neutrophil fraction against clinical `Total.neutrophil.count...mm3.` — high correlation confirms biological plausibility
8. Test condition difference: Student's t-test per cell type between COVID and healthy; boxplot the most-significant cell type

The validation step (correlating against an externally measured cell count) is the model for how to pick between competing deconvolution results — pick the method whose output tracks independent measurements.

## Key Takeaways
1. **Reference comprehensiveness dominates accuracy** — a missing cell type in `X` corrupts every proportion estimate; invest in the reference before tuning the method
2. **Keep inputs in linear scale, library-size normalized**; avoid z-score / quantile / row-scaling normalizations
3. **Run ≥2 methods** (e.g. MuSiC + DWLS or CIBERSORTx + SCDC) and select the result that best matches external biology, not the one that "looks cleanest"
4. **Drop rare cell types** from the reference rather than hoping a method will recover them
5. **Validate against external measurements** when possible (clinical cell counts, flow cytometry, known biology) — deconvolution output without validation is a hypothesis, not a result
6. **For sub-state composition changes within a cell type, use CPM**, not standard bulk deconvolution

## Connects To
- **Ch 3 (Annotation)**: cell-type labels in the scRNA-seq reference feed directly into the signature matrix `X`; misannotation propagates into bulk proportions
- **Ch 5 (Compositional analysis)**: deconvolution outputs are fractions summing to 1 — use compositional-aware statistics for condition comparisons, not naive t-tests across all cell types independently
- **Ch 9 (Spatial deconvolution)**: same `y = bX` framework applied to spatial transcriptomics spots, with spatial-specific reference constraints
