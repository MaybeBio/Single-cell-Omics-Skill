# Usage Principles

> Import Scanpy as: Workflow: The typical workflow consists of subsequent calls of data analysis tools in sc.tl, e.g.: where adata is an AnnData object. Each of these calls adds annotation to an expr...

# Usage Principles[#](https://scanpy.scverse.org/en/stable/usage-principles.html#usage-principles)

Import Scanpy as:

```
import scanpy as sc
```

## Workflow[#](https://scanpy.scverse.org/en/stable/usage-principles.html#workflow)

The typical workflow consists of subsequent calls of data analysis tools in `sc.tl`, e.g.:

```
sc.tl.umap(adata, **tool_params)  # embed a neighborhood graph of the data using UMAP
```

where `adata` is an [`AnnData`](https://anndata.scverse.org/en/stable/generated/anndata.AnnData.html#anndata.AnnData) object. Each of these calls adds annotation to an expression matrix *X*, which stores *n_obs* observations (cells) of *n_vars* variables (genes). For each tool, there typically is an associated plotting function in `sc.pl`:

```
sc.pl.umap(adata, **plotting_params)
```

If you pass `show=False`, a [`Axes`](https://matplotlib.org/stable/api/_as_gen/matplotlib.axes.Axes.html#matplotlib.axes.Axes) instance is returned and you have all of matplotlib’s detailed configuration possibilities.

To facilitate writing memory-efficient pipelines, by default, Scanpy tools operate *inplace* on `adata` and return `None` – this also allows to easily transition to [out-of-memory pipelines](https://falexwolf.de/blog/171223_AnnData_indexing_views_HDF5-backing/). If you want to return a copy of the [`AnnData`](https://anndata.scverse.org/en/stable/generated/anndata.AnnData.html#anndata.AnnData) object and leave the passed `adata` unchanged, pass `copy=True` or `inplace=False`.

## AnnData[#](https://scanpy.scverse.org/en/stable/usage-principles.html#anndata)

Scanpy is based on [`anndata`](https://anndata.scverse.org/en/stable/api.html#module-anndata), which provides the [`AnnData`](https://anndata.scverse.org/en/stable/generated/anndata.AnnData.html#anndata.AnnData) class.

[![https://falexwolf.de/img/scanpy/anndata.svg](https://falexwolf.de/img/scanpy/anndata.svg)](https://falexwolf.de/img/scanpy/anndata.svg)

At the most basic level, an [`AnnData`](https://anndata.scverse.org/en/stable/generated/anndata.AnnData.html#anndata.AnnData) object `adata` stores a data matrix `adata.X`, annotation of observations `adata.obs` and variables `adata.var` as `pd.DataFrame` and unstructured annotation `adata.uns` as `dict`. Names of observations and variables can be accessed via `adata.obs_names` and `adata.var_names`, respectively. [`AnnData`](https://anndata.scverse.org/en/stable/generated/anndata.AnnData.html#anndata.AnnData) objects can be sliced like dataframes, for example, `adata_subset=adata[:,list_of_gene_names]`. For more, see this [blog post](https://falexwolf.de/blog/171223_AnnData_indexing_views_HDF5-backing/).

To read a data file to an [`AnnData`](https://anndata.scverse.org/en/stable/generated/anndata.AnnData.html#anndata.AnnData) object, call:

```
adata = sc.read_h5ad(filename).  # or sc.read_zarr(filename)
```

to initialize an [`AnnData`](https://anndata.scverse.org/en/stable/generated/anndata.AnnData.html#anndata.AnnData) object. Possibly add further annotation using, e.g., `pd.read_csv`:

```
import pandas as pd

anno = pd.read_csv(filename_sample_annotation)
# annotations of types pandas.Categorical and float
adata.obs[["cell_groups", "time"]] = anno[["cell_groups", "time"]]
# alternatively, you could also set the whole dataframe
# adata.obs = anno
```

To write, use:

```
adata.write_h5ad(filename)
adata.write_zarr(filename)
adata.write_csvs(filename)
adata.write_loom(filename)
```
