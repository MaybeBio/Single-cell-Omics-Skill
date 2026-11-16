# Chapter 21: Scanpy — Core Workflow (pp / tl / pl)

## Core Idea
Scanpy is the scverse reference toolkit for single-cell gene expression, organized into three modules on top of AnnData: **`sc.pp`** (preprocessing — transforms the data matrix), **`sc.tl`** (tools — compute interpretable annotations like clusters/embeddings/markers), and **`sc.pl`** (plotting — mirrors `tl.*` one-to-one). Functions mutate `adata` in place and return `None`; results land in `adata.obs`/`obsm`/`uns`/`obsp`. The canonical pipeline (filter → normalize → log1p → HVG → PCA → neighbors → UMAP → Leiden → rank_genes_groups) is ~10 calls.

## Frameworks Introduced

- **pp / tl / pl separation**
  - When to use: every scanpy script
  - How: `sc.pp` = data-matrix transformations (filter, normalize, log1p, scale, HVG, PCA); `sc.tl` = tools that add a named annotation (umap, leiden, dpt, paga, rank_genes_groups); `sc.pl` = visualization with a function matching each `tl` function
  - Why: the division tells you *where a result lives* — preprocessing reshapes `X`, tools annotate `obs`/`obsm`/`uns`, plotting reads those annotations

- **In-place, annotation-accumulating convention**
  - When to use: writing memory-efficient pipelines
  - How: `sc.tl.umap(adata, **params)` mutates `adata` and returns `None`; pass `copy=True` (or `inplace=False`) to return a copy and leave `adata` unchanged; `show=False` returns the matplotlib `Axes`
  - Why: in-place mutation enables out-of-memory/backed pipelines and avoids duplicating large objects

- **Standard 10-call pipeline**
  - When to use: any new scRNA-seq dataset
  - How: `filter_cells(min_genes)` → `filter_genes(min_cells)` → `normalize_total` → `log1p` → `highly_variable_genes(n_top_genes)` → `pca` → `neighbors` → `umap` → `leiden` → `rank_genes_groups(adata, "leiden")`
  - Why: this ordering is load-bearing — neighbors consumes PCA (`use_rep`), UMAP/Leiden consume the neighbor graph, rank_genes_groups consumes the cluster labels

## Key Concepts
- **Module index**: `sc.pp` preprocessing (filter_cells/filter_genes, normalize_total, log1p, scale, regress_out, highly_variable_genes, pca, harmony_integrate, bbknn, scrublet, neighbors); `sc.tl` tools (umap, tsne, diffmap, draw_graph, leiden, dpt, paga, dendrogram, rank_genes_groups, score_genes_cell_cycle, marker_gene_overlap); `sc.pl` plotting (heatmap, dotplot, violin, stacked_violin, matrixplot, tracksplot, rank_genes_groups_*, paga, embedding).
- **Leiden requires igraph**: install `scanpy[leiden]` (igraph + leidenalg); conda needs `python-igraph leidenalg` added manually. Use `flavor="igraph"`.
- **Two plotting backends**: new HoloViews backend + legacy matplotlib; `pl.ranking` replaces `pca_loadings`/`pca_variance_ratio` in the HoloViews backend; legacy-only functions include `pl.dendrogram`, `pl.embedding`.
- **Output types**: multi-output functions return a `...Result` object when `out=True`, or write to a caller-supplied `...Out` mapping via `out=` (e.g. `pp.PcaResult`/`pp.PcaOut`, `pp.NeighborsResult`/`NeighborsOut`).
- **GPU**: `rapids-singlecell` mirrors the scanpy API with order-of-magnitude speedups on preprocessing/neighbors/embedding/clustering.
- **dask compatibility** (experimental): many `sc.pp`/`sc.tl` functions accept dask-backed AnnData for >RAM datasets.

## Mental Models
- **Think of scanpy as "functions that annotate an AnnData, not return values"** — always check `adata.obs`/`adata.obsm`/`adata.uns`/`adata.obsp` for the output.
- **`sc.tl` = compute, `sc.pl` = draw the same thing** — if you ran `sc.tl.X`, the plot is almost always `sc.pl.X`.
- **Preprocessing before tools, PCA before neighbors before embeddings** — the pipeline is a DAG; neighbors is the choke point everything downstream depends on.

