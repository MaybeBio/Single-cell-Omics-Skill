# Chapter 24: Scanpy — Advanced Methods (Pearson residuals, PAGA, dask, cell cycle, marsilea)

## Core Idea
Beyond the core workflow, scanpy ships experimental/advanced tools: **analytic Pearson residuals** (`sc.experimental.pp`) as a one-shot replacement for normalize+log1p, **PAGA** for coarse-grained trajectory abstraction, **dask** for out-of-core/1M+-cell scaling, **cell-cycle scoring/regression**, swappable **kNN transformers** for faster neighbors, and **Marsilea** for composed, publication-grade figures. These are opt-in and mostly flagged experimental — use them when the standard pipeline hits a ceiling (scale, noise, or figure control).

## Frameworks Introduced

- **Pearson residuals as a single-step normalization+HGV**
  - When to use: replacing `normalize_total`+`log1p` when you want variance-stabilized, technically-decorrelated data in one shot
  - How: `sc.experimental.pp.highly_variable_genes(adata, flavor="pearson_residuals", n_top_genes=2000)` → `sc.experimental.pp.normalize_pearson_residuals(adata)` (dense); wrappers `normalize_pearson_residuals_pca` and `recipe_pearson_residuals` do it all
  - Why: residuals remove depth technical variation, stabilize mean-variance, and amplify DE genes (residual variance >1) — so PCA is driven by biology, not technical noise. Save `layers["raw"]` and a `sqrt_norm` layer for plotting

- **PAGA + diffusion pseudotime**
  - When to use: mapping coarse structure of a developmental manifold
  - How: `sc.tl.paga(adata, groups=...)` builds the abstracted graph; `sc.pl.paga(threshold=...)` prunes weak edges; optionally denoise via `sc.tl.diffmap` + neighbors on `X_diffmap`; set `uns["iroot"]` then `sc.tl.dpt` for pseudotime; `sc.pl.paga_path` for gene changes along a path
  - Why: PAGA gives a topology (connectivities between clusters) that survives noisy single-cell edges; `draw_graph(init_pos="paga")` re-lays out the embedding from that topology

- **dask for out-of-core scaling (experimental)**
  - When to use: >1M cells that don't fit in RAM
  - How: `scanpy[dask]`; load `X` via `anndata.experimental.read_elem_lazy(..., chunks=(n_cells_chunk, n_vars))`; `normalize_total`/`log1p` are lazy, `highly_variable_genes`/`pca` compute immediately; `obsm["X_pca"]` needs a manual `.compute()`
  - Why: sparse chunks let you use fewer, larger chunks and skip zero-sums; dask itself poorly supports scipy.sparse — densify per chunk with `map_blocks` if you touch arrays directly

## Key Concepts
- **Pearson-residual hyperparameters**: `theta` (overdispersion; default 100, `np.Inf` = Poisson), `clip` (residuals clipped to ±√n_cells by default; `np.Inf` disables), `chunksize` (RAM-efficient gene selection for large data). Output is **dense** — subset genes first.
- **Cell-cycle scoring**: `sc.tl.score_genes_cell_cycle(adata, s_genes, g2m_genes)` (Tirosh et al. 2015, 97 genes: 43 S + 54 G2M) → `obs["S_score"/"G2M_score"/"phase"]`; `sc.pp.regress_out(adata, ["S_score", "G2M_score"])` removes it. Score on the full dataset (not HVG), log1p first.
- **kNN transformers**: `sc.pp.neighbors(transformer=AnnoyTransformer(n_neighbors))` or `transformer="pynndescent"` via `sklearn-ann`; Annoy is faster than PyNNDescent, brute-force still wins on small data.
- **Marsilea compose model**: `ma.Heatmap(adata, A.X[:, markers])` + `group_rows/cols` + `add_left/top/bottom/right(plotter)` + `add_dendrogram` + `add_legends` + `render()` last; `+` (side-by-side) and `/` (stack) combine boards; `SizedHeatmap`=dotplot, `ClusterBoard`=tracksplot/stacked-violin.
- **`sc.get.aggregate(..., by=A.obs[...], func=["mean","count_nonzero"])`**: the Marsilea workflow aggregates to a cell-type-level AnnData first; fraction = count_nonzero / `n_obs_aggregated`.

## Mental Models
- **Think of Pearson residuals as "variance stabilization without explicit depth normalization"** — the null model *expects* technical variance and you keep only the deviation.
- **Think of PAGA as a coarse-grained UMAP** — clusters are nodes, connectivity is edges; pseudotime/DPT flows along it.
- **Think of dask-scanpy as "lazy until you touch `obs`"** — `X` ops defer, but any `obs`-writing call (HVG) forces compute.
- **Think of Marsilea as "compose plots, don't configure them"** — you assemble plotters on a board instead of passing 30 kwargs.

