# Chapter 22: Scanpy — API Reference & Plotting

## Core Idea
Scanpy's full API spans `pp`/`tl`/`pl` (Ch 21) plus satellite modules: **`sc.io`** (10x/mtx/csv readers), **`sc.get`** (extract results as DataFrames), **`sc.queries`** (mito genes, enrichment), **`sc.metrics`** (Moran's I, Geary's C, confusion matrix), **`sc.experimental`** (Pearson-residual methods), **`sc.datasets`** (bundled benchmarks), and **`scanpy.external`** (deprecated wrappers). Plotting is a first-class surface: `sc.pl` mirrors `sc.tl`, returns matplotlib `Axes`/`Figure` for customization, and every plot type has a specific job (dotplot for overview, violin for distribution, heatmap for per-cell, matrixplot for scaled means).

## Frameworks Introduced

- **Result-extraction via `sc.get`**
  - When to use: pulling computed results into pandas for downstream/table output
  - How: `sc.get.rank_genes_groups_df(adata, group=...)` flattens DE results; `sc.get.pca(adata)` returns PCA as an AnnData indexed by component; `sc.get.var_df` / `sc.get.aggregate` for obs tables and group aggregation
  - Why: `uns["rank_genes_groups"]` is a nested dict; `sc.get` gives it a tabular shape

- **Plot-type selection by goal**
  - When to use: choosing the right visualization for markers/clusters
  - How: **dotplot** = color mean expression + size fraction expressing (best overview); **violin**/stacked_violin = full value distribution; **matrixplot** = heatmap of mean expression (often scaled); **heatmap** = per-cell rows (no collapse); **tracksplot** = expression as height; **rank_genes_groups_*** = the same shapes for DE output
  - Why: each collapses cells differently — dotplot/matrixplot collapse to group means, heatmap shows every cell

- **matplotlib interop for customization**
  - When to use: any non-default plot
  - How: `show=False` returns `Axes` (or dict/list for multi-axes), `return_fig=True` returns the `Figure`; `ax=` accepts pre-built `Axes`; `rc_context({"figure.figsize": ...})` scopes sizing
  - Why: continuous plots have a second colorbar `Axes`; `dotplot` returns a `DotPlot` object with `mainplot_ax`/`size_legend_ax`/`color_legend_ax`

## Key Concepts
- **Array type support**: most functions accept `ndarray`, `csr`/`csc`; a subset accept Array-API/dask (⚡ = dask chunk). Check the API support table before using a GPU/dask array with a given function.
- **`sc.metrics`**: `morans_i` / `gearys_c` (spatial autocorrelation), `confusion_matrix` (label agreement).
- **`sc.queries`**: `mitochondrial_genes` (BioMart lookup), `enrich` (DE enrichment).
- **`sc.experimental`**: `pp.normalize_pearson_residuals_pca`, `pp.highly_variable_genes` (analytic Pearson), `pp.recipe_pearson_residuals` (full pipeline; Lause et al. 2021).
- **`sc.datasets`**: `pbmc3k`, `pbmc3k_processed`, `pbmc68k_reduced`, `paul15`, `moignard15`, `krumsiek11`, `visium_sge`, `ebi_expression_atlas`.
- **Deprecated**: `tl.louvain` (→ `tl.leiden`), `pp.recipe_weinreb17`, `tl.sim`/`pl.sim`, `queries.gene_coordinates`, `read_loom`, `read_visium`.
- **`scanpy.external`** (deprecated since 1.13.0): wraps PHATE/trimap/SAM/wishbone/SPRING/cellbrowser; being moved to the scverse ecosystem.

## Mental Models
- **Think of `sc.get` as "`.to_dataframe()` for scanpy results"** — when `uns` is nested, `sc.get` is the flat view.
- **Think of the plot family as a spectrum of cell-collapsing** — dotplot (means+frequency) → matrixplot (means) → heatmap (all cells) → violin (distribution).
- **Color param = gene name OR `.obs` column** — `sc.pl.umap(adata, color=...)` accepts either, so marker genes and annotations plot identically.

## Anti-patterns
- **Using `scanpy.external` in new code** — deprecated since 1.13.0; use scverse ecosystem packages directly.
- **Using `tl.louvain`** — deprecated; use `tl.leiden` (needs igraph).
- **Forgetting `show=False` before customizing `Axes`** — without it the figure renders early and later edits don't apply.
- **Log-scale values without a log `norm`** — pass `norm=matplotlib.colors.LogNorm()` for log-normal data.
- **Symmetric/diverging data without centering** — use `vcenter=0` (or symmetric `vmin`/`vmax`) with a diverging `cmap` (`RdBu_r`, `bwr`, `coolwarm`).

## Code Examples

