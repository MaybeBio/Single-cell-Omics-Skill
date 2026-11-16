# Chapter 23: Scanpy — Clustering & Integration (leiden + ingest/BBKNN)

## Core Idea
The canonical clustering-and-annotation loop is: per-batch QC → Scrublet doublets → save `layers["counts"]` → normalize_total+log1p → HVG (`batch_key`) → PCA → neighbors → UMAP → **multi-resolution Leiden** → dotplot marker annotation → `rank_genes_groups` (Wilcoxon). For batch integration, two lightweight scanpy-native paths exist: **`sc.tl.ingest`** (asymmetric reference→query label/embedding mapping via PCA + neighbor search) and **BBKNN** (symmetric graph-level correction) — both leave the data matrix untouched, so raw counts stay intact for downstream DE.

## Frameworks Introduced

- **Permissive-first, re-assess-later QC**
  - When to use: every clustering run
  - How: start with loose thresholds (`filter_cells(min_genes=100)`, `filter_genes(min_cells=3)`), run clustering, then re-inspect QC metrics (doublet_score, pct_counts_mt, total_counts) on the UMAP and filter clusters
  - Why: "poor" QC metrics are often real biology; aggressive upfront filtering discards valid cell states. Do QC **per sample** — thresholds vary across batches

- **Multi-resolution Leiden + dotplot annotation**
  - When to use: labeling clusters without a reference
  - How: run `sc.tl.leiden` at several resolutions (e.g. `0.02/0.5/2.0`, `key_added="leiden_res_..."`), then `sc.pl.dotplot(adata, marker_genes, groupby=..., standard_scale="var")` and map clusters → cell types by hand
  - Why: cluster count is arbitrary (resolution-controlled); the "right" resolution is the one that matches stable, marker-distinguishable groups. `standard_scale="var"` z-scores genes so low/high expressors are comparable

- **ingest vs BBKNN for integration**
  - When to use: choosing a lightweight integration method
  - How: `sc.tl.ingest(adata, adata_ref, obs="celltype")` fits PCA+neighbors+UMAP on `adata_ref` and projects labels+embeddings onto query; `sc.external.pp.bbknn(adata_concat, batch_key=...)` recomputes the neighbor graph across batches
  - Why: **ingest** is asymmetric and solves the *label-mapping* problem (like scmap/MNN), preserving a reference structure/trajectory; **BBKNN** is symmetric and mixes batches more homogeneously but doesn't transfer labels. Both leave `X` invariant

## Key Concepts
- **`sc.pp.scrublet(adata, batch_key=...)`**: adds `doublet_score` + `predicted_doublet` to `.obs`; filter now or flag clusters later.
- **`sc.tl.leiden(flavor="igraph", n_iterations=2)`**: igraph flavor is significantly faster, especially on large data; clusters the neighbor graph directly.
- **`rank_genes_groups(..., method="wilcoxon")`**: one-vs-rest marker test per cluster; results in `uns["rank_genes_groups"]`, flattened by `sc.get.rank_genes_groups_df(adata, group=...)`.
- **Pseudobulk caveat**: `rank_genes_groups` treats each cell as independent → p-values are wildly inflated; for condition comparisons aggregate via `sc.get.aggregate(..., by=["sample","cell_type"], layer="counts")` + PyDESeq2.
- **ingest requirements**: same `var_names` (intersect), reference must capture the biological variation of interest; reorder/recolor categories after concat (`cat.reorder_categories` + copy `uns["..._colors"]`).
- **`sc.tl.embedding_density(groupby=...)`**: batch density overlays for comparing experiments.

## Mental Models
- **Think of `ingest` as "train a reference manifold, then ask 'where does this new cell go?'"** — it transfers labels *and* embeddings, not just a correction.
- **Think of resolution as a zoom dial, not a truth** — no single resolution is correct; coarse for broad lineages, fine for subtypes, annotate at the level your markers resolve.
- **Think of QC as a loop, not a gate** — filter permissively, cluster, then let the UMAP reveal which clusters are debris.

## Anti-patterns
- **Global QC thresholds across batches** — batch-specific thresholds are mandatory; a threshold right for one sample can gut another.
- **Trusting per-cell `rank_genes_groups` p-values for condition comparison** — pseudoreplication; pseudobulk first.
- **Aggressive upfront filtering** — real biology (e.g. erythroid, plasma cells) can look like "bad QC"; revisit after clustering.
- **`ingest` with a reference lacking key cell types** — query cells with no reference counterpart map incorrectly (e.g. Megakaryocytes vanish); always question reference coverage.
- **Forgetting to reorder/color categories after `ingest`+concat** — category order/colors must be copied from the reference for comparable plots.

