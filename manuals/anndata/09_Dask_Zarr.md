# Dask + Zarr, but Remote!

> Author: Ilan Gold To begin we need to create a dataset on disk to be used with dask in the zarr format. We will edit the chunk_size argument so that we make fetching expression data for groups of c...

# Dask + Zarr, but Remote![#](https://anndata.scverse.org/tutorials/notebooks/%7Bread%2Cwrite%7D_dispatched.html#dask-zarr-but-remote)

**Author**: Ilan Gold

To begin we need to create a dataset on disk to be used with `dask` in the `zarr` format. We will edit the `chunk_size` argument so that we make fetching expression data for groups of cells more efficient i.e., each access-per-gene over a contiguous group of cells (within the `obs` ordering) will be fast and efficient.

```
from __future__ import annotations

import scanpy as sc
import zarr

from anndata.experimental import read_dispatched, read_elem_lazy, write_dispatched
from anndata.io import read_elem
```

```
rel_zarr_path = "data/pbmc3k_processed.zarr"
```

```
adata = sc.datasets.pbmc3k_processed()
adata.write_zarr(f"./{rel_zarr_path}", chunks=[adata.shape[0], 5])
zarr.consolidate_metadata(f"./{rel_zarr_path}")
```

```
<zarr.hierarchy.Group '/'>
```

```
def read_dask(store):
    f = zarr.open(store, mode="r")

    def callback(func, elem_name: str, elem, iospec):
        if iospec.encoding_type in (
            "dataframe",
            "csr_matrix",
            "csc_matrix",
            "awkward-array",
        ):
            # Preventing recursing inside of these types
            return read_elem(elem)
        elif iospec.encoding_type == "array":
            return read_elem_lazy(elem)
        else:
            return func(elem)

    adata = read_dispatched(f, callback=callback)

    return adata
```

Before continuing, go to a shell and run `python3-mhttp.server8080` out of the directory containing this notebook. This will allow you to observe how different requests are handled by a file server. After this, run the next cell to load the data via the server, using dask arrays “over the wire” – note that this functionality is enabled by `dask`’s deep integration with `zarr`, not `hdf5`!

```
adata_dask = read_dask(f"http://127.0.0.1:8080/{rel_zarr_path}")
adata_dask.X
```

| Array | Chunk | Bytes | Shape | Dask graph | Data type |
|-------|-------|-------|-------|------------|-----------|
|  |  |  |  |  |  |
| 18.50 MiB | 51.52 kiB |  |  |  |  |
| (2638, 1838) | (2638, 5) |  |  |  |  |
| 368 chunks in 2 graph layers |  |  |  |  |  |
| float32 numpy.ndarray |  |  |  |  |  |

```
adata_dask.obsm["X_draw_graph_fr"]
```

| Array | Chunk | Bytes | Shape | Dask graph | Data type |
|-------|-------|-------|-------|------------|-----------|
|  |  |  |  |  |  |
| 41.22 kiB | 41.22 kiB |  |  |  |  |
| (2638, 2) | (2638, 2) |  |  |  |  |
| 1 chunks in 2 graph layers |  |  |  |  |  |
| float64 numpy.ndarray |  |  |  |  |  |

Now let’s make some requests – slicing over the `obs` axis should be efficient.

```
adata_dask.X[:, adata.var.index == "C1orf86"].compute()
```

```
array([[-0.4751688 ],
       [-0.68339145],
       [-0.52097213],
       ...,
       [-0.40973732],
       [-0.35466102],
       [-0.42529213]], dtype=float32)
```

Indeed, you should only have one additional request now, which looks something like this:

```
::ffff:127.0.0.1 - - [13/Feb/2023 20:00:36] "GET /data/pbmc3k_processed.zarr/X/0.0 HTTP/1.1" 200 -
```

What about over multiple genes? `adata.var['n_cells']&gt;1000==59` so this should be less than 59 requests (indeed there are)!

```
adata_dask.X[:, adata.var["n_cells"] > 1000].compute()
```

