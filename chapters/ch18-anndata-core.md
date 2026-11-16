# Chapter 18: anndata — Core Structure & API

## Core Idea
AnnData is the scverse standard container for annotated matrix data: a single object holding a sparse/dense count matrix `X`, axis-aligned metadata (`obs` for cells, `var` for genes), multi-dimensional embeddings (`obsm`/`varm`), pairwise graphs (`obsp`/`varp`), alternative matrices (`layers`), and free-form config (`uns`). Everything in scanpy/muon/scirpy/squidpy flows through it — knowing its layout and view semantics is prerequisite to all downstream work.

## Frameworks Introduced

- **The AnnData 10-slot schema**
  - When to use: any time you touch a single-cell object
  - How: `X` (n_obs × n_vars), `obs`/`var` (pandas DataFrames, row/col metadata), `obsm`/`varm` (key→n_obs×d / n_vars×d aligned matrices, e.g. `X_umap`, `PCs`), `obsp`/`varp` (key→square pairwise graphs, e.g. `connectivities`, `distances`), `layers` (key→matrices same shape as `X`), `uns` (OrderedDict, arbitrary), `raw` (pre-normalization snapshot)
  - Why: each slot has a contract — `obsm` rows == `n_obs`, `varm` rows == `n_vars`, `obsp`/`varp` are square

- **Views vs copies (copy-on-modify)**
  - When to use: subsetting (`adata[obs_subset, var_subset]`) or per-batch slicing
  - How: subsetting always returns a *view* (zero copy); reading from a view is lazy; writing any attribute (except `X`) triggers an automatic `.copy()` that materializes a real object
  - Why: `adata[...]` is near-free, so you can slice liberally without memory blow-up — but a view that is modified silently detaches

- **String vs integer indexing (loc/iloc contract)**
  - When to use: every slice
  - How: string/boolean → label-based (`loc`-like); integer → positional (`iloc`-like). AnnData always coerces indices to strings
  - Why: removes the classic pandas numeric-index ambiguity

## Key Concepts
- **Backed mode**: `ad.read_h5ad(path, backed="r")` keeps data on disk, loads on access; `adata.isbacked`, `adata.filename`, `adata.file` (must `.close()` in read-only mode). Equivalent to constructing from a `zarr` group.
- **`read_lazy`**: `ad.experimental.read_lazy(path)` returns dask-backed `X` plus lazy `obs`/`var` (`Dataset2D`) — call `adata.obs = adata.obs.to_memory()` before scanpy use.
- **`anndata.acc` accessors**: `A` global accessor builds `AdRef` references (`A.X[:, "gene-3"]`, `A.obsm["pca"][:, 0]`); `A.varm["PCs"][:, 30] in adata` checks presence, `adata[ref]` extracts. JSON round-trip via `A.to_json` / `A.from_json`.
- **Categoricals**: use `pd.Categorical` for `obs`/`var` columns (memory + speed); `adata.strings_to_categoricals()` converts all strings.
- **zarr vs h5ad**: zarr is cloud-native, parallel, async-capable (default sharded since 0.13); h5ad is a single HDF5 file, not cloud-native, serial. Prefer zarr for large/remote data.

## Mental Models
- **Think of AnnData as "a DataFrame where cells are rows and genes are columns, plus structured side-channels"** — `obs` is the cell metadata table, `var` the gene table, `obsm` the per-cell embeddings drawer.
- **Use `layers` to preserve raw counts** — keep `X` as the working matrix and stash `layers["counts"]` before normalizing; downstream methods (scVI, DESeq2) need raw.
- **Use `.copy()` only when you must detach** — views are cheap and usually what you want; force a real object only before mutating a slice you'll pass elsewhere.

## Anti-patterns
- **Importing `anndata._core` / `anndata._io`** — internal, undocumented, unstable; use the public API only.
- **Forgetting to close a backed h5ad handle** — `read_h5ad(backed="r")` holds an open `h5py.File`; call `adata.file.close()` (or prefer lazy/zarr construction).
- **Treating `.obsm` values as plottable columns** — scanpy plots `obs` columns, not `obsm` columns; store per-cell labels in `obs`.
- **Mutating a view and expecting the parent to change** — copy-on-modify detaches it; read `is_view`/`isview` if unsure.

