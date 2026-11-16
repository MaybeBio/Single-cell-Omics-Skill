# Using dask with Scanpy

> dask is a popular out-of-core, distributed array processing library that scanpy is beginning to support. Here we walk through a quick tutorial of using dask in a simple analysis task. This notebook...

# Using `dask` with Scanpy[#](https://scanpy.scverse.org/en/stable/tutorials/experimental/dask.html#using-dask-with-scanpy)

Warning

🔪 **Beware sharp edges!** 🔪

`dask` support in `scanpy` is new and highly experimental!

Many functions in `scanpy`**do not** support `dask` and may exhibit unexpected behaviour if dask arrays are passed to them. Stick to what’s outlined in this tutorial and you should be fine!

Please report any issues you run into over on the issue tracker.

[`dask`](https://www.dask.org) is a popular out-of-core, distributed array processing library that scanpy is beginning to support. Here we walk through a quick tutorial of using `dask` in a simple analysis task.

This notebook relies on optional dependencies in dask, `sklearn-ann` and `annoy`. Install them with:

```
pip install -U "anndata[dask]" "scanpy[dask,leiden]" "dask[distributed,diagnostics]" sklearn-ann annoy
```

(`scanpy[dask]` means to install scanpy together with `dask[array]`, but will also always make sure that a compatible dask version is selected)

```
from __future__ import annotations

from pathlib import Path

import anndata as ad
import dask.distributed as dd
import h5py
import pooch

import scanpy as sc

sc.logging.print_header()
```

Here, we’ll be working with a moderately large dataset of 1.4 million cells taken from: [COVID-19 immune features revealed by a large-scale single-cell transcriptome atlas](https://cellxgene.cziscience.com/collections/0a839c4b-10d0-4d64-9272-684c49a2c8ba)

```
cell_atlas_path = Path(
    pooch.retrieve(
        "https://datasets.cellxgene.cziscience.com/82eac9c1-485f-4e21-ab21-8510823d4f6e.h5ad",
        known_hash="sha256:0b24babfb34b4af87a76806039afa3513c2c04c9045e2a9fb31a6e9350b1fabe",
        fname="cell_atlas.h5ad",
        path="data",
    )
)
```

For more information on using distributed computing via `dask`, please see their [documentation](https://docs.dask.org/en/stable/deploying-python.html). In short, one needs to define both a cluster and a client to have some degree of control over the compute resources dask will use. It’s very likely you will have to tune the number of workers and amount of memory per worker along with your chunk sizes.

```
cluster = dd.LocalCluster(n_workers=3)
client = dd.Client(cluster)
```

Note

In this notebook we will be demonstrating some computations in scanpy that use `scipy.sparse` classes within each dask chunk. Be aware that this is currently poorly supported by `dask`, and that if you want to interact with the `dask` arrays in any way other than though the `anndata` and `scanpy` libraries you will likely need to densify each chunk.

All operations in `scanpy` and `anndata` that work with sparse chunks also work with dense chunks.

The advantage of using sparse chunks are:

- The ability to work with fewer, larger chunks
- Accelerated computations per chunk (e.g. don’t need to `sum` all those extra zeros)

You can convert from `sparse` to `dense` chunks via:

```
X = X.map_blocks(lambda x: x.toarray(), dtype=X.dtype, meta=np.array([]))
```

And in reverse:

```
X = X.map_blocks(sparse.csr_matrix)
```

Note that you will likely have to work with smaller chunks when doing this, via a rechunking operation. We suggest using a factor of the larger chunk size to achieve the most efficient rechunking.

```
SPARSE_CHUNK_SIZE = 100_000
```

Dask provides extensive tooling for monitoring your computation. You can access that via the dashboard started when using any of their distributed clusters.

```
client
```

### Client

Client-dd1a682c-6c7f-11f0-979d-95f6366ae0fa

| Dashboard:http://127.0.0.1:43531/status |  |
|-----------------------------------------|--|

### Cluster Info

### LocalCluster

2504ead7

| Total threads:18 | Total memory:62.79 GiB |
|------------------|------------------------|
| Status:running | Using processes:True |

### Scheduler Info

### Scheduler

Scheduler-2c72d7c3-d2f2-4c49-b175-d01048567e07

| Dashboard:http://127.0.0.1:43531/status | Total threads:18 |
|-----------------------------------------|------------------|
| Started:Just now | Total memory:62.79 GiB |

### Workers

#### Worker: 0

| Dashboard:http://127.0.0.1:37035/status | Memory:20.93 GiB |
|-----------------------------------------|------------------|
| Nanny:tcp://127.0.0.1:43817 |  |
| Local directory:/mnt/volume/tmp/dask-scratch-space/worker-3kn90gqx |  |

#### Worker: 1

| Dashboard:http://127.0.0.1:37541/status | Memory:20.93 GiB |
|-----------------------------------------|------------------|
| Nanny:tcp://127.0.0.1:37117 |  |
| Local directory:/mnt/volume/tmp/dask-scratch-space/worker-lmafs1m9 |  |

#### Worker: 2

| Dashboard:http://127.0.0.1:46165/status | Memory:20.93 GiB |
|-----------------------------------------|------------------|
| Nanny:tcp://127.0.0.1:36781 |  |
| Local directory:/mnt/volume/tmp/dask-scratch-space/worker-221nzh2z |  |

We’ll convert the `X` representation to `dask` using `anndata.experimental.read_elem_lazy`.

The file we’ve retrieved from cellxgene has already been processed. Since this tutorial is demonstrating processing from counts, we’re just going to access the counts matrix and annotations.

```
%%time
with h5py.File(cell_atlas_path, "r") as f:
    adata = ad.AnnData(
        obs=ad.io.read_elem(f["obs"]),
        var=ad.io.read_elem(f["var"]),
    )
    adata.X = ad.experimental.read_elem_lazy(
        f["raw/X"], chunks=(SPARSE_CHUNK_SIZE, adata.shape[1])
    )
```

```
CPU times: user 2.34 s, sys: 180 ms, total: 2.52 s
Wall time: 2.48 s
```

We’ve optimized a number of scanpy functions to be completely lazy. That means it will look like nothing is computed when you call an operation on a dask array, but only later when you hit compute.

In some cases it’s currently unavoidable to skip all computation, and these cases will kick off compute for all the delayed operations immediately.

```
%%time
adata.layers["counts"] = adata.X.copy()  # Making sure we keep access to the raw counts
sc.pp.normalize_total(adata, target_sum=1e4)
```

```
CPU times: user 12.7 ms, sys: 59 μs, total: 12.8 ms
Wall time: 12.2 ms
```

```
%%time
sc.pp.log1p(adata)
```

```
CPU times: user 1.09 ms, sys: 0 ns, total: 1.09 ms
Wall time: 1.09 ms
```

Highly variable genes needs to add entries into `obs`, which currently does not support lazy column. So computation will occur immediately on call.

```
%%time
sc.pp.highly_variable_genes(adata)
```

```
CPU times: user 1.27 s, sys: 341 ms, total: 1.61 s
Wall time: 43.5 s
```

Scanpy 1.11+ supports computing PCA on `dask` arrays with sparse chunks.

```
%%time
sc.pp.pca(adata)
```

```
CPU times: user 7.58 s, sys: 441 ms, total: 8.02 s
Wall time: 53.1 s
```

While most of the PCA computation runs immediately, the last step (computing the observation loadings) is lazy, so must be triggered manually to avoid recomputation.

```
%%time
adata.obsm["X_pca"] = adata.obsm["X_pca"].compute()
```

```
CPU times: user 952 ms, sys: 497 ms, total: 1.45 s
Wall time: 39.4 s
```

```
adata
```

```
AnnData object with n_obs × n_vars = 1462702 × 27714
    obs: 'celltype', 'majorType', 'City', 'sampleID', 'donor_id', 'Sample type', 'CoVID-19 severity', 'Sample time', 'Sampling day (Days after symptom onset)', 'BCR single cell sequencing', 'TCR single cell sequencing', 'Outcome', 'Comorbidities', 'COVID-19-related medication and anti-microbials', 'Leukocytes [G over L]', 'Neutrophils [G over L]', 'Lymphocytes [G over L]', 'Unpublished', 'disease_ontology_term_id', 'cell_type_ontology_term_id', 'tissue_ontology_term_id', 'development_stage_ontology_term_id', 'self_reported_ethnicity_ontology_term_id', 'assay_ontology_term_id', 'sex_ontology_term_id', 'is_primary_data', 'organism_ontology_term_id', 'suspension_type', 'tissue_type', 'cell_type', 'assay', 'disease', 'organism', 'sex', 'tissue', 'self_reported_ethnicity', 'development_stage', 'observation_joinid'
    var: 'feature_is_filtered', 'feature_name', 'feature_reference', 'feature_biotype', 'feature_length', 'highly_variable', 'means', 'dispersions', 'dispersions_norm'
    uns: 'log1p', 'hvg', 'pca'
    obsm: 'X_pca'
    varm: 'PCs'
    layers: 'counts'
```

Now that we’ve computed our PCA let’s take a look at it:

```
sc.pl.pca(adata, color="majorType")
```

[![../../_images/7280e7628c2d51572ba50bdeb1600d6b8579ff7c9fab4536e4a730b1904a789a.png](https://scanpy.scverse.org/en/stable/_images/7280e7628c2d51572ba50bdeb1600d6b8579ff7c9fab4536e4a730b1904a789a.png)](https://scanpy.scverse.org/en/stable/_images/7280e7628c2d51572ba50bdeb1600d6b8579ff7c9fab4536e4a730b1904a789a.png)

Further support for `dask` is a work in progress. However, many operations past this point can work with the dimensionality reduction directly in memory. With scanpy `1.10` many of these operations can be accelerated to make working with large datasets significantly easier. For example:

- Using alternative KNN backends for faster neighbor calculation [Using other kNN libraries in Scanpy](https://scanpy.scverse.org/en/stable/how-to/knn-transformers.html)
- Using the `igraph` backend for clustering

```
%%time
from sklearn_ann.kneighbors.annoy import AnnoyTransformer

transformer = AnnoyTransformer(n_neighbors=15)
sc.pp.neighbors(adata, transformer=transformer)
```

```
CPU times: user 1min 39s, sys: 2.33 s, total: 1min 41s
Wall time: 1min 18s
```

```
%%time
sc.tl.leiden(adata, flavor="igraph", n_iterations=2)
```

```
CPU times: user 2min 14s, sys: 4 s, total: 2min 18s
Wall time: 2min 17s
```

UMAP computation can still be rather slow, taking longer than the rest of this notebook combined:

```
%%time
sc.tl.umap(adata)
sc.pl.umap(adata, color=["leiden", "majorType"])
```

[![../../_images/7892b8f497b868bf4353f873b5762e96e73c1fa06356dbf9e971f51c5e41aa37.png](https://scanpy.scverse.org/en/stable/_images/7892b8f497b868bf4353f873b5762e96e73c1fa06356dbf9e971f51c5e41aa37.png)](https://scanpy.scverse.org/en/stable/_images/7892b8f497b868bf4353f873b5762e96e73c1fa06356dbf9e971f51c5e41aa37.png)

```
CPU times: user 2h 37min 54s, sys: 27 s, total: 2h 38min 21s
Wall time: 30min 46s
```
