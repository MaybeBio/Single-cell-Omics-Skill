# Chapter 2: Preprocessing and Visualization

## Core Idea
Convert a raw count matrix (cells × genes) into a denoised, normalized, low-dimensional representation suitable for downstream analysis. Order matters: filter cells → remove ambient RNA → detect doublets → normalize → select features → reduce dimensionality. Be permissive at every cutoff — over-filtering removes rare populations that no downstream trick can recover.

## Frameworks Introduced

- **MAD-based cell filtering** (replaces hard count thresholds)
  - When to use: any scRNA-seq dataset where manual thresholding feels arbitrary
  - How: compute median ± N·MAD for each QC metric; flag outliers. Use 5 MAD for counts/genes/top-20, 3 MAD + 8% hard cap for mitochondrial fraction
  - Why: data-dependent, automated, robust to dataset scale; preserves rare small populations better than fixed thresholds

- **Three normalization strategies, matched to task**
  1. **Shifted logarithm** `log(y/s + y₀)` — best for PCA, dimensionality reduction, DGE
  2. **Scran pooling** — deconvolution-based size factors; best for batch correction
  3. **Analytic Pearson residuals** — regularized NB regression; best for HVG selection and rare cell-type recovery
  - The choice is not neutral — each preserves a different signal; store all three as layers and pick per task

- **Deviance-based feature selection** (replaces HVG-on-log-data)
  - When to use: feature selection step, before PCA
  - How: compute binomial deviance on raw counts (closed form, `scry::devianceFeatureSelection`); rank genes; take top N (e.g. 4,000)
  - Why: avoids pseudo-count bias that log+HVG introduces; works on raw counts directly

- **Doublet detection with scDblFinder**
  - When to use: per-sample, never on aggregated multi-batch data
  - How: simulate artificial doublets by averaging cell pairs; compute doublet score as fraction of artificial doublet neighbors in PC space; classify via `scDblFinder.class`

## Key Concepts
- **Barcode vs cell**: a barcode may tag 0 cells (empty droplet), 1 cell (singlet), or ≥2 cells (doublet) — use "barcode" until doublet detection is done
- **Ambient RNA ("the soup")**: cell-free mRNA in the dilution, barcoded alongside cellular mRNA; estimated from empty droplets
- **MAD**: `median(|Xᵢ - median(X)|)`, robust variability statistic
- **Size factor `s_c`**: per-cell scaling factor to normalize for library size; `s_c = Σ_g y_gc / L` with target sum `L`
- **Overdispersion α**: quadratic mean-variance relationship `Var[Y] = μ + αμ²` (Gamma-Poisson); scRNA UMI data is overdispersed
- **Pearson residual**: `(observed - predicted) / √predicted_variance` — measures deviation from a regularized NB model of technical noise
- **DORC**: Domain of Regulatory Chromatin — multiple correlated peaks around one gene (covered in ch08)

## Mental Models
- **Filter cells once, permissively** — re-assess QC after annotation; you can always filter more later, you cannot put cells back
- **Three covariates jointly** (counts, genes, %mito) — filtering on %mito alone kills respiring cells; filtering on counts alone kills quiescent small cells
- **Match normalization to the downstream task** — log for PCA + DGE, scran for batch correction, Pearson for HVG + rare types
- **Use raw counts for deviance** — normalization choices inject arbitrary pseudo-count variance that biases HVG selection
- **Initialize t-SNE from PCA** — reduces noise, improves stability vs random init
- **UMAP preserves more global structure than t-SNE** — both are valid; UMAP is the default for visualization

## Anti-patterns
- **Aggressive QC cutoffs**: bias against smaller / quiescent / metabolically active subpopulations
- **Feature-based filtering** (selecting "interesting" genes before QC): no demonstrated downstream benefit
- **Running doublet detection on aggregated multi-batch data**: artificial doublets span batch effects, scores become unreliable
- **CPM with fixed L=1e5 or 1e6**: produces larger overdispersion than typical scRNA data; prefer median library size (scanpy default `target_sum=None`)
- **Pseudo-count = 1 + log + HVG**: pseudo-count choice dominates HVG results, especially for low-count genes
- **Using raw counts for PCA**: log-transform first — multiplicative biology becomes additive, variance stabilizes, highly-expressed genes no longer dominate distances
- **Ignoring mitochondrial fraction**: high %mito = broken membrane = dying cell; but also legitimate respiratory cells — set the cap (8%) thoughtfully

