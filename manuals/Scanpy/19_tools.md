# Tools: tl

> Any transformation of the data matrix that is not preprocessing. In contrast to a preprocessing function, a tool usually adds an easily interpretable annotation to the data matrix, which can then b...

# Tools: `tl`[#](https://scanpy.scverse.org/en/stable/api/tools.html#module-scanpy.tl)

Any transformation of the data matrix that is not *preprocessing*. In contrast to a *preprocessing* function, a *tool* usually adds an easily interpretable annotation to the data matrix, which can then be visualized with a corresponding plotting function.

## Embeddings[#](https://scanpy.scverse.org/en/stable/api/tools.html#embeddings)

| tl.tsne | t-SNE[Amiret al., 2013,Pedregosaet al., 2011,van der Maaten and Hinton, 2008]. |
|---------|--------------------------------------------------------------------------------|
| tl.umap | Embed the neighborhood graph using UMAP[McInneset al., 2018]. |
| tl.draw_graph | Force-directed graph drawing[Chippada, 2018,Islamet al., 2011,Jacomyet al., 2014]. |
| tl.diffmap | Diffusion Maps[Coifmanet al., 2005,Haghverdiet al., 2015,Wolfet al., 2018]. |

Compute densities on embeddings.

|  |
|--|

## Clustering and trajectory inference[#](https://scanpy.scverse.org/en/stable/api/tools.html#clustering-and-trajectory-inference)

| tl.dendrogram | Compute a hierarchical clustering for the givengroupbycategories. |
|---------------|-------------------------------------------------------------------|
| tl.dpt | Infer progression of cells through geodesic distance along the graph[Haghverdiet al., 2016,Wolfet al., 2019]. |
| tl.paga | Map out the coarse-grained connectivity structures of complex manifolds[Wolfet al., 2019]. |

## Data integration[#](https://scanpy.scverse.org/en/stable/api/tools.html#data-integration)

|  |
|--|

## Marker genes[#](https://scanpy.scverse.org/en/stable/api/tools.html#marker-genes)

| tl.filter_rank_genes_groups | Filter out genes based on two criteria. |
|-----------------------------|-----------------------------------------|
| tl.marker_gene_overlap | Calculate an overlap score between data-derived marker genes and provided markers. |

## Gene scores, Cell cycle[#](https://scanpy.scverse.org/en/stable/api/tools.html#gene-scores-cell-cycle)

| tl.score_genes_cell_cycle | Score cell cycle genes[Satijaet al., 2015]. |
|---------------------------|---------------------------------------------|

## Output types[#](https://scanpy.scverse.org/en/stable/api/tools.html#output-types)

Functions with several outputs return a `…Result` if passed `out=True`, and take a `…Out` mapping as `out` to write each output to a place of choice.

| tl.CellCycleOut | Where a mappingoutofscore_genes_cell_cycle()writes each output, seeCellCycleResult. |
|-----------------|-------------------------------------------------------------------------------------|
| tl.DptResult | Outputs ofdpt(). |
| tl.DptOut | Where a mappingoutofdpt()writes each output, seeDptResult. |
