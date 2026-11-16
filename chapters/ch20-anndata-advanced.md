# Chapter 20: anndata — Advanced Containers (dask, awkward, concat, AnnBatch/AnnCollection)

## Core Idea
Beyond a single in-memory AnnData, anndata scales three ways: **dask arrays** (chunked, lazy, cluster-distributable in `X`/`obsm`/`layers`), **awkward arrays** (ragged/nested data in `varm`/`layers`), and **lazy multi-object containers** (`ad.concat` for eager merging, `AnnCollection`/`annbatch.Loader` for zero-copy streaming to ML frameworks). The core decision is *when to materialize*: keep everything lazy until you need a concrete slice.

## Frameworks Introduced

- **Dask-backed AnnData**
  - When to use: matrices larger than RAM, or distributed compute
  - How: put `da.Array` in any of `X`/`obsm`/`varm`/`obsp`/`varp`/`layers`/`uns`/`raw` (`pip install anndata[dask]`); operations build a lazy graph, `.compute()` or `to_memory()` materializes
  - Why: `concat`, views, `copy`, `to_memory`, h5ad/zarr IO all work transparently on dask arrays; `to_memory(copy=False)` materializes in place, `copy=True` duplicates

- **Concatenation = stacking + merging**
  - When to use: combining batches/samples/datasets into one object
  - How: `ad.concat(adatas, axis="obs"|"var", join="inner"|"outer", merge=..., uns_merge=..., label=..., keys=..., index_unique=...)`
  - Why: two orthogonal controls — `join` aligns the *other* axis (intersection vs union), `merge`/`uns_merge` decide which *not-aligned-to-concat-axis* elements (e.g. `varm`, `uns`) survive

- **Lazy multi-object containers (AnnCollection)**
  - When to use: many large/backed `.h5ad`s that must never be fully concatenated in RAM
  - How: `AnnCollection([a, b], join_vars="inner", join_obs=None|"inner"|"outer", join_obsm="inner", label=..., convert=...)`; subset → `AnnCollectionView`; `iterate_axis(n)` yields batches; `to_adata()` returns only `obs`/`obsm` (no `X`)
  - Why: it keeps a *joint index* and copies nothing from `X`/`layers`; conversions happen on subset

- **annbatch.Loader for minibatch streaming**
  - When to use: feeding PyTorch/JAX models from many AnnData without in-memory concat
  - How: `Loader(chunk_size=, preload_nchunks=, batch_size=, shuffle=, preload_to_gpu=).add_adata(dataset)`; streams contiguous runs with on-disk preshuffle
  - Why: `chunk_size=1` gives perfect random sampling for in-memory data; larger `chunk_size` + preshuffled data maximizes speed for on-disk

## Key Concepts
- **DaskArrayView**: slicing a dask-backed AnnData returns a `DaskArrayView`, *not* a dask array — not lazy, but materialized on access; the source object is unchanged.
- **Chunk-size trade-off**: `chunks=[n_obs, 5]` makes per-gene column access cheap (few requests); larger chunks amortize fetch overhead for PCA/row ops. Tune via `write_zarr(chunks=...)` or `read_elem_lazy(..., chunks=...)`.
- **`read_dispatched`/`write_dispatched`**: recursive IO with a `callback(func, key, elem, iospec)` that can lazy-read `array` elements but eagerly read `dataframe`/`csr_matrix`/`awkward-array`, or set per-element chunk/shard specs.
- **awkward arrays** (experimental, ≥0.9): `ak.Array` in `layers`/`obsm`/`varm`/`uns`; e.g. store per-gene ragged transcript lists in `adata.varm["transcripts"]`.
- **merge strategies**: `None` (drop), `"same"` (identical across all), `"unique"` (single possible value), `"first"` (first seen), `"only"` (appears in exactly one). `uns_merge` applies the same rules *recursively* into nested `uns`.

## Mental Models
- **Think of `ad.concat` as pandas.concat (axis) + xarray.merge (compat)** — the two dials are independent.
- **Think of `AnnCollection` as a lazy `ad.concat`** — it computes the joint `obs`/`var` index but defers all data materialization to subset time.
- **Dask makes AnnData a lazy pipeline** — build the graph, then `.compute()` only the slice you actually plot/analyze.

