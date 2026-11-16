# anndata.AnnData

# anndata.AnnData[#](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata-anndata)

classanndata.AnnData(*X=None*, *obs=None*, *var=None*, *uns=None*, ***, *obsm=None*, *varm=None*, *layers=None*, *raw=None*, *shape=None*, *filename=None*, *filemode=None*, *asview=False*, *obsp=None*, *varp=None*, *oidx=None*, *vidx=None*)[#](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData)

An annotated data matrix.

[![../_images/anndata_schema.svg](https://anndata.scverse.org/_images/anndata_schema.svg)](https://anndata.scverse.org/_images/anndata_schema.svg)

[`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData) stores a data matrix [`X`](https://anndata.scverse.org/generated/anndata.AnnData.X.html#anndata.AnnData.X) together with annotations of observations [`obs`](https://anndata.scverse.org/generated/anndata.AnnData.obs.html#anndata.AnnData.obs) ([`obsm`](https://anndata.scverse.org/generated/anndata.AnnData.obsm.html#anndata.AnnData.obsm), [`obsp`](https://anndata.scverse.org/generated/anndata.AnnData.obsp.html#anndata.AnnData.obsp)), variables [`var`](https://anndata.scverse.org/generated/anndata.AnnData.var.html#anndata.AnnData.var) ([`varm`](https://anndata.scverse.org/generated/anndata.AnnData.varm.html#anndata.AnnData.varm), [`varp`](https://anndata.scverse.org/generated/anndata.AnnData.varp.html#anndata.AnnData.varp)), and unstructured annotations [`uns`](https://anndata.scverse.org/generated/anndata.AnnData.uns.html#anndata.AnnData.uns).

An [`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData) object `adata` can be sliced like a [`DataFrame`](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.html#pandas.DataFrame), for instance `adata_subset=adata[:,list_of_variable_names]`. [`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData)’s basic structure is similar to R’s ExpressionSet [[Huber15]](https://anndata.scverse.org/references.html#huber15). If setting an `.h5ad`-formatted HDF5 backing file `.filename`, data remains on the disk but is automatically loaded into memory if needed.

Parameters:**X**[`AlignedArray`](https://anndata.scverse.org/typing.html#anndata.typing.AlignedArray) | [`None`](https://docs.python.org/3/builtins/constants.html#None)

A #observations × #variables data matrix. A view of the data is used if the data type matches, otherwise, a copy is made.

**obs**[`Mapping`](https://docs.python.org/3/library/collections.abc.html#collections.abc.Mapping)[[`str`](https://docs.python.org/3/builtins/stdtypes.html#str), [`Iterable`](https://docs.python.org/3/library/collections.abc.html#collections.abc.Iterable)[[`Any`](https://docs.python.org/3/library/typing.html#typing.Any)]] | [`DataFrame`](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.html#pandas.DataFrame) | [`Series`](https://pandas.pydata.org/docs/reference/api/pandas.Series.html#pandas.Series) | [`Index`](https://pandas.pydata.org/docs/reference/api/pandas.Index.html#pandas.Index) | [`Dataset`](https://docs.xarray.dev/en/stable/generated/xarray.Dataset.html#xarray.Dataset) | [`Dataset2D`](https://anndata.scverse.org/generated/anndata.experimental.backed.Dataset2D.html#anndata.experimental.backed.Dataset2D) | [`None`](https://docs.python.org/3/builtins/constants.html#None)

Key-indexed one-dimensional observations annotation of length #observations.

**var**[`Mapping`](https://docs.python.org/3/library/collections.abc.html#collections.abc.Mapping)[[`str`](https://docs.python.org/3/builtins/stdtypes.html#str), [`Iterable`](https://docs.python.org/3/library/collections.abc.html#collections.abc.Iterable)[[`Any`](https://docs.python.org/3/library/typing.html#typing.Any)]] | [`DataFrame`](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.html#pandas.DataFrame) | [`Series`](https://pandas.pydata.org/docs/reference/api/pandas.Series.html#pandas.Series) | [`Index`](https://pandas.pydata.org/docs/reference/api/pandas.Index.html#pandas.Index) | [`Dataset`](https://docs.xarray.dev/en/stable/generated/xarray.Dataset.html#xarray.Dataset) | [`Dataset2D`](https://anndata.scverse.org/generated/anndata.experimental.backed.Dataset2D.html#anndata.experimental.backed.Dataset2D) | [`None`](https://docs.python.org/3/builtins/constants.html#None)

Key-indexed one-dimensional variables annotation of length #variables.

**uns**[`Mapping`](https://docs.python.org/3/library/collections.abc.html#collections.abc.Mapping)[[`str`](https://docs.python.org/3/builtins/stdtypes.html#str), [`Any`](https://docs.python.org/3/library/typing.html#typing.Any)] | [`None`](https://docs.python.org/3/builtins/constants.html#None)

Key-indexed unstructured annotation.

**obsm**[`Mapping`](https://docs.python.org/3/library/collections.abc.html#collections.abc.Mapping)[[`str`](https://docs.python.org/3/builtins/stdtypes.html#str), [`AlignedArray`](https://anndata.scverse.org/typing.html#anndata.typing.AlignedArray)] | [`NDArray`](https://numpy.org/doc/stable/reference/typing.html#numpy.typing.NDArray)[[`void`](https://numpy.org/doc/stable/reference/arrays.scalars.html#numpy.void)] | [`None`](https://docs.python.org/3/builtins/constants.html#None)

Key-indexed multi-dimensional observations annotation of length #observations. If passing a [`ndarray`](https://numpy.org/doc/stable/reference/generated/numpy.ndarray.html#numpy.ndarray), it needs to have a structured datatype.

**varm**[`Mapping`](https://docs.python.org/3/library/collections.abc.html#collections.abc.Mapping)[[`str`](https://docs.python.org/3/builtins/stdtypes.html#str), [`AlignedArray`](https://anndata.scverse.org/typing.html#anndata.typing.AlignedArray)] | [`NDArray`](https://numpy.org/doc/stable/reference/typing.html#numpy.typing.NDArray)[[`void`](https://numpy.org/doc/stable/reference/arrays.scalars.html#numpy.void)] | [`None`](https://docs.python.org/3/builtins/constants.html#None)

Key-indexed multi-dimensional variables annotation of length #variables. If passing a [`ndarray`](https://numpy.org/doc/stable/reference/generated/numpy.ndarray.html#numpy.ndarray), it needs to have a structured datatype.

**layers**[`Mapping`](https://docs.python.org/3/library/collections.abc.html#collections.abc.Mapping)[[`str`](https://docs.python.org/3/builtins/stdtypes.html#str), [`AlignedArray`](https://anndata.scverse.org/typing.html#anndata.typing.AlignedArray)] | [`Mapping`](https://docs.python.org/3/library/collections.abc.html#collections.abc.Mapping)[[`str`](https://docs.python.org/3/builtins/stdtypes.html#str) | [`None`](https://docs.python.org/3/builtins/constants.html#None), [`AlignedArray`](https://anndata.scverse.org/typing.html#anndata.typing.AlignedArray)] | [`None`](https://docs.python.org/3/builtins/constants.html#None)

Key-indexed multi-dimensional arrays aligned to dimensions of `X`.

**shape**[`tuple`](https://docs.python.org/3/builtins/stdtypes.html#tuple)[[`int`](https://docs.python.org/3/builtins/functions.html#int), [`int`](https://docs.python.org/3/builtins/functions.html#int)] | [`None`](https://docs.python.org/3/builtins/constants.html#None)

Shape tuple (#observations, #variables). Can only be provided if `X` is `None`.

**filename**[`PathLike`](https://docs.python.org/3/library/os.html#os.PathLike)[[`str`](https://docs.python.org/3/builtins/stdtypes.html#str)] | [`str`](https://docs.python.org/3/builtins/stdtypes.html#str) | [`None`](https://docs.python.org/3/builtins/constants.html#None)

Name of backing file. See [`h5py.File`](https://docs.h5py.org/en/latest/high/file.html#h5py.File).

**filemode**[`Literal`](https://docs.python.org/3/library/typing.html#typing.Literal)[`'r'`, `'r+'`] | [`None`](https://docs.python.org/3/builtins/constants.html#None)

Open mode of backing file. See [`h5py.File`](https://docs.h5py.org/en/latest/high/file.html#h5py.File).

See also

[`io.read_h5ad`](https://anndata.scverse.org/generated/anndata.io.read_h5ad.html#anndata.io.read_h5ad), [`io.read_csv`](https://anndata.scverse.org/generated/anndata.io.read_csv.html#anndata.io.read_csv), [`io.read_excel`](https://anndata.scverse.org/generated/anndata.io.read_excel.html#anndata.io.read_excel), [`io.read_hdf`](https://anndata.scverse.org/generated/anndata.io.read_hdf.html#anndata.io.read_hdf), [`io.read_zarr`](https://anndata.scverse.org/generated/anndata.io.read_zarr.html#anndata.io.read_zarr), [`io.read_mtx`](https://anndata.scverse.org/generated/anndata.io.read_mtx.html#anndata.io.read_mtx), [`io.read_text`](https://anndata.scverse.org/generated/anndata.io.read_text.html#anndata.io.read_text), [`io.read_umi_tools`](https://anndata.scverse.org/generated/anndata.io.read_umi_tools.html#anndata.io.read_umi_tools)

Notes

[`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData) stores observations (samples) of variables/features in the rows of a matrix. This is the convention of the modern classics of statistics [[Hastie09]](https://anndata.scverse.org/references.html#hastie09) and machine learning [[Murphy12]](https://anndata.scverse.org/references.html#murphy12), the convention of dataframes both in R and Python and the established statistics and machine learning packages in Python ([statsmodels](http://www.statsmodels.org/stable/index.html), [scikit-learn](http://scikit-learn.org/)).

Single dimensional annotations of the observation and variables are stored in the [`obs`](https://anndata.scverse.org/generated/anndata.AnnData.obs.html#anndata.AnnData.obs) and [`var`](https://anndata.scverse.org/generated/anndata.AnnData.var.html#anndata.AnnData.var) attributes as [`DataFrame`](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.html#pandas.DataFrame)s. This is intended for metrics calculated over their axes. Multi-dimensional annotations are stored in [`obsm`](https://anndata.scverse.org/generated/anndata.AnnData.obsm.html#anndata.AnnData.obsm) and [`varm`](https://anndata.scverse.org/generated/anndata.AnnData.varm.html#anndata.AnnData.varm), which are aligned to the objects observation and variable dimensions respectively. Square matrices representing graphs are stored in [`obsp`](https://anndata.scverse.org/generated/anndata.AnnData.obsp.html#anndata.AnnData.obsp) and [`varp`](https://anndata.scverse.org/generated/anndata.AnnData.varp.html#anndata.AnnData.varp), with both of their own dimensions aligned to their associated axis. Additional measurements across both observations and variables are stored in [`layers`](https://anndata.scverse.org/generated/anndata.AnnData.layers.html#anndata.AnnData.layers).

Indexing into an AnnData object can be performed by relative position with numeric indices (like pandas’ [`iloc()`](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.iloc.html#pandas.DataFrame.iloc)), or by labels (like [`loc()`](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.loc.html#pandas.DataFrame.loc)). To avoid ambiguity with numeric indexing into observations or variables, indexes of the AnnData object are converted to strings by the constructor.

Subsetting an AnnData object by indexing into it will also subset its elements according to the dimensions they were aligned to. This means an operation like `adata[list_of_obs,:]` will also subset [`obs`](https://anndata.scverse.org/generated/anndata.AnnData.obs.html#anndata.AnnData.obs), [`obsm`](https://anndata.scverse.org/generated/anndata.AnnData.obsm.html#anndata.AnnData.obsm), and [`layers`](https://anndata.scverse.org/generated/anndata.AnnData.layers.html#anndata.AnnData.layers).

Subsetting an AnnData object returns a view into the original object, meaning very little additional memory is used upon subsetting. This is achieved lazily, meaning that the constituent arrays are subset on access. Copying a view causes an equivalent “real” AnnData object to be generated. Attempting to modify a view (at any attribute except X) is handled in a copy-on-modify manner, meaning the object is initialized in place. Here’s an example:

```
batch1 = adata[adata.obs["batch"] == "batch1", :]
batch1.obs["value"] = 0  # This makes batch1 a “real” AnnData object
```

At the end of this snippet: `adata` was not modified, and `batch1` is its own AnnData object with its own data.

Similar to Bioconductor’s `ExpressionSet` and [`scipy.sparse`](https://docs.scipy.org/doc/scipy/reference/sparse.html#module-scipy.sparse) matrices, subsetting an AnnData object retains the dimensionality of its constituent arrays. Therefore, unlike with the classes exposed by [`pandas`](https://pandas.pydata.org/docs/index.html#module-pandas), [`numpy`](https://numpy.org/doc/stable/reference/index.html#module-numpy), and `xarray`, there is no concept of a one dimensional AnnData object. AnnDatas always have two inherent dimensions, [`obs`](https://anndata.scverse.org/generated/anndata.AnnData.obs.html#anndata.AnnData.obs) and [`var`](https://anndata.scverse.org/generated/anndata.AnnData.var.html#anndata.AnnData.var). Additionally, maintaining the dimensionality of the AnnData object allows for consistent handling of [`scipy.sparse`](https://docs.scipy.org/doc/scipy/reference/sparse.html#module-scipy.sparse) matrices and [`numpy`](https://numpy.org/doc/stable/reference/index.html#module-numpy) arrays.

Attributes

| X | Data matrix of shapen_obs×n_vars. |
|---|-----------------------------------|
| filename | Change to backing mode by setting the filename of a.h5adfile. |
| is_view | Trueif object is view of another AnnData object,Falseotherwise. |
| isbacked | Trueif object is backed on disk,Falseotherwise. |
| isview | Whether or not this object is a view. |
| layers | Dictionary-like object with values of the same dimensions asX. |
| n_obs | Number of observations. |
| n_vars | Number of variables/features. |
| obs | One-dimensional annotation of observations (pd.DataFrame). |
| obs_names | Names of observations (alias for.obs.index). |
| obsm | Multi-dimensional annotation of observations (mutable structuredndarray). |
| obsp | Pairwise annotation of observations, a mutable mapping with array-like values. |
| raw | Store raw version ofXandvaras.raw.Xand.raw.var. |
| shape | Shape of data matrix (n_obs,n_vars). |
| uns | Unstructured annotation (ordered dictionary). |
| var | One-dimensional annotation of variables/ features (pd.DataFrame). |
| var_names | Names of variables (alias for.var.index). |
| varm | Multi-dimensional annotation of variables/features (mutable structuredndarray). |
| varp | Pairwise annotation of variables/features, a mutable mapping with array-like values. |

Methods

| __getitem__(index) | Slice AnnData object or retrieve an array using an accessor. |
|--------------------|--------------------------------------------------------------|
| __setitem__(index, value) | Set an array or element using an accessor. |
| __sizeof__(*[, show_stratified, with_disk]) | Get the size of the object’s data in bytes. |
| chunk_X([select, replace]) | Return a chunk of the data matrixXwith random or specified indices. |
| chunked_X([chunk_size]) | Return an iterator over the rows of the data matrixX. |
| copy([filename]) | Full copy, optionally on disk. |
| obs_keys() | List keys of observation annotationobs. |
| obs_names_make_unique([join]) | Makes the index unique by appending a number string to each duplicate index element: '1', '2', etc. |
| obs_vector(k, /, *[, layer]) | Convenience function for returning a 1 dimensional ndarray of values fromX,layers[k], orobs. |
| obsm_keys() | List keys of observation annotationobsm. |
| rename_categories(key, categories) | Rename categories of annotationkeyinobs,var, anduns. |
| strings_to_categoricals([df]) | Transform string annotations to categoricals. |
| to_df([layer]) | Generate shallowDataFrame. |
| to_memory(*[, copy]) | Return a new AnnData object with all non-in-memory arrays loaded into memory. |
| transpose() | Transpose whole object. |
| uns_keys() | List keys of unstructured annotation. |
| unwriteable(*[, store_type, compat]) | Whether or not anAnnDataobject can be written to disk for a given store type. |
| var_keys() | List keys of variable annotationvar. |
| var_names_make_unique([join]) | Makes the index unique by appending a number string to each duplicate index element: '1', '2', etc. |
| var_vector(k, /, *[, layer]) | Convenience function for returning a 1 dimensional ndarray of values fromX,layers[k], orobs. |
| varm_keys() | List keys of variable annotationvarm. |
| write([filename, ...]) | Write.h5ad-formatted hdf5 file. |
| write_csvs(dirname, *[, skip_data, sep]) | Write annotation to.csvfiles. |
| write_h5ad([filename, ...]) | Write.h5ad-formatted hdf5 file. |
| write_loom(filename, *[, write_obsm_varm]) | Write.loom-formatted hdf5 file. |
| write_zarr(store, *[, chunks, ...]) | Write a hierarchical Zarr array store. |