## Anti-patterns
- **Running `normalize_pearson_residuals` before gene selection** — the output is dense; reduce to HVGs first or you blow memory.
- **Scoring cell cycle on HVG subset** — ~70 of the 97 cell-cycle genes aren't "highly variable"; score on the full matrix.
- **`sc.pl.paga` without a `threshold`** — weak edges clutter the graph; prune by edge-weight outliers.
- **Touching dask-backed `X` directly** — scipy.sparse chunks are poorly supported by dask; go through scanpy/anndata or `map_blocks` densify.
- **Marsilea stats tests as biological evidence** — per-cell tests (Mann-Whitney) inflate significance; pass precomputed p-values with `pvalues=` instead.

## Code Examples

```python
import scanpy as sc, numpy as np

# Pearson residuals pipeline (Lause et al. 2021)
sc.experimental.pp.highly_variable_genes(adata, flavor="pearson_residuals", n_top_genes=2000)
adata = adata[:, adata.var["highly_variable"]]
adata.layers["raw"] = adata.X.copy()
sc.experimental.pp.normalize_pearson_residuals(adata)   # X becomes dense residuals
sc.pp.pca(adata, n_comps=50); sc.pp.neighbors(adata, n_neighbors=10, n_pcs=50)
sc.tl.leiden(adata, flavor="igraph", n_iterations=2, directed=False)

# PAGA + diffusion pseudotime
sc.tl.paga(adata, groups="leiden")
sc.pl.paga(adata, threshold=0.03)
adata.uns["iroot"] = np.flatnonzero(adata.obs["leiden"] == "0")[0]
sc.tl.dpt(adata)                        # -> obs["dpt_pseudotime"]

# Cell cycle: score then regress
sc.tl.score_genes_cell_cycle(adata, s_genes=s_genes, g2m_genes=g2m_genes)
sc.pp.regress_out(adata, ["S_score", "G2M_score"])

# Faster neighbors (Annoy)
from sklearn_ann.kneighbors.annoy import AnnoyTransformer
sc.pp.neighbors(adata, transformer=AnnoyTransformer(15))
```
- **What it demonstrates**: the four advanced paths — residuals, PAGA/DPT, cell cycle, and kNN backends.

## Reference Tables

### Pearson residual functions (`sc.experimental.pp`)

| Function | Role |
|---|---|
| `highly_variable_genes(flavor="pearson_residuals")` | HVG by residual variance |
| `normalize_pearson_residuals` | raw counts → dense residuals |
| `normalize_pearson_residuals_pca` | residuals + PCA in one |
| `recipe_pearson_residuals` | full HVG + residuals + PCA pipeline |

### Trajectory / PAGA API

| Function | Output |
|---|---|
| `tl.paga(groups=...)` | `uns["paga/connectivities"|"connectivities_tree"]` |
| `pl.paga(threshold=...)` | pruned abstracted graph |
| `tl.diffmap` + neighbors(`use_rep="X_diffmap"`) | optional graph denoising |
| `tl.dpt` (after `uns["iroot"]`) | `obs["dpt_pseudotime"]` |
| `tl.draw_graph(init_pos="paga")` | PAGA-initialized embedding |
| `pl.paga_path(path, genes)` | gene expression along a path |

### kNN transformer choice

| Setting | Behavior |
|---|---|
| (default) | brute-force all distances (small data) |
| `transformer="pynndescent"` | approximate (default for large) |
| `transformer=AnnoyTransformer(k)` | Annoy approximate (fastest) |

## Key Takeaways
1. Pearson residuals replace normalize+log1p and fold in HVG; output is dense, so subset genes first and keep `layers["raw"]`.
2. PAGA + DPT give topology and pseudotime; prune with `threshold`, optionally denoise via diffmap.
3. dask-scanpy is lazy for `X` but forces compute on any `obs` write (HVG/PCA); sparse chunks are poorly supported by dask.
4. Cell-cycle scoring needs the full (non-HVG) matrix and log1p; regress with `pp.regress_out(["S_score","G2M_score"])`.
5. Marsilea composes plots from an AnnData via the `A` accessor — call `render()` last, and pass precomputed p-values for stats.

## Connects To
- **Ch 02**: Pearson residuals are the third normalization layer (with log1p and scran) in the book's framework.
- **Ch 04**: PAGA/DPT are the scanpy-native trajectory methods; scVelo is the RNA-velocity extension.
- **Ch 18–20**: `A` accessor, dask arrays, and `read_elem_lazy` are anndata features this chapter builds on.
- **Ch 21 / Ch 22**: `score_genes`, `regress_out`, `sc.get.aggregate` are core-API functions applied here.
