# zarr-v3 Guide/Roadmap

> anndata now uses the much improved zarr v3 package and format. Users should notice a significant performance improvement, especially for cloud data, but also likely for local data as well. Here is ...

# zarr-v3 Guide/Roadmap[#](https://anndata.scverse.org/tutorials/zarr-v3.html#zarr-v3-guide-roadmap)

`anndata` now uses the much improved [`zarr`](https://zarr.readthedocs.io/en/stable/api/zarr/#zarr) v3 package and format. Users should notice a significant performance improvement, especially for cloud data, but also likely for local data as well. Here is a quick guide on some of our learnings so far:

## Consolidated Metadata[#](https://anndata.scverse.org/tutorials/zarr-v3.html#consolidated-metadata)

All `zarr` stores are now consolidated by default when written via [`anndata.io.write_zarr()`](https://anndata.scverse.org/generated/anndata.io.write_zarr.html#anndata.io.write_zarr) or [`anndata.AnnData.write_zarr()`](https://anndata.scverse.org/generated/anndata.AnnData.write_zarr.html#anndata.AnnData.write_zarr). For more information on this topic, please see the zarr [consolidated metadata](https://zarr.readthedocs.io/en/latest/user-guide/consolidated_metadata/) user guide. Practcally, this changes means that once a store has been written, it should be treated as immutable **unless you remove the consolidated metadata and/or rewrite after the mutating operation** i.e., if you wish to use [`anndata.io.write_elem()`](https://anndata.scverse.org/generated/anndata.io.write_elem.html#anndata.io.write_elem) to add a column to `obs`, a `layer` etc. to an existing store. For example, to mutate an existing store on-disk, you may do:

```
g = zarr.open_group(orig_path, mode="a", use_consolidated=False)
ad.io.write_elem(
    g,
    "obs",
    obs,
    dataset_kwargs=dict(chunks=(250,)),
)
zarr.consolidate_metadata(g.store)
```

In this example, the store was opened unconsolidated (trying to open it as a consolidated store would error out), edited, and then reconsolidated. Alternatively, one could simple delete the file containing the consolidated metadata first at the root, `.zmetadata`.

## Remote data[#](https://anndata.scverse.org/tutorials/zarr-v3.html#remote-data)

We now provide the [`anndata.experimental.read_lazy()`](https://anndata.scverse.org/generated/anndata.experimental.read_lazy.html#anndata.experimental.read_lazy) feature for reading as much of the [`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData) object as lazily as possible, using `dask` and [`xarray`](https://docs.xarray.dev/en/stable/index.html#module-xarray). Please note that this feature is experimental and subject to change. To enable this functionality in a performant and feature-complete way for remote data sources, we use [consolidated metadata](https://zarr.readthedocs.io/en/latest/user-guide/consolidated_metadata/) on the `zarr` store (written by default). Please note that this introduces consistency issues – if you update the structure of the underlying `zarr` store i.e., remove a column from `obs`, the consolidated metadata will no longer be valid. Further, note that without consolidated metadata, we cannot guarantee your stored `AnnData` object will be fully readable. And even if it is fully readable, it will almost certainly be much slower to read.

There are two ways of opening remote `zarr` stores from the `zarr-python` package, [`zarr.storage.FsspecStore`](https://zarr.readthedocs.io/en/stable/api/zarr/storage/#zarr.storage.FsspecStore) and [`zarr.storage.ObjectStore`](https://zarr.readthedocs.io/en/stable/api/zarr/storage/#zarr.storage.ObjectStore), and both can be used with `read_lazy`. [`obstore` claims](https://developmentseed.org/obstore/latest/performance) to be more performant out-of-the-box, but notes that this claim has not been benchmarked with the `uvloop` event loop, which itself claims to be 2× more performant than the default event loop for `python`.

[`zarr`](https://zarr.readthedocs.io/en/stable/api/zarr/#zarr) ships a `FusedCodecPipeline` that is likely *much* (~2-4x) faster for some use-cases, like small chunks, and is no worse than the current default. We strongly encourage users to switch to it via `zarr.config.set({"codec_pipeline.path":"zarr.core.codec_pipeline.FusedCodecPipeline"})` for remote data. If [`zarr`](https://zarr.readthedocs.io/en/stable/api/zarr/#zarr) is new enough, it will be used *by default* for [`read_zarr()`](https://anndata.scverse.org/generated/anndata.io.read_zarr.html#anndata.io.read_zarr) and [`write_zarr()`](https://anndata.scverse.org/generated/anndata.AnnData.write_zarr.html#anndata.AnnData.write_zarr).

## Local data[#](https://anndata.scverse.org/tutorials/zarr-v3.html#local-data)

Local data generally poses a different set of challenges. First, write speeds can be somewhat slow and second, the creation of many small files on a file system can slow down a filesystem. For the “many small files” problem, `zarr` has introduced [sharding](https://zarr.readthedocs.io/en/stable/user-guide/arrays/#sharding) in the v3 file format. We offer [`anndata.settings.auto_shard_zarr_v3`](https://anndata.scverse.org/generated/anndata.settings.html#anndata.settings.auto_shard_zarr_v3) to hook into zarr’s ability to automatically compute shards, which is experimental at the moment. Manual sharding requires knowledge of the array element you are writing (such as shape or data type), though, and therefore you will need to use [`anndata.experimental.write_dispatched()`](https://anndata.scverse.org/generated/anndata.experimental.write_dispatched.html#anndata.experimental.write_dispatched) to use custom sharding. For example, you cannot shard a 1D array with `shard` sizes `(256,256)`. Here is a short example, although you should tune the sizes to your own use-case and also use the compression that makes the most sense for you:

```
import zarr
import anndata as ad
from collections.abc import Mapping
from typing import Any

g = zarr.open_group(
    orig_path, mode="a", use_consolidated=False, zarr_format=3
)  # zarr_format 3 is default but note that sharding only works with v3!


def write_sharded(group: zarr.Group, adata: ad.AnnData):
    def callback(
        func: ad.experimental.Write,
        g: zarr.Group,
        k: str,
        elem: ad.typing.RWAble,
        dataset_kwargs: Mapping[str, Any],
        iospec: ad.experimental.IOSpec,
    ):
        if iospec.encoding_type in {"array"}:
            dataset_kwargs = {
                "shards": tuple(int(2 ** (16 / len(elem.shape))) for _ in elem.shape),
                **dataset_kwargs,
            }
            dataset_kwargs["chunks"] = tuple(i // 2 for i in dataset_kwargs["shards"])
        elif iospec.encoding_type in {"csr_matrix", "csc_matrix"}:
            dataset_kwargs = {"shards": (2**16,), "chunks": (2**8,), **dataset_kwargs}
        func(g, k, elem, dataset_kwargs=dataset_kwargs)

    return ad.experimental.write_dispatched(group, "/", adata, callback=callback)
```

However, `zarr-python` can be slow with sharding throughput as well as writing throughput. Thus if you wish to speed up either writing, sharding, or both (or receive a modest speed-boost for reading), a bridge to the `zarr` implementation in Rust [zarrs-python](https://zarrs-python.readthedocs.io/en/stable/index.html) can help with that (see the [zarr-benchmarks](https://github.com/LDeakin/zarr_benchmarks)):

```
uv pip install zarrs
```

```
import zarr
import zarrs

zarr.config.set({"codec_pipeline.path": "zarrs.ZarrsCodecPipeline"})
```

However, this pipeline is not compatible with all types of zarr store, especially remote stores and there are limitations on where rust can give a performance boost for indexing. We therefore recommend this pipeline for writing full datasets and reading contiguous regions of said written data locally. If installed, it will be used *by default* for [`read_zarr()`](https://anndata.scverse.org/generated/anndata.io.read_zarr.html#anndata.io.read_zarr) and [`write_zarr()`](https://anndata.scverse.org/generated/anndata.AnnData.write_zarr.html#anndata.AnnData.write_zarr).

## Codecs[#](https://anndata.scverse.org/tutorials/zarr-v3.html#codecs)

The default `zarr-python` v3 codec for the v3 format is no longer `blosc` but `zstd`. While `zstd` is more widespread, you may find its performance to not meet your old expectations. Therefore, we recommend passing in the [`zarr.codecs.BloscCodec`](https://zarr.readthedocs.io/en/stable/api/zarr/codecs/#zarr.codecs.BloscCodec) to `compressor` on [`write_zarr()`](https://anndata.scverse.org/generated/anndata.AnnData.write_zarr.html#anndata.AnnData.write_zarr) if you wish to return to the old behavior.

## Dask[#](https://anndata.scverse.org/tutorials/zarr-v3.html#dask)

Zarr v3 should be compatible with dask, although the default behavior is to use zarr’s chunking for dask’s own. With sharding, this behavior may be undesirable as shards can often contain many small chunks, thereby slowing down i/o as dask will need to index into the zarr store for every chunk. Therefore it may be better to customize this behavior by passing `chunks=my_zarr_array.shards` as an argument to [`dask.array.from_zarr()`](https://docs.dask.org/en/stable/generated/dask.array.from_zarr.html#dask.array.from_zarr) or similar.

## GPU i/o[#](https://anndata.scverse.org/tutorials/zarr-v3.html#gpu-i-o)

At the moment, it is unlikely your `anndata` i/o will work if you use [`zarr.config.enable_gpu`](https://zarr.readthedocs.io/en/stable/user-guide/gpu/). It’s *possible* dense data i/o i.e., using [`anndata.io.read_elem()`](https://anndata.scverse.org/generated/anndata.io.read_elem.html#anndata.io.read_elem) will work as expected, but this functionality is untested – sparse data, awkward arrays, and dataframes will not. `kvikio` currently provides a [`kvikio.zarr.GDSStore`](https://docs.nvidia.com/kvikio/latest/python/#kvikio.zarr.GDSStore) although there are no working compressors at the moment exported from the `zarr-python` package (work is underway for `Zstd`: [zarr-developers/zarr-python#2863](https://github.com/zarr-developers/zarr-python/pull/2863).

We anticipate enabling officially supporting this functionality officially for dense data, sparse data, and possibly awkward arrays in the next minor release, 0.13.

## Asynchronous i/o[#](https://anndata.scverse.org/tutorials/zarr-v3.html#asynchronous-i-o)

At the moment, `anndata` exports no `async` functions. However, `zarr-python` has a fully `async` API and provides its own event-loop so that users like `anndata` can interact with a synchronous API while still beenfitting from `zarr-python`’s asynchronous functionality under that API. We anticipate providing `async` versions of [`anndata.io.read_elem()`](https://anndata.scverse.org/generated/anndata.io.read_elem.html#anndata.io.read_elem) and [`anndata.experimental.read_dispatched()`](https://anndata.scverse.org/generated/anndata.experimental.read_dispatched.html#anndata.experimental.read_dispatched) so that users can download data asynchronously without using the `zarr-python` event loop. We also would like to create an asynchronous partial reader to enable iterative streaming of a dataset.