```
array([[ 0.53837276, -0.862139  , -1.1624558 , ...,  0.02576654,
        -0.7214901 , -0.86157244],
       [-0.39546633, -1.4468503 , -0.23953451, ..., -1.8439665 ,
        -0.95835304, -0.04634313],
       [ 1.036884  , -0.82907706,  0.13356175, ..., -0.91740227,
         1.2407869 , -0.95057184],
       ...,
       [ 0.9374183 , -0.63782793,  1.4828881 , ..., -0.74470884,
         1.4084249 ,  1.8403655 ],
       [ 1.4825792 , -0.48758882,  1.2520502 , ..., -0.54854494,
        -0.61547786, -0.68133515],
       [ 1.2934785 ,  1.2127419 ,  1.2300901 , ..., -0.5996045 ,
         1.1535971 , -0.8018701 ]], dtype=float32)
```

Now what if we chunk differently, larger? There should be fewer requests made to the server, although now each request will be larger – a tradeoff that needs to be tailored to each use-case!

```
adata.write_zarr(f"./{rel_zarr_path}", chunks=[adata.shape[0], 25])
zarr.consolidate_metadata(f"./{rel_zarr_path}")
adata_dask = read_dask(f"http://127.0.0.1:8080/{rel_zarr_path}")

adata_dask.X[:, adata.var["n_cells"] > 1000].compute()
```

```
array([[ 0.53837276, -0.862139  , -1.1624558 , ...,  0.02576654,
        -0.7214901 , -0.86157244],
       [-0.39546633, -1.4468503 , -0.23953451, ..., -1.8439665 ,
        -0.95835304, -0.04634313],
       [ 1.036884  , -0.82907706,  0.13356175, ..., -0.91740227,
         1.2407869 , -0.95057184],
       ...,
       [ 0.9374183 , -0.63782793,  1.4828881 , ..., -0.74470884,
         1.4084249 ,  1.8403655 ],
       [ 1.4825792 , -0.48758882,  1.2520502 , ..., -0.54854494,
        -0.61547786, -0.68133515],
       [ 1.2934785 ,  1.2127419 ,  1.2300901 , ..., -0.5996045 ,
         1.1535971 , -0.8018701 ]], dtype=float32)
```

Now what if we had a `layer` that we wanted to chunk in a custom way, e.g. chunked across all cells by gene)? Just use `write_dispatched` as we did with `read_dispatched`!

```
adata.layers["scaled"] = adata.X.copy()
sc.pp.scale(adata, layer="scaled")
```

```
def write_chunked(func, store, k, elem, dataset_kwargs, *, iospec):
    """Write callback that chunks X and layers"""

    def set_chunks(d, chunks=None):
        """Helper function for setting dataset_kwargs. Makes a copy of d."""
        d = dict(d)
        if chunks is not None:
            d["chunks"] = chunks
        else:
            d.pop("chunks", None)
        return d

    if iospec.encoding_type == "array":
        if "layers" in k or k.endswith("X"):
            dataset_kwargs = set_chunks(dataset_kwargs, (adata.shape[0], 25))
        else:
            dataset_kwargs = set_chunks(dataset_kwargs, None)

    func(store, k, elem, dataset_kwargs=dataset_kwargs)


output_zarr_path = "data/pbmc3k_scaled.zarr"
z = zarr.open_group(output_zarr_path)

write_dispatched(z, "/", adata, callback=write_chunked)
zarr.consolidate_metadata(f"./{rel_zarr_path}")
```

```
<zarr.hierarchy.Group '/'>
```

```
adata_dask = read_dask(f"http://127.0.0.1:8080/{output_zarr_path}")
```

```
adata_dask.layers["scaled"]
```

| Array | Chunk | Bytes | Shape | Dask graph | Data type |
|-------|-------|-------|-------|------------|-----------|
|  |  |  |  |  |  |
| 18.50 MiB | 257.62 kiB |  |  |  |  |
| (2638, 1838) | (2638, 25) |  |  |  |  |
| 74 chunks in 2 graph layers |  |  |  |  |  |
| float32 numpy.ndarray |  |  |  |  |  |
