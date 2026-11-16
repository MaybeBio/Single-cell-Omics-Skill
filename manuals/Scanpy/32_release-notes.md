# Release notes

> Version 1.13: 1.13.0 2026-11-13: Pre-releases: b1 2026-11-05, a2 2026-08-28, a1 2026-07-24. Breaking changes: a1 Remove compatibility helper for passing parameters positionally P Angerer(#3984), a1...

# Release notes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#release-notes)

## Version 1.13[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-13)

### 1.13.0 2026-11-13[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-13-0)

Pre-releases:

- b1 2026-11-05
- a2 2026-08-28
- a1 2026-07-24

#### Breaking changes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#breaking-changes)

- a1 Remove compatibility helper for passing parameters positionally P Angerer ([#3984](https://github.com/scverse/scanpy/pull/3984))
- a1 Remove `use_highly_variable` parameter to [`scanpy.pp.pca()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.pca.html#scanpy.pp.pca) and [`scanpy.experimental.pp.normalize_pearson_residuals_pca()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.experimental.pp.normalize_pearson_residuals_pca.html#scanpy.experimental.pp.normalize_pearson_residuals_pca)P Angerer ([#4039](https://github.com/scverse/scanpy/pull/4039))
- a1 Remove `'rapids'` flavor for [`scanpy.pp.neighbors()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.neighbors.html#scanpy.pp.neighbors), [`scanpy.tl.louvain()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.louvain.html#scanpy.tl.louvain), [`scanpy.tl.umap()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.umap.html#scanpy.tl.umap)P Angerer ([#4052](https://github.com/scverse/scanpy/pull/4052))

#### Features[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#features)

- a1 Add [`scanpy.settings.preset`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.settings.html#scanpy.settings.preset) setting with the new preset [`ScanpyV2Preview`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.Preset.html#scanpy.Preset.ScanpyV2Preview)P Angerer ([#3653](https://github.com/scverse/scanpy/pull/3653))
- a1 Add [`scanpy.pp.harmony_integrate()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.harmony_integrate.html#scanpy.pp.harmony_integrate) with Harmony1 and Harmony2 support for batch correction S Dicks, P Angerer ([#3953](https://github.com/scverse/scanpy/pull/3953))
- a1 Add support for [`numpy.random.Generator`](https://numpy.org/doc/stable/reference/random/generator.html#numpy.random.Generator) to all functions previously accepting a `random_state` parameter P Angerer ([#3983](https://github.com/scverse/scanpy/pull/3983))
- a1 Add `layer` parameter to [`filter_rank_genes_groups()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.filter_rank_genes_groups.html#scanpy.tl.filter_rank_genes_groups)P Angerer ([#3999](https://github.com/scverse/scanpy/pull/3999))
- a1 add `sparse_format` parameter to [`read_10x_mtx()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.io.read_10x_mtx.html#scanpy.io.read_10x_mtx) defaulting to CSR E. Provo ([#4017](https://github.com/scverse/scanpy/pull/4017))
- a1 Add `mean_in_log_space` argument to [`scanpy.tl.rank_genes_groups()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.rank_genes_groups.html#scanpy.tl.rank_genes_groups) for customizing how log-fold-change is calculated [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#4037](https://github.com/scverse/scanpy/pull/4037))
- a1 Introduce blazing-fast [`illico`](https://github.com/remydubois/illico) as a new `wilcoxon``method` in [`rank_genes_groups()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.rank_genes_groups.html#scanpy.tl.rank_genes_groups). Use it via either `method="wilcoxon_illico"` or via [`ScanpyV2Preview`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.Preset.html#scanpy.Preset.ScanpyV2Preview) (preferred) I Gold
- a1 New `scanpy2` option-dependency extra to make it easier to transition to the new scanpy 2 defaults I Gold ([#4055](https://github.com/scverse/scanpy/pull/4055))
- a1 Add preset support to the `key_added` parameter of [`scanpy.pp.pca()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.pca.html#scanpy.pp.pca), [`scanpy.tl.umap()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.umap.html#scanpy.tl.umap), [`scanpy.tl.diffmap()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.diffmap.html#scanpy.tl.diffmap), and [`scanpy.tl.draw_graph()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.draw_graph.html#scanpy.tl.draw_graph)P Angerer ([#4076](https://github.com/scverse/scanpy/pull/4076))
- a1 Added [`scanpy.settings.override()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.settings.override.html#scanpy.settings.override) function for temporarily overriding settings. P Angerer ([#4081](https://github.com/scverse/scanpy/pull/4081))
- a1 Add `ncols` parameter to [`scanpy.pl.violin()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.violin.html#scanpy.pl.violin) for wrapping multi-key violins into a grid layout ([#1552](https://github.com/scverse/scanpy/issues/1552)). O Akinremi ([#4132](https://github.com/scverse/scanpy/pull/4132))
- a1 Add a new [Holoviews](https://holoviews.org)-based plotting backend for [`scanpy.pl`](https://scanpy.scverse.org/en/stable/api/plotting.html#module-scanpy.pl), used when [`scanpy.settings.preset`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.settings.html#scanpy.settings.preset) is set to [`ScanpyV2Preview`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.Preset.html#scanpy.Preset.ScanpyV2Preview)P Angerer ([#4188](https://github.com/scverse/scanpy/pull/4188))
- a1 Add [`anndata.acc`](https://anndata.scverse.org/en/stable/accessors.html#module-anndata.acc) support to [`scanpy.get.aggregate()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.get.aggregate.html#scanpy.get.aggregate) via its `use` parameter. P Angerer ([#4199](https://github.com/scverse/scanpy/pull/4199), [#4334](https://github.com/scverse/scanpy/pull/4334))
- a2 Fix multi-key [`scanpy.pp.harmony_integrate()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.harmony_integrate.html#scanpy.pp.harmony_integrate), remove its legacy correction path, and speed up large integrations S Dicks ([#4232](https://github.com/scverse/scanpy/pull/4232))
- a2 Add exact raw counts for [`pbmc68k_reduced()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.datasets.pbmc68k_reduced.html#scanpy.datasets.pbmc68k_reduced) and clarify its docstring on how the dataset was derived P Angerer ([#4269](https://github.com/scverse/scanpy/pull/4269))
- a2 Move [`scanpy.external.pp.hashsolo()`](https://scanpy.scverse.org/en/stable/external/generated/scanpy.external.pp.hashsolo.html#scanpy.external.pp.hashsolo) into scanpy’s main API as [`scanpy.pp.hashsolo()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.hashsolo.html#scanpy.pp.hashsolo) and deprecate the old location. The new function takes [`anndata.acc`](https://anndata.scverse.org/en/stable/accessors.html#module-anndata.acc) references, so the hashing counts can live in [`obs`](https://anndata.scverse.org/en/stable/generated/anndata.AnnData.obs.html#anndata.AnnData.obs), in the data matrix (e.g. `A.X[:,["Hash1","Hash2"]]`), or all together in [`obsm`](https://anndata.scverse.org/en/stable/generated/anndata.AnnData.obsm.html#anndata.AnnData.obsm) (e.g. `A.obsm["hto"]`), and writes its results to `.obs['hashsolo']` and `.obsm['hashsolo']` instead of six unnamespaced `.obs` columns P Angerer ([#4303](https://github.com/scverse/scanpy/pull/4303), [#4359](https://github.com/scverse/scanpy/pull/4359))
- a2 Add [`scanpy.pp.bbknn()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.bbknn.html#scanpy.pp.bbknn), a native implementation of batch balanced kNN [[Polański *et al.*, 2019](https://scanpy.scverse.org/en/stable/references.html#id56)]S Dicks ([#4306](https://github.com/scverse/scanpy/pull/4306))
- a2[`scanpy.pp.neighbors()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.neighbors.html#scanpy.pp.neighbors), [`scanpy.tl.tsne()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.tsne.html#scanpy.tl.tsne), and [`scanpy.tl.dendrogram()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.dendrogram.html#scanpy.tl.dendrogram) gain a `use` parameter taking [`anndata.acc`](https://anndata.scverse.org/en/stable/accessors.html#module-anndata.acc) accessors such as `A.X`, `A.layers["scaled"]`, or `A.obsm["pca"]`, deprecating `use_rep`P Angerer ([#4315](https://github.com/scverse/scanpy/pull/4315), [#4334](https://github.com/scverse/scanpy/pull/4334))
- b1 Add [`scanpy.pp.normalize_shifted_clr()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.normalize_shifted_clr.html#scanpy.pp.normalize_shifted_clr) for PFlog shifted centered log-ratio normalization, a variance-stabilizing, depth-invariant and rank-preserving count transform [[Booeshaghi *et al.*, 2026](https://scanpy.scverse.org/en/stable/references.html#id11)]. By default, it estimates negative-binomial overdispersion and uses the Anscombe pseudocount. The complete centered result is stored in `X` or the selected layer. Because CLR centering generally maps zeros to non-zero values, the output is dense. R Baber ([#4160](https://github.com/scverse/scanpy/pull/4160))
- b1 Add an `ncols` parameter to [`scanpy.pl.paga()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.paga.html#scanpy.pl.paga) so multi-color PAGA plots can be arranged in a configurable grid instead of a single row. A Moradniaei ([#4325](https://github.com/scverse/scanpy/pull/4325))
- b1 Rename `mask_var`/`mask_obs` to `mask` in [`pca()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.pca.html#scanpy.pp.pca), [`scale()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.scale.html#scanpy.pp.scale), [`rank_genes_groups()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.rank_genes_groups.html#scanpy.tl.rank_genes_groups), and [`normalize_pearson_residuals_pca()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.experimental.pp.normalize_pearson_residuals_pca.html#scanpy.experimental.pp.normalize_pearson_residuals_pca). The new parameter takes [`anndata.acc`](https://anndata.scverse.org/en/stable/accessors.html#module-anndata.acc) references, e.g. `A.var["highly_variable"]`P Angerer ([#4333](https://github.com/scverse/scanpy/pull/4333))
- b1 Functions now say where they read from with `use` and where they write to with `out`, taking [`anndata.acc`](https://anndata.scverse.org/en/stable/accessors.html#module-anndata.acc) accessors (e.g. `A.layers["counts"]`) or [`resolve()`](https://anndata.scverse.org/en/stable/generated/anndata.acc.AdAcc.html#anndata.acc.AdAcc.resolve)\ able strings (e.g. `"layers.counts"`). A function’s `use` takes what was passed to the `out` of the function producing its input, e.g. `sc.pp.pca(adata,out=A.obsm["p"])` and then `sc.pp.neighbors(adata,use=A.obsm["p"])`:
Single input
`use` is one accessor, deprecating `layer`, `obsm`, `use_rep` and `use_raw`, e.g. `sc.pp.log1p(adata,use=A.layers["counts"])`.
Outputs
`out` is an accessor to write to (e.g. `sc.tl.umap(adata,out=A.obsm["umap2"])`), `out=True` to return the result instead (replacing `inplace=False`, `score_name` and `copy`), or `out=False` (the default) to write it to its default place, which is the input itself for functions transforming a matrix, e.g. [`log1p()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.log1p.html#scanpy.pp.log1p). This deprecates `key_added`. [`highly_variable_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.highly_variable_genes.html#scanpy.pp.highly_variable_genes) only takes `out=True` or `out=False` until there are references to columns of DataFrames in [`varm`](https://anndata.scverse.org/en/stable/generated/anndata.AnnData.varm.html#anndata.AnnData.varm) ([scverse/anndata#2722](https://github.com/scverse/anndata/issues/2722)).
Several outputs
An accessor names the primary output `k`, e.g. `sc.pp.pca(adata,out=A.obsm["p"])`, and the others follow it: they are named `k` in kinds of places no other output goes to (`.varm["p"]` and `.uns["p"]`), and `k_{name}` otherwise (e.g. `sc.pp.neighbors(adata,out=A.obsp["nn"])` writes `.obsp["nn"]` and `.obsp["nn_distances"]`). This also applies to the primary output’s default place (e.g. `out=A.obsp["connectivities"]` writes `.obsp["connectivities_distances"]`), only `out=False` writes to the default places. If the primary output goes to `A.X`, the others go to their default places. A mapping of output names to accessors writes only the outputs it names, e.g. `out={"connectivities":A.obsp["conns"]}` (typed as e.g. [`NeighborsOut`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.NeighborsOut.html#scanpy.pp.NeighborsOut)), and two outputs can’t go to the same place. `out=True` returns them as a [`NamedTuple`](https://docs.python.org/3/library/typing.html#typing.NamedTuple) (e.g. [`PcaResult`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.PcaResult.html#scanpy.pp.PcaResult)). For now, [`scrublet()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.scrublet.html#scanpy.pp.scrublet), [`score_genes_cell_cycle()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.score_genes_cell_cycle.html#scanpy.tl.score_genes_cell_cycle) and [`dpt()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.dpt.html#scanpy.tl.dpt) only take a mapping, as none of their outputs is the primary one. Once there are `.uns` accessors ([scverse/anndata#2657](https://github.com/scverse/anndata/issues/2657)), they will take `out=A.uns["k"]` and store the derived mapping of accessors in there.
Neighborhood graphs
Functions using [`neighbors()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.neighbors.html#scanpy.pp.neighbors)’ results take the graph as `use`, e.g. `sc.tl.umap(adata,use=A.obsp["nn"])`, deprecating `neighbors_key`, `obsp` and `adjacency` ([`ingest()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.ingest.html#scanpy.tl.ingest) keeps `neighbors_key` until there are `.uns` accessors). Distances and parameters are found where [`neighbors()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.neighbors.html#scanpy.pp.neighbors) stored them alongside the graph, if they exist. [`umap()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.umap.html#scanpy.tl.umap) lays out a graph with more than `2*n_components` connected components using data: by default the representation [`neighbors()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.neighbors.html#scanpy.pp.neighbors) used, otherwise pass e.g. `rep=A.obsm["pca"]`. [`gearys_c()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.metrics.gearys_c.html#scanpy.metrics.gearys_c) and [`morans_i()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.metrics.morans_i.html#scanpy.metrics.morans_i) also take the graph as `use`, deprecating `use_graph`, and their `vals` takes an accessor (e.g. `vals=A.obsm["pca"]`), deprecating `layer`, `obsm` and `obsp`.
Groupings and covariates
take references like `A.obs["batch"]`: `batch` replaces `batch_key` in [`highly_variable_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.highly_variable_genes.html#scanpy.pp.highly_variable_genes), [`scrublet()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.scrublet.html#scanpy.pp.scrublet) and the experimental Pearson residual functions, [`combat()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.combat.html#scanpy.pp.combat)’s `key` and [`regress_out()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.regress_out.html#scanpy.pp.regress_out)’s `keys` are renamed to `batch` and `covariates`, [`harmony_integrate()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.harmony_integrate.html#scanpy.pp.harmony_integrate) takes `batch`, and `groupby` of [`rank_genes_groups()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.rank_genes_groups.html#scanpy.tl.rank_genes_groups), [`filter_rank_genes_groups()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.filter_rank_genes_groups.html#scanpy.tl.filter_rank_genes_groups), [`dendrogram()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.dendrogram.html#scanpy.tl.dendrogram) and [`embedding_density()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.embedding_density.html#scanpy.tl.embedding_density) as well as [`paga()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.paga.html#scanpy.tl.paga)’s `groups` accept them.
Outputs only in `.uns`
[`rank_genes_groups()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.rank_genes_groups.html#scanpy.tl.rank_genes_groups), [`filter_rank_genes_groups()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.filter_rank_genes_groups.html#scanpy.tl.filter_rank_genes_groups), [`dendrogram()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.dendrogram.html#scanpy.tl.dendrogram) and [`paga()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.paga.html#scanpy.tl.paga) take `out=True` to return their results. Until there are `.uns` accessors ([scverse/anndata#2657](https://github.com/scverse/anndata/issues/2657)), they keep `key_added` (where they had one) and the functions reading their results keep `key`.

Strings passed instead of accessors are [`resolve()`](https://anndata.scverse.org/en/stable/generated/anndata.acc.AdAcc.html#anndata.acc.AdAcc.resolve)\ d (e.g. `"obs.batch"`), except where a parameter already took strings with another meaning (e.g. column or `.obsm` key names): there they keep it, unless [`scanpy.settings.preset`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.settings.html#scanpy.settings.preset) is [`ScanpyV2Preview`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.Preset.html#scanpy.Preset.ScanpyV2Preview). Deprecated parameters (e.g. `batch_key`) keep their meaning regardless of preset.

`copy` and `inplace` are deprecated everywhere: copy the [`AnnData`](https://anndata.scverse.org/en/stable/generated/anndata.AnnData.html#anndata.AnnData) yourself before calling a function, or pass `out=True`. Only the subsetting functions [`filter_cells()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.filter_cells.html#scanpy.pp.filter_cells), [`filter_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.filter_genes.html#scanpy.pp.filter_genes) and [`sample()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.sample.html#scanpy.pp.sample) keep `copy`, since copying the whole object first would be wasteful. [`calculate_qc_metrics()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.calculate_qc_metrics.html#scanpy.pp.calculate_qc_metrics) returns its metrics by default, but writes them with [`ScanpyV2Preview`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.Preset.html#scanpy.Preset.ScanpyV2Preview). Since [`raw`](https://anndata.scverse.org/en/stable/generated/anndata.AnnData.raw.html#anndata.AnnData.raw) is on its way out, `use_raw` is deprecated: keep the data in a layer instead. [`embedding_density()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.embedding_density.html#scanpy.tl.embedding_density)’s `basis` is renamed to `use`, and [`obs_df()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.get.obs_df.html#scanpy.get.obs_df) and [`var_df()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.get.var_df.html#scanpy.get.var_df) take `use` instead of `layer`.

[`dpt()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.dpt.html#scanpy.tl.dpt) now works with diffusion maps written under a key, e.g. with [`ScanpyV2Preview`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.Preset.html#scanpy.Preset.ScanpyV2Preview), and batched [`scrublet()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.scrublet.html#scanpy.pp.scrublet) now only writes its outputs instead of replacing all of [`obs`](https://anndata.scverse.org/en/stable/generated/anndata.AnnData.obs.html#anndata.AnnData.obs)P Angerer ([#4359](https://github.com/scverse/scanpy/pull/4359))
- Add `flavor="networkit"` to [`leiden()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.leiden.html#scanpy.tl.leiden) for parallel Leiden clustering via NetworKit (install `scanpy[networkit]`) amalia-k510 ([#4170](https://github.com/scverse/scanpy/pull/4170))
- Add Array-API support, enabling JAX and other array-api backends in `adata.X`A. Karesh ([#4179](https://github.com/scverse/scanpy/pull/4179))
- Allow `directed=True` with `flavor="igraph"` in [`leiden()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.leiden.html#scanpy.tl.leiden), and default to it with [`ScanpyV2Preview`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.Preset.html#scanpy.Preset.ScanpyV2Preview); the `leiden` extra now requires `igraph&gt;=1` and `leidenalg&gt;=0.11`M Kuehl ([#4490](https://github.com/scverse/scanpy/pull/4490))
- Add preset support to the `tie_correct` parameter of [`scanpy.tl.rank_genes_groups()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.rank_genes_groups.html#scanpy.tl.rank_genes_groups): [`ScanpyV2Preview`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.Preset.html#scanpy.Preset.ScanpyV2Preview) enables tie correction for `wilcoxon`, which changes its scores, p-values, and gene ranking on sparse data P Angerer ([#4504](https://github.com/scverse/scanpy/pull/4504))
- Add preset support for umap-learn 0.6’s new layout pipeline: [`scanpy.tl.umap()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.umap.html#scanpy.tl.umap) gets a `compatibility_layout` parameter, which [`ScanpyV2Preview`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.Preset.html#scanpy.Preset.ScanpyV2Preview) sets to `False`, changing the embedding. [`scanpy.tl.ingest()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.ingest.html#scanpy.tl.ingest) now transforms with the pipeline the reference embedding was computed with P Angerer ([#4506](https://github.com/scverse/scanpy/pull/4506))

#### Performance[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#performance)

- a1 Run [`scanpy.tl.umap()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.umap.html#scanpy.tl.umap) in parallel when possible P Angerer ([#4036](https://github.com/scverse/scanpy/pull/4036))
- a1 Changed default [`scanpy.settings.n_jobs`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.settings.html#scanpy.settings.n_jobs) from 1 to 4. P Angerer ([#4081](https://github.com/scverse/scanpy/pull/4081))
- a2 Derive unset [`scanpy.pp.harmony_integrate()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.harmony_integrate.html#scanpy.pp.harmony_integrate) stopping rules from `flavor`, speeding up `harmony2` considerably: it now follows the Harmony2 defaults, while `harmony1` still follows harmony-pytorch S Dicks ([#4290](https://github.com/scverse/scanpy/pull/4290))
- Speed up [`harmony_integrate()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.harmony_integrate.html#scanpy.pp.harmony_integrate) 6–13× with parallel numba kernels, and add the `seeding`, `stop_rule`, `penalty_combine`, `seeding_lloyd_steps`, `seeding_cells_per_cluster`, `recompute_counts` and `low_memory` parameters. For `flavor="harmony2"`, the default seeding is now `"afkmc2"`, and the default stop rule for both flavors is `"reference_record"`; pass `seeding="kmeans",stop_rule="reference"` for the previous behaviour. Cells with an all-zero embedding are now left out and returned unchanged T Treis ([#4424](https://github.com/scverse/scanpy/pull/4424))
- Speed up igraph-based functions ([`leiden()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.leiden.html#scanpy.tl.leiden) with `flavor="igraph"`, [`draw_graph()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.draw_graph.html#scanpy.tl.draw_graph), [`paga()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.paga.html#scanpy.pl.paga)) when `rng` is a [`numpy.random.Generator`](https://numpy.org/doc/stable/reference/random/generator.html#numpy.random.Generator)P. Angerer ([#4518](https://github.com/scverse/scanpy/pull/4518))

#### Miscellaneous changes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#miscellaneous-changes)

- a1 Remove the superseded functions `sc.pp.filter_genes_dispersion` and `sc.pp.normalize_per_cell`P Angerer ([#4010](https://github.com/scverse/scanpy/pull/4010))
- a1[`anndata`](https://anndata.scverse.org/en/stable/api.html#module-anndata) is lower bounded by `0.11.2`I Gold ([#4191](https://github.com/scverse/scanpy/pull/4191))
- a2 Deprecate [`scanpy.external`](https://scanpy.scverse.org/en/stable/external/index.html#module-scanpy.external) and each of its functions. Each function’s deprecation message names its replacement: an upstream package, a successor project, or the scanpy function it will move to. See also the [scverse ecosystem](https://scverse.org/packages/#ecosystem)P Angerer ([#3645](https://github.com/scverse/scanpy/pull/3645))
- a2 Create [`scanpy.io`](https://scanpy.scverse.org/en/stable/api/io.html#module-scanpy.io) and move `scanpy.{read*,write*}` into it. Deprecate `scanpy.read_loom` and `sc.external.exporting.*`P Angerer ([#4299](https://github.com/scverse/scanpy/pull/4299))
- a2 Fetch the bundled datasets through [`scverse_misc.datasets.fetch()`](https://scverse-misc.readthedocs.io/stable/generated/scverse_misc.datasets.fetch.html#scverse_misc.datasets.fetch), adding hash verification, retries, and fall back to their pre-S3 upstream URLs. [`read_10x_h5()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.io.read_10x_h5.html#scanpy.io.read_10x_h5) and friends now use `pooch` for downloading, also gaining retries P Angerer ([#4310](https://github.com/scverse/scanpy/pull/4310))
- b1 Deprecate [`sim()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.sim.html#scanpy.tl.sim), [`sim()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.sim.html#scanpy.pl.sim), [`toggleswitch()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.datasets.toggleswitch.html#scanpy.datasets.toggleswitch), [`recipe_weinreb17()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.recipe_weinreb17.html#scanpy.pp.recipe_weinreb17), [`gene_coordinates()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.queries.gene_coordinates.html#scanpy.queries.gene_coordinates), and the undocumented `sc.datasets.burczynski06`, `sc.tl.paga_compare_paths`, `sc.tl.paga_degrees`, `sc.tl.paga_expression_entropies`, and `sc.pl.paga_adjacency`, which have next to no users. Remove unreachable code paths from [`dpt()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.dpt.html#scanpy.tl.dpt)L Heumos ([#4390](https://github.com/scverse/scanpy/pull/4390))

#### Documentation[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#documentation)

- b1 Group the docs sidebar into Getting started, Guides, Reference and Project like the other scverse core packages, with new quickstart and citation pages L Heumos ([#4464](https://github.com/scverse/scanpy/pull/4464))

#### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#bug-fixes)

- b1 Fix [`combat()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.combat.html#scanpy.pp.combat) with categorical `covariates`, which produced a singular design matrix and either raised or silently returned wrong values L Heumos ([#4389](https://github.com/scverse/scanpy/pull/4389))
- b1 Fix `sc.pp.downsample_counts(total_counts=...)` silently doing nothing for dense `X` that is not C-contiguous, e.g. Fortran-ordered arrays. ([#4408](https://github.com/scverse/scanpy/pull/4408))
- b1 Fix [`downsample_counts()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.downsample_counts.html#scanpy.pp.downsample_counts) assigning reads to the wrong entries for float32 data with more than 2**24 total counts P Paladugu ([#4414](https://github.com/scverse/scanpy/pull/4414))
- b1 Fix [`embedding_density()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.embedding_density.html#scanpy.tl.embedding_density) raising for `groupby` categories with fewer than 3 cells or degenerate coordinates; their density is now `NaN` and a warning is shown A Asok ([#4431](https://github.com/scverse/scanpy/pull/4431))
- Fix [`scanpy.tl.rank_genes_groups()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.rank_genes_groups.html#scanpy.tl.rank_genes_groups) running the `t-test` and `t-test_overestim_var` methods on exponentiated values when `mean_in_log_space=False`; the test now always uses the (log-transformed) input and `mean_in_log_space` only affects `logfoldchanges`C Krafft ([#4337](https://github.com/scverse/scanpy/pull/4337))
- Fix [`scanpy.tl.ingest()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.ingest.html#scanpy.tl.ingest) failing with a `KeyError` when the reference’s PCA was computed by an old scanpy version and has no `.uns["pca"]["params"]`P Angerer ([#4516](https://github.com/scverse/scanpy/pull/4516))

## Version 1.12[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-12)

### 1.12.6 2026-11-05[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-12-6)

#### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id6)

- Fix [`ingest()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.ingest.html#scanpy.tl.ingest) truncating to the hardcoded default `settings.N_PCS` instead of the number of PCs that `pp.neighbors` actually used, whenever a reference dataset’s `neighbors` params recorded neither `use_rep` nor `n_pcs`Siddhardha Nanda ([#4401](https://github.com/scverse/scanpy/pull/4401))
- Return zero residuals for zero-count cells and genes in [`scanpy.experimental.pp.normalize_pearson_residuals()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.experimental.pp.normalize_pearson_residuals.html#scanpy.experimental.pp.normalize_pearson_residuals), avoiding undefined values in subsequent analysis [@HamzaAhmed-bio](https://github.com/sponsors/HamzaAhmed-bio) ([#4404](https://github.com/scverse/scanpy/pull/4404))
- Fix [`highly_variable_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.highly_variable_genes.html#scanpy.pp.highly_variable_genes) selecting more than `n_top_genes` genes with duplicate variable names when using `seurat_v3` or `seurat_v3_paper`. ([#4409](https://github.com/scverse/scanpy/pull/4409))
- Fix [`highly_variable_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.highly_variable_genes.html#scanpy.pp.highly_variable_genes) with `flavor="cell_ranger"` and `n_top_genes` selecting no genes when the top genes have infinite normalized dispersion L Jorgensen ([#4419](https://github.com/scverse/scanpy/pull/4419))
- [`highly_variable_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.highly_variable_genes.html#scanpy.pp.highly_variable_genes) with `batch_key` (`flavor="seurat"` or `"cell_ranger"`) no longer flags more than `n_top_genes` genes or merges the statistics of genes that share a name R García Revett ([#4439](https://github.com/scverse/scanpy/pull/4439))
- Fix [`highly_variable_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.highly_variable_genes.html#scanpy.pp.highly_variable_genes) (`flavor='seurat_v3'`), [`calculate_qc_metrics()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.calculate_qc_metrics.html#scanpy.pp.calculate_qc_metrics), and [`rank_genes_groups()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.rank_genes_groups.html#scanpy.tl.rank_genes_groups) loading entire row-chunked [`dask.array.Array`](https://docs.dask.org/en/stable/generated/dask.array.Array.html#dask.array.Array)s into memory when checking for count data or counting nonzero cells per gene Y Su ([#4446](https://github.com/scverse/scanpy/pull/4446))
- Fix [`calculate_qc_metrics()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.calculate_qc_metrics.html#scanpy.pp.calculate_qc_metrics) for sparse matrices of small integer dtypes, which gave negative `pct_counts_in_top_N_genes` (e.g. `int16`) or raised (e.g. `uint16`) P Paladugu ([#4461](https://github.com/scverse/scanpy/pull/4461))
- Fix [`neighbors()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.neighbors.html#scanpy.pp.neighbors) and [`ingest()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.ingest.html#scanpy.tl.ingest) with `umap-learn` 0.6 P Angerer ([#4480](https://github.com/scverse/scanpy/pull/4480))
- Multi-column `groupby` in [`dotplot()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.dotplot.html#scanpy.pl.dotplot), [`matrixplot()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.matrixplot.html#scanpy.pl.matrixplot), [`stacked_violin()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.stacked_violin.html#scanpy.pl.stacked_violin), [`tracksplot()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.tracksplot.html#scanpy.pl.tracksplot), and [`heatmap()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.heatmap.html#scanpy.pl.heatmap) now accepts integer-categorical and numeric observation columns instead of raising `TypeError`A Barlow ([#4385](https://github.com/scverse/scanpy/pull/4385))
- Fix [`combat()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.combat.html#scanpy.pp.combat) with categorical `covariates`, which produced a singular design matrix and either raised or silently returned wrong values L Heumos ([#4496](https://github.com/scverse/scanpy/pull/4496))

### 1.12.5 2026-10-29[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-12-5)

#### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id8)

- Fix [`log1p()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.log1p.html#scanpy.pp.log1p) warning that `adata.X` is already log-transformed when transforming a layer or obsm M Arvind Khakharia ([#4298](https://github.com/scverse/scanpy/pull/4298))
- [`read_10x_mtx()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.io.read_10x_mtx.html#scanpy.io.read_10x_mtx) no longer treats the gene symbol `nan` as missing. ([#4329](https://github.com/scverse/scanpy/pull/4329))
- Support pynndescent 0.7 in [`neighbors()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.neighbors.html#scanpy.pp.neighbors) and [`ingest()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.ingest.html#scanpy.tl.ingest)E Roellin ([#4451](https://github.com/scverse/scanpy/pull/4451))

### 1.12.4 2026-08-27[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-12-4)

#### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id10)

- Fix [`normalize_total()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.normalize_total.html#scanpy.pp.normalize_total) raising `UnboundLocalError` for `CSR` matrices when `exclude_highly_expressed=False` and Numba JIT is disabled S Dicks ([#4231](https://github.com/scverse/scanpy/pull/4231))
- Keep list-based boolean categorical colors stable after subsetting by storing colors for both boolean values. ([#4252](https://github.com/scverse/scanpy/pull/4252))
- Fix [`normalize_total()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.normalize_total.html#scanpy.pp.normalize_total) computing a different `target_sum` for sparse and dense input when `target_sum=None`, so matrices containing zero-count cells normalized inconsistently JOhnsonKC201 ([#4256](https://github.com/scverse/scanpy/pull/4256))
- Fix [`scrublet()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.scrublet.html#scanpy.pp.scrublet) failing with non-unique [`obs_names`](https://anndata.scverse.org/en/stable/generated/anndata.AnnData.obs_names.html#anndata.AnnData.obs_names), and with cells dropped by its internal filtering when `batch_key` is used P Angerer ([#4260](https://github.com/scverse/scanpy/pull/4260))
- Make [`sim()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.sim.html#scanpy.tl.sim) raise instead of silently returning a truncated data matrix when it can’t simulate the requested number of branching realizations P Angerer ([#4263](https://github.com/scverse/scanpy/pull/4263))

### 1.12.3 2026-07-24[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-12-3)

#### Documentation[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id12)

- Document that [`scale()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.scale.html#scanpy.pp.scale) computes variance/std with Bessel’s correction (`ddof=1`) A McKenna ([#4196](https://github.com/scverse/scanpy/pull/4196))

#### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id13)

- Fix [`scanpy.get.aggregate()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.get.aggregate.html#scanpy.get.aggregate) on `dask` arrays when `func` is a list containing `count_nonzero` or `sum`Z Boldyga ([#4192](https://github.com/scverse/scanpy/pull/4192))
- Fix [`ingest()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.ingest.html#scanpy.tl.ingest) PCA projection to center query data with the reference mean S Dicks ([#4208](https://github.com/scverse/scanpy/pull/4208))

#### Performance[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id14)

- Speed up [`scanpy.tl.rank_genes_groups()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.rank_genes_groups.html#scanpy.tl.rank_genes_groups)`vs_rest` statistics via a single [`scanpy.get.aggregate()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.get.aggregate.html#scanpy.get.aggregate) pass with [Chan’s](https://en.wikipedia.org/wiki/Algorithms_for_calculating_variance#Parallel_algorithm) cancellation-free leave-one-out variance Z Boldyga

### 1.12.2 2026-06-29[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-12-2)

#### Documentation[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id16)

- Clarify that `method` selects the connectivity kernel and `transformer` selects the kNN search backend in [`neighbors()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.neighbors.html#scanpy.pp.neighbors)C Gao ([#4079](https://github.com/scverse/scanpy/pull/4079))
- Highlighting rapids-singlecell for GPU users & cleanup documentation. ([#4100](https://github.com/scverse/scanpy/pull/4100))

#### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id17)

- Make [`scanpy.metrics.modularity()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.metrics.modularity.html#scanpy.metrics.modularity) produce the exact same results as `igraph` itself, see [here](https://github.com/scverse/scanpy/blob/e4e43830ea3d5af0acd99648612612551fa7b83e/tests/test_metrics.py#L340). In Scanpy 2.0, all graphs will be calculated slightly differently, using [`igraph.Graph.Weighted_Adjacency()`](https://python.igraph.org/en/stable/api/igraph.Graph.html#Weighted_Adjacency). P Angerer ([#4112](https://github.com/scverse/scanpy/pull/4112))
- Download cached dataset files atomically so that parallel processes sharing a dataset directory (e.g. `pytest-xdist` workers) can no longer read a partially-written file gaoflow ([#4142](https://github.com/scverse/scanpy/pull/4142))
- Fix usage of [`scanpy.get.aggregate()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.get.aggregate.html#scanpy.get.aggregate) with `mask` argument mis-assigning values that are masked out to category 0 + correct group size calculation with `mask`Z Boldyga ([#4178](https://github.com/scverse/scanpy/pull/4178))
- Fix single-observation (or `n_obs==dof`) aggregated variance in [`scanpy.get.aggregate()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.get.aggregate.html#scanpy.get.aggregate)Z Boldyga ([#4184](https://github.com/scverse/scanpy/pull/4184))

#### Performance[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id18)

- [`scanpy.pp.highly_variable_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.highly_variable_genes.html#scanpy.pp.highly_variable_genes) now does only two passes over the data sequentially for `seurat_v3` flavors, greatly reducing `dask` input usage time I Gold ([#4013](https://github.com/scverse/scanpy/pull/4013))
- Add `numba` kernels for mean/var/count-nonzero/sum arregation of sparse data in [`scanpy.get.aggregate()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.get.aggregate.html#scanpy.get.aggregate)I Gold ([#4062](https://github.com/scverse/scanpy/pull/4062))
- Improve memory usage of [`scanpy.pp.pca()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.pca.html#scanpy.pp.pca) when run on sparse [`dask.array.Array`](https://docs.dask.org/en/stable/generated/dask.array.Array.html#dask.array.Array)I Gold ([#4126](https://github.com/scverse/scanpy/pull/4126))
- Speed up [`score_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.score_genes.html#scanpy.tl.score_genes) by computing the sparse `nanmean` in a single pass instead of copying the matrix twice L Darrow ([#4141](https://github.com/scverse/scanpy/pull/4141))
- Use [Chan’s mean-var](https://en.wikipedia.org/wiki/Algorithms_for_calculating_variance#Parallel_algorithm) algorithm for acceleration of dask-backed [`scanpy.get.aggregate()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.get.aggregate.html#scanpy.get.aggregate)I Gold
- Use [Welford’s algorithm](https://en.wikipedia.org/wiki/Algorithms_for_calculating_variance#Welford's_online_algorithm) for mean-var calculation in [`scanpy.get.aggregate()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.get.aggregate.html#scanpy.get.aggregate) for in-memory (i.e., non-dask) arrays I Gold

### 1.12.1 2026-04-10[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-12-1)

#### Features[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id20)

- Use scverse S3 Cloudfront URLs for datasets zethson ([#4011](https://github.com/scverse/scanpy/pull/4011))

#### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id21)

- Prevent segfault when running [`scanpy.pp.highly_variable_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.highly_variable_genes.html#scanpy.pp.highly_variable_genes) with `flavor='seurat_v3{,_paper}'` and some all-zero genes P Angerer ([#3980](https://github.com/scverse/scanpy/pull/3980))
- Clarify and deduplicate shared parameter documentation between [`scanpy.tl.leiden()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.leiden.html#scanpy.tl.leiden) and [`scanpy.tl.louvain()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.louvain.html#scanpy.tl.louvain)Abi ([#3986](https://github.com/scverse/scanpy/pull/3986))
- [`scanpy.pp.combat()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.combat.html#scanpy.pp.combat) now raises a [`ValueError`](https://docs.python.org/3/builtins/exceptions.html#ValueError) when a batch contains fewer than 2 cells, instead of silently producing NaN values in the corrected data L Zhang ([#3994](https://github.com/scverse/scanpy/pull/3994))
- Fix crashes when running numba in dask by using the threadsafe [`fast_array_utils.numba.njit()`](https://fast-array-utils.scverse.org/en/stable/numba.html#fast_array_utils.numba.njit)P Angerer ([#4015](https://github.com/scverse/scanpy/pull/4015))
- [`scanpy.tl.umap()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.umap.html#scanpy.tl.umap) no longer silently mutates `adata.obsp['connectivities']` via a shared sparse buffer N Justice ([#4031](https://github.com/scverse/scanpy/pull/4031))
- Make [`scanpy.metrics.modularity()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.metrics.modularity.html#scanpy.metrics.modularity) actually use edge weights P Angerer ([#4045](https://github.com/scverse/scanpy/pull/4045))

### 1.12.0 2026-01-23[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-12-0)

Release candidates:

- rc1 2025-11-10

#### Breaking changes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id23)

- rc1 Adopt the Scientific Python [deprecation schedule](https://scientific-python.org/specs/spec-0000/): remove Python ≤3.11 support and officially add Python 3.13 and 3.14 support, require anndata≥0.10 P Angerer ([#3485](https://github.com/scverse/scanpy/pull/3485), [#3874](https://github.com/scverse/scanpy/pull/3874))

#### Features[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id24)

- rc1 Added `n_components` parameter to [`tsne()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.tsne.html#scanpy.tl.tsne)Kitsune ([#2803](https://github.com/scverse/scanpy/pull/2803))
- rc1[`scanpy.pp.highly_variable_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.highly_variable_genes.html#scanpy.pp.highly_variable_genes) flavors `seurat_v3` and `seurat_v3_paper` are now `dask`-compatible I Gold ([#3340](https://github.com/scverse/scanpy/pull/3340))
- rc1 Add [`zarr`](https://zarr.readthedocs.io/en/stable/api/zarr/#zarr) support and `convert_strings_to_categoricals` parameter to [`scanpy.io.write()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.io.write.html#scanpy.io.write)P Angerer ([#3498](https://github.com/scverse/scanpy/pull/3498))
- rc1 Add support for [`scipy.sparse.csr_array`](https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.csr_array.html#scipy.sparse.csr_array) and [`scipy.sparse.csc_array`](https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.csc_array.html#scipy.sparse.csc_array)P Angerer ([#3563](https://github.com/scverse/scanpy/pull/3563))
- rc1 Added a new `compressed` parameter to [`read_10x_mtx()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.io.read_10x_mtx.html#scanpy.io.read_10x_mtx) to support reading uncompressed matrix files produced by tools like STARsolo T Kuthuru ([#3564](https://github.com/scverse/scanpy/pull/3564))
- rc1 Make [`scanpy.get.aggregate()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.get.aggregate.html#scanpy.get.aggregate)[Dask](https://docs.dask.org/en/stable/index.html) compatible with all aggregations except median I Gold ([#3700](https://github.com/scverse/scanpy/pull/3700))
- rc1 Add `filter_unexpressed_genes` parameter to [`scanpy.pp.highly_variable_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.highly_variable_genes.html#scanpy.pp.highly_variable_genes)M Mueller ([#3767](https://github.com/scverse/scanpy/pull/3767))
- rc1 Add column `"n_obs_aggregated"` to [`scanpy.get.aggregate()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.get.aggregate.html#scanpy.get.aggregate) output to show the total number of observations aggregated per group R Disa ([#3824](https://github.com/scverse/scanpy/pull/3824))
- rc1 Add `method='jaccard'` for generating connectivities in [`scanpy.pp.neighbors()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.neighbors.html#scanpy.pp.neighbors)J Pintar ([#3831](https://github.com/scverse/scanpy/pull/3831))
- rc1 Add in `csc`-in-[Dask](https://docs.dask.org/en/stable/index.html) support for [`scanpy.get.aggregate()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.get.aggregate.html#scanpy.get.aggregate)I Gold ([#3872](https://github.com/scverse/scanpy/pull/3872))
- rc1 Raised warnings now always point at user code, not internal scanpy code P Angerer ([#3876](https://github.com/scverse/scanpy/pull/3876))
- Add modularity scoring via [`scanpy.metrics.modularity()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.metrics.modularity.html#scanpy.metrics.modularity) with support for directed/undirected graphs A. Karesh ([#3613](https://github.com/scverse/scanpy/pull/3613))
- Added `distances` parameter to [`scanpy.pp.neighbors()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.neighbors.html#scanpy.pp.neighbors), allowing to compute graphs from a precomputed distance matrix A. Karesh ([#3627](https://github.com/scverse/scanpy/pull/3627))
- [`scanpy.pl.dotplot()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.dotplot.html#scanpy.pl.dotplot) now supports a `group_colors` parameter for custom per-group coloring with perceptually uniform color gradients via OKLab interpolation. R Baber ([#3764](https://github.com/scverse/scanpy/pull/3764))
- Allow specifying graphs in [`scanpy.metrics`](https://scanpy.scverse.org/en/stable/api/metrics.html#module-scanpy.metrics) functions P Angerer ([#3898](https://github.com/scverse/scanpy/pull/3898))

#### Miscellaneous changes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id25)

- rc1 Replaced several internal utilities with their [`fast_array_utils`](https://fast-array-utils.scverse.org/en/stable/index.html#module-fast_array_utils) counterparts P Angerer ([#3598](https://github.com/scverse/scanpy/pull/3598))
- rc1 Deprecate [`scanpy.tl.louvain()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.louvain.html#scanpy.tl.louvain)P Angerer ([#3658](https://github.com/scverse/scanpy/pull/3658))
- rc1 Deprecate `save` parameter of plotting functions. zethson ([#3675](https://github.com/scverse/scanpy/pull/3675))
- rc1 Remove support for the experimental and unmaintained `zappy` library P Angerer ([#3884](https://github.com/scverse/scanpy/pull/3884))
- Remove development extras (`scanpy[test,dev]`) in favor of dependency groups P Angerer ([#3905](https://github.com/scverse/scanpy/pull/3905))

#### Documentation[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id26)

- rc1 Update tutorial [Using dask with Scanpy](https://scanpy.scverse.org/en/stable/tutorials/experimental/dask.html) for [`anndata`](https://anndata.scverse.org/en/stable/api.html#module-anndata) 0.12 (see [scverse/scanpy-tutorials#186](https://github.com/scverse/scanpy-tutorials/pull/186)) I Gold ([#3675](https://github.com/scverse/scanpy/pull/3675))
- rc1 Add a warning message to [`rank_genes_groups()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.rank_genes_groups.html#scanpy.tl.rank_genes_groups) that it is not appropriate for most use-cases G Sturm ([#3792](https://github.com/scverse/scanpy/pull/3792))
- Remove a few falsely advertised parameters from [`scanpy.pl.scatter()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.scatter.html#scanpy.pl.scatter)P Angerer ([#3894](https://github.com/scverse/scanpy/pull/3894))
- Document array type support for most functions in [`pp`](https://scanpy.scverse.org/en/stable/api/preprocessing.html#module-scanpy.pp) and [`tl`](https://scanpy.scverse.org/en/stable/api/tools.html#module-scanpy.tl)P Angerer ([#3895](https://github.com/scverse/scanpy/pull/3895))
- Add cruft config & update authors zethson ([#3896](https://github.com/scverse/scanpy/pull/3896))

#### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id27)

- rc1 Raise fewer redundant warnings, mainly in [`scanpy.pl`](https://scanpy.scverse.org/en/stable/api/plotting.html#module-scanpy.pl) and [`scanpy.datasets`](https://scanpy.scverse.org/en/stable/api/datasets.html#module-scanpy.datasets) functions P Angerer ([#3724](https://github.com/scverse/scanpy/pull/3724))
- rc1 Fix `swap_axes` argument used with `standard_scale` in [`scanpy.pl.dotplot()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.dotplot.html#scanpy.pl.dotplot)I Gold ([#3851](https://github.com/scverse/scanpy/pull/3851))
- rc1 Allow operating on [`obsm`](https://anndata.scverse.org/en/stable/generated/anndata.AnnData.obsm.html#anndata.AnnData.obsm) arrays in [`scanpy.pp.normalize_total()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.normalize_total.html#scanpy.pp.normalize_total) and [`scanpy.pp.pca()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.pca.html#scanpy.pp.pca)P Angerer ([#3863](https://github.com/scverse/scanpy/pull/3863))
- rc1 Fix [`dpt_groups_pseudotime()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.dpt_groups_pseudotime.html#scanpy.pl.dpt_groups_pseudotime)P Angerer ([#3864](https://github.com/scverse/scanpy/pull/3864))
- Fix [`scanpy.get.aggregate()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.get.aggregate.html#scanpy.get.aggregate) when a `by` column misses data P Angerer ([#3906](https://github.com/scverse/scanpy/pull/3906))
- Fix compatibility with pandas 3.0 P Angerer ([#3929](https://github.com/scverse/scanpy/pull/3929))
- Allow [`scanpy.io.read_10x_mtx()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.io.read_10x_mtx.html#scanpy.io.read_10x_mtx) to read numeric gene IDs P Angerer ([#3932](https://github.com/scverse/scanpy/pull/3932))
- Ensure [`scanpy.pp.scrublet()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.scrublet.html#scanpy.pp.scrublet) preserves (categorical) data types within `.obs`etlioglu ([#3939](https://github.com/scverse/scanpy/pull/3939))

## Version 1.11[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-11)

### 1.11.5 2025-10-20[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-11-5)

#### Documentation[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id29)

- Add [Cell-Cycle Scoring and Regression](https://scanpy.scverse.org/en/stable/how-to/cell-cycle.html)P Angerer ([#3816](https://github.com/scverse/scanpy/pull/3816))

#### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id30)

- Deprecate `__version__` and use standard [`version()`](https://docs.python.org/3/library/importlib.metadata.html#importlib.metadata.version) API P Angerer ([#3811](https://github.com/scverse/scanpy/pull/3811))

#### Performance[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id31)

- Optimise [`scanpy.pp.highly_variable_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.highly_variable_genes.html#scanpy.pp.highly_variable_genes) with `batch_key` set for dask arrays M Mueller ([#3735](https://github.com/scverse/scanpy/pull/3735))

### 1.11.4 2025-07-30[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-11-4)

### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id33)

- Recalculate colors when color keys length does not match number of categories e.g., in [`scanpy.pl.umap()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.umap.html#scanpy.pl.umap)Ilan G ([#3725](https://github.com/scverse/scanpy/pull/3725))
- Use `dask` version from `anndata` for the `[dask]` extra in `scanpy`I Gold ([#3737](https://github.com/scverse/scanpy/pull/3737))
- Fix [`leiden()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.leiden.html#scanpy.tl.leiden) with igraph backend on Windows P Angerer ([#3745](https://github.com/scverse/scanpy/pull/3745))
- Fix setup of initial log stream P Angerer ([#3747](https://github.com/scverse/scanpy/pull/3747))
- Remove upper bound from `scipy` version P Angerer ([#3752](https://github.com/scverse/scanpy/pull/3752))

### Documentation[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id34)

- Update [Preprocessing and clustering 3k PBMCs (legacy workflow)](https://scanpy.scverse.org/en/stable/tutorials/basics/clustering-2017.html) for more compatibility I Gold ([#3748](https://github.com/scverse/scanpy/pull/3748))

### 1.11.3 2025-07-01[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-11-3)

### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id36)

- Ensure `axis_nnz` calculates its chunk size/shape correctly with `dask`I Gold ([#3667](https://github.com/scverse/scanpy/pull/3667))
- Replace deprecated `np.in1d` with `np.isin` to silence deprecation warnings. E Ferdman ([#3685](https://github.com/scverse/scanpy/pull/3685))
- Upperbound [`scipy`](https://docs.scipy.org/doc/scipy/index.html#module-scipy) to `1.16.0` due to [statsmodels/statsmodels#9584](https://github.com/statsmodels/statsmodels/issues/9584)I Gold ([#3695](https://github.com/scverse/scanpy/pull/3695))

### Documentation[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id37)

- Fix documentation location for [`scanpy.settings`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.settings.html#scanpy.settings)P Angerer ([#3672](https://github.com/scverse/scanpy/pull/3672))

### 1.11.2 2025-05-28[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-11-2)

### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id39)

- Fix zappy compatibility for clip_array P Angerer ([#3351](https://github.com/scverse/scanpy/pull/3351))
- Fixes an error where `regress_out` would fail to work with `integer` types S Dicks ([#3461](https://github.com/scverse/scanpy/pull/3461))
- Prevent plotting with `mask_obs` from mutating data V Menon ([#3496](https://github.com/scverse/scanpy/pull/3496))
- Prevent [`scanpy.pp.scale()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.scale.html#scanpy.pp.scale) from creating a dask [`Array`](https://docs.dask.org/en/stable/generated/dask.array.Array.html#dask.array.Array) with [`numpy.matrix`](https://numpy.org/doc/stable/reference/generated/numpy.matrix.html#numpy.matrix) chunks P Angerer ([#3597](https://github.com/scverse/scanpy/pull/3597))
- Allow using [`sklearn`](https://scikit-learn.org/stable/api/sklearn.html#module-sklearn) ≥1.6, [Dask](https://docs.dask.org/en/stable/index.html) ≥2024.8, and `sphinx` ≥8.2.1 P Angerer ([#3611](https://github.com/scverse/scanpy/pull/3611))
- Fixed handling of `ext` argument in [`scanpy.io.read()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.io.read.html#scanpy.io.read)I Gold[#3643](https://github.com/scverse/scanpy/pull/3643)
- Fix error message when trying to use `sc.pp.pca(x,zero_center=False)` with a sparse dask array. P Angerer ([#3646](https://github.com/scverse/scanpy/pull/3646))

### Documentation[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id40)

- Clarify use of implementations in [`scanpy.pp.pca()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.pca.html#scanpy.pp.pca) docs. P Angerer ([#3655](https://github.com/scverse/scanpy/pull/3655))

### Performance[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id41)

- Speed up for a categorical regressor in [`regress_out()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.regress_out.html#scanpy.pp.regress_out)S DicksI Gold ([#3353](https://github.com/scverse/scanpy/pull/3353))
- In `pp.normalize_total`, the median is now computed in-memory when using Dask S Dicks ([#3379](https://github.com/scverse/scanpy/pull/3379))
- Speed up `pp.normalize_total` with a numba kernel for `csr-matrices`S Dicks ([#3571](https://github.com/scverse/scanpy/pull/3571))

### 1.11.1 2025-03-31[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-11-1)

#### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id43)

- Fix compatibility with IPython 9 P Angerer ([#3499](https://github.com/scverse/scanpy/pull/3499))
- Prevent too-low matplotlib version from being used P Angerer ([#3534](https://github.com/scverse/scanpy/pull/3534))

#### Features[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id44)

- Allow `covariance_eigh` as a solver option for [`pca()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.pca.html#scanpy.pp.pca) with [`dask.array.Array`](https://docs.dask.org/en/stable/generated/dask.array.Array.html#dask.array.Array) dense data ilan-gold ([#3528](https://github.com/scverse/scanpy/pull/3528))

#### Performance[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id45)

- Speed up wilcoxon rank-sum test with numba G Wu ([#3529](https://github.com/scverse/scanpy/pull/3529))

### 1.11.0 2025-02-14[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-11-0)

Release candidates:

- rc2 2025-01-24
- rc1 2024-12-20

#### Features[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id47)

- rc1[`sample()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.sample.html#scanpy.pp.sample) supports both upsampling and downsampling of observations and variables. [`subsample()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.subsample.html#scanpy.pp.subsample) is now deprecated. G Eraslan & P Angerer ([#943](https://github.com/scverse/scanpy/pull/943))
- rc1 Add `layer` argument to [`scanpy.tl.score_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.score_genes.html#scanpy.tl.score_genes) and [`scanpy.tl.score_genes_cell_cycle()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.score_genes_cell_cycle.html#scanpy.tl.score_genes_cell_cycle)L Zappia ([#2921](https://github.com/scverse/scanpy/pull/2921))
- rc1 Prevent `raw` conflict with `layer` in [`score_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.score_genes.html#scanpy.tl.score_genes)S Dicks ([#3155](https://github.com/scverse/scanpy/pull/3155))
- rc1 Add support for `median` as an aggregation function to [`aggregate()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.get.aggregate.html#scanpy.get.aggregate). This allows for median-based aggregation of data (e.g., pseudobulk), complementing existing methods like mean- and sum-based aggregation M Dehkordi (Farhad) ([#3180](https://github.com/scverse/scanpy/pull/3180))
- rc1 Add `key_added` argument to [`pca()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.pca.html#scanpy.pp.pca), [`tsne()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.tsne.html#scanpy.tl.tsne) and [`umap()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.umap.html#scanpy.tl.umap)P Angerer ([#3184](https://github.com/scverse/scanpy/pull/3184))
- rc1 Support running [`scanpy.pp.pca()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.pca.html#scanpy.pp.pca) on sparse Dask arrays with the `'covariance_eigh'` solver P Angerer ([#3263](https://github.com/scverse/scanpy/pull/3263))
- rc1 Use upstreamed [`PCA`](https://scikit-learn.org/stable/modules/generated/sklearn.decomposition.PCA.html#sklearn.decomposition.PCA) implementation for [`csr_array`](https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.csr_array.html#scipy.sparse.csr_array) and [`csr_matrix`](https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.csr_matrix.html#scipy.sparse.csr_matrix) (see scikit-learn [Version 1.4.0](https://scikit-learn.org/stable/whats_new/v1.4.html#changes-1-4)) P Angerer ([#3267](https://github.com/scverse/scanpy/pull/3267))
- rc1 Add explicit support to [`scanpy.pp.pca()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.pca.html#scanpy.pp.pca) for `svd_solver='covariance_eigh'`P Angerer ([#3296](https://github.com/scverse/scanpy/pull/3296))
- rc1 Add support for [`dask.array.Array`](https://docs.dask.org/en/stable/generated/dask.array.Array.html#dask.array.Array) to [`scanpy.pp.calculate_qc_metrics()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.calculate_qc_metrics.html#scanpy.pp.calculate_qc_metrics)I Gold ([#3307](https://github.com/scverse/scanpy/pull/3307))
- rc1 Support `layer` parameter in [`scanpy.pl.highest_expr_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.highest_expr_genes.html#scanpy.pl.highest_expr_genes)P Angerer ([#3324](https://github.com/scverse/scanpy/pull/3324))
- rc1 Run numba functions single-threaded when called from inside of a [`ThreadPool`](https://docs.python.org/3/library/multiprocessing.html#multiprocessing.pool.ThreadPool)P Angerer ([#3335](https://github.com/scverse/scanpy/pull/3335))
- rc1 Switch [`print_header()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.logging.print_header.html#scanpy.logging.print_header) and [`print_versions()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.logging.print_versions.html#scanpy.logging.print_versions) to [`session_info2`](https://session-info2.readthedocs.io/en/stable/api.html#module-session_info2)P Angerer ([#3384](https://github.com/scverse/scanpy/pull/3384))
- rc1 Add sampling probabilities/mask parameter `p` to [`sample()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.sample.html#scanpy.pp.sample)P Angerer ([#3410](https://github.com/scverse/scanpy/pull/3410))

#### Performance[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id48)

- rc1 Speed up [`regress_out()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.regress_out.html#scanpy.pp.regress_out)P Ashish, P Angerer & S Dicks ([#3284](https://github.com/scverse/scanpy/pull/3284))

#### Documentation[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id49)

- rc1 Improve [`harmony_integrate()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.harmony_integrate.html#scanpy.pp.harmony_integrate) docs D Kühl ([#3362](https://github.com/scverse/scanpy/pull/3362))
- rc1 Raise [`FutureWarning`](https://docs.python.org/3/builtins/exceptions.html#FutureWarning) when calling deprecated [`scanpy.pp`](https://scanpy.scverse.org/en/stable/api/preprocessing.html#module-scanpy.pp) functions P Angerer ([#3380](https://github.com/scverse/scanpy/pull/3380))
- rc1P Angerer ([#3407](https://github.com/scverse/scanpy/pull/3407))

| Deprecate … | in favor of … |
|-------------|---------------|
| scanpy.read_visium() | squidpy.read.visium() |
| scanpy.datasets.visium_sge() | squidpy.datasets.visium() |
| scanpy.pl.spatial() | squidpy.pl.spatial_scatter() |
- rc2 Fix reference in [`scanpy.pp`](https://scanpy.scverse.org/en/stable/api/preprocessing.html#module-scanpy.pp) page D Kazemi ([#3418](https://github.com/scverse/scanpy/pull/3418))

#### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id50)

- rc1 Upper-bound [`sklearn`](https://scikit-learn.org/stable/api/sklearn.html#module-sklearn)`&lt;1.6.0` due to [dask/dask-ml#1002](https://github.com/dask/dask-ml/issues/1002)Ilan Gold ([#3393](https://github.com/scverse/scanpy/pull/3393))
- rc2 Fix [`rank_genes_groups()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.rank_genes_groups.html#scanpy.tl.rank_genes_groups) compatibility with data &gt;10M cells P Angerer ([#3426](https://github.com/scverse/scanpy/pull/3426))
- rc2 Fix [`scanpy.pl.rank_genes_groups()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.rank_genes_groups.html#scanpy.pl.rank_genes_groups)’s `ax` parameter P Angerer ([#3428](https://github.com/scverse/scanpy/pull/3428))

#### Development Process[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#development-process)

- rc2 Fix version number inference in development environments (CI and local) P Angerer ([#3441](https://github.com/scverse/scanpy/pull/3441))

## Version 1.10[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-10)

### 1.10.4 2024-11-12[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-10-4)

#### Breaking changes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id52)

- Remove Python 3.9 support P Angerer ([#3283](https://github.com/scverse/scanpy/pull/3283))

#### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id53)

- Fix [`scanpy.pl.DotPlot.style()`](https://scanpy.scverse.org/en/stable/api/generated/classes/scanpy.pl.DotPlot.style.html#scanpy.pl.DotPlot.style), [`scanpy.pl.MatrixPlot.style()`](https://scanpy.scverse.org/en/stable/api/generated/classes/scanpy.pl.MatrixPlot.style.html#scanpy.pl.MatrixPlot.style), and [`scanpy.pl.StackedViolin.style()`](https://scanpy.scverse.org/en/stable/api/generated/classes/scanpy.pl.StackedViolin.style.html#scanpy.pl.StackedViolin.style) resetting all non-specified parameters P Angerer ([#3206](https://github.com/scverse/scanpy/pull/3206))
- Accept `'group'` instead of `'obs'` for `standard_scale` parameter in [`stacked_violin()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.stacked_violin.html#scanpy.pl.stacked_violin)P Angerer ([#3243](https://github.com/scverse/scanpy/pull/3243))
- Use `density_norm` instead of of `scale` (cont. from [#2844](https://github.com/scverse/scanpy/pull/2844)) in [`violin()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.violin.html#scanpy.pl.violin) and [`stacked_violin()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.stacked_violin.html#scanpy.pl.stacked_violin)P Angerer ([#3244](https://github.com/scverse/scanpy/pull/3244))
- Switched all compatibility adapters for positional parameters to [`FutureWarning`](https://docs.python.org/3/builtins/exceptions.html#FutureWarning)P Angerer ([#3264](https://github.com/scverse/scanpy/pull/3264))
- Catch `PerfectSeparationWarning` during [`regress_out()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.regress_out.html#scanpy.pp.regress_out)J Wagner ([#3275](https://github.com/scverse/scanpy/pull/3275))
- Fix [`scanpy.pp.highly_variable_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.highly_variable_genes.html#scanpy.pp.highly_variable_genes) for batches of size 1 P Angerer ([#3286](https://github.com/scverse/scanpy/pull/3286))
- Fix [`scanpy.pl.scatter()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.scatter.html#scanpy.pl.scatter)’s `color` parameter to take collections as advertised P Angerer ([#3299](https://github.com/scverse/scanpy/pull/3299))
- Fix [`scanpy.pl.highest_expr_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.highest_expr_genes.html#scanpy.pl.highest_expr_genes) when used with a categorical gene symbol column P Angerer ([#3302](https://github.com/scverse/scanpy/pull/3302))

### 1.10.3 2024-09-17[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-10-3)

#### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id55)

- Prevent empty control gene set in [`score_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.score_genes.html#scanpy.tl.score_genes)M Müller ([#2875](https://github.com/scverse/scanpy/pull/2875))
- Fix `subset=True` of [`highly_variable_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.highly_variable_genes.html#scanpy.pp.highly_variable_genes) when `flavor` is `seurat` or `cell_ranger`, and `batch_key!=None`E Roellin ([#3042](https://github.com/scverse/scanpy/pull/3042))
- Add compatibility with [`numpy`](https://numpy.org/doc/stable/reference/index.html#module-numpy) 2.0 P Angerer[#3065](https://github.com/scverse/scanpy/pull/3065) and ([#3115](https://github.com/scverse/scanpy/pull/3115))
- Fix `legend_loc` argument in [`scanpy.pl.embedding()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.embedding.html#scanpy.pl.embedding) not accepting matplotlib parameters P Angerer ([#3163](https://github.com/scverse/scanpy/pull/3163))
- Fix dispersion cutoff in [`highly_variable_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.highly_variable_genes.html#scanpy.pp.highly_variable_genes) in presence of `NaN`s P Angerer ([#3176](https://github.com/scverse/scanpy/pull/3176))
- Fix axis labeling for swapped axes in [`rank_genes_groups_stacked_violin()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.rank_genes_groups_stacked_violin.html#scanpy.pl.rank_genes_groups_stacked_violin)Ilan Gold ([#3196](https://github.com/scverse/scanpy/pull/3196))
- Upper bound dask on account of [scverse/anndata#1579](https://github.com/scverse/anndata/issues/1579)Ilan Gold ([#3217](https://github.com/scverse/scanpy/pull/3217))
- The [fa2-modified](https://github.com/AminAlam/fa2_modified) package replaces [forceatlas2](https://github.com/bhargavchippada/forceatlas2) for the latter’s lack of maintenance A Alam ([#3220](https://github.com/scverse/scanpy/pull/3220))

### 1.10.2 2024-06-25[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-10-2)

#### Development Process[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id57)

- Add performance benchmarking [#2977](https://github.com/scverse/scanpy/pull/2977)R Shrestha, P Angerer

#### Documentation[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id58)

- Document several missing parameters in docstring [#2888](https://github.com/scverse/scanpy/pull/2888)S Cheney
- Fixed incorrect instructions in “testing” dev docs [#2994](https://github.com/scverse/scanpy/pull/2994)I Virshup
- Update marsilea tutorial to use `group_` methods [#3001](https://github.com/scverse/scanpy/pull/3001)I Virshup
- Fixed citations [#3032](https://github.com/scverse/scanpy/pull/3032)P Angerer
- Improve dataset documentation [#3060](https://github.com/scverse/scanpy/pull/3060)P Angerer

#### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id59)

- Compatibility with `matplotlib` 3.9 [#2999](https://github.com/scverse/scanpy/pull/2999)I Virshup
- Add clear errors where `backed` mode-like matrices (i.e., from `sparse_dataset`) are not supported [#3048](https://github.com/scverse/scanpy/pull/3048)I gold
- Write out full pca results when `_choose_representation` is called i.e., [`neighbors()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.neighbors.html#scanpy.pp.neighbors) without [`pca()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.pca.html#scanpy.pp.pca)[#3078](https://github.com/scverse/scanpy/pull/3078)I gold
- Fix deprecated use of `.A` with sparse matrices [#3084](https://github.com/scverse/scanpy/pull/3084)P Angerer
- Fix zappy support [#3089](https://github.com/scverse/scanpy/pull/3089)P Angerer
- Fix dotplot group order with [`pandas`](https://pandas.pydata.org/pandas-docs/stable/index.html#module-pandas) 1.x [#3101](https://github.com/scverse/scanpy/pull/3101)P Angerer

#### Performance[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id60)

- `sparse_mean_variance_axis` now uses all cores for the calculations [#3015](https://github.com/scverse/scanpy/pull/3015)S Dicks
- `pp.highly_variable_genes` with `flavor=seurat_v3` now uses a numba kernel [#3017](https://github.com/scverse/scanpy/pull/3017)S Dicks
- Speed up [`scrublet()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.scrublet.html#scanpy.pp.scrublet)[#3044](https://github.com/scverse/scanpy/pull/3044)S Dicks and [#3056](https://github.com/scverse/scanpy/pull/3056)P Angerer
- Speed up clipping of array in [`scale()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.scale.html#scanpy.pp.scale)[#3100](https://github.com/scverse/scanpy/pull/3100)P Ashish & S Dicks

### 1.10.1 2024-04-09[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-10-1)

#### Documentation[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id62)

- Added [how-to example](https://scanpy.scverse.org/en/stable/how-to/plotting-with-marsilea.html) on plotting with [Marsilea](https://marsilea.readthedocs.io/en/stable/index.html)[#2974](https://github.com/scverse/scanpy/pull/2974)Y Zheng

#### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id63)

- Fix `aggregate` when aggregating by more than two groups [#2965](https://github.com/scverse/scanpy/pull/2965)I Virshup

#### Performance[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id64)

- [`scale()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.scale.html#scanpy.pp.scale) now uses numba kernels for `sparse.csr_matrix` and `sparse.csc_matrix` when `zero_center==False` and `mask_obs` is provided. This greatly speed up execution [#2942](https://github.com/scverse/scanpy/pull/2942)S Dicks

### 1.10.0 2024-03-26[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-10-0)

`scanpy` 1.10 brings a large amount of new features, performance improvements, and improved documentation.

Some highlights:

- Improved support for out-of-core workflows via `dask`. See new tutorial: [Using dask with Scanpy](https://scanpy.scverse.org/en/stable/tutorials/experimental/dask.html) demonstrating counts-to-clusters for 1.4 million cells in &lt;10 min.
- A new [basic clustering tutorial](https://scanpy.scverse.org/en/stable/tutorials/basics/clustering.html) demonstrating an updated workflow.
- Opt-in increased performance for neighbor search and clustering ([how to guide](https://scanpy.scverse.org/en/stable/how-to/knn-transformers.html)).
- Ability to `mask` observations or variables from a number of methods (see [Customizing Scanpy plots](https://scanpy.scverse.org/en/stable/tutorials/plotting/advanced.html) for an example with plotting embeddings)
- A new function [`aggregate()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.get.aggregate.html#scanpy.get.aggregate) for computing aggregations of your data, very useful for pseudo bulking!

#### Features[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id66)

- [`scrublet()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.scrublet.html#scanpy.pp.scrublet) and [`scrublet_simulate_doublets()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.scrublet_simulate_doublets.html#scanpy.pp.scrublet_simulate_doublets) were moved from [`scanpy.external.pp`](https://scanpy.scverse.org/en/stable/external/preprocessing.html#module-scanpy.external.pp) to [`scanpy.pp`](https://scanpy.scverse.org/en/stable/api/preprocessing.html#module-scanpy.pp). The `scrublet` implementation is now maintained as part of scanpy [#2703](https://github.com/scverse/scanpy/pull/2703)P Angerer
- [`scanpy.pp.pca()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.pca.html#scanpy.pp.pca), [`scanpy.pp.scale()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.scale.html#scanpy.pp.scale), [`scanpy.pl.embedding()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.embedding.html#scanpy.pl.embedding), and [`scanpy.experimental.pp.normalize_pearson_residuals_pca()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.experimental.pp.normalize_pearson_residuals_pca.html#scanpy.experimental.pp.normalize_pearson_residuals_pca) now support a `mask` parameter [#2272](https://github.com/scverse/scanpy/pull/2272)C Bright, T Marcella, & P Angerer
- Enhanced dask support for some internal utilities, paving the way for more extensive dask support [#2696](https://github.com/scverse/scanpy/pull/2696)P Angerer
- [`scanpy.pp.highly_variable_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.highly_variable_genes.html#scanpy.pp.highly_variable_genes) supports dask for the default `seurat` and `cell_ranger` flavors [#2809](https://github.com/scverse/scanpy/pull/2809)P Angerer
- New function [`scanpy.get.aggregate()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.get.aggregate.html#scanpy.get.aggregate) which allows grouped aggregations over your data. Useful for pseudobulking! [#2590](https://github.com/scverse/scanpy/pull/2590)Isaac VirshupIlan GoldJon Bloom
- [`scanpy.pp.neighbors()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.neighbors.html#scanpy.pp.neighbors) now has a `transformer` argument allowing the use of different ANN/ KNN libraries [#2536](https://github.com/scverse/scanpy/pull/2536)P Angerer
- [`scanpy.experimental.pp.highly_variable_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.experimental.pp.highly_variable_genes.html#scanpy.experimental.pp.highly_variable_genes) using `flavor='pearson_residuals'` now uses numba for variance computation and is faster [#2612](https://github.com/scverse/scanpy/pull/2612)S Dicks & P Angerer
- [`scanpy.tl.leiden()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.leiden.html#scanpy.tl.leiden) now offers `igraph`’s implementation of the leiden algorithm via via `flavor` when set to `igraph`. `leidenalg`’s implementation is still default, but discouraged. [#2815](https://github.com/scverse/scanpy/pull/2815)I Gold
- [`scanpy.pp.highly_variable_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.highly_variable_genes.html#scanpy.pp.highly_variable_genes) has new flavor `seurat_v3_paper` that is in its implementation consistent with the paper description in Stuart et al 2018. [#2792](https://github.com/scverse/scanpy/pull/2792)E Roellin
- [`scanpy.datasets.blobs()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.datasets.blobs.html#scanpy.datasets.blobs) now accepts a `random_state` argument [#2683](https://github.com/scverse/scanpy/pull/2683)E Roellin
- [`scanpy.pp.pca()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.pca.html#scanpy.pp.pca) and [`scanpy.pp.regress_out()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.regress_out.html#scanpy.pp.regress_out) now accept a layer argument [#2588](https://github.com/scverse/scanpy/pull/2588)S Dicks
- [`scanpy.pp.subsample()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.subsample.html#scanpy.pp.subsample) with `copy=True` can now be called in backed mode [#2624](https://github.com/scverse/scanpy/pull/2624)E Roellin
- [`scanpy.pp.harmony_integrate()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.harmony_integrate.html#scanpy.pp.harmony_integrate) now runs with 64 bit floats improving reproducibility [#2655](https://github.com/scverse/scanpy/pull/2655)S Dicks
- [`scanpy.tl.rank_genes_groups()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.rank_genes_groups.html#scanpy.tl.rank_genes_groups) no longer warns that it’s default was changed from t-test_overestim_var to t-test [#2798](https://github.com/scverse/scanpy/pull/2798)L Heumos
- `scanpy.pp.calculate_qc_metrics` now allows `qc_vars` to be passed as a string [#2859](https://github.com/scverse/scanpy/pull/2859)N Teyssier
- [`scanpy.tl.leiden()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.leiden.html#scanpy.tl.leiden) and [`scanpy.tl.louvain()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.louvain.html#scanpy.tl.louvain) now store clustering parameters in the key provided by the `key_added` parameter instead of always writing to (or overwriting) a default key [#2864](https://github.com/scverse/scanpy/pull/2864)J Fan
- [`scanpy.pp.scale()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.scale.html#scanpy.pp.scale) now clips `np.ndarray` also at `-max_value` for zero-centering [#2913](https://github.com/scverse/scanpy/pull/2913)S Dicks
- Support sparse chunks in dask [`scale()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.scale.html#scanpy.pp.scale), [`normalize_total()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.normalize_total.html#scanpy.pp.normalize_total) and [`highly_variable_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.highly_variable_genes.html#scanpy.pp.highly_variable_genes) (`seurat` and `cell-ranger` tested) [#2856](https://github.com/scverse/scanpy/pull/2856)ilan-gold

#### Documentation[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id67)

- Doc style overhaul [#2220](https://github.com/scverse/scanpy/pull/2220)A Gayoso
- Re-add search-as-you-type, this time via `readthedocs-sphinx-search`[#2805](https://github.com/scverse/scanpy/pull/2805)P Angerer
- Fixed a lot of broken usage examples [#2605](https://github.com/scverse/scanpy/pull/2605)P Angerer
- Improved harmonization of return field of `sc.pp` and `sc.tl` functions [#2742](https://github.com/scverse/scanpy/pull/2742)E Roellin
- Improved docs for `percent_top` argument of [`calculate_qc_metrics()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.calculate_qc_metrics.html#scanpy.pp.calculate_qc_metrics)[#2849](https://github.com/scverse/scanpy/pull/2849)I Virshup
- New basic clustering tutorial ([Preprocessing and clustering](https://scanpy.scverse.org/en/stable/tutorials/basics/clustering.html)), based on one from [scverse-tutorials](https://scverse-tutorials.readthedocs.io/en/latest/notebooks/basic-scrna-tutorial.html)[#2901](https://github.com/scverse/scanpy/pull/2901)I Virshup
- Overhauled [Tutorials](https://scanpy.scverse.org/en/stable/tutorials/index.html) page, and added new [How to](https://scanpy.scverse.org/en/stable/how-to/index.html) section to docs [#2901](https://github.com/scverse/scanpy/pull/2901)I Virshup
- Added a new tutorial on working with dask ([Using dask with Scanpy](https://scanpy.scverse.org/en/stable/tutorials/experimental/dask.html)) [#2901](https://github.com/scverse/scanpy/pull/2901)I GoldI Virshup

#### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id68)

- Updated [`read_visium()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.read_visium.html#scanpy.read_visium) such that it can read spaceranger 2.0 files L Lehner
- Fix [`normalize_total()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.normalize_total.html#scanpy.pp.normalize_total) for dask [#2466](https://github.com/scverse/scanpy/pull/2466)P Angerer
- Fix setting :attr:`scanpy.settings.verbosity` in some cases [#2605](https://github.com/scverse/scanpy/pull/2605)P Angerer
- Fix all remaining pandas warnings [#2789](https://github.com/scverse/scanpy/pull/2789)P Angerer
- Fix some annoying plotting warnings around violin plots [#2844](https://github.com/scverse/scanpy/pull/2844)P Angerer
- Scanpy now has a test job which tests against the minumum versions of the dependencies. In the process of implementing this, many bugs associated with using older versions of `pandas`, `anndata`, `numpy`, and `matplotlib` were fixed. [#2816](https://github.com/scverse/scanpy/pull/2816)I Virshup
- Fix warnings caused by internal usage of `pandas.DataFrame.stack` with `pandas&gt;=2.1`[#2864](https://github.com/scverse/scanpy/pull/2864)I Virshup
- [`scanpy.get.aggregate()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.get.aggregate.html#scanpy.get.aggregate) now always returns [`numpy.ndarray`](https://numpy.org/doc/stable/reference/generated/numpy.ndarray.html#numpy.ndarray)[#2893](https://github.com/scverse/scanpy/pull/2893)S Dicks
- Removes self from array of neighbors for `use_approx_neighbors=True` in [`scrublet()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.scrublet.html#scanpy.pp.scrublet)[#2896](https://github.com/scverse/scanpy/pull/2896)S Dicks
- Compatibility with scipy 1.13 [#2943](https://github.com/scverse/scanpy/pull/2943)I Virshup
- Fix use of [`dendrogram()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.dendrogram.html#scanpy.tl.dendrogram) on highly correlated low precision data [#2928](https://github.com/scverse/scanpy/pull/2928)P Angerer
- Fix pytest deprecation warning [#2879](https://github.com/scverse/scanpy/pull/2879)P Angerer

#### Development Process[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id69)

- Scanpy is now tested against python 3.12 [#2863](https://github.com/scverse/scanpy/pull/2863)ivirshup
- Fix testing package build [#2468](https://github.com/scverse/scanpy/pull/2468)P Angerer

#### Deprecations[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#deprecations)

- Dropped support for Python 3.8. [More details here](https://numpy.org/neps/nep-0029-deprecation_policy.html). [#2695](https://github.com/scverse/scanpy/pull/2695)P Angerer
- Deprecated specifying large numbers of function parameters by position as opposed to by name/keyword in all public APIs. e.g. prefer `sc.tl.umap(adata,min_dist=0.1,spread=0.8)` over `sc.tl.umap(adata,0.1,0.8)`[#2702](https://github.com/scverse/scanpy/pull/2702)P Angerer
- Dropped support for `umap&lt;0.5` for performance reasons. [#2870](https://github.com/scverse/scanpy/pull/2870)P Angerer

## Version 1.9[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-9)

### 1.9.8 2024-01-26[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-9-8)

#### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id71)

- Fix handling of numpy array palettes for old numpy versions [#2832](https://github.com/scverse/scanpy/pull/2832)P Angerer

### 1.9.7 2024-01-25[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-9-7)

#### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id73)

- Fix handling of numpy array palettes (e.g. after write-read cycle) [#2734](https://github.com/scverse/scanpy/pull/2734)P Angerer
- Specify correct version of `matplotlib` dependency [#2733](https://github.com/scverse/scanpy/pull/2733)P Fisher
- Fix [`scanpy.pl.violin()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.violin.html#scanpy.pl.violin) usage of `seaborn.catplot`[#2739](https://github.com/scverse/scanpy/pull/2739)E Roellin
- Fix [`scanpy.pp.highly_variable_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.highly_variable_genes.html#scanpy.pp.highly_variable_genes) to handle the combinations of `inplace` and `subset` consistently [#2757](https://github.com/scverse/scanpy/pull/2757)E Roellin
- Replace usage of various deprecated functionality from [`anndata`](https://anndata.scverse.org/en/stable/api.html#module-anndata) and [`pandas`](https://pandas.pydata.org/pandas-docs/stable/index.html#module-pandas)[#2678](https://github.com/scverse/scanpy/pull/2678)[#2779](https://github.com/scverse/scanpy/pull/2779)P Angerer
- Allow to use default `n_top_genes` when using [`scanpy.pp.highly_variable_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.highly_variable_genes.html#scanpy.pp.highly_variable_genes) flavor `'seurat_v3'`[#2782](https://github.com/scverse/scanpy/pull/2782)P Angerer
- Fix [`scanpy.io.read_10x_mtx()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.io.read_10x_mtx.html#scanpy.io.read_10x_mtx)’s `gex_only=True` mode [#2801](https://github.com/scverse/scanpy/pull/2801)P Angerer

### 1.9.6 2023-10-31[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-9-6)

#### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id75)

- Allow [`scanpy.pl.scatter()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.scatter.html#scanpy.pl.scatter) to accept a [`str`](https://docs.python.org/3/builtins/stdtypes.html#str) palette name [#2571](https://github.com/scverse/scanpy/pull/2571)P Angerer
- Make [`scanpy.external.tl.palantir()`](https://scanpy.scverse.org/en/stable/external/generated/scanpy.external.tl.palantir.html#scanpy.external.tl.palantir) compatible with palantir &gt;=1.3 [#2672](https://github.com/scverse/scanpy/pull/2672)DJ Otto
- Fix [`scanpy.pl.pca()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.pca.html#scanpy.pl.pca) when `return_fig=True` and `annotate_var_explained=True`[#2682](https://github.com/scverse/scanpy/pull/2682)J Wagner
- Temp fix for [#2680](https://github.com/scverse/scanpy/issues/2680) by skipping `seaborn` version 0.13.0 [#2661](https://github.com/scverse/scanpy/pull/2661)P Angerer
- Fix [`scanpy.pp.highly_variable_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.highly_variable_genes.html#scanpy.pp.highly_variable_genes) to not modify the used layer when `flavor=seurat`[#2698](https://github.com/scverse/scanpy/pull/2698)E Roellin
- Prevent pandas from causing infinite recursion when setting a slice of a categorical column [#2719](https://github.com/scverse/scanpy/pull/2719)P Angerer

### 1.9.5 2023-09-08[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-9-5)

#### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id77)

- Remove use of deprecated `dtype` argument to AnnData constructor [#2658](https://github.com/scverse/scanpy/pull/2658)Isaac Virshup

### 1.9.4 2023-08-24[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-9-4)

#### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id79)

- Support scikit-learn 1.3 [#2515](https://github.com/scverse/scanpy/pull/2515)P Angerer
- Deal with `None` value vanishing from things like `.uns['log1p']`[#2546](https://github.com/scverse/scanpy/pull/2546)SP Shen
- Depend on `igraph` instead of `python-igraph`[#2566](https://github.com/scverse/scanpy/pull/2566)P Angerer
- [`rank_genes_groups()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.rank_genes_groups.html#scanpy.tl.rank_genes_groups) now handles unsorted groups as intended [#2589](https://github.com/scverse/scanpy/pull/2589)S Dicks
- [`rank_genes_groups_df()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.get.rank_genes_groups_df.html#scanpy.get.rank_genes_groups_df) now works for [`rank_genes_groups()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.rank_genes_groups.html#scanpy.tl.rank_genes_groups) with `method="logreg"`[#2601](https://github.com/scverse/scanpy/pull/2601)S Dicks
- `scanpy.tl._utils._choose_representation` now works with `n_pcs` if bigger than `settings.N_PCS`[#2610](https://github.com/scverse/scanpy/pull/2610)S Dicks

### 1.9.3 2023-03-02[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-9-3)

#### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id81)

- Variety of fixes against pandas 2.0.0rc0 [#2434](https://github.com/scverse/scanpy/pull/2434)I Virshup

### 1.9.2 2023-02-16[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-9-2)

#### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id83)

- [`highly_variable_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.highly_variable_genes.html#scanpy.pp.highly_variable_genes)`layer` argument now works in tandem with `batches`[#2302](https://github.com/scverse/scanpy/pull/2302)D Schaumont
- [`highly_variable_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.highly_variable_genes.html#scanpy.pp.highly_variable_genes) with `flavor='cell_ranger'` now handles the case in [#2230](https://github.com/scverse/scanpy/issues/2230) where the number of calculated dispersions is less than `n_top_genes`[#2231](https://github.com/scverse/scanpy/pull/2231)L Zappia
- Fix compatibility with matplotlib 3.7 [#2414](https://github.com/scverse/scanpy/pull/2414)I VirshupP Fisher
- Fix scrublet numpy matrix compatibility issue [#2395](https://github.com/scverse/scanpy/pull/2395)A Gayoso

### 1.9.1 2022-04-05[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-9-1)

#### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id85)

- [`normalize_total()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.normalize_total.html#scanpy.pp.normalize_total) works when Dask is not installed [#2209](https://github.com/scverse/scanpy/pull/2209)R Cannoodt
- Fix embedding plots by bumping matplotlib dependency to version 3.4 [#2212](https://github.com/scverse/scanpy/pull/2212)I Virshup

### 1.9.0 2022-04-01[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-9-0)

#### Tutorials[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#tutorials)

- New tutorial on the usage of Pearson Residuals: [How to preprocess UMI count data with analytic Pearson residuals](https://scanpy.scverse.org/en/stable/tutorials/experimental/pearson_residuals.html)J Lause, G Palla
- [Materials](https://github.com/scverse/scanpy-tutorials/tree/master/scanpy_workshop) and [recordings](https://www.youtube.com/playlist?list=PL4rcQcNPLZxWQQH7LlRBMkAo5NWuHX1e3) for Scanpy workshops by Maren Büttner

#### Experimental module[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#experimental-module)

- Added [`scanpy.experimental`](https://scanpy.scverse.org/en/stable/api/experimental.html#module-scanpy.experimental) module! Currently contains functionality related to pearson residuals in [`scanpy.experimental.pp`](https://scanpy.scverse.org/en/stable/api/experimental.html#module-scanpy.experimental.pp)[#1715](https://github.com/scverse/scanpy/pull/1715)J Lause, G Palla, I Virshup. This includes:

- [`normalize_pearson_residuals()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.experimental.pp.normalize_pearson_residuals.html#scanpy.experimental.pp.normalize_pearson_residuals) for Pearson Residuals normalization
- [`highly_variable_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.experimental.pp.highly_variable_genes.html#scanpy.experimental.pp.highly_variable_genes) for HVG selection with Pearson Residuals
- [`normalize_pearson_residuals_pca()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.experimental.pp.normalize_pearson_residuals_pca.html#scanpy.experimental.pp.normalize_pearson_residuals_pca) for Pearson Residuals normalization and dimensionality reduction with PCA
- [`recipe_pearson_residuals()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.experimental.pp.recipe_pearson_residuals.html#scanpy.experimental.pp.recipe_pearson_residuals) for Pearson Residuals normalization, HVG selection and dimensionality reduction with PCA

#### Features[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id87)

- [`filter_rank_genes_groups()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.filter_rank_genes_groups.html#scanpy.tl.filter_rank_genes_groups) now allows to filter with absolute values of log fold change [#1649](https://github.com/scverse/scanpy/pull/1649)S Rybakov
- `_choose_representation` now subsets the provided representation to n_pcs, regardless of the name of the provided representation (should affect mostly [`neighbors()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.neighbors.html#scanpy.pp.neighbors)) [#2179](https://github.com/scverse/scanpy/pull/2179)I VirshupPG Majev
- [`scanpy.pp.scrublet()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.scrublet.html#scanpy.pp.scrublet) (and related functions) can now be used on `AnnData` objects containing multiple batches [#1965](https://github.com/scverse/scanpy/pull/1965)J Manning
- Number of variables plotted with [`pca_loadings()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.pca_loadings.html#scanpy.pl.pca_loadings) can now be controlled with `n_points` argument. Additionally, variables are no longer repeated if the anndata has less than 30 variables [#2075](https://github.com/scverse/scanpy/pull/2075)Yves33
- Dask arrays now work with [`scanpy.pp.normalize_total()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.normalize_total.html#scanpy.pp.normalize_total)[#1663](https://github.com/scverse/scanpy/pull/1663)G Buckley, I Virshup
- [`embedding_density()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.embedding_density.html#scanpy.pl.embedding_density) now allows more than 10 groups [#1936](https://github.com/scverse/scanpy/pull/1936)A Wolf
- Embedding plots can now pass `colorbar_loc` to specify the location of colorbar legend, or pass `None` to not show a colorbar [#1821](https://github.com/scverse/scanpy/pull/1821)A SchaarI Virshup
- Embedding plots now have a `dimensions` argument, which lets users select which dimensions of their embedding to plot and uses the same broadcasting rules as other arguments [#1538](https://github.com/scverse/scanpy/pull/1538)I Virshup
- [`print_versions()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.logging.print_versions.html#scanpy.logging.print_versions) now uses `session_info`[#2089](https://github.com/scverse/scanpy/pull/2089)P AngererI Virshup

#### Ecosystem[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#ecosystem)

Multiple packages have been added to our ecosystem page, including:

- [decoupler](https://github.com/saezlab/decoupler-py) a for footprint analysis and pathway enrichement [#2186](https://github.com/scverse/scanpy/pull/2186)PB Mompel
- [dandelion](https://github.com/zktuong/dandelion) for B-cell receptor analysis [#1953](https://github.com/scverse/scanpy/pull/1953)Z Tuong
- [CIARA](https://github.com/ScialdoneLab/CIARA_python) a feature selection tools for identifying rare cell types [#2175](https://github.com/scverse/scanpy/pull/2175)M Stock

#### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id88)

- Fixed finding variables with `use_raw=True` and `basis=None` in [`scanpy.pl.scatter()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.scatter.html#scanpy.pl.scatter)[#2027](https://github.com/scverse/scanpy/pull/2027)E Rice
- Fixed [`scanpy.pp.scrublet()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.scrublet.html#scanpy.pp.scrublet) to address [#1957](https://github.com/scverse/scanpy/issues/1957)FlMai and ensure raw counts are used for simulation
- Functions in [`scanpy.datasets`](https://scanpy.scverse.org/en/stable/api/datasets.html#module-scanpy.datasets) no longer throw `OldFormatWarnings` when using `anndata``0.8`[#2096](https://github.com/scverse/scanpy/pull/2096)I Virshup
- Fixed use of [`scanpy.pp.neighbors()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.neighbors.html#scanpy.pp.neighbors) with `method='rapids'`: RAPIDS cuML no longer returns a squared Euclidean distance matrix, so we should not square-root the kNN distance matrix. [#1828](https://github.com/scverse/scanpy/pull/1828)M Zaslavsky
- Removed `pytables` dependency by implementing `read_10x_h5` with `h5py` due to installation errors on Windows [#2064](https://github.com/scverse/scanpy/pull/2064)
- Fixed bug in [`scanpy.external.pp.hashsolo()`](https://scanpy.scverse.org/en/stable/external/generated/scanpy.external.pp.hashsolo.html#scanpy.external.pp.hashsolo) where default value was set improperly [#2190](https://github.com/scverse/scanpy/pull/2190)B Reiz
- Fixed bug in [`scanpy.pl.embedding()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.embedding.html#scanpy.pl.embedding) functions where an error could be raised when there were missing values and large numbers of categories [#2187](https://github.com/scverse/scanpy/pull/2187)I Virshup

## Version 1.8[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-8)

### 1.8.2 2021-11-3[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-8-2)

#### Documentation[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id90)

- Update conda installation instructions [#1974](https://github.com/scverse/scanpy/pull/1974)L Heumos

#### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id91)

- Fix plotting after [`scanpy.tl.filter_rank_genes_groups()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.filter_rank_genes_groups.html#scanpy.tl.filter_rank_genes_groups)[#1942](https://github.com/scverse/scanpy/pull/1942)S Rybakov
- Fix `use_raw=None` using [`anndata.AnnData.var_names`](https://anndata.scverse.org/en/stable/generated/anndata.AnnData.var_names.html#anndata.AnnData.var_names) if [`anndata.AnnData.raw`](https://anndata.scverse.org/en/stable/generated/anndata.AnnData.raw.html#anndata.AnnData.raw) is present in [`scanpy.tl.score_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.score_genes.html#scanpy.tl.score_genes)[#1999](https://github.com/scverse/scanpy/pull/1999)M Klein
- Fix compatibility with UMAP 0.5.2 [#2028](https://github.com/scverse/scanpy/pull/2028)L Mcinnes
- Fixed non-determinism in [`scanpy.pl.paga()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.paga.html#scanpy.pl.paga) node positions [#1922](https://github.com/scverse/scanpy/pull/1922)I Virshup

#### Ecosystem[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id92)

- Added PASTE (a tool to align and integrate spatial transcriptomics data) to scanpy ecosystem.

### 1.8.1 2021-07-07[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-8-1)

#### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id94)

- Fixed reproducibility of [`scanpy.tl.score_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.score_genes.html#scanpy.tl.score_genes). Calculation and output is now float64 type. [#1890](https://github.com/scverse/scanpy/pull/1890)I Kucinski
- Workarounds for some changes/ bugs in pandas 1.3 [#1918](https://github.com/scverse/scanpy/pull/1918)I Virshup
- Fixed bug where `sc.pl.paga_compare` could mislabel nodes on the paga graph [#1898](https://github.com/scverse/scanpy/pull/1898)I Virshup
- Fixed handling of `use_raw` with [`scanpy.tl.rank_genes_groups()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.rank_genes_groups.html#scanpy.tl.rank_genes_groups)[#1934](https://github.com/scverse/scanpy/pull/1934)I Virshup

### 1.8.0 2021-06-28[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-8-0)

#### Metrics module[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#metrics-module)

- Added [`scanpy.metrics`](https://scanpy.scverse.org/en/stable/api/metrics.html#module-scanpy.metrics) module!

- Added [`scanpy.metrics.gearys_c()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.metrics.gearys_c.html#scanpy.metrics.gearys_c) for spatial autocorrelation [#915](https://github.com/scverse/scanpy/pull/915)I Virshup
- Added [`scanpy.metrics.morans_i()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.metrics.morans_i.html#scanpy.metrics.morans_i) for global spatial autocorrelation [#1740](https://github.com/scverse/scanpy/pull/1740)I Virshup, G Palla
- Added [`scanpy.metrics.confusion_matrix()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.metrics.confusion_matrix.html#scanpy.metrics.confusion_matrix) for comparing labellings [#915](https://github.com/scverse/scanpy/pull/915)I Virshup

#### Features[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id96)

- Added `layer` and `copy` kwargs to [`normalize_total()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.normalize_total.html#scanpy.pp.normalize_total)[#1667](https://github.com/scverse/scanpy/pull/1667)I Virshup
- Added `vcenter` and `norm` arguments to the plotting functions [#1551](https://github.com/scverse/scanpy/pull/1551)G Eraslan
- Standardized and expanded available arguments to the `sc.pl.rank_genes_groups*` family of functions. [#1529](https://github.com/scverse/scanpy/pull/1529)F RamirezI Virshup

- See examples sections of [`rank_genes_groups_dotplot()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.rank_genes_groups_dotplot.html#scanpy.pl.rank_genes_groups_dotplot) and [`rank_genes_groups_matrixplot()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.rank_genes_groups_matrixplot.html#scanpy.pl.rank_genes_groups_matrixplot) for demonstrations.
- [`scanpy.tl.tsne()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.tsne.html#scanpy.tl.tsne) now supports the metric argument and records the passed parameters [#1854](https://github.com/scverse/scanpy/pull/1854)I Virshup
- [`scanpy.pl.scrublet_score_distribution()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.scrublet_score_distribution.html#scanpy.pl.scrublet_score_distribution) now uses same API as other scanpy functions for saving/ showing plots [#1741](https://github.com/scverse/scanpy/pull/1741)J Manning

#### Ecosystem[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id97)

- Added [Cubé](https://github.com/connerlambden/Cube) to ecosystem page [#1878](https://github.com/scverse/scanpy/pull/1878)C Lambden
- Added `triku` a feature selection method to the ecosystem page [#1722](https://github.com/scverse/scanpy/pull/1722)AM Ascensión
- Added `dorothea` and `progeny` to the ecosystem page [#1767](https://github.com/scverse/scanpy/pull/1767)P Badia-i-Mompel

#### Documentation[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id98)

- Added [Community](https://scanpy.scverse.org/en/stable/community.html) page to docs [#1856](https://github.com/scverse/scanpy/pull/1856)I Virshup
- Added rendered examples to many plotting functions [#1664](https://github.com/scverse/scanpy/issues/1664)A SchaarL Zappiabio-laL HetzelL DonyM ButtnerK HrovatinF RamirezI VirshupLouisK92mayarali
- Integrated [DocSearch](https://docsearch.algolia.com/), a find-as-you-type documentation index search. [#1754](https://github.com/scverse/scanpy/pull/1754)P Angerer
- Reorganized reference docs [#1753](https://github.com/scverse/scanpy/pull/1753)I Virshup
- Clarified docs issues for [`neighbors()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.neighbors.html#scanpy.pp.neighbors), [`diffmap()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.diffmap.html#scanpy.tl.diffmap), [`calculate_qc_metrics()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.calculate_qc_metrics.html#scanpy.pp.calculate_qc_metrics)[#1680](https://github.com/scverse/scanpy/pull/1680)G Palla
- Fixed typos in grouped plot doc-strings [#1877](https://github.com/scverse/scanpy/pull/1877)C Rands
- Extended examples for differential expression plotting. [#1529](https://github.com/scverse/scanpy/pull/1529)F Ramirez

- See [`rank_genes_groups_dotplot()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.rank_genes_groups_dotplot.html#scanpy.pl.rank_genes_groups_dotplot) or [`rank_genes_groups_matrixplot()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.rank_genes_groups_matrixplot.html#scanpy.pl.rank_genes_groups_matrixplot) for examples.

#### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id99)

- Fix [`scanpy.pl.paga_path()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.paga_path.html#scanpy.pl.paga_path)`TypeError` with recent versions of anndata [#1047](https://github.com/scverse/scanpy/pull/1047)P Angerer
- Fix detection of whether IPython is running [#1844](https://github.com/scverse/scanpy/pull/1844)I Virshup
- Fixed reproducibility of [`scanpy.tl.diffmap()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.diffmap.html#scanpy.tl.diffmap) (added random_state) [#1858](https://github.com/scverse/scanpy/pull/1858)I Kucinski
- Fixed errors and warnings from embedding plots with small numbers of categories after `sns.set_palette` was called [#1886](https://github.com/scverse/scanpy/pull/1886)I Virshup
- Fixed handling of `gene_symbols` argument in a number of `sc.pl.rank_genes_groups*` functions [#1529](https://github.com/scverse/scanpy/pull/1529)F RamirezI Virshup
- Fixed handling of `use_raw` for `sc.tl.rank_genes_groups` when no `.raw` is present [#1895](https://github.com/scverse/scanpy/pull/1895)I Virshup
- [`scanpy.pl.rank_genes_groups_violin()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.rank_genes_groups_violin.html#scanpy.pl.rank_genes_groups_violin) now works for `raw=False`[#1669](https://github.com/scverse/scanpy/pull/1669)M van den Beek
- [`scanpy.pl.dotplot()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.dotplot.html#scanpy.pl.dotplot) now uses `smallest_dot` argument correctly [#1771](https://github.com/scverse/scanpy/pull/1771)S Flemming

#### Development Process[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id100)

- Switched to [flit](https://flit.pypa.io/en/stable/index.html) for building and deploying the package, a simple tool with an easy to understand command line interface and metadata [#1527](https://github.com/scverse/scanpy/pull/1527)P Angerer
- Use [pre-commit](https://pre-commit.com) for style checks [#1684](https://github.com/scverse/scanpy/pull/1684)[#1848](https://github.com/scverse/scanpy/pull/1848)L HeumosI Virshup

#### Deprecations[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id101)

- Dropped support for Python 3.6. [More details here](https://numpy.org/neps/nep-0029-deprecation_policy.html). [#1897](https://github.com/scverse/scanpy/pull/1897)I Virshup
- Deprecated `layers` and `layers_norm` kwargs to [`normalize_total()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.normalize_total.html#scanpy.pp.normalize_total)[#1667](https://github.com/scverse/scanpy/pull/1667)I Virshup
- Deprecated `MulticoreTSNE` backend for [`scanpy.tl.tsne()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.tsne.html#scanpy.tl.tsne)[#1854](https://github.com/scverse/scanpy/pull/1854)I Virshup

## Version 1.7[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-7)

### 1.7.2 2021-04-07[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-7-2)

#### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id103)

- [`scanpy.logging.print_versions()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.logging.print_versions.html#scanpy.logging.print_versions) now works when `python&lt;3.8`[#1691](https://github.com/scverse/scanpy/pull/1691)I Virshup
- [`scanpy.pp.regress_out()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.regress_out.html#scanpy.pp.regress_out) now uses `joblib` as the parallel backend, and should stop oversubscribing threads [#1694](https://github.com/scverse/scanpy/pull/1694)I Virshup
- [`scanpy.pp.highly_variable_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.highly_variable_genes.html#scanpy.pp.highly_variable_genes) with `flavor="seurat_v3"` now returns correct gene means and -variances when used with `batch_key`[#1732](https://github.com/scverse/scanpy/pull/1732)J Lause
- [`scanpy.pp.highly_variable_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.highly_variable_genes.html#scanpy.pp.highly_variable_genes) now throws a warning instead of an error when non-integer values are passed for method `"seurat_v3"`. The check can be skipped by passing `check_values=False`. [#1679](https://github.com/scverse/scanpy/pull/1679)G Palla

#### Ecosystem[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id104)

- Added `triku` a feature selection method to the ecosystem page [#1722](https://github.com/scverse/scanpy/pull/1722)AM Ascensión
- Added `dorothea` and `progeny` to the ecosystem page [#1767](https://github.com/scverse/scanpy/pull/1767)P Badia-i-Mompel

### 1.7.1 2021-02-24[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-7-1)

#### Documentation[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id106)

- More twitter handles for core devs [#1676](https://github.com/scverse/scanpy/pull/1676)G Eraslan

#### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id107)

- [`dendrogram()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.dendrogram.html#scanpy.tl.dendrogram) use `1-correlation` as distance matrix to compute the dendrogram [#1614](https://github.com/scverse/scanpy/pull/1614)F Ramirez
- Fixed [`obs_df()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.get.obs_df.html#scanpy.get.obs_df)/ [`var_df()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.get.var_df.html#scanpy.get.var_df) erroring when `keys` not passed [#1637](https://github.com/scverse/scanpy/pull/1637)I Virshup
- Fixed argument handling for [`scanpy.pp.scrublet()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.scrublet.html#scanpy.pp.scrublet)J Manning
- Fixed passing of `kwargs` to [`scanpy.pl.violin()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.violin.html#scanpy.pl.violin) when `stripplot` was also used [#1655](https://github.com/scverse/scanpy/pull/1655)M van den Beek
- Fixed colorbar creation in `scanpy.pl.timeseries_as_heatmap`[#1654](https://github.com/scverse/scanpy/pull/1654)M van den Beek

### 1.7.0 2021-02-03[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-7-0)

#### Features[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id109)

- Add new 10x Visium datasets to [`visium_sge()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.datasets.visium_sge.html#scanpy.datasets.visium_sge)[#1473](https://github.com/scverse/scanpy/pull/1473)G Palla
- Enable download of source image for 10x visium datasets in [`visium_sge()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.datasets.visium_sge.html#scanpy.datasets.visium_sge)[#1506](https://github.com/scverse/scanpy/pull/1506)H Spitzer
- Refactor of [`scanpy.pl.spatial()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.spatial.html#scanpy.pl.spatial). Better support for plotting without an image, as well as directly providing images [#1512](https://github.com/scverse/scanpy/pull/1512)G Palla
- Dict input for [`scanpy.queries.enrich()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.queries.enrich.html#scanpy.queries.enrich)[#1488](https://github.com/scverse/scanpy/pull/1488)G Eraslan
- [`rank_genes_groups_df()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.get.rank_genes_groups_df.html#scanpy.get.rank_genes_groups_df) can now return fraction of cells in a group expressing a gene, and allows retrieving values for multiple groups at once [#1388](https://github.com/scverse/scanpy/pull/1388)G Eraslan
- Color annotations for gene sets in [`heatmap()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.heatmap.html#scanpy.pl.heatmap) are now matched to color for cluster [#1511](https://github.com/scverse/scanpy/pull/1511)L Sikkema
- PCA plots can now annotate axes with variance explained [#1470](https://github.com/scverse/scanpy/pull/1470)bfurtwa
- Plots with `groupby` arguments can now group by values in the index by passing the index’s name (like `pd.DataFrame.groupby`). [#1583](https://github.com/scverse/scanpy/pull/1583)F Ramirez
- Added `na_color` and `na_in_legend` keyword arguments to [`embedding()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.embedding.html#scanpy.pl.embedding) plots. Allows specifying color for missing or filtered values in plots like [`umap()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.umap.html#scanpy.pl.umap) or [`spatial()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.spatial.html#scanpy.pl.spatial)[#1356](https://github.com/scverse/scanpy/pull/1356)I Virshup
- [`embedding()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.embedding.html#scanpy.pl.embedding) plots now support passing `dict` of `{cluster_name:cluster_color,...}` for palette argument [#1392](https://github.com/scverse/scanpy/pull/1392)I Virshup

#### External tools (new)[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#external-tools-new)

- Add [Scanorama](https://github.com/brianhie/scanorama) integration to scanpy external API ([`scanorama_integrate()`](https://scanpy.scverse.org/en/stable/external/generated/scanpy.external.pp.scanorama_integrate.html#scanpy.external.pp.scanorama_integrate), Hie *et al.* [[2019](https://scanpy.scverse.org/en/stable/references.html#id27)]) [#1332](https://github.com/scverse/scanpy/pull/1332)B Hie
- Scrublet [[Wolock *et al.*, 2019](https://scanpy.scverse.org/en/stable/references.html#id80)] integration: [`scrublet()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.scrublet.html#scanpy.pp.scrublet), [`scrublet_simulate_doublets()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.scrublet_simulate_doublets.html#scanpy.pp.scrublet_simulate_doublets), and plotting method [`scrublet_score_distribution()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.scrublet_score_distribution.html#scanpy.pl.scrublet_score_distribution)[#1476](https://github.com/scverse/scanpy/pull/1476)J Manning
- [`hashsolo()`](https://scanpy.scverse.org/en/stable/external/generated/scanpy.external.pp.hashsolo.html#scanpy.external.pp.hashsolo) for HTO demultiplexing [[Bernstein *et al.*, 2020](https://scanpy.scverse.org/en/stable/references.html#id9)][#1432](https://github.com/scverse/scanpy/pull/1432)NJ Bernstein
- Added [scirpy](https://github.com/icbi-lab/scirpy) (sc-AIRR analysis) to ecosystem page [#1453](https://github.com/scverse/scanpy/pull/1453)G Sturm
- Added [scvi-tools](https://scvi-tools.org) to ecosystem page [#1421](https://github.com/scverse/scanpy/pull/1421)A Gayoso

#### External tools (changes)[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#external-tools-changes)

- Updates for [`palantir()`](https://scanpy.scverse.org/en/stable/external/generated/scanpy.external.tl.palantir.html#scanpy.external.tl.palantir) and [`palantir_results()`](https://scanpy.scverse.org/en/stable/external/generated/scanpy.external.tl.palantir_results.html#scanpy.external.tl.palantir_results)[#1245](https://github.com/scverse/scanpy/pull/1245)A Mousa
- Fixes to [`harmony_timeseries()`](https://scanpy.scverse.org/en/stable/external/generated/scanpy.external.tl.harmony_timeseries.html#scanpy.external.tl.harmony_timeseries) docs [#1248](https://github.com/scverse/scanpy/pull/1248)A Mousa
- Support for `leiden` clustering by [`scanpy.external.tl.phenograph()`](https://scanpy.scverse.org/en/stable/external/generated/scanpy.external.tl.phenograph.html#scanpy.external.tl.phenograph)[#1080](https://github.com/scverse/scanpy/pull/1080)A Mousa
- Deprecate `scanpy.external.pp.scvi`[#1554](https://github.com/scverse/scanpy/pull/1554)G Xing
- Updated default params of [`sam()`](https://scanpy.scverse.org/en/stable/external/generated/scanpy.external.tl.sam.html#scanpy.external.tl.sam) to work with larger data [#1540](https://github.com/scverse/scanpy/pull/1540)A Tarashansky

#### Documentation[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id113)

- [New contribution guide](https://scanpy.scverse.org/en/stable/dev/index.html#contribution-guide)[#1544](https://github.com/scverse/scanpy/pull/1544)I Virshup
- `zsh` installation instructions [#1444](https://github.com/scverse/scanpy/pull/1444)P Angerer

#### Performance[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id114)

- Speed up [`read_10x_h5()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.io.read_10x_h5.html#scanpy.io.read_10x_h5)[#1402](https://github.com/scverse/scanpy/pull/1402)P Weiler
- Speed ups for [`obs_df()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.get.obs_df.html#scanpy.get.obs_df)[#1499](https://github.com/scverse/scanpy/pull/1499)F Ramirez

#### Bugfixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#bugfixes)

- Consistent fold-change, fractions calculation for filter_rank_genes_groups [#1391](https://github.com/scverse/scanpy/pull/1391)S Rybakov
- Fixed bug where `score_genes` would error if one gene was passed [#1398](https://github.com/scverse/scanpy/pull/1398)I Virshup
- Fixed `log1p` inplace on integer dense arrays [#1400](https://github.com/scverse/scanpy/pull/1400)I Virshup
- Fix docstring formatting for [`rank_genes_groups()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.rank_genes_groups.html#scanpy.tl.rank_genes_groups)[#1417](https://github.com/scverse/scanpy/pull/1417)P Weiler
- Removed `PendingDeprecationWarning`sfromuseof`np.matrix`[#1424](https://github.com/scverse/scanpy/pull/1424)P Weiler
- Fixed indexing byg in `~scanpy.pp.highly_variable_genes`[#1456](https://github.com/scverse/scanpy/pull/1456)V Bergen
- Fix default number of genes for marker_genes_overlap [#1464](https://github.com/scverse/scanpy/pull/1464)MD Luecken
- Fixed passing `groupby` and `dendrogram_key` to [`dendrogram()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.dendrogram.html#scanpy.tl.dendrogram)[#1465](https://github.com/scverse/scanpy/pull/1465)M Varma
- Fixed download path of `pbmc3k_processed`[#1472](https://github.com/scverse/scanpy/pull/1472)D Strobl
- Better error message when computing DE with a group of size 1 [#1490](https://github.com/scverse/scanpy/pull/1490)J Manning
- Update cugraph API usage for v0.16 [#1494](https://github.com/scverse/scanpy/pull/1494)R Ilango
- Fixed `marker_gene_overlap` default value for `top_n_markers`[#1464](https://github.com/scverse/scanpy/pull/1464)MD Luecken
- Pass `random_state` to RAPIDs UMAP [#1474](https://github.com/scverse/scanpy/pull/1474)C Nolet
- Fixed `anndata` version requirement for [`concat()`](https://anndata.scverse.org/en/stable/generated/anndata.concat.html#anndata.concat) (re-exported from scanpy as `sc.concat`) [#1491](https://github.com/scverse/scanpy/pull/1491)I Virshup
- Fixed the width of the progress bar when downloading data [#1507](https://github.com/scverse/scanpy/pull/1507)M Klein
- Updated link for `moignard15` dataset [#1542](https://github.com/scverse/scanpy/pull/1542)I Virshup
- Fixed bug where calling `set_figure_params` could block if IPython was installed, but not used. [#1547](https://github.com/scverse/scanpy/pull/1547)I Virshup
- [`violin()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.violin.html#scanpy.pl.violin) no longer fails if `.raw` not present [#1548](https://github.com/scverse/scanpy/pull/1548)I Virshup
- [`spatial()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.spatial.html#scanpy.pl.spatial) refactoring and better handling of spatial data [#1512](https://github.com/scverse/scanpy/pull/1512)G Palla
- [`pca()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.pca.html#scanpy.pp.pca) works with `chunked=True` again [#1592](https://github.com/scverse/scanpy/pull/1592)I Virshup
- [`ingest()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.ingest.html#scanpy.tl.ingest) now works with umap-learn 0.5.0 [#1601](https://github.com/scverse/scanpy/pull/1601)S Rybakov

## Version 1.6[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-6)

### 1.6.0 2020-08-15[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-6-0)

This release includes an overhaul of [`dotplot()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.dotplot.html#scanpy.pl.dotplot), [`matrixplot()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.matrixplot.html#scanpy.pl.matrixplot), and [`stacked_violin()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.stacked_violin.html#scanpy.pl.stacked_violin) ([#1210](https://github.com/scverse/scanpy/pull/1210)F Ramirez), and of the internals of [`rank_genes_groups()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.rank_genes_groups.html#scanpy.tl.rank_genes_groups) ([#1156](https://github.com/scverse/scanpy/pull/1156)S Rybakov).

#### Overhaul of [`dotplot()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.dotplot.html#scanpy.pl.dotplot), [`matrixplot()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.matrixplot.html#scanpy.pl.matrixplot), and [`stacked_violin()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.stacked_violin.html#scanpy.pl.stacked_violin)[#1210](https://github.com/scverse/scanpy/pull/1210)F Ramirez[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#overhaul-of-dotplot-matrixplot-and-stacked-violin-1210-f-ramirez)

- An overhauled tutorial [Core plotting functions](https://scanpy.scverse.org/en/stable/tutorials/plotting/core.html).
- New plotting classes can be accessed directly (e.g., [`DotPlot`](https://scanpy.scverse.org/en/stable/api/generated/classes/scanpy.pl.DotPlot.html#scanpy.pl.DotPlot)) or using the `return_fig` param.
- It is possible to plot log fold change and p-values in the [`rank_genes_groups_dotplot()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.rank_genes_groups_dotplot.html#scanpy.pl.rank_genes_groups_dotplot) family of functions.
- Added `ax` parameter which allows embedding the plot in other images.
- Added option to include a bar plot instead of the dendrogram containing the cell/observation totals per category.
- Return a dictionary of axes for further manipulation. This includes the main plot, legend and dendrogram to totals
- Legends can be removed.
- The `groupby` param can take a list of categories, e.g., `groupby=[‘tissue’,‘celltype’]`.
- Added padding parameter to `dotplot` and `stacked_violin`. [#1270](https://github.com/scverse/scanpy/pull/1270)
- Added title for colorbar and positioned as in dotplot for [`matrixplot()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.matrixplot.html#scanpy.pl.matrixplot).
- [`dotplot()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.dotplot.html#scanpy.pl.dotplot) changes:

- Improved the colorbar and size legend for dotplots. Now the colorbar and size have titles, which can be modified using the `colorbar_title` and `size_title` params. They also align at the bottom of the image and do not shrink if the dotplot image is smaller.
- Allow plotting genes in rows and categories in columns (`swap_axes`).
- Using [`DotPlot`](https://scanpy.scverse.org/en/stable/api/generated/classes/scanpy.pl.DotPlot.html#scanpy.pl.DotPlot), the `dot_edge_color` and line width can be modified, a grid can be added, and other modifications are enabled.
- A new style was added in which the dots are replaced by an empty circle and the square behind the circle is colored (like in matrixplots).
- [`stacked_violin()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.stacked_violin.html#scanpy.pl.stacked_violin) changes:

- Violin colors can be colored based on average gene expression as in dotplots.
- The linewidth of the violin plots is thinner.
- Removed the tics for the y-axis as they tend to overlap with each other. Using the style method they can be displayed if needed.

#### Additions[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#additions)

- [`concat()`](https://anndata.scverse.org/en/stable/generated/anndata.concat.html#anndata.concat) is now exported from scanpy, see [Concatenation](https://anndata.scverse.org/en/stable/tutorials/concatenation.html) for more info. [#1338](https://github.com/scverse/scanpy/pull/1338)I Virshup
- Added highly variable gene selection strategy from Seurat v3 [#1204](https://github.com/scverse/scanpy/pull/1204)A Gayoso
- Added [CellRank](https://github.com/theislab/cellrank/) to scanpy ecosystem [#1304](https://github.com/scverse/scanpy/pull/1304)giovp
- Added `backup_url` param to [`read_10x_h5()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.io.read_10x_h5.html#scanpy.io.read_10x_h5)[#1296](https://github.com/scverse/scanpy/pull/1296)A Gayoso
- Allow prefix for [`read_10x_mtx()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.io.read_10x_mtx.html#scanpy.io.read_10x_mtx)[#1250](https://github.com/scverse/scanpy/pull/1250)G Sturm
- Optional tie correction for the `'wilcoxon'` method in [`rank_genes_groups()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.rank_genes_groups.html#scanpy.tl.rank_genes_groups)[#1330](https://github.com/scverse/scanpy/pull/1330)S Rybakov
- Use `sinfo` for [`print_versions()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.logging.print_versions.html#scanpy.logging.print_versions) and add [`print_header()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.logging.print_header.html#scanpy.logging.print_header) to do what it previously did. [#1338](https://github.com/scverse/scanpy/pull/1338)I Virshup[#1373](https://github.com/scverse/scanpy/pull/1373)

#### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id116)

- Avoid warning in [`rank_genes_groups()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.rank_genes_groups.html#scanpy.tl.rank_genes_groups) if ‘t-test’ is passed [#1303](https://github.com/scverse/scanpy/pull/1303)A Wolf
- Restrict sphinx version to &lt;3.1, &gt;3.0 [#1297](https://github.com/scverse/scanpy/pull/1297)I Virshup
- Clean up `_ranks` and fix `dendrogram` for scipy 1.5 [#1290](https://github.com/scverse/scanpy/pull/1290)S Rybakov
- Use `.raw` to translate gene symbols if applicable [#1278](https://github.com/scverse/scanpy/pull/1278)E Rice
- Fix `diffmap` ([#1262](https://github.com/scverse/scanpy/issues/1262)) G Eraslan
- Fix `neighbors` in `spring_project`[#1260](https://github.com/scverse/scanpy/issues/1260)S Rybakov
- Fix default size of dot in spatial plots [#1255](https://github.com/scverse/scanpy/pull/1255)[#1253](https://github.com/scverse/scanpy/issues/1253)giovp
- Bumped version requirement of `scipy` to `scipy&gt;1.4` to support `rmatmat` argument of `LinearOperator`[#1246](https://github.com/scverse/scanpy/issues/1246)I Virshup
- Fix asymmetry of scores for the `'wilcoxon'` method in [`rank_genes_groups()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.rank_genes_groups.html#scanpy.tl.rank_genes_groups)[#754](https://github.com/scverse/scanpy/issues/754)S Rybakov
- Avoid trimming of gene names in [`rank_genes_groups()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.rank_genes_groups.html#scanpy.tl.rank_genes_groups)[#753](https://github.com/scverse/scanpy/issues/753)S Rybakov

## Version 1.5[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-5)

### 1.5.1 2020-05-21[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-5-1)

#### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id118)

- Fixed a bug in [`pca()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.pca.html#scanpy.pp.pca), where `random_state` did not have an effect for sparse input [#1240](https://github.com/scverse/scanpy/pull/1240)I Virshup
- Fixed docstring in [`pca()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.pca.html#scanpy.pp.pca) which included an unused argument [#1240](https://github.com/scverse/scanpy/pull/1240)I Virshup

### 1.5.0 2020-05-15[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-5-0)

The `1.5.0` release adds a lot of new functionality, much of which takes advantage of [`anndata`](https://anndata.scverse.org/en/stable/api.html#module-anndata) updates `0.7.0-0.7.2`. Highlights of this release include support for spatial data, dedicated handling of graphs in AnnData, sparse PCA, an interface with scvi, and others.

#### Spatial data support[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#spatial-data-support)

- Tutorials for basic analysis and integration with single cell data G Palla
- [`read_visium()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.read_visium.html#scanpy.read_visium) read 10x Visium data [#1034](https://github.com/scverse/scanpy/pull/1034)G Palla, P Angerer, I Virshup
- [`visium_sge()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.datasets.visium_sge.html#scanpy.datasets.visium_sge) load Visium data directly from 10x Genomics [#1013](https://github.com/scverse/scanpy/pull/1013)M Mirkazemi, G Palla, P Angerer
- [`spatial()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.spatial.html#scanpy.pl.spatial) plot spatial data [#1012](https://github.com/scverse/scanpy/pull/1012)G Palla, P Angerer

#### New functionality[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#new-functionality)

- Many functions, like [`neighbors()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.neighbors.html#scanpy.pp.neighbors) and [`umap()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.umap.html#scanpy.tl.umap), now store cell-by-cell graphs in [`obsp`](https://anndata.scverse.org/en/stable/generated/anndata.AnnData.obsp.html#anndata.AnnData.obsp)[#1118](https://github.com/scverse/scanpy/pull/1118)S Rybakov
- [`scale()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.scale.html#scanpy.pp.scale) and [`log1p()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.log1p.html#scanpy.pp.log1p) can be used on any element in [`layers`](https://anndata.scverse.org/en/stable/generated/anndata.AnnData.layers.html#anndata.AnnData.layers) or [`obsm`](https://anndata.scverse.org/en/stable/generated/anndata.AnnData.obsm.html#anndata.AnnData.obsm)[#1173](https://github.com/scverse/scanpy/pull/1173)I Virshup

#### External tools[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#external-tools)

- `scanpy.external.pp.scvi` for preprocessing with scVI [#1085](https://github.com/scverse/scanpy/pull/1085)G Xing
- Guide for using `ScanpyinR`[#1186](https://github.com/scverse/scanpy/pull/1186)L Zappia

#### Performance[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id120)

- [`pca()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.pca.html#scanpy.pp.pca) now uses efficient implicit centering for sparse matrices. This can lead to signifigantly improved performance for large datasets [#1066](https://github.com/scverse/scanpy/pull/1066)A Tarashansky
- [`score_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.score_genes.html#scanpy.tl.score_genes) now has an efficient implementation for sparse matrices with missing values [#1196](https://github.com/scverse/scanpy/pull/1196)redst4r.

Warning

The new [`pca()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.pca.html#scanpy.pp.pca) implementation can result in slightly different results for sparse matrices. See the pr ([#1066](https://github.com/scverse/scanpy/pull/1066)) and documentation for more info.

#### Code design[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#code-design)

- [`stacked_violin()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.stacked_violin.html#scanpy.pl.stacked_violin) can now be used as a subplot [#1084](https://github.com/scverse/scanpy/pull/1084)P Angerer
- [`score_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.score_genes.html#scanpy.tl.score_genes) has improved logging [#1119](https://github.com/scverse/scanpy/pull/1119)G Eraslan
- [`scale()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.scale.html#scanpy.pp.scale) now saves mean and standard deviation in the [`var`](https://anndata.scverse.org/en/stable/generated/anndata.AnnData.var.html#anndata.AnnData.var)[#1173](https://github.com/scverse/scanpy/pull/1173)A Wolf
- [`harmony_timeseries()`](https://scanpy.scverse.org/en/stable/external/generated/scanpy.external.tl.harmony_timeseries.html#scanpy.external.tl.harmony_timeseries)[#1091](https://github.com/scverse/scanpy/pull/1091)A Mousa

#### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id121)

- [`combat()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.combat.html#scanpy.pp.combat) now works when `obs_names` aren’t unique. [#1215](https://github.com/scverse/scanpy/pull/1215)I Virshup
- [`scale()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.scale.html#scanpy.pp.scale) can now be used on dense arrays without centering [#1160](https://github.com/scverse/scanpy/pull/1160)simonwm
- [`regress_out()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.regress_out.html#scanpy.pp.regress_out) now works when some features are constant [#1194](https://github.com/scverse/scanpy/pull/1194)simonwm
- [`normalize_total()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.normalize_total.html#scanpy.pp.normalize_total) errored if the passed object was a view [#1200](https://github.com/scverse/scanpy/pull/1200)I Virshup
- [`neighbors()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.neighbors.html#scanpy.pp.neighbors) sometimes ignored the `n_pcs` param [#1124](https://github.com/scverse/scanpy/pull/1124)V Bergen
- [`ebi_expression_atlas()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.datasets.ebi_expression_atlas.html#scanpy.datasets.ebi_expression_atlas) which contained some out-of-date URLs [#1102](https://github.com/scverse/scanpy/pull/1102)I Virshup
- [`ingest()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.ingest.html#scanpy.tl.ingest) for UMAP `0.4`[#1165](https://github.com/scverse/scanpy/pull/1165)S Rybakov
- [`louvain()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.louvain.html#scanpy.tl.louvain) for Louvain `0.6`[#1197](https://github.com/scverse/scanpy/pull/1197)I Virshup
- [`highly_variable_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.highly_variable_genes.html#scanpy.pp.highly_variable_genes) which could lead to incorrect results when the `batch_key` argument was used [#1180](https://github.com/scverse/scanpy/pull/1180)G Eraslan
- [`ingest()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.ingest.html#scanpy.tl.ingest) where an inconsistent number of neighbors was used [#1111](https://github.com/scverse/scanpy/pull/1111)S Rybakov

## Version 1.4[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-4)

### 1.4.6 2020-03-17[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-4-6)

#### Functionality in `external`[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#functionality-in-external)

- [`sam()`](https://scanpy.scverse.org/en/stable/external/generated/scanpy.external.tl.sam.html#scanpy.external.tl.sam) self-assembling manifolds [[Tarashansky *et al.*, 2019](https://scanpy.scverse.org/en/stable/references.html#id67)][#903](https://github.com/scverse/scanpy/pull/903)A Tarashansky
- [`harmony_timeseries()`](https://scanpy.scverse.org/en/stable/external/generated/scanpy.external.tl.harmony_timeseries.html#scanpy.external.tl.harmony_timeseries) for trajectory inference on discrete time points [#994](https://github.com/scverse/scanpy/pull/994)A Mousa
- [`wishbone()`](https://scanpy.scverse.org/en/stable/external/generated/scanpy.external.tl.wishbone.html#scanpy.external.tl.wishbone) for trajectory inference (bifurcations) [#1063](https://github.com/scverse/scanpy/pull/1063)A Mousa

#### Code design[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id124)

- [`violin`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.violin.html#scanpy.pl.violin) now reads `.uns['colors_...']`[#1029](https://github.com/scverse/scanpy/pull/1029)michalk8

#### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id125)

- adapt [`ingest()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.ingest.html#scanpy.tl.ingest) for UMAP 0.4 [#1038](https://github.com/scverse/scanpy/pull/1038)[#1106](https://github.com/scverse/scanpy/pull/1106)S Rybakov
- compat with matplotlib 3.1 and 3.2 [#1090](https://github.com/scverse/scanpy/pull/1090)I Virshup, P Angerer
- fix PAGA for new igraph [#1037](https://github.com/scverse/scanpy/pull/1037)P Angerer
- fix rapids compat of louvain [#1079](https://github.com/scverse/scanpy/pull/1079)LouisFaure

### 1.4.5 2019-12-30[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-4-5)

Please install `scanpy==1.4.5.post3` instead of `scanpy==1.4.5`.

#### New functionality[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id127)

- [`ingest()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.ingest.html#scanpy.tl.ingest) maps labels and embeddings of reference data to new data [Integrating data using ingest and BBKNN](https://scanpy.scverse.org/en/stable/tutorials/basics/integrating-data-using-ingest.html)[#651](https://github.com/scverse/scanpy/pull/651)S Rybakov, A Wolf
- [`queries`](https://scanpy.scverse.org/en/stable/api/queries.html#module-scanpy.queries) recieved many updates including enrichment through [gprofiler](https://biit.cs.ut.ee/gprofiler/) and more advanced biomart queries [#467](https://github.com/scverse/scanpy/pull/467)I Virshup
- [`set_figure_params()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.set_figure_params.html#scanpy.set_figure_params) allows setting `figsize` and accepts `facecolor='white'`, useful for working in dark mode A Wolf

#### Code design[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id128)

- [`downsample_counts`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.downsample_counts.html#scanpy.pp.downsample_counts) now always preserves the dtype of it’s input, instead of converting floats to ints [#865](https://github.com/scverse/scanpy/pull/865)I Virshup
- allow specifying a base for [`log1p()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.log1p.html#scanpy.pp.log1p)[#931](https://github.com/scverse/scanpy/pull/931)G Eraslan
- run neighbors on a GPU using rapids [#830](https://github.com/scverse/scanpy/pull/830)T White
- param docs from typed params P Angerer
- [`embedding_density()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.embedding_density.html#scanpy.tl.embedding_density) now only takes one positional argument; similar for [`embedding_density()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.embedding_density.html#scanpy.pl.embedding_density), which gains a param `groupby`[#965](https://github.com/scverse/scanpy/pull/965)A Wolf
- webpage overhaul, ecosystem page, release notes, tutorials overhaul [#960](https://github.com/scverse/scanpy/pull/960)[#966](https://github.com/scverse/scanpy/pull/966)A Wolf

Warning

- changed default `solver` in [`pca()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.pca.html#scanpy.pp.pca) from `auto` to `arpack`
- changed default `use_raw` in [`score_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.score_genes.html#scanpy.tl.score_genes) from `False` to `None`

### 1.4.4 2019-07-20[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-4-4)

#### New functionality[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id130)

- [`scanpy.get`](https://scanpy.scverse.org/en/stable/api/get.html#module-scanpy.get) adds helper functions for extracting data in convenient formats [#619](https://github.com/scverse/scanpy/pull/619)I Virshup

#### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id131)

- Stopped deprecations warnings from AnnData `0.6.22`I Virshup

#### Code design[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id132)

- [`normalize_total()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.normalize_total.html#scanpy.pp.normalize_total) gains param `exclude_highly_expressed`, and `fraction` is renamed to `max_fraction` with better docs A Wolf

### 1.4.3 2019-05-14[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-4-3)

#### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id134)

- [`neighbors()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.neighbors.html#scanpy.pp.neighbors) correctly infers `n_neighbors` again from `params`, which was temporarily broken in `v1.4.2`I Virshup

#### Code design[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id135)

- [`calculate_qc_metrics()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.calculate_qc_metrics.html#scanpy.pp.calculate_qc_metrics) is single threaded by default for datasets under 300,000 cells – allowing cached compilation [#615](https://github.com/scverse/scanpy/pull/615)I Virshup

### 1.4.2 2019-05-06[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-4-2)

#### New functionality[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id137)

- [`combat()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.combat.html#scanpy.pp.combat) supports additional covariates which may include adjustment variables or biological condition [#618](https://github.com/scverse/scanpy/pull/618)G Eraslan
- [`highly_variable_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.highly_variable_genes.html#scanpy.pp.highly_variable_genes) has a `batch_key` option which performs HVG selection in each batch separately to avoid selecting genes that vary strongly across batches [#622](https://github.com/scverse/scanpy/pull/622)G Eraslan

#### Bug fixes[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id138)

- [`rank_genes_groups()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.rank_genes_groups.html#scanpy.tl.rank_genes_groups) t-test implementation doesn’t return NaN when variance is 0, also changed to scipy’s implementation [#621](https://github.com/scverse/scanpy/pull/621)I Virshup
- [`umap()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.umap.html#scanpy.tl.umap) with `init_pos='paga'` detects correct `dtype`A Wolf
- [`louvain()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.louvain.html#scanpy.tl.louvain) and [`leiden()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.leiden.html#scanpy.tl.leiden) auto-generate `key_added=louvain_R` upon passing `restrict_to`, which was temporarily changed in `1.4.1`A Wolf

#### Code design[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id139)

- [`neighbors()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.neighbors.html#scanpy.pp.neighbors) and [`umap()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.umap.html#scanpy.tl.umap) got rid of UMAP legacy code and introduced UMAP as a dependency [#576](https://github.com/scverse/scanpy/pull/576)S Rybakov

### 1.4.1 2019-04-26[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-4-1)

#### New functionality[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id141)

- Scanpy has a command line interface again. Invoking it with `scanpysomecommand[args]` calls `scanpy-somecommand[args]`, except for builtin commands (currently `scanpysettings`) [#604](https://github.com/scverse/scanpy/pull/604)P Angerer
- [`ebi_expression_atlas()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.datasets.ebi_expression_atlas.html#scanpy.datasets.ebi_expression_atlas) allows convenient download of EBI expression atlas I Virshup
- [`marker_gene_overlap()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.marker_gene_overlap.html#scanpy.tl.marker_gene_overlap) computes overlaps of marker genes M Luecken
- [`filter_rank_genes_groups()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.filter_rank_genes_groups.html#scanpy.tl.filter_rank_genes_groups) filters out genes based on fold change and fraction of cells expressing genes F Ramirez
- [`normalize_total()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.normalize_total.html#scanpy.pp.normalize_total) replaces `sc.pp.normalize_per_cell`, is more efficient and provides a parameter to only normalize using a fraction of expressed genes S Rybakov
- [`downsample_counts()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.downsample_counts.html#scanpy.pp.downsample_counts) has been sped up, changed default value of `replace` parameter to `False`[#474](https://github.com/scverse/scanpy/pull/474)I Virshup
- [`embedding_density()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.embedding_density.html#scanpy.tl.embedding_density) computes densities on embeddings [#543](https://github.com/scverse/scanpy/pull/543)M Luecken
- [`palantir()`](https://scanpy.scverse.org/en/stable/external/generated/scanpy.external.tl.palantir.html#scanpy.external.tl.palantir) interfaces Palantir [[Setty *et al.*, 2019](https://scanpy.scverse.org/en/stable/references.html#id62)][#493](https://github.com/scverse/scanpy/pull/493)A Mousa

#### Code design[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id143)

- `.layers` support of scatter plots F Ramirez
- fix double-logarithmization in compute of log fold change in [`rank_genes_groups()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.rank_genes_groups.html#scanpy.tl.rank_genes_groups)A Muñoz-Rojas
- fix return sections of docs P Angerer

## Version 1.3[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-3)

### 1.3.8 2019-02-05[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-3-8)

- various documentation and dev process improvements
- Added [`combat()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.combat.html#scanpy.pp.combat) function for batch effect correction [[Johnson *et al.*, 2006](https://scanpy.scverse.org/en/stable/references.html#id32), [Leek *et al.*, 2017](https://scanpy.scverse.org/en/stable/references.html#id39), [Pedersen, 2012](https://scanpy.scverse.org/en/stable/references.html#id53)][#398](https://github.com/scverse/scanpy/pull/398)M Lange

### 1.3.7 2019-01-02[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-3-7)

- API changed from `importscanpyassc` to `importscanpy.apiassc`.
- [`phenograph()`](https://scanpy.scverse.org/en/stable/external/generated/scanpy.external.tl.phenograph.html#scanpy.external.tl.phenograph) wraps the graph clustering package Phenograph [[Levine *et al.*, 2015](https://scanpy.scverse.org/en/stable/references.html#id40)]thanks to A Mousa

### 1.3.6 2018-12-11[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-3-6)

#### Major updates[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#major-updates)

- a new plotting gallery for `visualizing-marker-genes`F Ramirez
- tutorials are integrated on ReadTheDocs, `pbmc3k` and `paga-paul15`A Wolf

#### Interactive exploration of analysis results through *manifold viewers*[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#interactive-exploration-of-analysis-results-through-manifold-viewers)

- CZI’s [cellxgene](https://github.com/chanzuckerberg/cellxgene) directly reads `.h5ad` files the cellxgene developers
- the [UCSC Single Cell Browser](https://github.com/maximilianh/cellBrowser) requires exporting via [`cellbrowser()`](https://scanpy.scverse.org/en/stable/external/generated/scanpy.external.exporting.cellbrowser.html#scanpy.external.exporting.cellbrowser)M Haeussler

#### Code design[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id149)

- [`highly_variable_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.highly_variable_genes.html#scanpy.pp.highly_variable_genes) supersedes `sc.pp.filter_genes_dispersion`, it gives the same results but, by default, expects logarithmized data and doesn’t subset A Wolf

### 1.3.5 2018-12-09[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-3-5)

- uncountable figure improvements [#369](https://github.com/scverse/scanpy/pull/369)F Ramirez

### 1.3.4 2018-11-24[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-3-4)

- [`leiden()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.leiden.html#scanpy.tl.leiden) wraps the recent graph clustering package by Traag *et al.* [[2019](https://scanpy.scverse.org/en/stable/references.html#id70)]K Polanski
- [`bbknn()`](https://scanpy.scverse.org/en/stable/external/generated/scanpy.external.pp.bbknn.html#scanpy.external.pp.bbknn) wraps the recent batch correction package [[Polański *et al.*, 2019](https://scanpy.scverse.org/en/stable/references.html#id56)]K Polanski
- [`calculate_qc_metrics()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.calculate_qc_metrics.html#scanpy.pp.calculate_qc_metrics) caculates a number of quality control metrics, similar to `calculateQCMetrics` from *Scater*[[McCarthy *et al.*, 2017](https://scanpy.scverse.org/en/stable/references.html#id44)]I Virshup

### 1.3.3 2018-11-05[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-3-3)

#### Major updates[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id156)

- a fully distributed preprocessing backend T White and the Laserson Lab

#### Code design[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id157)

- [`read_10x_h5()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.io.read_10x_h5.html#scanpy.io.read_10x_h5) and [`read_10x_mtx()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.io.read_10x_mtx.html#scanpy.io.read_10x_mtx) read Cell Ranger 3.0 outputs [#334](https://github.com/scverse/scanpy/pull/334)Q Gong

Note

Also see changes in anndata 0.6.

- changed default compression to `None` in [`write_h5ad()`](https://anndata.scverse.org/en/stable/generated/anndata.AnnData.write_h5ad.html#anndata.AnnData.write_h5ad) to speed up read and write, disk space use is usually less critical
- performance gains in [`write_h5ad()`](https://anndata.scverse.org/en/stable/generated/anndata.AnnData.write_h5ad.html#anndata.AnnData.write_h5ad) due to better handling of strings and categories S Rybakov

### 1.3.1 2018-09-03[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-3-1)

#### RNA velocity in single cells [[La Manno *et al.*, 2018](https://scanpy.scverse.org/en/stable/references.html#id36)][#](https://scanpy.scverse.org/en/stable/release-notes/index.html#rna-velocity-in-single-cells-lamanno2018)

- Scanpy and AnnData support loom’s layers so that computations for single-cell RNA velocity [[La Manno *et al.*, 2018](https://scanpy.scverse.org/en/stable/references.html#id36)] become feasible S Rybakov and V Bergen
- [scvelo](https://github.com/theislab/scvelo) harmonizes with Scanpy and is able to process loom files with splicing information produced by Velocyto [[La Manno *et al.*, 2018](https://scanpy.scverse.org/en/stable/references.html#id36)], it runs a lot faster than the count matrix analysis of Velocyto and provides several conceptual developments

#### Plotting ([Generic](https://scanpy.scverse.org/en/stable/api/plotting.html#pl-generic))[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#plotting-pl-generic)

- [`dotplot()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.dotplot.html#scanpy.pl.dotplot) for visualizing genes across conditions and clusters, see [here](https://gist.github.com/fidelram/2289b7a8d6da055fb058ac9a79ed485c)[#199](https://github.com/scverse/scanpy/pull/199)F Ramirez
- [`heatmap()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.heatmap.html#scanpy.pl.heatmap) for pretty heatmaps [#175](https://github.com/scverse/scanpy/pull/175)F Ramirez
- [`violin()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.violin.html#scanpy.pl.violin) produces very compact overview figures with many panels [#175](https://github.com/scverse/scanpy/pull/175)F Ramirez

#### There now is a section on imputation in [external](https://scanpy.scverse.org/en/stable/external/index.html):[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#there-now-is-a-section-on-imputation-in-external)

- [`magic()`](https://scanpy.scverse.org/en/stable/external/generated/scanpy.external.pp.magic.html#scanpy.external.pp.magic) for imputation using data diffusion [[van Dijk *et al.*, 2018](https://scanpy.scverse.org/en/stable/references.html#id84)][#187](https://github.com/scverse/scanpy/pull/187)S Gigante
- `sc.external.pp.dca` for imputation and latent space construction using an autoencoder [[Eraslan *et al.*, 2019](https://scanpy.scverse.org/en/stable/references.html#id18)][#186](https://github.com/scverse/scanpy/pull/186)G Eraslan

## Version 1.2[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-2)

### 1.2.1 2018-06-08[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-2-1)

#### Plotting of [Generic](https://scanpy.scverse.org/en/stable/api/plotting.html#pl-generic) marker genes and quality control.[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#plotting-of-pl-generic-marker-genes-and-quality-control)

- [`highest_expr_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.highest_expr_genes.html#scanpy.pl.highest_expr_genes) for quality control; plot genes with highest mean fraction of cells, similar to `plotQC` of *Scater*[[McCarthy *et al.*, 2017](https://scanpy.scverse.org/en/stable/references.html#id44)][#169](https://github.com/scverse/scanpy/pull/169)F Ramirez

### 1.2.0 2018-06-08[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-2-0)

- [`paga()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.paga.html#scanpy.tl.paga) improved, see [PAGA](https://github.com/theislab/paga); the default model changed, restore the previous default model by passing `model='v1.0'`

## Version 1.1[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-1)

### 1.1.0 2018-06-01[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-1-0)

- [`set_figure_params()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.set_figure_params.html#scanpy.set_figure_params) by default passes `vector_friendly=True` and allows you to produce reasonablly sized pdfs by rasterizing large scatter plots A Wolf
- [`draw_graph()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.draw_graph.html#scanpy.tl.draw_graph) defaults to the ForceAtlas2 layout [[Chippada, 2018](https://scanpy.scverse.org/en/stable/references.html#id14), [Jacomy *et al.*, 2014](https://scanpy.scverse.org/en/stable/references.html#id31)], which is often more visually appealing and whose computation is much faster S Wollock
- [`scatter()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.scatter.html#scanpy.pl.scatter) also plots along variables axis MD Luecken
- [`pca()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.pca.html#scanpy.pp.pca) and [`log1p()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.log1p.html#scanpy.pp.log1p) support chunk processing S Rybakov
- [`regress_out()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.regress_out.html#scanpy.pp.regress_out) is back to multiprocessing F Ramirez
- [`read()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.io.read.html#scanpy.io.read) reads compressed text files G Eraslan
- [`mitochondrial_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.queries.mitochondrial_genes.html#scanpy.queries.mitochondrial_genes) for querying mito genes FG Brundu
- [`mnn_correct()`](https://scanpy.scverse.org/en/stable/external/generated/scanpy.external.pp.mnn_correct.html#scanpy.external.pp.mnn_correct) for batch correction [[Haghverdi *et al.*, 2018](https://scanpy.scverse.org/en/stable/references.html#id25), [Kang, 2018](https://scanpy.scverse.org/en/stable/references.html#id33)]
- [`phate()`](https://scanpy.scverse.org/en/stable/external/generated/scanpy.external.tl.phate.html#scanpy.external.tl.phate) for low-dimensional embedding [[Moon *et al.*, 2019](https://scanpy.scverse.org/en/stable/references.html#id47)]S Gigante
- [`sandbag()`](https://scanpy.scverse.org/en/stable/external/generated/scanpy.external.tl.sandbag.html#scanpy.external.tl.sandbag), [`cyclone()`](https://scanpy.scverse.org/en/stable/external/generated/scanpy.external.tl.cyclone.html#scanpy.external.tl.cyclone) for scoring genes [[Fechtner, 2018](https://scanpy.scverse.org/en/stable/references.html#id19), [Scialdone *et al.*, 2015](https://scanpy.scverse.org/en/stable/references.html#id59)]

## Version 1.0[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-0)

### 1.0.0 2018-03-30[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v1-0-0)

#### Major updates[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#id173)

- Scanpy is much faster and more memory efficient: preprocess, cluster and visualize 1.3M cells in [6h](https://github.com/scverse/scanpy_usage/blob/master/170522_visualizing_one_million_cells/), 130K cells in [14min](https://github.com/scverse/scanpy_usage/blob/master/170522_visualizing_one_million_cells/logfile_130K.txt), and 68K cells in [3min](https://nbviewer.jupyter.org/github/scverse/scanpy_usage/blob/master/170503_zheng17/zheng17.ipynb)A Wolf
- the API gained a preprocessing function [`neighbors()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.neighbors.html#scanpy.pp.neighbors) and a class [`Neighbors()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.Neighbors.html#scanpy.Neighbors) to which all basic graph computations are delegated A Wolf

Warning

Upgrading to 1.0 isn’t fully backwards compatible in the following changes

- the graph-based tools [`louvain()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.louvain.html#scanpy.tl.louvain)[`dpt()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.dpt.html#scanpy.tl.dpt)[`draw_graph()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.draw_graph.html#scanpy.tl.draw_graph)[`umap()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.umap.html#scanpy.tl.umap)[`diffmap()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.diffmap.html#scanpy.tl.diffmap)[`paga()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.paga.html#scanpy.tl.paga) require prior computation of the graph: `sc.pp.neighbors(adata,n_neighbors=5);sc.tl.louvain(adata)` instead of previously `sc.tl.louvain(adata,n_neighbors=5)`
- install `numba` via `condainstallnumba`, which replaces cython
- the default connectivity measure (dpt will look different using default settings) changed. setting `method='gauss'` in `sc.pp.neighbors` uses gauss kernel connectivities and reproduces the previous behavior, see, for instance in the example [paul15](https://nbviewer.jupyter.org/github/scverse/scanpy_usage/blob/master/170502_paul15/paul15.ipynb).
- namings of returned annotation have changed for less bloated AnnData objects, which means that some of the unstructured annotation of old AnnData files is not recognized anymore
- replace occurances of `group_by` with `groupby` (consistency with `pandas`)
- it is worth checking out the notebook examples to see changes, e.g. the [seurat](https://nbviewer.jupyter.org/github/scverse/scanpy_usage/blob/master/170505_seurat/seurat.ipynb) example.
- upgrading scikit-learn from 0.18 to 0.19 changed the implementation of PCA, some results might therefore look slightly different

#### Further updates[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#further-updates)

- UMAP [[McInnes *et al.*, 2018](https://scanpy.scverse.org/en/stable/references.html#id45)] can serve as a first visualization of the data just as tSNE, in contrast to tSNE, UMAP directly embeds the single-cell graph and is faster; UMAP is also used for measuring connectivities and computing neighbors, see [`neighbors()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.neighbors.html#scanpy.pp.neighbors)A Wolf
- graph abstraction: AGA is renamed to [PAGA](https://github.com/theislab/paga): [`paga()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.paga.html#scanpy.tl.paga); now, it only measures connectivities between partitions of the single-cell graph, pseudotime and clustering need to be computed separately via [`louvain()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.louvain.html#scanpy.tl.louvain) and [`dpt()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.dpt.html#scanpy.tl.dpt), the connectivity measure has been improved A Wolf
- logistic regression for finding marker genes [`rank_genes_groups()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.rank_genes_groups.html#scanpy.tl.rank_genes_groups) with parameter `method='logreg'`A Wolf
- [`louvain()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.louvain.html#scanpy.tl.louvain) provides a better implementation for reclustering via `restrict_to`A Wolf
- scanpy no longer modifies rcParams upon import, call :func:`scanpy.set_figure_params` to set the ‘scanpy style’ A Wolf
- default cache directory is `./cache/`, set `settings.cachedir` to change this; nested directories in this are avoided A Wolf
- show edges in scatter plots based on graph visualization [`draw_graph()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.draw_graph.html#scanpy.tl.draw_graph) and [`umap()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.umap.html#scanpy.tl.umap) by passing `edges=True`A Wolf
- [`downsample_counts()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pp.downsample_counts.html#scanpy.pp.downsample_counts) for downsampling counts MD Luecken
- default `'louvain_groups'` are called `'louvain'`A Wolf
- `'X_diffmap'` contains the zero component, plotting remains unchanged A Wolf

## Version 0.4[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v0-4)

### 0.4.4 2018-02-26[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v0-4-4)

- embed cells using [`umap()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.umap.html#scanpy.tl.umap)[[McInnes *et al.*, 2018](https://scanpy.scverse.org/en/stable/references.html#id45)][#92](https://github.com/scverse/scanpy/pull/92)G Eraslan
- score sets of genes, e.g. for cell cycle, using [`score_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.score_genes.html#scanpy.tl.score_genes)[[Satija *et al.*, 2015](https://scanpy.scverse.org/en/stable/references.html#id57)]: [notebook](https://nbviewer.jupyter.org/github/theislab/scanpy_usage/blob/master/180209_cell_cycle/cell_cycle.ipynb)

### 0.4.3 2018-02-09[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v0-4-3)

- [`clustermap()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.clustermap.html#scanpy.pl.clustermap): heatmap from hierarchical clustering, based on [`seaborn.clustermap()`](https://seaborn.pydata.org/generated/seaborn.clustermap.html#seaborn.clustermap)[[Waskom *et al.*, 2016](https://scanpy.scverse.org/en/stable/references.html#id74)]A Wolf
- only return [`matplotlib.axes.Axes`](https://matplotlib.org/stable/api/_as_gen/matplotlib.axes.Axes.html#matplotlib.axes.Axes) in plotting functions of `sc.pl` when `show=False`, otherwise `None`A Wolf

### 0.4.2 2018-01-07[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v0-4-2)

- amendments in [PAGA](https://github.com/theislab/paga) and its plotting functions A Wolf

### 0.4.0 2017-12-23[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v0-4-0)

- export to [SPRING](https://github.com/AllonKleinLab/SPRING/)[[Weinreb *et al.*, 2017](https://scanpy.scverse.org/en/stable/references.html#id75)] for interactive visualization of data: [spring tutorial](https://github.com/scverse/scanpy_usage/tree/master/171111_SPRING_export)S Wollock

## Version 0.3[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v0-3)

### 0.3.2 2017-11-29[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v0-3-2)

- finding marker genes via [`rank_genes_groups_violin()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.rank_genes_groups_violin.html#scanpy.pl.rank_genes_groups_violin) improved, see [#51](https://github.com/scverse/scanpy/issues/51)F Ramirez

### 0.3.0 2017-11-16[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v0-3-0)

- [`AnnData`](https://anndata.scverse.org/en/stable/generated/anndata.AnnData.html#anndata.AnnData) gains method `AnnData.concatenate`A Wolf
- [`AnnData`](https://anndata.scverse.org/en/stable/generated/anndata.AnnData.html#anndata.AnnData) is available as the separate [anndata](https://pypi.org/project/anndata/) package P Angerer, A Wolf
- results of [PAGA](https://github.com/theislab/paga) simplified A Wolf

## Version 0.2[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v0-2)

### 0.2.9 2017-10-25[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v0-2-9)

#### Initial release of the new trajectory inference method [PAGA](https://github.com/theislab/paga)[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#initial-release-of-the-new-trajectory-inference-method-paga)

- [`paga()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.paga.html#scanpy.tl.paga) computes an abstracted, coarse-grained (PAGA) graph of the neighborhood graph A Wolf
- [`paga_compare()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.paga_compare.html#scanpy.pl.paga_compare) plot this graph next an embedding A Wolf
- [`paga_path()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.pl.paga_path.html#scanpy.pl.paga_path) plots a heatmap through a node sequence in the PAGA graph A Wolf

### 0.2.1 2017-07-24[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v0-2-1)

Scanpy includes preprocessing, visualization, clustering, pseudotime and trajectory inference, differential expression testing and simulation of gene regulatory networks. The implementation efficiently deals with [datasets of more than one million cells](https://github.com/scverse/scanpy_usage/tree/master/170522_visualizing_one_million_cells). A Wolf, P Angerer

## Version 0.1[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v0-1)

### 0.1.0 2017-05-17[#](https://scanpy.scverse.org/en/stable/release-notes/index.html#v0-1-0)

Scanpy computationally outperforms and allows reproducing both the [Cell Ranger R kit’s](https://github.com/scverse/scanpy_usage/tree/master/170503_zheng17) and most of [Seurat’s](https://github.com/scverse/scanpy_usage/tree/master/170505_seurat) clustering workflows. A Wolf, P Angerer