## Code Examples

```python
import anndata as ad
import numpy as np
from scipy.sparse import csr_matrix

# Init: sparse counts, categorical cell_type
adata = ad.AnnData(csr_matrix(np.random.poisson(1, (100, 2000)), dtype=np.float32))
adata.obs_names = [f"Cell_{i}" for i in range(adata.n_obs)]
adata.var_names = [f"Gene_{i}" for i in range(adata.n_vars)]
adata.obs["cell_type"] = pd.Categorical(np.random.choice(["B", "T", "Mono"], adata.n_obs))

# Aligned matrices + layers + uns
adata.obsm["X_umap"] = np.random.normal(size=(adata.n_obs, 2))
adata.layers["log"] = np.log1p(adata.X)
adata.uns["params"] = {"n_pcs": 30}

# Subsetting -> view (string=loc, int=iloc); .copy() to materialize
bdata = adata[adata.obs.cell_type == "B"]            # view
sub = adata[:5, ["Gene_1", "Gene_3"]].copy()          # real object

# Backed vs lazy
b = ad.read_h5ad("x.h5ad", backed="r"); b.file.close()
lazy = ad.experimental.read_lazy("x.zarr")
lazy.obs = lazy.obs.to_memory(); lazy.var = lazy.var.to_memory()

# Accessors (anndata.acc)
from anndata.acc import A
A.varm["PCs"][:, 30] in adata          # presence check
adata[A.obsm["X_umap"][:, 0]]          # extract referenced vector
```
- **What it demonstrates**: the full lifecycle — construct → annotate → subset → persist/backed → accessor introspection.

## Reference Tables

### AnnData attribute contract

| Slot | Type | Shape / constraint |
|---|---|---|
| `X` | sparse/dense array | n_obs × n_vars |
| `obs` / `var` | pd.DataFrame | len n_obs / n_vars; index == `obs_names`/`var_names` |
| `obsm` / `varm` | key→array | row length == n_obs / n_vars |
| `obsp` / `varp` | key→array | n_obs×n_obs / n_vars×n_vars square |
| `layers` | key→array | each n_obs × n_vars |
| `uns` | OrderedDict | arbitrary |
| `raw` | AnnData | pre-normalization `X`+`var` |

### Key methods

| Method | Purpose |
|---|---|
| `copy(filename=None)` | full copy, optionally to disk |
| `to_df(layer=None)` | shallow DataFrame from `X`/layer |
| `to_memory()` | load backed/lazy arrays into RAM |
| `chunk_X(select, replace)` / `chunked_X(chunk_size)` | chunked access to `X` |
| `obs_vector(k, layer)` / `var_vector(k, layer)` | 1-D values from `X`/layer/`obs` |
| `write_h5ad` / `write_zarr` / `write` / `write_loom` / `write_csvs` | persistence |
| `transpose()` | swap obs/var |
| `strings_to_categoricals` / `rename_categories` | dtype hygiene |
| `obs_names_make_unique` / `var_names_make_unique` | deduplicate indices |

## Key Takeaways
1. AnnData = `X` + `obs`/`var` + `obsm`/`varm` + `obsp`/`varp` + `layers` + `uns` + `raw`; each slot has a fixed alignment contract.
2. Subsetting returns a cheap **view**; mutating it copy-on-modify detaches — call `.copy()` to materialize.
3. String index = label-based, integer = positional; indices are always strings.
4. Keep raw counts in `layers["counts"]`; use `pd.Categorical` for metadata columns.
5. Prefer **zarr** over h5ad for large/remote data; use backed mode or `read_lazy` for on-disk access.

## Connects To
- **Ch 01**: AnnData/MuData introduced as the book's data structures — this chapter is the full reference.
- **Ch 02**: normalization writes to `layers`/`obsm` (log1p, PCA, UMAP all live in these slots).
- **Ch 13**: scverse tool ecosystem assumes `adata` as the interchange format.
- **Ch 19**: file-format spec and IO (h5ad/zarr on-disk layout).
- **Ch 20**: lazy/dask arrays, concatenation, AnnBatch/AnnCollection extend this core.
