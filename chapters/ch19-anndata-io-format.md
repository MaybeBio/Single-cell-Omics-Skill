# Chapter 19: anndata — File Format & IO

## Core Idea
AnnData persists to two hierarchical array stores — HDF5 (`.h5ad`) and zarr — with an identical on-disk layout: a root `anndata` group plus `obs`/`var`/`X`/`layers`/`obsm`/`varm`/`obsp`/`varp`/`uns`. Every element tags itself with `encoding-type` + `encoding-version` attributes, which drive dispatch on read. The practical decisions are: **prefer zarr** (cloud-native, parallel, sharded, async), use `read_lazy`/`read_elem_lazy` for remote/partial access, and pin `compat` when writing for cross-version/cross-language readers.

## Frameworks Introduced

- **Encoding-type dispatch model**
  - When to use: understanding or debugging any on-disk AnnData
  - How: each element (group or array) carries `encoding-type` and `encoding-version`; the reader dispatches on these keys, so the format is self-describing and versioned per-element
  - Why: the root is `encoding-type: "anndata"` v0.1.0; child elements version *their own* layout independently

- **zarr-over-h5ad selection**
  - When to use: choosing a persistence format
  - How: zarr = cloud-native, parallel, async backend, sharded (≥0.13); h5ad = single HDF5 file, serial, not cloud-native. Use zarr for large/remote; h5ad for single-file portability
  - Why: zarr's directory structure enables parallel + partial chunk reads; h5ad's single file is convenient but blocks that

- **Lazy/backed access (read_lazy)**
  - When to use: datasets too big for RAM, or remote stores
  - How: `ad.experimental.read_lazy(store)` returns dask-backed `X` + lazy `obs`/`var` (`Dataset2D` wrapping xarray); `load_annotation_index=True` skips index load; `to_memory()` materializes for scanpy
  - Why: you can subset/filter on metadata without ever loading `X` into memory — a confirmatory analysis on 275k cells can run in seconds

- **Write compatibility profiles**
  - When to use: when the file will be read by older anndata or another language
  - How: pass `compat="0.13"` (or a `WriteConfig`) to every write function; profile = oldest python-anndata version guaranteed to read the file
  - Why: `encoding-version` versions individual elements, `compat` promises *every* element is readable by a target version

## Key Concepts
- **Sparse encoding**: CSR/CSC stored as a group of `data`/`indices`/`indptr` arrays with `shape` attr; two formats only (CSR for row access, CSC for column access — `read_lazy` exposes `X_csc` for gene-subset queries).
- **DataFrame encoding**: `dataframe` v0.2.0, attrs `_index` + `column-order`; each column is its own array; categorical columns split into `categories` + `codes` (code `-1` = missing).
- **Nullable arrays**: pandas nullable int/bool/string → `nullable-*` v0.1.0 group with `values` + boolean `mask`; `"na-value"` records `"NA"` vs `"NaN"` missing semantics.
- **Consolidated metadata (zarr v3)**: written by default; store becomes immutable — to mutate, open with `use_consolidated=False`, edit, then `zarr.consolidate_metadata()` (or delete `.zmetadata`).
- **Sharding (zarr v3 only)**: `anndata.settings.auto_shard_zarr_v3` or `write_dispatched` callback for custom shards; solves "many small files" and improves throughput.
- **zstd vs blosc**: zarr v3 default codec is `zstd`; pass `zarr.codecs.BloscCodec` to `compressor` to restore old behavior.

## Mental Models
- **Think of an `.h5ad`/`.zarr` as a self-describing tree** — the exact same object you'd hold in memory, one directory/group per slot, each node stamped with its type.
- **Use `X_csc` for column (gene) queries, `X_csr` for row (cell) queries** — sparse format determines which axis is cheap to slice.
- **Chunk size is a latency/compute trade-off** — big chunks amortize fetch overhead (good for PCA on dense/CSR), small chunks make single-gene/column reads cheap.

## Anti-patterns
- **Mutating a consolidated zarr store in place** — consolidated metadata goes stale; open unconsolidated and re-consolidate, or the file becomes unreadable/slow.
- **Reading a remote zarr without consolidated metadata** — can't guarantee full readability and is far slower.
- **Forgetting `obs_names`/`var_names` when `load_annotation_index=False`** — those columns live outside xarray coords and get dropped, breaking gene-name subsetting.
- **Indexing lazy data without materializing** — use `.data[()]` / `.compute()` on the dask-backed column before boolean-subsetting the AnnData.

