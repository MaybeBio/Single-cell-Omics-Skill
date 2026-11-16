# Preprocessing: pp

> Filtering of highly-variable genes, batch-effect correction, per-cell normalization, preprocessing recipes. Any transformation of the data matrix that is not a tool. Other than tools, preprocessing...

# Preprocessing: `pp`[#](https://scanpy.scverse.org/en/stable/api/preprocessing.html#module-scanpy.pp)

Filtering of highly-variable genes, batch-effect correction, per-cell normalization, preprocessing recipes.

Any transformation of the data matrix that is not a *tool*. Other than *tools*, preprocessing steps usually don’t return an easily interpretable annotation, but perform a basic transformation on the data matrix.

## Basic Preprocessing[#](https://scanpy.scverse.org/en/stable/api/preprocessing.html#basic-preprocessing)

For visual quality control, see [`highest_expr_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.highest_expr_genes.html#scanpy.pl.highest_expr_genes) and [`highly_variable_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.highly_variable_genes.html#scanpy.pl.highly_variable_genes) in [`scanpy.pl`](https://scanpy.scverse.org/en/stable/api/plotting.html#module-scanpy.pl).

| pp.filter_cells | Filter cell outliers based on counts and numbers of genes expressed. |
|-----------------|----------------------------------------------------------------------|
| pp.filter_genes | Filter genes based on number of cells or counts. |
| pp.highly_variable_genes | Annotate highly variable genes[Satijaet al., 2015,Stuartet al., 2019,Zhenget al., 2017]. |
| pp.log1p | Logarithmize the data matrix. |
| pp.pca | Principal component analysis[Pedregosaet al., 2011]. |
| pp.normalize_total | Normalize counts per cell. |
| pp.normalize_shifted_clr | Normalize counts with the shifted centered log-ratio (PFlog) transform. |
| pp.regress_out | Regress out (mostly) unwanted sources of variation. |
| pp.scale | Scale data to unit variance and zero mean. |
| pp.sample | Sample observations or variables with or without replacement. |
| pp.downsample_counts | Downsample counts from count matrix. |

## Recipes[#](https://scanpy.scverse.org/en/stable/api/preprocessing.html#recipes)

| pp.recipe_seurat | Normalize and filter as of Seurat[Satijaet al., 2015]. |
|------------------|--------------------------------------------------------|

## Data integration[#](https://scanpy.scverse.org/en/stable/api/preprocessing.html#pp-data-integration)

Batch effect correction and other data integration. Note that a simple batch correction method is available via [`pp.regress_out()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.regress_out.html#scanpy.pp.regress_out).

| pp.harmony_integrate | Integrate different experiments using the Harmony algorithm[Korsunskyet al., 2019,Patikaset al., 2026]. |
|----------------------|---------------------------------------------------------------------------------------------------------|

Batches can also be integrated at the level of the neighbor graph using [`pp.bbknn()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.bbknn.html#scanpy.pp.bbknn). Also see [data integration tools](https://scanpy.scverse.org/en/stable/api/tools.html#data-integration) and external [external data integration](https://scanpy.scverse.org/en/stable/external/preprocessing.html#external-data-integration).

## Doublet detection[#](https://scanpy.scverse.org/en/stable/api/preprocessing.html#doublet-detection)

| pp.scrublet_simulate_doublets | Simulate doublets by adding the counts of random observed transcriptome pairs. |
|-------------------------------|--------------------------------------------------------------------------------|

## Sample demultiplexing[#](https://scanpy.scverse.org/en/stable/api/preprocessing.html#sample-demultiplexing)

|  |
|--|

## Neighbors[#](https://scanpy.scverse.org/en/stable/api/preprocessing.html#neighbors)

| pp.neighbors | Compute the nearest neighbors distance matrix and a neighborhood graph of observations[McInneset al., 2018]. |
|--------------|--------------------------------------------------------------------------------------------------------------|

## Output types[#](https://scanpy.scverse.org/en/stable/api/preprocessing.html#output-types)

Functions with several outputs return a `…Result` if passed `out=True`, and take a `…Out` mapping as `out` to write each output to a place of choice.

| pp.HashsoloOut | Where a mappingoutofhashsolo()writes each output, seeHashsoloResult. |
|----------------|----------------------------------------------------------------------|
| pp.NeighborsResult | Outputs ofneighbors()andbbknn(). |
| pp.NeighborsOut | Where a mappingoutofneighbors()andbbknn()writes each output, seeNeighborsResult. |
| pp.NormalizeTotalResult | Outputs ofnormalize_total(). |
| pp.NormalizeTotalOut | Where a mappingoutofnormalize_total()writes each output, seeNormalizeTotalResult. |
| pp.PcaResult | Outputs ofpca(). |
| pp.PcaOut | Where a mappingoutofpca()writes each output, seePcaResult. |
| pp.QcMetricsResult | Outputs ofcalculate_qc_metrics(). |
| pp.QcMetricsOut | Where a mappingoutofcalculate_qc_metrics()writes each output, seeQcMetricsResult. |
| pp.ScaleResult | Outputs ofscale(). |
| pp.ScaleOut | Where a mappingoutofscale()writes each output, seeScaleResult. |
| pp.ScrubletResult | Outputs ofscrublet(). |
| pp.ScrubletOut | Where a mappingoutofscrublet()writes each output, seeScrubletResult. |
