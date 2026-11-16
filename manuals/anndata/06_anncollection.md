# Lazily concatenating multiple AnnData objects

> Author: Sergei Rybakov This tutorial shows how to use the AnnCollection class with one or multiple AnnData objects. AnnCollection initialization: AnnCollection is a wrapper class which allows to la...

# Lazily concatenating multiple AnnData objects[#](https://anndata.scverse.org/tutorials/notebooks/anncollection.html#lazily-concatenating-multiple-anndata-objects)

**Author:**[Sergei Rybakov](https://github.com/koncopd)

This tutorial shows how to use the `AnnCollection` class with one or multiple `AnnData` objects.

```
from __future__ import annotations

import warnings

import gdown
import numpy as np
import pandas as pd
import scanpy as sc
from sklearn.decomposition import IncrementalPCA
from sklearn.model_selection import train_test_split

from anndata.experimental.multi_files import AnnCollection

warnings.simplefilter(action="ignore", category=pd.core.common.SettingWithCopyWarning)
```

```
# the data is from this scvi reproducibility notebook
# https://yoseflab.github.io/scvi-tools-reproducibility/scarches_totalvi_seurat_data/
gdown.download(
    url="https://drive.google.com/uc?id=1X5N9rOaIqiGxZRyr1fyZ6NpDPeATXoaC",
    output="pbmc_seurat_v4.h5ad",
    quiet=False,
)
gdown.download(
    url="https://drive.google.com/uc?id=1JgaXNwNeoEqX7zJL-jJD3cfXDGurMrq9",
    output="covid_cite.h5ad",
    quiet=False,
)
```

```
covid = sc.read("covid_cite.h5ad")
pbmc = sc.read("pbmc_seurat_v4.h5ad")
```

## AnnCollection initialization[#](https://anndata.scverse.org/tutorials/notebooks/anncollection.html#anncollection-initialization)

AnnCollection is a wrapper class which allows to lazily concatenate and jointly subset AnnData objects along the observations axis.

You can initialize AnnCollection with one AnnData object.

```
dataset = AnnCollection([covid])
```

```
dataset
```

```
AnnCollection object with n_obs × n_vars = 57669 × 33538
  constructed from 1 AnnData objects
    view of obsm: 'pro_exp'
    obs: 'orig.ident', 'nCount_RNA', 'nFeature_RNA', 'RNA_snn_res.0.4', 'seurat_clusters', 'set', 'Resp', 'disease', 'subj_code', 'covidpt_orhealth', 'mito', 'ncount', 'nfeat', 'bust_21', 'og_clust', 'severmod_other', 'og_clusts', 'nCount_ADT', 'nFeature_ADT', 'UMAP1', 'UMAP2', 'final_clust', 'final_clust_v2', 'new_pt_id', 'Resp_og', 'final_clust_withnum', 'final_clust_review', 'Age', 'Gender', 'Gender_num'
```

`viewofobsm` means that the wrapper object doesn’t copy anything from .obsm of the underlying AnnData object. `obs` instead of `viewofobs` means that the object copied .obs from the AnnData object. You can configure what is copied, please see the AnnCollection tutorial for details. The wrapper object (AnnCollection) never copies the full .X or .layers from the underlying AnnData object, all conversions happen when the AnnCollection object is subset.

But most importantly you can lazily join several AnnData objects without copying them. Set `join_vars='inner'` if you have different variables in the AnnData objects to create a joint index of the intersection of the variables

```
dataset = AnnCollection(
    [covid, pbmc], join_vars="inner", join_obs=None, label="dataset"
)
```

`label='dataset'` means the AnnCollection object will have the column ‘dataset’ in its own .obs.

```
dataset
```

```
AnnCollection object with n_obs × n_vars = 219433 × 20729
  constructed from 2 AnnData objects
    view of obs: 'orig.ident', 'nCount_RNA', 'nFeature_RNA', 'nCount_ADT', 'nFeature_ADT'
    own obs: 'dataset'
```

This object doesn’t copy data from AnnData objects, it uses joint index of observations and variables of the AnnData objects to allow joint subsetting. AnnCollection object has its own .obs and also view of inner join (you can select the type of join for .obs on initialization) of .obs in the passed AnnData objects. An AnnCollection object can also have its own .obsm if `join_obsm='inner'` is passed.

You can also initialize an AnnCollection object with backed AnnData objects

```
covid = sc.read("covid_cite.h5ad", backed="r")
pbmc = sc.read("pbmc_seurat_v4.h5ad", backed="r")
```

```
dataset = AnnCollection(
    [covid, pbmc], join_vars="inner", join_obs=None, label="dataset"
)
```

```
dataset
```

```
AnnCollection object with n_obs × n_vars = 219433 × 20729
  constructed from 2 AnnData objects
    view of obs: 'orig.ident', 'nCount_RNA', 'nFeature_RNA', 'nCount_ADT', 'nFeature_ADT'
    own obs: 'dataset'
```

You can subset AnnCollection to get observation related attributes of the lazily concatenated AnnData objects.

```
subset = dataset[56000:58000]
```

```
subset
```

```
AnnCollectionView object with n_obs × n_vars = 2000 × 20729
    obs: 'orig.ident', 'nCount_RNA', 'nFeature_RNA', 'nCount_ADT', 'nFeature_ADT'
```

```
subset.X.shape
```

```
(2000, 20729)
```

```
subset.obs
```

```
View of obs with keys: 'orig.ident', 'nCount_RNA', 'nFeature_RNA', 'nCount_ADT', 'nFeature_ADT'
```

```
subset.obs["orig.ident"][:2]
```

```
array(['SeuratProject', 'SeuratProject'], dtype=object)
```

Subsetting of an AnnCollection object returns AnnCollectionView. Only these subset objects have views of .obs, .obsm, .layers, .X from the passed AnnData objects. When an AnnCollection object is initialized with `join_obs=None` and `join_obsm=None`, the subset objects only give you access to the views of .obs, .obsm, .layers, .X from the passed AnnData objects, and not to .obs of AnnCollection. This is why you don’t see dataset key among the .obs keys of the subset.

However, if you initialize an AnnCollection object with `join_obs='inner'` (for inner join of .obs in the passed AnnData objects) or `join_obs='outer'`, the AnnCollection with join and copy the result to its own .obs instead of creating a view. Then subset objects (AnnCollectionView) will give access to the .obs attribute of the AnnCollection object instead of the view of the lazily joined .obs of the underlying AnnData objects. When `join_obs` is not `None`, you have a copy of the concatenated .obs attributes in the AnnCollection object.

```
dataset = AnnCollection(
    [covid, pbmc], join_vars="inner", join_obs="inner", label="dataset"
)
```

```
dataset
```

```
AnnCollection object with n_obs × n_vars = 219433 × 20729
  constructed from 2 AnnData objects
    obs: 'orig.ident', 'nCount_RNA', 'nFeature_RNA', 'nCount_ADT', 'nFeature_ADT', 'dataset'
```

As you see, now the concatenated .obs attributes are copied to .obs of the AnnCollection object.

```
subset = dataset[56000:58000]
```

```
subset
```

```
AnnCollectionView object with n_obs × n_vars = 2000 × 20729
    obs: 'orig.ident', 'nCount_RNA', 'nFeature_RNA', 'nCount_ADT', 'nFeature_ADT', 'dataset'
```

Note that now you can have access to the ‘dataset’.obs column from subsets.

```
subset.obs["dataset"][:2]
```

```
GTTGTCCAGCAACAAT-1    0
GTTGTCCAGCGAGAAA-1    0
Name: dataset, dtype: category
Categories (2, object): ['0', '1']
```

Same can be done for .obsm, though only with inner join. `join_obsm='inner'` means do inner join and copy AnnCollection’s own obsm, `join_obsm=None` means do inner join without copying and do not create obsm in an AnnCollection object, though AnncollectionView will still have access to the lazily inner joined .obsm attributes of the underlying AnnData objects.

```
# inner join of .obsm of the datasets covid and pbmc is empty
dataset = AnnCollection(
    [covid, pbmc], join_vars="inner", join_obsm=None, label="dataset"
)
```

```
dataset.obsm is None
```

```
True
```

```
# inner join of .obsm of the datasets covid and pbmc is empty
dataset = AnnCollection(
    [covid, pbmc], join_vars="inner", join_obsm="inner", label="dataset"
)
```

```
dataset.obsm is None
```

```
False
```

```
dataset.obsm
```

```
AxisArrays with keys:
```

```
subset = dataset[56000:58000]
```

```
subset.obs
```

```
View of obs with keys: 'orig.ident', 'nCount_RNA', 'nFeature_RNA', 'nCount_ADT', 'nFeature_ADT', 'dataset'
```

.obs of AnnCollectionView objects is a MapObsView object, but it can be converted to pandas DataFrame.

```
subset.obs.df
```

|  | orig.ident | nCount_RNA | nFeature_RNA | nCount_ADT | nFeature_ADT | dataset |
|--|------------|------------|--------------|------------|--------------|---------|
| SeuratProject | 3763.0 | 1353 | 268.0 | 35 | 0 |  |
| SeuratProject | 5883.0 | 1931 | 515.0 | 32 | 0 |  |
| SeuratProject | 1479.0 | 750 | 147.0 | 34 | 0 |  |
| SeuratProject | 8920.0 | 2588 | 788.0 | 37 | 0 |  |
| SeuratProject | 3016.0 | 1261 | 257.0 | 34 | 0 |  |
| ... | ... | ... | ... | ... | ... |  |
| P1_0 | 4571.0 | 1838 | 4667.0 | 218 | 1 |  |
| P4_7 | 4351.0 | 1234 | 6718.0 | 213 | 1 |  |
| P1_0 | 3922.0 | 1602 | 4267.0 | 219 | 1 |  |
| P3_7 | 6033.0 | 1395 | 5277.0 | 213 | 1 |  |
| P4_0 | 9649.0 | 2610 | 7159.0 | 221 | 1 |  |

2000 rows × 6 columns

Yo can also create a converter with a function or a Mapping of functions which will be applied to the values of attributes (`.obs`, `.obsm`, `.layers`, `.X`) or to specific keys of these attributes in the subset object. Specify an attribute and a key (if needed) as keys of the passed Mapping and a function to be applied as a value (see also the AnnLoader tutorial)

For example, now .X is a sparse matrix.

```
subset.X
```

```
<2000x20729 sparse matrix of type '<class 'numpy.float32'>'
	with 3935425 stored elements in Compressed Sparse Row format>
```

You can set up a converter to densify it on the fly.

```
convert = {"X": lambda a: a.toarray()}

dataset.convert = (
    convert  # or during the initialization AnnCollection(..., convert=convert)
)
```

```
dataset[56000:58000].X
```

```
array([[0., 0., 0., ..., 0., 0., 0.],
       [0., 0., 0., ..., 0., 0., 0.],
       [0., 0., 0., ..., 0., 0., 0.],
       ...,
       [0., 0., 0., ..., 0., 0., 0.],
       [0., 0., 0., ..., 0., 0., 0.],
       [0., 0., 0., ..., 0., 0., 0.]], dtype=float32)
```

## Use with AnnLoader[#](https://anndata.scverse.org/tutorials/notebooks/anncollection.html#use-with-annloader)

You can directly pass Anncollection to AnnLoader (see also the AnnLoader tutorial).

```
from anndata.experimental.pytorch import AnnLoader
```

```
dataloader = AnnLoader(dataset, batch_size=128)
```

By default AnnLoader adds its own converter on top of converters in AnnCollection. It densifies sparse arrays, converts dense numeric arrays to pytorch tensors, transfers to the default cuda device (if `use_cuda=True`), does memory pinning (if `pin_memory=True`). If you pass an AnnCollection object with prespecified converters, the default converter won’t overwrite these converters but will be applied on top of them.

```
dataloader.dataset[:2].X.dtype
```

```
torch.float32
```

You can also pass an AnncollectionView object (subset object) to AnnLoader. This can be useful when you want to do a train test split without creating copies of your huge AnnData objects.

```
train, test = train_test_split(dataset)
```

```
train
```

```
AnnCollectionView object with n_obs × n_vars = 164574 × 20729
    obs: 'orig.ident', 'nCount_RNA', 'nFeature_RNA', 'nCount_ADT', 'nFeature_ADT', 'dataset'
```

```
test
```

```
AnnCollectionView object with n_obs × n_vars = 54859 × 20729
    obs: 'orig.ident', 'nCount_RNA', 'nFeature_RNA', 'nCount_ADT', 'nFeature_ADT', 'dataset'
```

```
dataloader = AnnLoader(train, batch_size=128)
```

```
dataloader.dataset[:2].X.dtype
```

```
torch.float32
```

## Example: use AnnCollection for iterative PCA[#](https://anndata.scverse.org/tutorials/notebooks/anncollection.html#example-use-anncollection-for-iterative-pca)

This example shows how to get joint PCA of both AnnData objects without in-memory concatenation.

```
covid = sc.read("covid_cite.h5ad")
pbmc = sc.read("pbmc_seurat_v4.h5ad")
```

```
sc.pp.normalize_total(covid)
sc.pp.log1p(covid)

sc.pp.normalize_total(pbmc)
sc.pp.log1p(pbmc)
```

Setting `join_obsm='inner'` to have a separate .obsm in AnnCollection object.

```
dataset = AnnCollection(
    dict(covid=covid, pbmc=pbmc),
    convert=dict(X=lambda a: a.toarray()),
    join_vars="inner",
    join_obs="outer",
    join_obsm="inner",
    label="dataset",
)
```

```
dataset.obsm["X_pca"] = np.empty(
    (dataset.shape[0], 50), dtype="float64"
)  # for future pca projections
```

```
dataset
```

```
AnnCollection object with n_obs × n_vars = 219433 × 20729
  constructed from 2 AnnData objects
    obs: 'orig.ident', 'nCount_RNA', 'nFeature_RNA', 'RNA_snn_res.0.4', 'seurat_clusters', 'set', 'Resp', 'disease', 'subj_code', 'covidpt_orhealth', 'mito', 'ncount', 'nfeat', 'bust_21', 'og_clust', 'severmod_other', 'og_clusts', 'nCount_ADT', 'nFeature_ADT', 'UMAP1', 'UMAP2', 'final_clust', 'final_clust_v2', 'new_pt_id', 'Resp_og', 'final_clust_withnum', 'final_clust_review', 'Age', 'Gender', 'Gender_num', 'lane', 'donor', 'time', 'celltype.l1', 'celltype.l2', 'celltype.l3', 'Phase', 'nCount_SCT', 'nFeature_SCT', 'X_index', 'dataset'
    obsm: 'X_pca'
```

As `join_obs='outer'`, the subsets (AnnCollectionView objects) will point to the .obs attribute of the dataset.

```
dataset[:2].obs
```

```
View of obs with keys: 'orig.ident', 'nCount_RNA', 'nFeature_RNA', 'RNA_snn_res.0.4', 'seurat_clusters', 'set', 'Resp', 'disease', 'subj_code', 'covidpt_orhealth', 'mito', 'ncount', 'nfeat', 'bust_21', 'og_clust', 'severmod_other', 'og_clusts', 'nCount_ADT', 'nFeature_ADT', 'UMAP1', 'UMAP2', 'final_clust', 'final_clust_v2', 'new_pt_id', 'Resp_og', 'final_clust_withnum', 'final_clust_review', 'Age', 'Gender', 'Gender_num', 'lane', 'donor', 'time', 'celltype.l1', 'celltype.l2', 'celltype.l3', 'Phase', 'nCount_SCT', 'nFeature_SCT', 'X_index', 'dataset'
```

Use the ‘orig.ident’ column for joint batch information.

```
dataset.obs["orig.ident"].loc[covid.obs_names] = covid.obs["set"]
```

```
ipca = IncrementalPCA(n_components=50)
```

Use the built-in iterator over the AnnCollection object to get bacthes of data with 6000 observations each and fit partial PCA.

```
for batch, _idx in dataset.iterate_axis(6000):
    ipca.partial_fit(batch.X)
```

Use the iterator to transform the normalized counts.

```
for batch, idx in dataset.iterate_axis(6000):
    dataset.obsm["X_pca"][idx] = ipca.transform(batch.X)
```

You can then convert an AnnCollecion object to an AnnData object. The AnnData object won’t have .X or .layers, only .obs and .obsm. In order to get also .X and .layers, you need to call to_adata() for an AnnCollectionView object (subset of an AnnCollection object).

```
adata = dataset.to_adata()
```

```
adata
```

```
AnnData object with n_obs × n_vars = 219433 × 20729
    obs: 'orig.ident', 'nCount_RNA', 'nFeature_RNA', 'RNA_snn_res.0.4', 'seurat_clusters', 'set', 'Resp', 'disease', 'subj_code', 'covidpt_orhealth', 'mito', 'ncount', 'nfeat', 'bust_21', 'og_clust', 'severmod_other', 'og_clusts', 'nCount_ADT', 'nFeature_ADT', 'UMAP1', 'UMAP2', 'final_clust', 'final_clust_v2', 'new_pt_id', 'Resp_og', 'final_clust_withnum', 'final_clust_review', 'Age', 'Gender', 'Gender_num', 'lane', 'donor', 'time', 'celltype.l1', 'celltype.l2', 'celltype.l3', 'Phase', 'nCount_SCT', 'nFeature_SCT', 'X_index', 'dataset'
    obsm: 'X_pca'
```

```
adata.X is None
```

```
True
```

```
sc.pp.neighbors(adata)
```

```
sc.tl.umap(adata)
```

```
sc.pl.umap(adata, color=["dataset", "orig.ident"])
```

![../../_images/6f787644a193af699d5d8f7ba77ee8efd989b4a32cfdb05c78b8a2dc977b89e8.png](https://anndata.scverse.org/_images/6f787644a193af699d5d8f7ba77ee8efd989b4a32cfdb05c78b8a2dc977b89e8.png)

```
sc.pl.umap(adata, color=["celltype.l2", "final_clust_v2"], wspace=0.8)
```

![../../_images/7cbf3095d3b2040b1fe3a23416dbc37d2c50fd634afaf8f7aae3f1dbd0fec679.png](https://anndata.scverse.org/_images/7cbf3095d3b2040b1fe3a23416dbc37d2c50fd634afaf8f7aae3f1dbd0fec679.png)