```python
import scanpy as sc
import matplotlib.pyplot as plt

adata = sc.datasets.pbmc68k_reduced()
sc.tl.rank_genes_groups(adata, "bulk_labels", method="wilcoxon")

# Extract DE results as a flat DataFrame
de = sc.get.rank_genes_groups_df(adata, group="B-cells")

# Marker overview: dotplot with dendrogram + scaled layer for matrixplot
markers = {"B": ["CD79A", "MS4A1"], "T": ["CD3D"], "NK": ["GNLY", "NKG7"]}
sc.pl.dotplot(adata, markers, "bulk_labels", dendrogram=True)
adata.layers["scaled"] = sc.pp.scale(adata, copy=True).X
sc.pl.matrixplot(adata, markers, "bulk_labels", layer="scaled",
                 vmin=-2, vmax=2, cmap="RdBu_r", dendrogram=True)

# Customize: return Axes, scope figsize, centered diverging palette
ax = sc.pl.umap(adata, color="n_counts", show=False)   # Axes
ax.set_xlabel("UMAP 1", fontsize=20)
with plt.rc_context({"figure.figsize": (4, 4)}):
    sc.pl.umap(adata, color="louvain", legend_loc="on data", frameon=False)

# Spatial autocorrelation of a gene across embedding
sc.metrics.morans_i(adata)             # -> adata.uns["moransI"]
```
- **What it demonstrates**: result extraction, plot-type selection, scaling a layer, matplotlib customization, and metrics.

## Reference Tables

### Scanpy module map (beyond pp/tl/pl)

| Module | Key functions |
|---|---|
| `sc.io` | `read_10x_mtx`, `read_h5ad`/`read_zarr`, `read_csv`/`read_mtx`/`read_text`/`read_hdf`/`read_excel` |
| `sc.get` | `rank_genes_groups_df`, `pca`, `var_df`, `aggregate` |
| `sc.queries` | `mitochondrial_genes`, `enrich` |
| `sc.metrics` | `morans_i`, `gearys_c`, `confusion_matrix` |
| `sc.experimental` | `pp.normalize_pearson_residuals_pca`, `pp.highly_variable_genes`, `pp.recipe_pearson_residuals` |
| `sc.datasets` | `pbmc3k(_processed)`, `pbmc68k_reduced`, `paul15`, `moignard15`, `krumsiek11`, `visium_sge`, `ebi_expression_atlas` |
| `scanpy.external` | deprecated (1.13.0) → scverse ecosystem |

### Plot-type quick pick

| Goal | Function |
|---|---|
| Gene/category overview across groups | `pl.dotplot` |
| Full distribution of expression | `pl.violin` / `pl.stacked_violin` |
| Mean expression heatmap (scaled) | `pl.matrixplot` |
| Every cell, one row/col | `pl.heatmap` |
| Expression as bar height | `pl.tracksplot` |
| DE results | `pl.rank_genes_groups_{dotplot,violin,heatmap,matrixplot,tracksplot,stacked_violin}` |
| Category hierarchy | `pl.dendrogram`, `pl.correlation_matrix` |

### Plot customization knobs

| Knob | Effect |
|---|---|
| `show=False` / `return_fig=True` | return `Axes` / `Figure` |
| `ax=` | draw on pre-built Axes |
| `rc_context({"figure.figsize": ...})` | scoped size |
| `color` | gene name or `.obs` column |
| `layer=` | plot a named layer (e.g. `"scaled"`) |
| `vmin`/`vmax`/`vcenter` | palette limits / diverging center |
| `cmap` / `palette` | continuous / discrete colors |
| `dendrogram=True` | cluster categories hierarchically |
| `standard_scale="var"` | per-gene z-score in matrixplot |
| `ncols`, `wspace`, `frameon`, `s` | layout/spacing/dots |

## Key Takeaways
1. `sc.get` flattens nested `uns` results into DataFrames — prefer it over hand-parsing `uns`.
2. Match plot type to goal: dotplot (overview), violin (distribution), heatmap (per-cell), matrixplot (scaled means).
3. `color=` accepts either a gene name or an `.obs` column; `layer=` switches the plotted matrix.
4. `show=False`/`return_fig=True`/`ax=` give full matplotlib control; `rc_context` scopes sizing.
5. Avoid `scanpy.external` and `tl.louvain` (both deprecated); use scverse ecosystem and `tl.leiden`.

## Connects To
- **Ch 21**: the pp/tl/pl core this chapter completes with the satellite modules.
- **Ch 23**: `rank_genes_groups` + marker plotting are used to annotate clusters in the clustering tutorial.
- **Ch 24**: `sc.experimental` Pearson-residual methods and `sc.metrics` (Moran's I) are used in advanced analyses.
- **Ch 18**: AnnData slots are the storage target for every `sc.get`/`sc.metrics`/`sc.queries` result.
