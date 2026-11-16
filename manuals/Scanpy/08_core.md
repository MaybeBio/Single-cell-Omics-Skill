# Core plotting functions

> Author: Fidel Ramírez This tutorial explores the visualization possibilities of scanpy and is divided into three sections: Scatter plots for embeddings (eg. UMAP, t-SNE), Identification of clusters...

# Core plotting functions[#](https://scanpy.scverse.org/en/stable/tutorials/plotting/core.html#core-plotting-functions)

**Author:**[Fidel Ramírez](https://github.com/fidelram)

This tutorial explores the visualization possibilities of scanpy and is divided into three sections:

- Scatter plots for embeddings (eg. UMAP, t-SNE)
- Identification of clusters using known marker genes
- Visualization of differentially expressed genes

In this tutorial, we will use a dataset from 10x containing 68k cells from [PBMC](https://support.10xgenomics.com/single-cell-gene-expression/datasets). Scanpy, includes in its distribution a reduced sample of this dataset consisting of only 700 cells and 765 highly variable genes. This dataset has been already preprocessed and UMAP computed.

In this tutorial, we will also use the following literature markers:

- **B-cell**: CD79A, MS4A1
- **Plasma**: IGJ (JCHAIN)
- **T-cell**: CD3D
- **NK**: GNLY, NKG7
- **Myeloid**: CST3, LYZ
- **Monocytes**: FCGR3A
- **Dendritic**: FCER1A

## Scatter plots for embeddings[#](https://scanpy.scverse.org/en/stable/tutorials/plotting/core.html#scatter-plots-for-embeddings)

With scanpy, scatter plots for tSNE, UMAP and several other embeddings are readily available using the `sc.pl.tsne`, `sc.pl.umap` etc. functions. See [Embeddings](https://scanpy.scverse.org/en/stable/api/plotting.html#pl-embeddings) for the list of options.

Those functions access the data stored in `adata.obsm`. For example `sc.pl.umap` uses the information stored in `adata.obsm['X_umap']`. For more flexibility, any key stored in `adata.obsm` can be used with the generic function [`scanpy.pl.embedding()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.embedding.html#scanpy.pl.embedding).

```
from __future__ import annotations

from matplotlib.pyplot import rc_context

import scanpy as sc
```

```
sc.set_figure_params(dpi=100, color_map="viridis_r")
sc.settings.verbosity = 0
sc.logging.print_header()
```

| Package | Version |
|---------|---------|
| matplotlib | 3.10.3 |
|  |  |
| Python | 3.13.2 | packaged by conda-forge | (main, Feb 17 2025, 14:10:22) [GCC 13.3.0] |
| OS | Linux-6.11.0-118026-tuxedo-x86_64-with-glibc2.39 |
| CPU | 16 logical CPU cores, x86_64 |
| GPU | No GPU found |
| Updated | 2025-06-06 16:28 |

Dependencies

| Dependency | Version |
|------------|---------|
| awkward_cpp | 45 |
| llvmlite | 0.44.0 |
| sphinxcontrib-htmlhelp | 2.1.0 |
| Jinja2 | 3.1.6 |
| executing | 2.2.0 |
| fsspec | 2025.3.2 |
| sphinx-jinja2-compat | 0.3.0 |
| charset-normalizer | 3.4.1 |
| sphinxcontrib-applehelp | 2.0.0 |
| packaging | 25.0 |
| comm | 0.2.2 |
| debugpy | 1.8.13 |
| parso | 0.8.4 |
| kiwisolver | 1.4.8 |
| numba | 0.61.2 |
| stack-data | 0.6.3 |
| traitlets | 5.14.3 |
| MarkupSafe | 3.0.2 |
| pytz | 2025.2 |
| numpy | 2.2.6 |
| crc32c | 2.7.1 |
| pandas | 2.3.0 |
| jupyter_client | 8.6.3 |
| natsort | 8.4.0 |
| tqdm | 4.67.1 |
| coverage | 7.8.0 |
| session-info2 | 0.1.2 |
| zarr | 3.0.8 |
| tblib | 3.1.0 |
| Pygments | 2.19.1 |
| joblib | 1.5.1 |
| sphinxcontrib-qthelp | 2.0.0 |
| wcwidth | 0.2.13 |
| donfig | 0.8.1.post1 |
| prompt_toolkit | 3.0.50 |
| ipykernel | 6.29.5 |
| sphinxcontrib-serializinghtml | 2.0.0 |
| scipy | 1.15.3 |
| ipython | 9.1.0 |
| threadpoolctl | 3.6.0 |
| PyYAML | 6.0.2 |
| platformdirs | 4.3.7 |
| cloudpickle | 3.1.1 |
| h5py | 3.14.0 |
| pyzmq | 26.4.0 |
| six | 1.17.0 |
| legacy-api-wrap | 1.4.1 |
| pyparsing | 3.2.3 |
| numcodecs | 0.16.1 |
| awkward | 2.8.1 |
| setuptools | 78.1.0 |
| scikit-learn | 1.7.0 |
| decorator | 5.2.1 |
| pure_eval | 0.2.3 |
| python-dateutil | 2.9.0.post0 |
| psutil | 7.0.0 |
| jedi | 0.19.2 |
| matplotlib-inline | 0.1.7 |
| jupyter_core | 5.7.2 |
| sphinxcontrib-devhelp | 2.0.0 |
| msgpack | 1.1.0 |
| fast-array-utils | 1.2.1 |
| dask | 2025.1.0 |
| typing_extensions | 4.14.0 |
| asttokens | 3.0.0 |
| toolz | 1.0.0 |
| sphinxcontrib-jsmath | 1.0.1 |
| tornado | 6.4.2 |
| cycler | 0.12.1 |
| torch | 2.6.0 (2.6.0+cu124) |
| pillow | 11.2.1 |
| pyarrow | 19.0.1 |

Copyable Markdown

```
| Package    | Version |
| ---------- | ------- |
| matplotlib | 3.10.3  |

| Dependency                    | Version             |
| ----------------------------- | ------------------- |
| awkward_cpp                   | 45                  |
| llvmlite                      | 0.44.0              |
| sphinxcontrib-htmlhelp        | 2.1.0               |
| Jinja2                        | 3.1.6               |
| executing                     | 2.2.0               |
| fsspec                        | 2025.3.2            |
| sphinx-jinja2-compat          | 0.3.0               |
| charset-normalizer            | 3.4.1               |
| sphinxcontrib-applehelp       | 2.0.0               |
| packaging                     | 25.0                |
| comm                          | 0.2.2               |
| debugpy                       | 1.8.13              |
| parso                         | 0.8.4               |
| kiwisolver                    | 1.4.8               |
| numba                         | 0.61.2              |
| stack-data                    | 0.6.3               |
| traitlets                     | 5.14.3              |
| MarkupSafe                    | 3.0.2               |
| pytz                          | 2025.2              |
| numpy                         | 2.2.6               |
| crc32c                        | 2.7.1               |
| pandas                        | 2.3.0               |
| jupyter_client                | 8.6.3               |
| natsort                       | 8.4.0               |
| tqdm                          | 4.67.1              |
| coverage                      | 7.8.0               |
| session-info2                 | 0.1.2               |
| zarr                          | 3.0.8               |
| tblib                         | 3.1.0               |
| Pygments                      | 2.19.1              |
| joblib                        | 1.5.1               |
| sphinxcontrib-qthelp          | 2.0.0               |
| wcwidth                       | 0.2.13              |
| donfig                        | 0.8.1.post1         |
| prompt_toolkit                | 3.0.50              |
| ipykernel                     | 6.29.5              |
| sphinxcontrib-serializinghtml | 2.0.0               |
| scipy                         | 1.15.3              |
| ipython                       | 9.1.0               |
| threadpoolctl                 | 3.6.0               |
| PyYAML                        | 6.0.2               |
| platformdirs                  | 4.3.7               |
| cloudpickle                   | 3.1.1               |
| h5py                          | 3.14.0              |
| pyzmq                         | 26.4.0              |
| six                           | 1.17.0              |
| legacy-api-wrap               | 1.4.1               |
| pyparsing                     | 3.2.3               |
| numcodecs                     | 0.16.1              |
| awkward                       | 2.8.1               |
| setuptools                    | 78.1.0              |
| scikit-learn                  | 1.7.0               |
| decorator                     | 5.2.1               |
| pure_eval                     | 0.2.3               |
| python-dateutil               | 2.9.0.post0         |
| psutil                        | 7.0.0               |
| jedi                          | 0.19.2              |
| matplotlib-inline             | 0.1.7               |
| jupyter_core                  | 5.7.2               |
| sphinxcontrib-devhelp         | 2.0.0               |
| msgpack                       | 1.1.0               |
| fast-array-utils              | 1.2.1               |
| dask                          | 2025.1.0            |
| typing_extensions             | 4.14.0              |
| asttokens                     | 3.0.0               |
| toolz                         | 1.0.0               |
| sphinxcontrib-jsmath          | 1.0.1               |
| tornado                       | 6.4.2               |
| cycler                        | 0.12.1              |
| torch                         | 2.6.0 (2.6.0+cu124) |
| pillow                        | 11.2.1              |
| pyarrow                       | 19.0.1              |

| Component | Info                                                                          |
| --------- | ----------------------------------------------------------------------------- |
| Python    | 3.13.2 | packaged by conda-forge | (main, Feb 17 2025, 14:10:22) [GCC 13.3.0] |
| OS        | Linux-6.11.0-118026-tuxedo-x86_64-with-glibc2.39                              |
| CPU       | 16 logical CPU cores, x86_64                                                  |
| GPU       | No GPU found                                                                  |
| Updated   | 2025-06-06 16:28                                                              |
```

### Load pbmc dataset[#](https://scanpy.scverse.org/en/stable/tutorials/plotting/core.html#load-pbmc-dataset)

```
pbmc = sc.datasets.pbmc68k_reduced()
```

```
# inspect pbmc contents
pbmc
```

```
AnnData object with n_obs × n_vars = 700 × 765
    obs: 'bulk_labels', 'n_genes', 'percent_mito', 'n_counts', 'S_score', 'G2M_score', 'phase', 'louvain'
    var: 'n_counts', 'means', 'dispersions', 'dispersions_norm', 'highly_variable'
    uns: 'bulk_labels_colors', 'louvain', 'louvain_colors', 'neighbors', 'pca', 'rank_genes_groups'
    obsm: 'X_pca', 'X_umap'
    varm: 'PCs'
    obsp: 'distances', 'connectivities'
```

### Visualization of gene expression and other variables[#](https://scanpy.scverse.org/en/stable/tutorials/plotting/core.html#visualization-of-gene-expression-and-other-variables)

For the scatter plots, the value to plot is given as the `color` argument. This can be any gene or any column in `.obs`, where `.obs` is a DataFrame containing the annotations per observation/cell, see [anndata - Annotated data](https://anndata.scverse.org/en/stable/index.html) for more information.

```
# rc_context is used for the figure size, in this case 4x4
with rc_context({"figure.figsize": (4, 4)}):
    sc.pl.umap(pbmc, color="CD79A")
```

[![../../_images/3a82d437cedadd52481ef022d5d5ac673e0870db60f66a000e7c06be9804b29d.png](https://scanpy.scverse.org/en/stable/_images/3a82d437cedadd52481ef022d5d5ac673e0870db60f66a000e7c06be9804b29d.png)](https://scanpy.scverse.org/en/stable/_images/3a82d437cedadd52481ef022d5d5ac673e0870db60f66a000e7c06be9804b29d.png)

Multiple values can be given to `color`. In the following example we will plot 6 genes: ‘CD79A’, ‘MS4A1’, ‘IGJ’, CD3D’, ‘FCER1A’, and ‘FCGR3A’ to get an idea on where those marker genes are being expressed.

Also, we will plot two other values: `n_counts` which is the number of UMI counts per cell (stored in `.obs`), and `bulk_labels` which is a categorical value containing the original labelling of the cells from 10X.

The number of plots per row is controlled using the `ncols` parameter. The maximum value plotted can be adjusted using `vmax` (similarly `vmin` can be used for the minimum value). In this case we use `p99`, which means to use as max value the 99 percentile. The max value can be a number or a list of numbers if the vmax wants to be set for multiple plots individually.

Also, we are using `frameon=False` to remove the boxes around the plots and `s=50` to set the dot size.

```
color_vars = [
    "CD79A",
    "MS4A1",
    "IGJ",
    "CD3D",
    "FCER1A",
    "FCGR3A",
    "n_counts",
    "bulk_labels",
]
with rc_context({"figure.figsize": (3, 3)}):
    sc.pl.umap(pbmc, color=color_vars, s=50, frameon=False, ncols=4, vmax="p99")
```

[![../../_images/a04173d282b7225af891853648454277c62aa658e3ed12b0e5a82cbdb2bc1bbf.png](https://scanpy.scverse.org/en/stable/_images/a04173d282b7225af891853648454277c62aa658e3ed12b0e5a82cbdb2bc1bbf.png)](https://scanpy.scverse.org/en/stable/_images/a04173d282b7225af891853648454277c62aa658e3ed12b0e5a82cbdb2bc1bbf.png)

In this plot we can see the groups of cells that express the marker genes and the agreement with the original cell labels.

The functions for scatterplots have many options that allow fine tuning of the images. For example, we can look at the clustering as follows:

```
# compute clusters using the leiden method and store the results with the name `clusters`
sc.tl.leiden(
    pbmc,
    key_added="clusters",
    resolution=0.5,
    n_iterations=2,
    flavor="igraph",
    directed=False,
)
```

```
with rc_context({"figure.figsize": (5, 5)}):
    sc.pl.umap(
        pbmc,
        color="clusters",
        add_outline=True,
        legend_loc="on data",
        legend_fontsize=12,
        legend_fontoutline=2,
        frameon=False,
        title="clustering of cells",
        palette="Set1",
    )
```

[![../../_images/c1e530b06deed8e1a2861304d545d88ceaa563443748219fc5782a11bfed8bab.png](https://scanpy.scverse.org/en/stable/_images/c1e530b06deed8e1a2861304d545d88ceaa563443748219fc5782a11bfed8bab.png)](https://scanpy.scverse.org/en/stable/_images/c1e530b06deed8e1a2861304d545d88ceaa563443748219fc5782a11bfed8bab.png)

## Identification of clusters based on known marker genes[#](https://scanpy.scverse.org/en/stable/tutorials/plotting/core.html#identification-of-clusters-based-on-known-marker-genes)

Frequently, clusters need to be labelled using well known marker genes. Using scatter plots we can see the expression of a gene and perhaps associate it with a cluster. Here, we will show other visual ways to associate marker genes to clusters using *dotplots*, *violin plots*, *heatmaps* and something that we call ‘*tracksplot*’. All of these visualizations summarize the same information, expression split by cluster, and the selection of the best results is left to the investigator do decide.

First, we set up a dictionary with the marker genes, as this will allow scanpy to automatically label the groups of genes:

```
marker_genes_dict = {
    "B-cell": ["CD79A", "MS4A1"],
    "Dendritic": ["FCER1A", "CST3"],
    "Monocytes": ["FCGR3A"],
    "NK": ["GNLY", "NKG7"],
    "Other": ["IGLL1"],
    "Plasma": ["IGJ"],
    "T-cell": ["CD3D"],
}
```

### dotplot[#](https://scanpy.scverse.org/en/stable/tutorials/plotting/core.html#dotplot)

A quick way to check the expression of these genes per cluster is to using a dotplot. This type of plot summarizes two types of information: the color represents the mean expression within each of the categories (in this case in each cluster) and the dot size indicates the fraction of cells in the categories expressing a gene.

Also, it is also useful to add a dendrogram to the graph to bring together similar clusters. The hierarchical clustering is computed automatically using the correlation of the PCA components between the clusters.

```
sc.pl.dotplot(pbmc, marker_genes_dict, "clusters", dendrogram=True)
```

[![../../_images/2c9701ef266c8b3b9490dcb11623e4e2a3d69eaadb9f187953a8c0613a65ad00.png](https://scanpy.scverse.org/en/stable/_images/2c9701ef266c8b3b9490dcb11623e4e2a3d69eaadb9f187953a8c0613a65ad00.png)](https://scanpy.scverse.org/en/stable/_images/2c9701ef266c8b3b9490dcb11623e4e2a3d69eaadb9f187953a8c0613a65ad00.png)

Using this plot, we can see that cluster 4 correspond to B-cells, cluster 2 is T-cells etc. This information can be used to manually annotate the cells as follows:

```
# create a dictionary to map cluster to annotation label
cluster2annotation = {
    "0": "Monocytes",
    "1": "NK",
    "2": "T-cell",
    "3": "Dendritic",
    "4": "Dendritic",
    "5": "Plasma",
    "6": "B-cell",
    "7": "Dendritic",
    "8": "Other",
}

# add a new `.obs` column called `cell type` by mapping clusters to annotation using pandas `map` function
pbmc.obs["cell type"] = pbmc.obs["clusters"].map(cluster2annotation).astype("category")
```

```
sc.pl.dotplot(pbmc, marker_genes_dict, "cell type", dendrogram=True)
```

[![../../_images/d4f91a929016b982fdee738f747f0a8d1809c43e7e3d33245259adb7b1338791.png](https://scanpy.scverse.org/en/stable/_images/d4f91a929016b982fdee738f747f0a8d1809c43e7e3d33245259adb7b1338791.png)](https://scanpy.scverse.org/en/stable/_images/d4f91a929016b982fdee738f747f0a8d1809c43e7e3d33245259adb7b1338791.png)

```
sc.pl.umap(
    pbmc,
    color="cell type",
    legend_loc="on data",
    frameon=False,
    legend_fontsize=10,
    legend_fontoutline=2,
)
```

[![../../_images/b083f58cdecea3e188f5448696676033caad5295288a66e6200fe35079cd4cec.png](https://scanpy.scverse.org/en/stable/_images/b083f58cdecea3e188f5448696676033caad5295288a66e6200fe35079cd4cec.png)](https://scanpy.scverse.org/en/stable/_images/b083f58cdecea3e188f5448696676033caad5295288a66e6200fe35079cd4cec.png)

### violin plot[#](https://scanpy.scverse.org/en/stable/tutorials/plotting/core.html#violin-plot)

A different way to explore the markers is with violin plots. Here we can see the expression of *CD79A* in clusters 5 and 8, and *MS4A1* in cluster 5.Compared to a dotplot, the violin plot gives us and idea of the distribution of gene expression values across cells.

```
with rc_context({"figure.figsize": (4.5, 3)}):
    sc.pl.violin(pbmc, ["CD79A", "MS4A1"], groupby="clusters")
```

[![../../_images/ab8ceac0ade73d69c35dae952929be9a16a21b7e5a9612e76511dcf341274cad.png](https://scanpy.scverse.org/en/stable/_images/ab8ceac0ade73d69c35dae952929be9a16a21b7e5a9612e76511dcf341274cad.png)](https://scanpy.scverse.org/en/stable/_images/ab8ceac0ade73d69c35dae952929be9a16a21b7e5a9612e76511dcf341274cad.png)

**Note** Violin plots can also be used to plot any numerical value stored in `.obs`. For example, here violin plots are used to compare the number of genes and the percentage of mitochondrial genes between the different clusters.

```
with rc_context({"figure.figsize": (4.5, 3)}):
    sc.pl.violin(
        pbmc,
        ["n_genes", "percent_mito"],
        groupby="clusters",
        stripplot=False,  # remove the internal dots
        inner="box",  # adds a boxplot inside violins
    )
```

[![../../_images/d9dcabc0cdd72d9908378b6689153e8b23c803c4078439c92b6aeb3e47787a58.png](https://scanpy.scverse.org/en/stable/_images/d9dcabc0cdd72d9908378b6689153e8b23c803c4078439c92b6aeb3e47787a58.png)](https://scanpy.scverse.org/en/stable/_images/d9dcabc0cdd72d9908378b6689153e8b23c803c4078439c92b6aeb3e47787a58.png)

### stacked-violin plot[#](https://scanpy.scverse.org/en/stable/tutorials/plotting/core.html#stacked-violin-plot)

To simultaneously look at the violin plots for all marker genes we use `sc.pl.stacked_violin`. As previously, a dendrogram was added to group similar clusters

```
ax = sc.pl.stacked_violin(
    pbmc, marker_genes_dict, groupby="clusters", swap_axes=False, dendrogram=True
)
```

[![../../_images/08940b0dd5b929575cafae1e9d952d7177831ff7796a50edc2331e78b7ec073d.png](https://scanpy.scverse.org/en/stable/_images/08940b0dd5b929575cafae1e9d952d7177831ff7796a50edc2331e78b7ec073d.png)](https://scanpy.scverse.org/en/stable/_images/08940b0dd5b929575cafae1e9d952d7177831ff7796a50edc2331e78b7ec073d.png)

### matrixplot[#](https://scanpy.scverse.org/en/stable/tutorials/plotting/core.html#matrixplot)

A simple way to visualize the expression of genes is with a `matrixplot`. This is a heatmap of the mean expression values per gene grouped by categories. This type plot basically shows the same information as the color in the dotplots.

Here, scale the expression of the genes from 0 to 1, being the maximum mean expression and 0 the minimum.

```
sc.pl.matrixplot(
    pbmc,
    marker_genes_dict,
    "clusters",
    dendrogram=True,
    cmap="Blues",
    standard_scale="var",
    colorbar_title="column scaled\nexpression",
)
```

[![../../_images/352f3ecd06ce5d90c523472f90e001df88d69becc556dba6dfd4aaaceea7e1f4.png](https://scanpy.scverse.org/en/stable/_images/352f3ecd06ce5d90c523472f90e001df88d69becc556dba6dfd4aaaceea7e1f4.png)](https://scanpy.scverse.org/en/stable/_images/352f3ecd06ce5d90c523472f90e001df88d69becc556dba6dfd4aaaceea7e1f4.png)

Other useful option is to normalize the gene expression using `sc.pp.scale`. Here we store this information under the `scale` layer. Afterwards we adjust the plot min and max and use a diverging color map (in this case `RdBu_r` where `_r` means reversed).

```
# scale and store results in layer
pbmc.layers["scaled"] = sc.pp.scale(pbmc, copy=True).X
```

```
sc.pl.matrixplot(
    pbmc,
    marker_genes_dict,
    "clusters",
    dendrogram=True,
    colorbar_title="mean z-score",
    layer="scaled",
    vmin=-2,
    vmax=2,
    cmap="RdBu_r",
)
```

[![../../_images/4d4980d62f791299b4db8748e42969abcf258c24a8e16b6dbc4c4f0b6f3c0954.png](https://scanpy.scverse.org/en/stable/_images/4d4980d62f791299b4db8748e42969abcf258c24a8e16b6dbc4c4f0b6f3c0954.png)](https://scanpy.scverse.org/en/stable/_images/4d4980d62f791299b4db8748e42969abcf258c24a8e16b6dbc4c4f0b6f3c0954.png)

## Combining plots in subplots[#](https://scanpy.scverse.org/en/stable/tutorials/plotting/core.html#combining-plots-in-subplots)

An `axis` can be passed to a plot to combine multiple outputs as in the following example

```
import matplotlib.pyplot as plt

fig, (ax1, ax2, ax3) = plt.subplots(1, 3, figsize=(20, 4), gridspec_kw={"wspace": 0.9})

ax1_dict = sc.pl.dotplot(
    pbmc, marker_genes_dict, groupby="bulk_labels", ax=ax1, show=False
)
ax2_dict = sc.pl.stacked_violin(
    pbmc, marker_genes_dict, groupby="bulk_labels", ax=ax2, show=False
)
ax3_dict = sc.pl.matrixplot(
    pbmc, marker_genes_dict, groupby="bulk_labels", ax=ax3, show=False, cmap="viridis"
)
```

[![../../_images/6fc7410a8fe298e3f12b23877016321dc26fc5084064a189722cb82abb675c57.png](https://scanpy.scverse.org/en/stable/_images/6fc7410a8fe298e3f12b23877016321dc26fc5084064a189722cb82abb675c57.png)](https://scanpy.scverse.org/en/stable/_images/6fc7410a8fe298e3f12b23877016321dc26fc5084064a189722cb82abb675c57.png)

## Heatmaps[#](https://scanpy.scverse.org/en/stable/tutorials/plotting/core.html#heatmaps)

Heatmaps do not collapse cells as in previous plots. Instead, each cells is shown in a row (or column if `swap_axes=True`). The groupby information can be added and is shown using the same color code found for `sc.pl.umap` or any other embedding.

```
ax = sc.pl.heatmap(
    pbmc, marker_genes_dict, groupby="clusters", cmap="viridis", dendrogram=True
)
```

[![../../_images/fc2449724f50fc3a4891f902f64112fc8d709eb0c11b3ea02648dfba0b8b49cb.png](https://scanpy.scverse.org/en/stable/_images/fc2449724f50fc3a4891f902f64112fc8d709eb0c11b3ea02648dfba0b8b49cb.png)](https://scanpy.scverse.org/en/stable/_images/fc2449724f50fc3a4891f902f64112fc8d709eb0c11b3ea02648dfba0b8b49cb.png)

The heatmap can also be plotted on scaled data. In the next image, similar to the previus matrixplot the min and max had been adjusted and a divergent color map is used.

```
ax = sc.pl.heatmap(
    pbmc,
    marker_genes_dict,
    groupby="clusters",
    layer="scaled",
    vmin=-2,
    vmax=2,
    cmap="RdBu_r",
    dendrogram=True,
    swap_axes=True,
    figsize=(11, 4),
)
```

[![../../_images/2991fe558766f086411c6f1949c4c664c819a0ed608a34f7d3f878fb2a8bd46d.png](https://scanpy.scverse.org/en/stable/_images/2991fe558766f086411c6f1949c4c664c819a0ed608a34f7d3f878fb2a8bd46d.png)](https://scanpy.scverse.org/en/stable/_images/2991fe558766f086411c6f1949c4c664c819a0ed608a34f7d3f878fb2a8bd46d.png)

## Tracksplot[#](https://scanpy.scverse.org/en/stable/tutorials/plotting/core.html#tracksplot)

The track plot shows the same information as the heatmap, but, instead of a color scale, the gene expression is represented by height.

```
ax = sc.pl.tracksplot(pbmc, marker_genes_dict, groupby="clusters", dendrogram=True)
```

[![../../_images/dd2fab22e25749d072c93ec385a1f448cf3fb7665eb2f992c9515ebdea1d4e70.png](https://scanpy.scverse.org/en/stable/_images/dd2fab22e25749d072c93ec385a1f448cf3fb7665eb2f992c9515ebdea1d4e70.png)](https://scanpy.scverse.org/en/stable/_images/dd2fab22e25749d072c93ec385a1f448cf3fb7665eb2f992c9515ebdea1d4e70.png)

## Visualization of marker genes[#](https://scanpy.scverse.org/en/stable/tutorials/plotting/core.html#visualization-of-marker-genes)

Instead of characterizing clusters by known gene markers as previously, we can identify genes that are differentially expressed in the clusters or groups.

To identify differentially expressed genes we run `sc.tl.rank_genes_groups`. This function will take each group of cells and compare the distribution of each gene in a group against the distribution in all other cells not in the group. Here, we will use the original cell labels given by 10x to identify marker genes for those cell types.

```
sc.tl.rank_genes_groups(pbmc, groupby="clusters", method="wilcoxon")
```

### Visualize marker genes using dotplot[#](https://scanpy.scverse.org/en/stable/tutorials/plotting/core.html#visualize-marker-genes-using-dotplot)

The dotplot visualization is useful to get an overview of the genes that show differential expression. To make the resulting image more compact we will use `n_genes=4` to show only the top 4 scoring genes.

```
sc.pl.rank_genes_groups_dotplot(pbmc, n_genes=4)
```

[![../../_images/2fdf057d113584ecc12239fb66397e9636b946d24fd9f6250b8e189a3c2dd238.png](https://scanpy.scverse.org/en/stable/_images/2fdf057d113584ecc12239fb66397e9636b946d24fd9f6250b8e189a3c2dd238.png)](https://scanpy.scverse.org/en/stable/_images/2fdf057d113584ecc12239fb66397e9636b946d24fd9f6250b8e189a3c2dd238.png)

In order to get a better representation we can plot *log fold changes* instead of gene expression. Also, we want to focus on genes that have a log fold change &gt;= 3 between the cell type expression and the rest of cells.

In this case we set `values_to_plot='logfoldchanges'` and `min_logfoldchange=3`.

Because log fold change is a divergent scale we also adjust the min and max to be plotted and use a divergent color map. Notice in the following plot that is rather difficult to distinguish between T-cell populations.

```
sc.pl.rank_genes_groups_dotplot(
    pbmc,
    n_genes=4,
    values_to_plot="logfoldchanges",
    min_logfoldchange=3,
    vmax=7,
    vmin=-7,
    cmap="bwr",
)
```

[![../../_images/bcf01e7a13a626a9d44a9f87b1a2c719f829981b46ec34b7f6ec6942e411ec64.png](https://scanpy.scverse.org/en/stable/_images/bcf01e7a13a626a9d44a9f87b1a2c719f829981b46ec34b7f6ec6942e411ec64.png)](https://scanpy.scverse.org/en/stable/_images/bcf01e7a13a626a9d44a9f87b1a2c719f829981b46ec34b7f6ec6942e411ec64.png)

#### Focusing on particular groups[#](https://scanpy.scverse.org/en/stable/tutorials/plotting/core.html#focusing-on-particular-groups)

Next, we use a dotplot focusing only on two groups (the groups option is also available for violin, heatmap and matrix plots). Here, we set `n_genes=30` as in this case it will show all the genes that have a `min_logfoldchange=4` up to 30.

```
sc.pl.rank_genes_groups_dotplot(
    pbmc,
    n_genes=30,
    values_to_plot="logfoldchanges",
    min_logfoldchange=4,
    vmax=7,
    vmin=-7,
    cmap="bwr",
    groups=["3", "7"],
)
```

[![../../_images/4e39081732f70423300980f2b389aa5ae21497dc3f9446681e22f5a8a1525a6f.png](https://scanpy.scverse.org/en/stable/_images/4e39081732f70423300980f2b389aa5ae21497dc3f9446681e22f5a8a1525a6f.png)](https://scanpy.scverse.org/en/stable/_images/4e39081732f70423300980f2b389aa5ae21497dc3f9446681e22f5a8a1525a6f.png)

### Visualize marker genes using matrixplot[#](https://scanpy.scverse.org/en/stable/tutorials/plotting/core.html#visualize-marker-genes-using-matrixplot)

For the following plot the we use the previously computed ‘scaled’ values (stored in layer `scaled`) and use a divergent color map.

```
sc.pl.rank_genes_groups_matrixplot(
    pbmc, n_genes=3, use_raw=False, vmin=-3, vmax=3, cmap="bwr", layer="scaled"
)
```

[![../../_images/50b2b99a1a055abb06df268721d9be07613f992127cda844f616625219955568.png](https://scanpy.scverse.org/en/stable/_images/50b2b99a1a055abb06df268721d9be07613f992127cda844f616625219955568.png)](https://scanpy.scverse.org/en/stable/_images/50b2b99a1a055abb06df268721d9be07613f992127cda844f616625219955568.png)

### Visualize marker genes using stacked violin plots[#](https://scanpy.scverse.org/en/stable/tutorials/plotting/core.html#visualize-marker-genes-using-stacked-violin-plots)

```
sc.pl.rank_genes_groups_stacked_violin(pbmc, n_genes=3, cmap="viridis_r")
```

[![../../_images/92c8c115b1c591c74b8cb6e9d0835a859d6707b376d57f5b4f946258c9fdd8a4.png](https://scanpy.scverse.org/en/stable/_images/92c8c115b1c591c74b8cb6e9d0835a859d6707b376d57f5b4f946258c9fdd8a4.png)](https://scanpy.scverse.org/en/stable/_images/92c8c115b1c591c74b8cb6e9d0835a859d6707b376d57f5b4f946258c9fdd8a4.png)

### Visualize marker genes using heatmap[#](https://scanpy.scverse.org/en/stable/tutorials/plotting/core.html#visualize-marker-genes-using-heatmap)

```
sc.pl.rank_genes_groups_heatmap(
    pbmc,
    n_genes=3,
    use_raw=False,
    swap_axes=True,
    vmin=-3,
    vmax=3,
    cmap="bwr",
    layer="scaled",
    figsize=(10, 7),
    show=False,
);
```

[![../../_images/c8a97f882cf8e155013d4ce88b30b12db459333f31fee35eb80d12633c02cddd.png](https://scanpy.scverse.org/en/stable/_images/c8a97f882cf8e155013d4ce88b30b12db459333f31fee35eb80d12633c02cddd.png)](https://scanpy.scverse.org/en/stable/_images/c8a97f882cf8e155013d4ce88b30b12db459333f31fee35eb80d12633c02cddd.png)

Showing 10 genes per category, turning the gene labels off and swapping the axes. Notice that when the image is swapped, a color code for the categories appear instead of the ‘brackets’.

```
sc.pl.rank_genes_groups_heatmap(
    pbmc,
    n_genes=10,
    use_raw=False,
    swap_axes=True,
    show_gene_labels=False,
    vmin=-3,
    vmax=3,
    cmap="bwr",
)
```

[![../../_images/1653ef8b3d91e89e509984dd8cc5d19145350beaedaaab86a3a6c07672a28ef6.png](https://scanpy.scverse.org/en/stable/_images/1653ef8b3d91e89e509984dd8cc5d19145350beaedaaab86a3a6c07672a28ef6.png)](https://scanpy.scverse.org/en/stable/_images/1653ef8b3d91e89e509984dd8cc5d19145350beaedaaab86a3a6c07672a28ef6.png)

### Visualize marker genes using tracksplot[#](https://scanpy.scverse.org/en/stable/tutorials/plotting/core.html#visualize-marker-genes-using-tracksplot)

```
sc.pl.rank_genes_groups_tracksplot(pbmc, n_genes=3)
```

[![../../_images/052f07fe780438b12d496b3ddaed3f958cc22f5a321991ae7ae593c773a35b76.png](https://scanpy.scverse.org/en/stable/_images/052f07fe780438b12d496b3ddaed3f958cc22f5a321991ae7ae593c773a35b76.png)](https://scanpy.scverse.org/en/stable/_images/052f07fe780438b12d496b3ddaed3f958cc22f5a321991ae7ae593c773a35b76.png)

## Comparison of marker genes using split violin plots[#](https://scanpy.scverse.org/en/stable/tutorials/plotting/core.html#comparison-of-marker-genes-using-split-violin-plots)

In scanpy, is very easy to compare marker genes using split violin plots for all groups at once.

```
with rc_context({"figure.figsize": (9, 1.5)}):
    sc.pl.rank_genes_groups_violin(pbmc, n_genes=20, jitter=False)
```

[![../../_images/e679f7ec42621cd951b12ed129b4f6474d24b5043fcb2fecdc96479a77531681.png](https://scanpy.scverse.org/en/stable/_images/e679f7ec42621cd951b12ed129b4f6474d24b5043fcb2fecdc96479a77531681.png)](https://scanpy.scverse.org/en/stable/_images/e679f7ec42621cd951b12ed129b4f6474d24b5043fcb2fecdc96479a77531681.png)[![../../_images/93488e44a5f2084884177480f5d1f8f1dbe87f53b44d9f1cc7058b983ff35938.png](https://scanpy.scverse.org/en/stable/_images/93488e44a5f2084884177480f5d1f8f1dbe87f53b44d9f1cc7058b983ff35938.png)](https://scanpy.scverse.org/en/stable/_images/93488e44a5f2084884177480f5d1f8f1dbe87f53b44d9f1cc7058b983ff35938.png)[![../../_images/f7e0871712a60e23b35c9f3379edb644d0dfeb0fd817a7e126b69d519253307f.png](https://scanpy.scverse.org/en/stable/_images/f7e0871712a60e23b35c9f3379edb644d0dfeb0fd817a7e126b69d519253307f.png)](https://scanpy.scverse.org/en/stable/_images/f7e0871712a60e23b35c9f3379edb644d0dfeb0fd817a7e126b69d519253307f.png)[![../../_images/72ca059296ecfe466026a5012c4c135420684573ae6e42aabc2968efb1290002.png](https://scanpy.scverse.org/en/stable/_images/72ca059296ecfe466026a5012c4c135420684573ae6e42aabc2968efb1290002.png)](https://scanpy.scverse.org/en/stable/_images/72ca059296ecfe466026a5012c4c135420684573ae6e42aabc2968efb1290002.png)[![../../_images/2a280bd3e22958f78430cfd4c41d5d37b44469e45a82ac05f5ccbca717b17457.png](https://scanpy.scverse.org/en/stable/_images/2a280bd3e22958f78430cfd4c41d5d37b44469e45a82ac05f5ccbca717b17457.png)](https://scanpy.scverse.org/en/stable/_images/2a280bd3e22958f78430cfd4c41d5d37b44469e45a82ac05f5ccbca717b17457.png)[![../../_images/daa55f3a8d5cc51906bb0897a81840e0c18cc86a174bf3eb891139934c50140d.png](https://scanpy.scverse.org/en/stable/_images/daa55f3a8d5cc51906bb0897a81840e0c18cc86a174bf3eb891139934c50140d.png)](https://scanpy.scverse.org/en/stable/_images/daa55f3a8d5cc51906bb0897a81840e0c18cc86a174bf3eb891139934c50140d.png)[![../../_images/f9429a2e1aaf6d889692fb40258efd524f12209e521b2b03f95b222f00c9e6df.png](https://scanpy.scverse.org/en/stable/_images/f9429a2e1aaf6d889692fb40258efd524f12209e521b2b03f95b222f00c9e6df.png)](https://scanpy.scverse.org/en/stable/_images/f9429a2e1aaf6d889692fb40258efd524f12209e521b2b03f95b222f00c9e6df.png)[![../../_images/5cfbe9eb92b9a937a933a0a604a872e7914012b0a7d2008e62bf8363a6cd5b42.png](https://scanpy.scverse.org/en/stable/_images/5cfbe9eb92b9a937a933a0a604a872e7914012b0a7d2008e62bf8363a6cd5b42.png)](https://scanpy.scverse.org/en/stable/_images/5cfbe9eb92b9a937a933a0a604a872e7914012b0a7d2008e62bf8363a6cd5b42.png)[![../../_images/600e370a6717ab2f4fea7b390eb08c45b8a20a5e3d0ad61a09a06adaed6e81d4.png](https://scanpy.scverse.org/en/stable/_images/600e370a6717ab2f4fea7b390eb08c45b8a20a5e3d0ad61a09a06adaed6e81d4.png)](https://scanpy.scverse.org/en/stable/_images/600e370a6717ab2f4fea7b390eb08c45b8a20a5e3d0ad61a09a06adaed6e81d4.png)

## Dendrogram options[#](https://scanpy.scverse.org/en/stable/tutorials/plotting/core.html#dendrogram-options)

Most of the visualizations can arrange the categories using a dendrogram. However, the dendrogram can also be plotted independently as follows:

```
# compute hierarchical clustering using PCs (several distance metrics and linkage methods are available).
sc.tl.dendrogram(pbmc, "bulk_labels")
```

```
ax = sc.pl.dendrogram(pbmc, "bulk_labels")
```

[![../../_images/4b1be5ff87b37ad710d192a2dd9b53793c01f155b84f1d13932229b4583f7a37.png](https://scanpy.scverse.org/en/stable/_images/4b1be5ff87b37ad710d192a2dd9b53793c01f155b84f1d13932229b4583f7a37.png)](https://scanpy.scverse.org/en/stable/_images/4b1be5ff87b37ad710d192a2dd9b53793c01f155b84f1d13932229b4583f7a37.png)

## Plot correlation[#](https://scanpy.scverse.org/en/stable/tutorials/plotting/core.html#plot-correlation)

Together with the dendrogram it is possible to plot the correlation (by default ‘pearson’) of the categories.

```
ax = sc.pl.correlation_matrix(pbmc, "bulk_labels", figsize=(5, 3.5))
```

[![../../_images/c018c4b6b1363e6601efbaa0dc27c3f9e12cdddfeb15d75739291ddf554fa91e.png](https://scanpy.scverse.org/en/stable/_images/c018c4b6b1363e6601efbaa0dc27c3f9e12cdddfeb15d75739291ddf554fa91e.png)](https://scanpy.scverse.org/en/stable/_images/c018c4b6b1363e6601efbaa0dc27c3f9e12cdddfeb15d75739291ddf554fa91e.png)

## Saving plots[#](https://scanpy.scverse.org/en/stable/tutorials/plotting/core.html#saving-plots)

You can save a plot by directly accessing the returned Matplotlib figure:

```
sc.pl.umap(pbmc, color="CD79A", show=False).figure.savefig("CD79A.png")
```

[![../../_images/3a82d437cedadd52481ef022d5d5ac673e0870db60f66a000e7c06be9804b29d.png](https://scanpy.scverse.org/en/stable/_images/3a82d437cedadd52481ef022d5d5ac673e0870db60f66a000e7c06be9804b29d.png)](https://scanpy.scverse.org/en/stable/_images/3a82d437cedadd52481ef022d5d5ac673e0870db60f66a000e7c06be9804b29d.png)
