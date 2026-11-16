# API

> The central class: Its attributes are reflected in the Accessors and paths API ( anndata.acc). If you want to write e.g. plotting or validation code for anndata objects, look there! Combining: Comb...

# API[#](https://anndata.scverse.org/api.html#module-anndata)

The central class:

|  |
|--|

Its attributes are reflected in the [Accessors and paths](https://anndata.scverse.org/accessors.html) API (`anndata.acc`). If you want to write e.g. plotting or validation code for anndata objects, look there!

## Combining[#](https://anndata.scverse.org/api.html#combining-api)

Combining [`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData) objects. See also the section on concatenation.

|  |
|--|

## Reading[#](https://anndata.scverse.org/api.html#reading-api)

Reading anndata’s native formats `.h5ad` and `zarr`.

| io.read_zarr(store) | Read from a hierarchical Zarr array store. |
|---------------------|--------------------------------------------|

Reading individual portions ([`obs`](https://anndata.scverse.org/generated/anndata.AnnData.obs.html#anndata.AnnData.obs), [`varm`](https://anndata.scverse.org/generated/anndata.AnnData.varm.html#anndata.AnnData.varm) etc.) of the [`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData) object.

| io.sparse_dataset(group, *[, ...]) | Generates a backed mode-compatible sparse dataset class. |
|------------------------------------|----------------------------------------------------------|

Reading file formats that cannot represent all aspects of [`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData) objects.

Tip

You might have more success by assembling the [`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData) object yourself from the individual parts.

| io.read_excel(filename, sheet[, dtype]) | Read.xlsx(Excel) file. |
|-----------------------------------------|------------------------|
| io.read_hdf(filename, key) | Read.h5(hdf5) file. |
| io.read_mtx(filename[, dtype]) | Read.mtxfile. |
| io.read_text(filename[, delimiter, ...]) | Read.txt,.tab,.data(text) file. |
| io.read_umi_tools(filename[, dtype]) | Read a gzipped condensed count matrix from umi_tools. |

## Writing[#](https://anndata.scverse.org/api.html#writing-api)

Writing a complete [`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData) object to disk in anndata’s native formats `.h5ad` and `zarr`. (These functions are also exported as [`io.write_h5ad()`](https://anndata.scverse.org/generated/anndata.io.write_h5ad.html#anndata.io.write_h5ad) and [`io.write_zarr()`](https://anndata.scverse.org/generated/anndata.io.write_zarr.html#anndata.io.write_zarr).)

| AnnData.write_zarr(store, *[, chunks, ...]) | Write a hierarchical Zarr array store. |
|---------------------------------------------|----------------------------------------|
| AnnData.unwriteable(*[, store_type, compat]) | Whether or not anAnnDataobject can be written to disk for a given store type. |

Writing individual portions ([`obs`](https://anndata.scverse.org/generated/anndata.AnnData.obs.html#anndata.AnnData.obs), [`varm`](https://anndata.scverse.org/generated/anndata.AnnData.varm.html#anndata.AnnData.varm) etc.) of the [`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData) object.

|  |
|--|

All of these take a `compat` argument, see [Write compatibility profiles](https://anndata.scverse.org/write-compat.html) for what the compatibility profiles do.

| WriteConfig(*[, accessor]) | Fine-grained write behavior: one flag per on-disk feature. |
|----------------------------|------------------------------------------------------------|

|  |
|--|

Writing formats that cannot represent all aspects of [`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData) objects.

|  |
|--|

## Experimental API[#](https://anndata.scverse.org/api.html#experimental-api)

Warning

APIs in the experimental module are currently in development and subject to change at any time.

Two classes for working with batched access to collections of many [`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData) objects or `.h5ad` files. In particular, for pytorch-based models.

| experimental.AnnLoader(*args, **kwargs) | PyTorch DataLoader for AnnData objects. |
|-----------------------------------------|-----------------------------------------|

classexperimental.AnnCollectionView[#](https://anndata.scverse.org/api.html#anndata.experimental.AnnCollectionView)

A view of [`experimental.AnnCollection`](https://anndata.scverse.org/generated/anndata.experimental.AnnCollection.html#anndata.experimental.AnnCollection) with most of the same functions and attributes.

Note

`AnnLoader` is deprecated and will be removed in 0.14. Please see [`annbatch.Loader`](https://annbatch.readthedocs.io/en/stable/generated/annbatch.Loader.html#annbatch.Loader) for the replacement.

Out of core concatenation

|  |
|--|

Low level methods for reading and writing elements of an [`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData) object to a store:

| experimental.read_lazy(store, *[, ...]) | Lazily read in on-disk/in-cloud AnnData stores, includingobsandvar. |
|-----------------------------------------|---------------------------------------------------------------------|

Utilities for customizing the IO process:

| experimental.write_dispatched(store, key, ...) | Write elem to store, recursively calling callback at each sub-element. |
|------------------------------------------------|------------------------------------------------------------------------|

Types used by the former:

| experimental.Read |  |
|-------------------|--|
| experimental.Write |  |
| experimental.ReadCallback |  |
| experimental.WriteCallback |  |
| experimental.backed.MaskedArray | A wrapper class meant to enable working with lazy masked data. |
| experimental.backed.CategoricalArray | A wrapper class meant to enable working with lazy categorical data. |
| experimental.backed.Dataset2D | A wrapper class meant to enable working with lazy dataframe data according toAnnData's internal API. |
| experimental.Dataset2DIlocIndexer |  |

|  |
|--|

## Extensions[#](https://anndata.scverse.org/api.html#extensions-api)

|  |
|--|

Types used by the former:

| types.SupportsArrayApi(*args, **kwargs) |  |
|-----------------------------------------|--|

[`scverse_misc.ExtensionNamespace`](https://scverse-misc.readthedocs.io/stable/generated/scverse_misc.ExtensionNamespace.html#scverse_misc.ExtensionNamespace)

## Errors and warnings[#](https://anndata.scverse.org/api.html#errors-api)

|  |
|--|

## Settings[#](https://anndata.scverse.org/api.html#settings-api)

| settings.override(*[, ...]) | Provides local override via keyword arguments as a context manager. |
|-----------------------------|---------------------------------------------------------------------|
| settings.reset(*names) | Reset passed settings to their default values. |

## Custom Types/Classes for Readable/Writeable Elements[#](https://anndata.scverse.org/api.html#types-api)

| abc.CSCDataset() | On disk CSC sparse matrix. |
|------------------|----------------------------|

| typing.Index | Index anAnnDataobject can be sliced with. |
|--------------|-------------------------------------------|
| typing.InMemoryArray | An Array that is possibly stored in Memory (Dask Arrays are possibly stored on disk). |
| typing.AlignedArray | Every array-like data structure anndata accepts, i.e. the full set of array types that can be stored inX,layers, and{obs,var}{m,p}. |
| typing.Storable | A serializable object, excludinganndata.AnnDataobjects i.e., something that can be stored inunsorobsm. |
| typing.RWAble | A superset ofanndata.typing.Storable(i.e., includinganndata.AnnData) which is everything can be read/written byanndata.io.read_elem()andanndata.io.write_elem(). |