## Code Examples

### MAD-based outlier detection (the canonical helper)

```python
from scipy.stats import median_abs_deviation
import numpy as np

def is_outlier(adata, metric: str, nmads: int):
    M = adata.obs[metric]
    outlier = (M < np.median(M) - nmads * median_abs_deviation(M)) | (
        np.median(M) + nmads * median_abs_deviation(M) < M
    )
    return outlier

# Apply per metric with different cutoffs
adata.obs["outlier"] = (
    is_outlier(adata, "log1p_total_counts", 5)
    | is_outlier(adata, "log1p_n_genes_by_counts", 5)
    | is_outlier(adata, "pct_counts_in_top_20_genes", 5)
)
adata.obs["mt_outlier"] = is_outlier(adata, "pct_counts_mt", 3) | (
    adata.obs["pct_counts_mt"] > 8
)
adata = adata[(~adata.obs.outlier) & (~adata.obs.mt_outlier)].copy()
```

- **What it demonstrates**: permissive 5-MAD filter on counts/genes/top-20; tighter 3-MAD + 8% cap on mitochondrial fraction. Annotation-aware — re-assess after clustering.

### Three normalization layers stored in one AnnData

```python
# 1. Shifted logarithm (size factor = median library size)
scales = sc.pp.normalize_total(adata, target_sum=None, inplace=False)
adata.layers["log1p_norm"] = sc.pp.log1p(scales["X"], copy=True)

# 2. Scran pooling (requires coarse clustering + raw counts)
sc.pp.normalize_total(adata_pp); sc.pp.log1p(adata_pp)
sc.pp.pca(adata_pp, n_comps=15); sc.pp.neighbors(adata_pp)
sc.tl.leiden(adata_pp, key_added="groups", flavor="igraph",
             n_iterations=2, directed=False)
# Pass RAW counts (not adata_pp.X) and the groups to scran::computeSumFactors
adata.obs["size_factors"] = size_factors  # from R
adata.layers["scran_normalization"] = sc.sparse.csr_matrix(
    np.log1p(adata.X / adata.obs["size_factors"].values[:, None]))

# 3. Analytic Pearson residuals
adata.layers["analytic_pearson_residuals"] = sc.sparse.csr_matrix(
    sc.experimental.pp.normalize_pearson_residuals(adata, inplace=False)["X"])
```

- **What it demonstrates**: all three methods as separate layers — pick per downstream task. Note `computeSumFactors` requires raw counts, not the log-normalized `adata_pp.X` used only for clustering.

### Deviance feature selection (raw counts, no pseudo-count)

```python
%%R
sce <- devianceFeatureSelection(sce, assay = "X")
```
```python
binomial_deviance = ro.r("rowData(sce)$binomial_deviance")
idx = binomial_deviance.argsort()[-4000:]
mask = np.zeros(adata.var_names.shape, dtype=bool); mask[idx] = True
adata.var["highly_deviant"] = mask
adata.var["binomial_deviance"] = binomial_deviance
```

- **What it demonstrates**: closed-form deviance on raw counts; no normalization, no pseudo-count, no log. Top 4,000 by deviance. Empirically selects high-mean genes (Townes 2019 observation).

### PCA → UMAP pipeline

```python
adata.var["highly_variable"] = adata.var["highly_deviant"]
sc.pp.pca(adata, svd_solver="arpack", mask_var="highly_variable")
sc.pp.neighbors(adata)  # KNN graph on X_pca
sc.tl.umap(adata)
# t-SNE alternative: sc.tl.tsne(adata, use_rep="X_pca")  # init from PCA
```

## Reference Tables

### Normalization method → best downstream use

