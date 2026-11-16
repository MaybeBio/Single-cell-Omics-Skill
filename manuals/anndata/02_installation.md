# Installation

> anndata needs Python 3.13 or newer. PyPI: conda-forge: Development version: To work on anndata itself, see Contributing. Optional dependencies: anndata[dask] lets an AnnData hold Dask arrays. annda...

# Installation[#](https://anndata.scverse.org/installation.html#installation)

anndata needs Python 3.13 or newer.

## PyPI[#](https://anndata.scverse.org/installation.html#pypi)

```
pip install anndata
```

## conda-forge[#](https://anndata.scverse.org/installation.html#conda-forge)

```
conda install -c conda-forge anndata
```

## Development version[#](https://anndata.scverse.org/installation.html#development-version)

```
pip install git+https://github.com/scverse/anndata.git
```

To work on anndata itself, see [Contributing](https://anndata.scverse.org/contributing.html).

## Optional dependencies[#](https://anndata.scverse.org/installation.html#optional-dependencies)

`anndata[dask]` lets an `AnnData` hold [Dask arrays](https://anndata.scverse.org/tutorials/notebooks/anndata_dask_array.html).

`anndata[lazy]` adds Dask, xarray and what reading remote zarr stores needs, for [`read_lazy()`](https://anndata.scverse.org/generated/anndata.experimental.read_lazy.html#anndata.experimental.read_lazy) and [`read_elem_lazy()`](https://anndata.scverse.org/generated/anndata.experimental.read_elem_lazy.html#anndata.experimental.read_elem_lazy).

`anndata[cu11]`, `anndata[cu12]` or `anndata[cu13]` installs the CuPy wheel for your CUDA version, so an `AnnData` can hold GPU arrays. `anndata[gpu]` installs CuPy from source instead.