## Anti-patterns
- **`join="outer"` with sparse data and assuming zeros are real** — outer join pads absent variables with **0** for sparse (NaN for dense); these zeros are indistinguishable from true absence.
- **Wrapping `scipy.sparse` inside a dask array** — sparse's inconsistent numpy API breaks dask ops; prefer dask-native arrays.
- **Forgetting `pairwise=True` when you need `obsp`/`varp` after concat** — graphs are dropped by default (padding 0s is rarely meaningful).
- **Concatenating with duplicate obs/var indices and not handling it** — use `index_unique="_"` (or `label`/`keys`) or you silently get non-unique indices.
- **Calling `to_adata()` on `AnnCollection` expecting `X`** — it returns only `obs`+`obsm`; call it on an `AnnCollectionView` to also get `X`/`layers`.

## Code Examples

```python
import anndata as ad, dask.array as da, numpy as np, pandas as pd
from anndata.experimental.multi_files import AnnCollection

# Dask-backed AnnData
adata = ad.AnnData(
    X=da.random.random((8192, 8192), chunks=(4096, 4096)),
    obs=pd.DataFrame({"batch": np.random.choice(["a","b"], 8192)}, index=[f"c{i}" for i in range(8192)]),
    var=pd.DataFrame(index=[f"g{i}" for i in range(8192)]),
)
mem = adata.to_memory(copy=False)          # materialize, no copy

# Concatenation: track source + avoid index collision
merged = ad.concat({"a": a, "b": b}, join="outer", merge="same",
                   label="dataset", index_unique="-", pairwise=False)

# Lazy multi-file: zero-copy joint subset + iterate for IncrementalPCA
ac = AnnCollection([covid, pbmc], join_vars="inner", join_obs="outer",
                   join_obsm="inner", label="dataset", convert={"X": lambda a: a.toarray()})
for batch, idx in ac.iterate_axis(6000):
    ipca.partial_fit(batch.X)
adata = ac.to_adata()                      # obs + obsm only, X is None
```
- **What it demonstrates**: dask init/materialize, labeled concat, and AnnCollection-based streaming without full concat.

## Reference Tables

### `ad.concat` arguments

| Arg | Values | Meaning |
|---|---|---|
| `axis` | `"obs"`/`0`, `"var"`/`1` | concat along cells or genes |
| `join` | `inner`, `outer` | other-axis intersection/union |
| `merge` | `None`/`same`/`unique`/`first`/`only` | how non-concat-axis elements combine |
| `uns_merge` | same strategies | same, applied recursively in `uns` |
| `label` | str | add a categorical column naming source |
| `keys` | list | source names for `label`/`index_unique` |
| `index_unique` | str | delimiter for making indices unique |
| `pairwise` | bool | include `obsp`/`varp` (default False) |
| `fill_value` | scalar | outer-join fill (default 0 sparse / NaN dense) |
| `force_lazy` | bool | dask-concat even when eager possible |

### `annbatch.Loader` key params

| Param | Guidance |
|---|---|
| `chunk_size` | 1 for in-memory perfect shuffle; larger for on-disk (needs preshuffle) |
| `preload_nchunks` | chunks buffered before yielding |
| `batch_size` | 256–8192 cells; larger = faster + more GPU mem |
| `shuffle` | bool |
| `preload_to_gpu` | needs `cupy-cudaxx` extra |
| `return_index` | bool (for writing predictions back) |

## Key Takeaways
1. Dask, awkward, `ad.concat`, `AnnCollection`, and `annbatch.Loader` are five distinct scaling tools — match the tool to whether you need chunked compute, ragged data, eager merge, lazy merge, or model streaming.
2. `ad.concat` = `join` (align other axis) + `merge`/`uns_merge` (which side-elements survive); `join="outer"` pads sparse with **0**, dense with NaN.
3. `AnnCollection` never copies `X`/`layers`; use `convert=` to transform on subset, `iterate_axis(n)` for out-of-core algorithms.
4. `annbatch.Loader` (not the deprecated `AnnLoader`) is the supported minibatch path; install `annbatch[numba]` for sparse.
5. Tune zarr/dask `chunks` to your access pattern — small chunks for gene queries, large for row/PCA ops.

## Connects To
- **Ch 18**: AnnData core slots this chapter scales up.
- **Ch 19**: `read_lazy`/`read_dispatched`/`write_dispatched` and chunking are the IO underpinnings of dask/AnnCollection.
- **Ch 12 / Ch 13**: scVI/totalVI models consume minibatches via these loaders; scverse ML relies on AnnData as the data contract.
- **Ch 24**: Scanpy's own dask integration (`sc.pp`/`sc.tl` with dask) mirrors this pattern.
