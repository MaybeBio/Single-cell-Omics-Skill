# Quickstart

> Every scanpy analysis works on an AnnData object: scanpy.pp preprocesses it, scanpy.tl computes on it and scanpy.pl plots it. The example below clusters 3k PBMCs and finds the marker genes of each ...

# Quickstart[#](https://scanpy.scverse.org/en/stable/quickstart.html#quickstart)

Every scanpy analysis works on an [`AnnData`](https://anndata.scverse.org/en/stable/generated/anndata.AnnData.html#anndata.AnnData) object: [`scanpy.pp`](https://scanpy.scverse.org/en/stable/api/preprocessing.html#module-scanpy.pp) preprocesses it, [`scanpy.tl`](https://scanpy.scverse.org/en/stable/api/tools.html#module-scanpy.tl) computes on it and [`scanpy.pl`](https://scanpy.scverse.org/en/stable/api/plotting.html#module-scanpy.pl) plots it. The example below clusters 3k PBMCs and finds the marker genes of each cluster.

```
import scanpy as sc

adata = sc.datasets.pbmc3k()

sc.pp.filter_cells(adata, min_genes=100)
sc.pp.filter_genes(adata, min_cells=3)
sc.pp.normalize_total(adata)
sc.pp.log1p(adata)
sc.pp.highly_variable_genes(adata, n_top_genes=2000)

sc.tl.pca(adata)
sc.pp.neighbors(adata)
sc.tl.umap(adata)
sc.tl.leiden(adata, flavor="igraph", n_iterations=2)
sc.tl.rank_genes_groups(adata, "leiden")

sc.pl.umap(adata, color="leiden")
sc.pl.rank_genes_groups_dotplot(adata, n_genes=3)
```

Most functions store their results in `adata` instead of returning them, e.g. the clusters in `adata.obs["leiden"]`. Leiden clustering needs igraph, which `pipinstall'scanpy[leiden]'` installs, see [Installation](https://scanpy.scverse.org/en/stable/installation.html).

[Preprocessing and clustering](https://scanpy.scverse.org/en/stable/tutorials/basics/clustering.html) walks through the same steps with quality control and doublet detection, [Usage Principles](https://scanpy.scverse.org/en/stable/usage-principles.html) explains the conventions behind the API and [API](https://scanpy.scverse.org/en/stable/api/index.html) lists every function.