| Method | Best for | Caveat |
|---|---|---|
| Shifted logarithm | PCA, DGE, visualization | Pseudo-count choice affects low-count genes |
| Scran pooling | Batch correction | Requires coarse clustering input; needs ≥2 samples |
| Analytic Pearson residuals | HVG selection, rare cell types | Output can be negative; less intuitive |

### QC metric → recommended threshold strategy

| Metric | Strategy |
|---|---|
| `log1p_total_counts` | 5 MAD |
| `log1p_n_genes_by_counts` | 5 MAD |
| `pct_counts_in_top_20_genes` | 5 MAD |
| `pct_counts_mt` | 3 MAD **and** hard cap at 8% |
| Gene detection (`min_cells`) | ≥20 cells (information threshold) |

### Mitochondrial prefix by species

| Species | Prefix |
|---|---|
| Human | `MT-` |
| Mouse | `mt-` |

## Worked Example

**NeurIPS 2021 bone marrow mononuclear cells** — sample s4d8 of donor 8, 10x Multiome assay.

Pipeline on 16,934 cells × 36,601 genes:
1. `var_names_make_unique()` — genes appear multiple times due to ambiguous annotation
2. Annotate `mt` / `ribo` / `hb` gene sets via prefix/regex
3. `calculate_qc_metrics` adds `n_genes_by_counts`, `total_counts`, `pct_counts_mt` to `.obs`
4. MAD filter (5/5/5/3+8%): drop cells failing any
5. SoupX ambient RNA correction: cluster cells (Leiden on PCA of log-normalized copy), feed `filtered_feature_bc_matrix` + raw `raw_feature_bc_matrix` to `SoupChannel()` → `autoEstCont()` → `adjustCounts()`; store as `adata.layers["soupX_counts"]` and overwrite `.X`
6. Filter genes (≥20 cells detected)
7. scDblFinder via rpy2: store `scDblFinder_score` and `scDblFinder_class` in `.obs` (do not remove doublets yet — visualize first)
8. Compute three normalization layers (`log1p_norm`, `scran_normalization`, `analytic_pearson_residuals`); keep raw counts in `layers["counts"]`
9. Deviance feature selection → top 4,000 highly deviant genes; set `highly_variable = highly_deviant`
10. PCA on HVGs → neighbors graph → UMAP
11. Inspect QC metrics on UMAP: doublets cluster together → revisit QC if needed

The "do not remove doublets yet" choice is deliberate — doublets often form their own UMAP cluster, which is the strongest validation that the score is meaningful. Remove them after visualization, not before.

## Key Takeaways
1. **Use MAD, not fixed thresholds** — 5 MAD for counts/genes/top-20, 3 MAD + 8% cap for mitochondrial
2. **Three normalizations, three layers, three purposes** — log for PCA/DGE, scran for batch, Pearson for HVG/rare types
3. **Deviance on raw counts for feature selection** — avoids pseudo-count bias that log+HVG introduces
4. **Run scDblFinder per sample, never on aggregated batches** — artificial doublets across batches are uninterpretable
5. **Initialize t-SNE from PCA**; default to UMAP for visualization (preserves more global structure)
6. **Keep doublets in the dataset through dimensionality reduction** — visualize their UMAP cluster before removing
7. **Re-assess QC after annotation** — first-pass filtering should be permissive; tightening later is cheap, recovering filtered cells is impossible
8. **Match normalization to the downstream task** — never default to one layer for everything

## Connects To
- **Ch 1 (Introduction)**: raw data processing produces the count matrix that enters this pipeline; AnnData/MuData structures store the layers
- **Ch 3 (Clustering, annotation, integration)**: operates on `X_pca` from this chapter; doublet clusters revealed in UMAP feed back into QC refinement
- **Ch 5 (DGE)**: uses the shifted-log layer; pseudobulk DGE is sensitive to normalization choice
- **Ch 8 (scATAC QC)**: parallel QC logic for chromatin data — different metrics (TSS enrichment, nucleosome signal) but same MAD-style filtering philosophy
- **Ch 10 (Surface protein)**: CITE-seq has its own QC + normalization, but the per-sample doublet detection rule applies