## Code Examples

```python
import scanpy as sc, anndata as ad
adata = ad.concat(adatas, label="sample"); adata.obs_names_make_unique()

# QC (per sample) + save raw counts + normalize
adata.var["mt"] = adata.var_names.str.startswith("MT-")
sc.pp.calculate_qc_metrics(adata, qc_vars=["mt"], inplace=True, log1p=True)
sc.pp.filter_cells(adata, min_genes=100); sc.pp.filter_genes(adata, min_cells=3)
sc.pp.scrublet(adata, batch_key="sample")
adata.layers["counts"] = adata.X.copy()
sc.pp.normalize_total(adata); sc.pp.log1p(adata)

# feature selection + dim red + clustering
sc.pp.highly_variable_genes(adata, n_top_genes=2000, batch_key="sample")
sc.tl.pca(adata); sc.pp.neighbors(adata); sc.tl.umap(adata)
for res in [0.02, 0.5, 2.0]:
    sc.tl.leiden(adata, key_added=f"leiden_{res:4.2f}", resolution=res, flavor="igraph")

# annotate: markers + DE
sc.tl.rank_genes_groups(adata, "leiden_0.50", method="wilcoxon")
sc.pl.rank_genes_groups_dotplot(adata, n_genes=5, standard_scale="var")
sc.get.rank_genes_groups_df(adata, group="7").head()

# ingest: map reference labels onto query
sc.pp.pca(adata_ref); sc.pp.neighbors(adata_ref); sc.tl.umap(adata_ref)
sc.tl.ingest(adata_query, adata_ref, obs="celltype")
```
- **What it demonstrates**: the full clustering→annotation loop plus reference-based label transfer.

## Reference Tables

### Clustering workflow order

| Step | Function | Output |
|---|---|---|
| QC metrics | `pp.calculate_qc_metrics` | `obs["total_counts"/"pct_counts_mt"/...]` |
| Filter | `pp.filter_cells`/`filter_genes` | subset in place |
| Doublets | `pp.scrublet(batch_key)` | `obs["predicted_doublet"/"doublet_score"]` |
| Normalize | `pp.normalize_total` + `pp.log1p` | `X` |
| HVG | `pp.highly_variable_genes(batch_key)` | `var["highly_variable"]` |
| Dim red | `pp.pca` → `pp.neighbors` → `tl.umap` | `obsm["X_pca"/"X_umap"]`, `obsp[...]` |
| Cluster | `tl.leiden(flavor="igraph")` | `obs["leiden"]` |
| Annotate | `pl.dotplot` + `tl.rank_genes_groups` | `uns["rank_genes_groups"]` |

### ingest vs BBKNN

| | `sc.tl.ingest` | BBKNN (`sc.external.pp.bbknn`) |
|---|---|---|
| Direction | asymmetric (ref → query) | symmetric (joint graph) |
| Transfers labels | ✅ (obs) | ❌ |
| Transfers embedding | ✅ (X_pca, X_umap) | ❌ (recompute) |
| Modifies `X` | ❌ | ❌ |
| Best for | reference annotation, preserving trajectory | mixing batches homogeneously |
| Key input | `adata, adata_ref, obs=` | `batch_key=` |

## Key Takeaways
1. QC permissively per sample, then re-assess on the UMAP — aggressive upfront filtering discards real biology.
2. Cluster at multiple Leiden resolutions and annotate with a marker dotplot (`standard_scale="var"`).
3. `rank_genes_groups` p-values are per-cell (pseudoreplication) — pseudobulk for condition DE.
4. `ingest` transfers labels+embeddings from a reference (asymmetric); BBKNN corrects the graph (symmetric); both keep `X` intact.
5. Always check reference coverage before `ingest` — missing cell types map incorrectly.

## Connects To
- **Ch 03**: clustering/annotation/integration rationale (Leiden, marker-based annotation, scIB benchmarking).
- **Ch 21**: the pp/tl/pl functions this workflow composes.
- **Ch 22**: `rank_genes_groups_df`, `sc.get.aggregate`, `sc.metrics.confusion_matrix` support this chapter.
- **Ch 05 / Ch 13**: pseudobulk DGE and scVI/Harmony are the heavier alternatives to ingest/BBKNN.