## Code Examples

```python
import anndata as ad, zarr, dask.array as da
from anndata.experimental import read_lazy, read_elem_lazy

# Write (zarr preferred), with a compat profile
adata.write_zarr("x.zarr")
adata.write_h5ad("old.h5ad", compat="0.13")          # readable by 0.13.x+

# Backed (h5ad) vs lazy (zarr, remote-capable)
b = ad.read_h5ad("x.h5ad", backed="r"); b.file.close()
lazy = read_lazy(zarr.open("x.zarr", mode="r"), load_annotation_index=True)
lazy.obs = lazy.obs.to_memory(); lazy.var = lazy.var.to_memory()

# Re-chunk a lazy layer for cheap per-gene (column) access
lazy.layers["X_csc"] = read_elem_lazy(
    zarr.open("x.zarr", mode="r")["layers"]["X_csc"], chunks=(lazy.shape[0], 1)
)
# subset by boolean mask — materialize the column first
mask = lazy.obs["cell_type"].data[()] == "T"
t_cells = lazy[mask, :]

# Mutate a consolidated store safely
g = zarr.open_group("x.zarr", mode="a", use_consolidated=False)
ad.io.write_elem(g, "obs", new_obs, dataset_kwargs=dict(chunks=(250,)))
zarr.consolidate_metadata(g.store)
```
- **What it demonstrates**: write/backed/lazy paths, re-chunking, safe store mutation.

## Reference Tables

### On-disk element encodings (encoding-type)

| encoding-type | version | Structure | Notes |
|---|---|---|---|
| `anndata` (root) | 0.1.0 | group | MUST contain `obs`+`var` |
| `array` (dense) | 0.2.0 | array | native |
| `csr_matrix` / `csc_matrix` | 0.1.0 | group: `data`,`indices`,`indptr` | + `shape` attr |
| `dataframe` | 0.2.0 | group: one array/col | + `_index`, `column-order` |
| `categorical` | 0.2.0 | group: `categories`+`codes` | + `ordered`; code `-1`=NA |
| `string-array` | 0.2.0 | array | VLenUTF8 (zarr) / vlen utf-8 (h5) |
| `string` / `numeric-scalar` | 0.2.0 | 0-d array | scalar in `uns` |
| `nullable-integer/boolean/string-array` | 0.1.0 | group: `values`+`mask` | + optional `na-value` |
| `dict` (mapping) | 0.1.0 | group | recursive (e.g. `uns`) |
| `awkward-array` | 0.1.0 | group of `node*-*` | + `form`,`length`; experimental |
| `accessor` | 0.1.0 | 0-d JSON string | `anndata.acc` refs |

### Write compatibility profiles

| Profile | Released | Effect |
|---|---|---|
| `"0.13"` | 2026-07-07 | default |
| `"0.14"` | unreleased | enables accessor encoding (older can't read) |

### Cross-language IO

| Language | Library |
|---|---|
| R | anndataR (h5ad+zarr, →Seurat/SCE), zellkonverter, MuData, MuDataSeurat |
| Julia | Muon.jl, scVI.jl |
| JS | Vitessce (zarr loaders) |
| Rust | anndata-rs |

## Key Takeaways
1. h5ad and zarr share one self-describing layout; pick zarr for cloud/parallel, h5ad for single-file portability.
2. Use `read_lazy`/`read_elem_lazy` for big or remote data — subset on metadata first, `to_memory()` only the slice you need.
3. Tune chunk size to your access pattern (`X_csc`/small chunks for genes, `X_csr`/big chunks for PCA).
4. Pass `compat="0.13"` when older anndata or another language must read the file.
5. Consolidated zarr stores are immutable — mutate via `use_consolidated=False` + re-consolidate.

## Connects To
- **Ch 18**: the in-memory AnnData schema this format mirrors.
- **Ch 20**: dask arrays, concatenation, AnnBatch/AnnCollection build on lazy/backed IO.
- **Ch 13 / Ch 01**: scanpy/scverse read/write through this format; AnnData is the interchange standard.