## Anti-patterns
- **`sudo pip install`** — never; use a venv or `pip install --user`.
- **Expecting scanpy functions to return results** — they return `None` by default; read the result from `adata`, or pass `copy=True`.
- **Forgetting `scanpy[leiden]` and then hitting `ImportError` on `sc.tl.leiden`** — igraph/leidenalg are extras, not core deps.
- **Treating `sc.pl` outputs as saved** — pass `save=".png"` (or set `settings.autosave`); in non-interactive scripts set `settings.autoshow=False` and `show=False`.
- **Importing internal modules (`scanpy.logging`)** — undocumented, unstable; use the public API.

## Code Examples

```python
import scanpy as sc
adata = sc.datasets.pbmc3k()

# canonical pipeline (each call mutates adata in place)
sc.pp.filter_cells(adata, min_genes=100)
sc.pp.filter_genes(adata, min_cells=3)
sc.pp.normalize_total(adata)            # default target_sum=1e4
sc.pp.log1p(adata)
sc.pp.highly_variable_genes(adata, n_top_genes=2000)
sc.tl.pca(adata)                        # -> adata.obsm['X_pca'], varm['PCs'], uns['pca']
sc.pp.neighbors(adata)                  # -> adata.obsp['distances'|'connectivities']
sc.tl.umap(adata)                       # -> adata.obsm['X_umap']
sc.tl.leiden(adata, flavor="igraph", n_iterations=2)   # -> adata.obs['leiden']
sc.tl.rank_genes_groups(adata, "leiden", method="wilcoxon")  # -> adata.uns['rank_genes_groups']

# plot: each mirrors a tl/pp function
sc.pl.umap(adata, color="leiden", save=".png")   # show=False returns Axes
sc.pl.rank_genes_groups_dotplot(adata, n_genes=3)
```
- **What it demonstrates**: the full pipeline and *where each result lands* in the AnnData slots.

## Reference Tables

### `sc.pp` core functions

| Function | Result location |
|---|---|
| `filter_cells` / `filter_genes` | subsets `adata` (in place) |
| `normalize_total` | `X` (per-cell sum normalization) |
| `log1p` | `X` (log1p transform) |
| `scale` | `X` (zero-mean unit-variance) |
| `highly_variable_genes` | `var["highly_variable"]` + `uns["hvg"]` |
| `pca` | `obsm["X_pca"]`, `varm["PCs"]`, `uns["pca"]` |
| `regress_out` | `X` (remove covariates) |
| `harmony_integrate` / `bbknn` | `obsm["X_pca_harmony"]` / graph-only |
| `neighbors` | `obsp["distances"|"connectivities"]`, `uns["neighbors"]` |
| `scrublet` | `obs["predicted_doublet"]`, `uns["scrublet"]` |

### `sc.tl` / `sc.pl` pairs (subset)

| Compute (`sc.tl`) | Plot (`sc.pl`) | Result |
|---|---|---|
| `umap` / `tsne` / `diffmap` / `draw_graph` | `umap` / `tsne` / `diffmap` / `draw_graph` | `obsm["X_*"]` |
| `leiden` / `louvain` | `umap(color=...)` | `obs["leiden"/"louvain"]` |
| `paga` | `paga`, `paga_path`, `paga_compare` | `uns["paga"]` |
| `dpt` | `dpt_timeseries` | `obs["dpt_pseudotime"]` |
| `rank_genes_groups` | `rank_genes_groups_dotplot/violin/heatmap/matrixplot/...` | `uns["rank_genes_groups"]` |
| `score_genes_cell_cycle` | — | `obs["S_score"/"G2M_score"/"phase"]` |
| `dendrogram` | `dendrogram` | `uns["dendrogram_*"]` |

## Key Takeaways
1. `sc.pp` reshapes the matrix, `sc.tl` annotates it, `sc.pl` plots it — results always land in `adata`, not return values.
2. The 10-call pipeline order is fixed: filter → normalize → log1p → HVG → PCA → neighbors → UMAP → Leiden → rank_genes_groups.
3. Install `scanpy[leiden]` for `sc.tl.leiden`; never `sudo pip`.
4. `neighbors` is the downstream choke point — embeddings, clustering, and PAGA all consume its graph.
5. Use `rapids-singlecell` for GPU scale; dask-backed scanpy is experimental.

## Connects To
- **Ch 18–20**: AnnData is the substrate — every `sc.pp`/`sc.tl` result lands in a specific AnnData slot.
- **Ch 02 / Ch 03**: the *why* behind QC, normalization, clustering that this chapter maps to the *how* (exact functions).
- **Ch 22**: full API reference for the modules summarized here.
- **Ch 23**: clustering tutorial + ingest reference mapping in depth.
- **Ch 24**: pearson residuals, PAGA, cell cycle, dask, and marsilea plotting extend this core.
