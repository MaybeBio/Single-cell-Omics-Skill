# Plotting: pl

> The plotting module scanpy.pl largely parallels the tl.* and a few of the pp.* functions. For most tools and for some preprocessing functions, you’ll find a plotting function with the same name. Se...

# Plotting: `pl`[#](https://scanpy.scverse.org/en/stable/api/plotting.html#module-scanpy.pl)

The plotting module [`scanpy.pl`](https://scanpy.scverse.org/en/stable/api/plotting.html#module-scanpy.pl) largely parallels the `tl.*` and a few of the `pp.*` functions. For most tools and for some preprocessing functions, you’ll find a plotting function with the same name.

See [Core plotting functions](https://scanpy.scverse.org/en/stable/tutorials/plotting/core.html) for an overview of how to use these functions.

Note

See the [Settings](https://scanpy.scverse.org/en/stable/api/settings.html#settings) section for all important plotting configurations.

## Generic[#](https://scanpy.scverse.org/en/stable/api/plotting.html#pl-generic)

Functions with both backends:

| pl.heatmap | Heatmap of the expression values of genes. |
|------------|--------------------------------------------|
| pl.dotplot | Make adot plotof the expression values ofvar_names. |
| pl.tracksplot | Compact plot of expression of a list of genes. |
| pl.violin | Violin plot. |
| pl.stacked_violin | Stacked violin plots. |
| pl.matrixplot | Create a heatmap of the mean expression values per group of each var_names. |
| pl.ranking | Plot rankings. |

Helper for computing dot areas, e.g. for [`dotplot()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.dotplot.html#scanpy.pl.dotplot):

|  |
|--|

Legacy (matplotlib) only:

| pl.dendrogram | Plot a dendrogram of the categories defined ingroupby. |
|---------------|--------------------------------------------------------|

## Classes[#](https://scanpy.scverse.org/en/stable/api/plotting.html#classes)

These classes allow fine tuning of visual parameters.

| pl.MatrixPlot | Allows the visualization of values using a color map. |
|---------------|-------------------------------------------------------|
| pl.StackedViolin | Stacked violin plots. |

## Preprocessing[#](https://scanpy.scverse.org/en/stable/api/plotting.html#preprocessing)

Methods for visualizing quality control and results of preprocessing functions.

| pl.highly_variable_genes | Plot dispersions or normalized variance versus means for genes. |
|--------------------------|-----------------------------------------------------------------|
| pl.scrublet_score_distribution | Plot histogram of doublet scores for observed transcriptomes and simulated doublets. |

## Tools[#](https://scanpy.scverse.org/en/stable/api/plotting.html#tools)

Methods that extract and visualize tool-specific annotation in an [`AnnData`](https://anndata.scverse.org/en/stable/generated/anndata.AnnData.html#anndata.AnnData) object. For any method in module `tl`, there is a method with the same name in `pl`.

### PCA[#](https://scanpy.scverse.org/en/stable/api/plotting.html#pca)

| pl.ranking | Plot rankings. |
|------------|----------------|

[`ranking()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.ranking.html#scanpy.pl.ranking) replaces [`pca_loadings()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.pca_loadings.html#scanpy.pl.pca_loadings) and [`pca_variance_ratio()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.pca_variance_ratio.html#scanpy.pl.pca_variance_ratio) in the new (HoloViews) backend.

| pl.pca_variance_ratio | Plot the variance ratio. |
|-----------------------|--------------------------|
| pl.pca_overview | Plot PCA results. |

### Embeddings[#](https://scanpy.scverse.org/en/stable/api/plotting.html#pl-embeddings)

Functions with both backends:

| pl.tsne | Scatter plot in tSNE basis. |
|---------|-----------------------------|
| pl.diffmap | Scatter plot in Diffusion Map basis. |
| pl.draw_graph | Scatter plot in graph-drawing basis. |

Compute densities on embeddings.

|  |
|--|

Legacy (matplotlib) only:

| pl.embedding | Scatter plot for user specified embedding basis (e.g. umap, pca, etc). |
|--------------|------------------------------------------------------------------------|

### Branching trajectories and pseudotime, clustering[#](https://scanpy.scverse.org/en/stable/api/plotting.html#branching-trajectories-and-pseudotime-clustering)

Visualize clusters using one of the embedding methods passing e.g. `color='leiden'`.

| pl.dpt_timeseries | Heatmap of pseudotime series. |
|-------------------|-------------------------------|
| pl.paga | Plot the PAGA graph through thresholding low-connectivity edges. |
| pl.paga_path | Gene expression and annotation changes along paths in the abstracted graph. |
| pl.paga_compare | Scatter and PAGA graph side-by-side. |

Visualize hierarchical clustering results as a heatmap.

|  |
|--|

### Marker genes[#](https://scanpy.scverse.org/en/stable/api/plotting.html#marker-genes)

| pl.rank_genes_groups_violin | Plot ranking of genes for all tested comparisons. |
|-----------------------------|---------------------------------------------------|
| pl.rank_genes_groups_stacked_violin | Plot ranking of genes using stacked_violin plot. |
| pl.rank_genes_groups_heatmap | Plot ranking of genes using heatmap plot (seeheatmap()). |
| pl.rank_genes_groups_dotplot | Plot ranking of genes using dotplot plot (seedotplot()). |
| pl.rank_genes_groups_matrixplot | Plot ranking of genes using matrixplot plot (seematrixplot()). |
| pl.rank_genes_groups_tracksplot | Plot ranking of genes using heatmap plot (seeheatmap()). |
