# Chapter 9: Spatial Omics

## Core Idea
Spatially resolved transcriptomics adds a spatial coordinate to each measurement, enabling analysis of tissue architecture. Three analysis layers stack on top of an annotated dataset: **neighborhood statistics** (which cell types are spatially associated), **spatial domains** (tissue regions defined by joint gene + spatial similarity), **spatially variable genes** (genes with non-random spatial patterns), and **deconvolution / imputation** (resolving spot-level mixed data into cell-type abundances, or mapping scRNA-seq onto targeted in-situ data).

## Frameworks Introduced

- **Three spatial resolution tiers**
  - When to use: pick analysis strategy by data type
  - How:
    1. **Multi-cell** (Visium 55μm, slide-seq 10μm) — spots contain mixed cells; needs deconvolution
    2. **Single-cell** (HDST, slide-seqV2, stereo-seq, MERFISH, IMC, cyCIF, CODEX) — each spot is one cell, possibly targeted gene panel
    3. **Sub-cellular** (MERFISH, seqFISH+) — individual transcript coordinates; aggregate to single-cell via segmentation
  - Why: method choice depends on whether you need deconvolution, segmentation, or imputation

- **Squidpy neighborhood statistics (3 tools)**
  - When to use: after cell-type annotation, as a first spatial analysis
  - How:
    1. `sq.gr.spatial_neighbors(adata)` — build spatial kNN graph from coordinates
    2. `sq.gr.nhood_enrichment(adata, cluster_key=...)` — permutation test of cluster pair spatial co-occurrence (z-score + count)
    3. `sq.gr.interaction_matrix(adata, cluster_key=...)` — descriptive count of cluster-pair edges (no test)
    4. `sq.gr.co_occurrence(adata, cluster_key=...)` — conditional probability across increasing radii: `p(exp|cond) / p(exp)`
  - Why: identifies candidate cell-cell communication pairs by spatial proximity; co-occurrence gives distance-dependent signal

- **Spatial domains (Squidpy joint graph + SpaGCN)**
  - When to use: identify tissue regions reflecting both gene expression and spatial proximity
  - How:
    1. **Squidpy approach** — compute kNN graph on PCA (`sc.pp.neighbors`) AND spatial graph (`sq.gr.spatial_neighbors`); combine as `(1-α)·G_genes + α·G_space`; Leiden on joint graph
    2. **SpaGCN** — graph convolutional network over joint graph of gene expression + spatial location + histology image; `calculate_adj_matrix` with `beta` (spot area ~55μm for Visium) and `alpha` (histology weight); find `l` (length scale) given `p` (neighborhood contribution, 0.5 for Visium); optional refinement step (majority vote of neighbors)
  - Why: expression-only clustering produces fragmented spatial maps; spatial smoothing via graph convolution yields coherent tissue regions

- **Spatially variable genes (SVGs)**
  - When to use: identify genes with non-random spatial patterns (orthogonal to HVGs)
  - How:
    1. **Moran's I** (Squidpy, fast) — spatial autocorrelation: `I = (n/W) · Σ_ij w_ij z_i z_j / Σ_i z_i²`; `sq.gr.spatial_autocorr(mode="moran")` adds `I`, `pval_norm`, `var_norm` to `adata.uns['moranI']`
    2. **SpatialDE** (slower, GP-based) — Gaussian process regression decomposes expression into spatial + non-spatial variance; Anscombe transform → regress out library size → `SpatialDE.run(coords, resid_expr)`; output `g`, `l` (distance scale), `pval`, `qval`
  - Why: SVGs are an orthogonal feature selection to HVGs; a gene can be highly variable without being spatially patterned, and vice versa

- **Spatial deconvolution (Cell2location / Stereoscope)**
  - When to use: spot-based spatial data (Visium) where each spot contains multiple cells; need cell-type proportions per spot
  - How:
    1. Prepare scRNA-seq reference (cell-type annotated) + spatial data; share ENSEMBL IDs; remove MT genes
    2. Filter genes (`filter_genes` with permissive thresholds: `cell_count_cutoff=5`, `cell_percentage_cutoff2=0.03`, `nonz_mean_cutoff=1.12`)
    3. **Reference model** (`RegressionModel`) — NB regression estimates cell-type signatures; `setup_anndata` with `batch_key` + `categorical_covariate_keys`; train ~250 epochs
    4. **Spatial model** (`Cell2location`) — `setup_anndata` with `batch_key`; specify `N_cells_per_location` (from histology; ~8 for Visium) and `detection_alpha` (default 20); train ~30000 epochs
    5. Export posterior quantiles (`q05_cell_abundance_w_sf`) → per-spot cell-type abundances
    6. Downstream: KNN on cell abundance → Leiden → spatial regions; or cell-type-specific expression per gene
  - Why: spots contain mixed cell types; deconvolution with NB model + hierarchical priors outperforms simpler NMF-style methods (Li et al. benchmark)

- **Tangram imputation (cell → voxel mapping)**
  - When to use: targeted in-situ data (MERFISH, smFISH, seqFISH+; ~100-200 genes) + whole-transcriptome scRNA-seq reference; want whole-genome spatial expression
  - How:
    1. Compute marker genes from scRNA-seq; intersect with spatial gene panel → training genes (~200)
    2. `tg.pp_adatas` stores `training_genes` in both adatas
    3. `tg.map_cells_to_space(adata_sc, adata_st, mode="cells", density_prior="rna_count_based")` — learns mapping matrix `M` where `M_ij` = probability cell `i` belongs to voxel `j`; objective = `KL[m, d] - Σ_k cos(M^T S_:k, G_:k) - Σ_j cos(M^T S_j:, G_j:)`
    4. `tg.project_genes(ad_map, adata_sc)` → spatial × all-genes matrix
    5. Validate: `tg.plot_training_scores` (cosine similarity per gene); compare measured vs predicted on shared marker genes
  - Why: Tangram won Li et al. benchmark for imputation; probabilistic alignment with KL+cosine objective outperforms gimVI and SpaGE

## Key Concepts
- **Spot**: spatial capture area (Visium 55μm); contains multiple cells
- **Voxel**: generic spatial unit; for Visium = spot, for MERFISH = cell
- **Spatial domain**: cluster reflecting both gene expression and spatial proximity
- **Spatial autocorrelation (Moran's I)**: correlation of signal between nearby spatial units; high = spatially patterned
- **Spatially variable gene (SVG)**: gene with significant spatial pattern; orthogonal concept to HVG
- **Deconvolution**: infer cell-type proportions per spot from mixed expression
- **Imputation**: infer unmeasured gene expression in spatial data via reference mapping
- **intBC**: not applicable here (see Ch 4)
- **Histology integration**: using the H&E image alongside gene expression for domain identification (SpaGCN, stLearn)
- **Stereoscope**: NB model assuming cell-type expression is constant globally; uses success probability `p_g` shared across genes
- **Cell2location**: NB model with mean parametrization; cell-type abundance itself modeled as linear combination of tissue prototypes (hierarchical Gamma prior)
- **Density prior** (Tangram): `d_j` estimated cell density per voxel; uniform for MERFISH (cells themselves), rna_count_based for Visium

## Mental Models
- **Spatial is an orthogonal axis, not a replacement** — still need cell-type annotation, HVGs, normalization; spatial adds a new question layer
- **HVG ≠ SVG** — a housekeeping gene can be spatially variable if cell types are spatially segregated; an HVG can have no spatial pattern
- **Co-occurrence is distance-dependent** — `p(exp|cond)/p(exp)` varies with radius; close-range enrichment is meaningful, far-range is baseline
- **Deconvolution quality depends on reference** — mismatches between reference and spatial cell types corrupt estimates; include a "dummy" cell type for technical shifts (Stereoscope)
- **Imputation needs shared markers** — ~200 shared genes are sufficient; fewer gives unstable mappings
- **Validate Tangram on training genes** — if cosine similarity is low on shared genes, the imputed genes are unreliable
- **SpaGCN with histology > SpaGCN without** for tissues where morphology reflects cell type (tumor, brain layers)
- **Joint graph domains = spatially smoothed expression clusters** — alpha controls smoothing strength; 0.2 is a typical starting point

## Anti-patterns
- **Running deconvolution with mismatched gene IDs** — must use ENSEMBL IDs not gene symbols to avoid mapping ambiguity
- **Skipping MT-gene removal in Cell2location** — MT genes are technical artifacts, not biological abundance
- **Forgetting `var_names_make_unique` before SpatialDE** — silent duplicate gene handling
- **Setting `N_cells_per_location` blindly** — derive from histology image segmentation; wrong value biases all abundance estimates
- **Using only Moran's I for SVGs** — fast but less sophisticated than SpatialDE; use both for cross-validation
- **Running SpaGCN without checking `l` value** — `l` controls spatial decay; `search_l(p=0.5)` is the recommended default for Visium
- **Trusting Tangram imputation on genes with low training scores** — if a gene's measured vs predicted cosine similarity is low, the imputed values for that gene are unreliable
- **Skipping the refinement step in SpaGCN** — when domains have clear boundaries, refinement removes stray spots
- **Treating spatial CCC as confirmed** — neighborhood enrichment is hypothesis-generating; validate with ligand-receptor methods (Ch 6) + spatial co-localization

## Code Examples

### Squidpy neighborhood statistics (Visium mouse brain)

```python
import scanpy as sc
import squidpy as sq

adata = sq.datasets.visium_hne_adata()
sq.gr.spatial_neighbors(adata)

# Permutation-based enrichment test
sq.gr.nhood_enrichment(adata, cluster_key="cluster")
sq.pl.nhood_enrichment(adata, cluster_key="cluster", method="average", figsize=(5, 5))

# Descriptive interaction counts
sq.gr.interaction_matrix(adata, cluster_key="cluster")
sq.pl.interaction_matrix(adata, cluster_key="cluster", method="average", figsize=(5, 5))

# Distance-dependent co-occurrence
sq.gr.co_occurrence(adata, cluster_key="cluster")
sq.pl.co_occurrence(adata, cluster_key="cluster", clusters="Cortex_1", figsize=(8, 5))
```

- **What it demonstrates**: three spatial statistics on the same dataset. `nhood_enrichment` is the test (z-score); `interaction_matrix` is the count (no test); `co_occurrence` shows how the score changes with radius.

### Squidpy joint-graph spatial domains

```python
sc.pp.neighbors(adata)  # kNN on PCA
nn_graph_genes = adata.obsp["connectivities"]
sq.gr.spatial_neighbors(adata)  # spatial kNN
nn_graph_space = adata.obsp["spatial_connectivities"]

alpha = 0.2
joint_graph = (1 - alpha) * nn_graph_genes + alpha * nn_graph_space
sc.tl.leiden(adata, adjacency=joint_graph, key_added="squidpy_domains")
sq.pl.spatial_scatter(adata, color=["cluster", "squidpy_domains"], wspace=0.9)
```

### SpaGCN with histology

```python
import SpaGCN as spg

adj = spg.calculate_adj_matrix(
    x=x_pixel, y=y_pixel, x_pixel=x_pixel, y_pixel=y_pixel,
    image=img, beta=55, alpha=1, histology=True)

p = 0.5
l = spg.search_l(p, adj)
res = spg.search_res(adata, adj, l, target_num=15)

model = spg.SpaGCN()
model.set_l(l)
model.train(adata, adj, res=res)
y_pred, prob = model.predict()

# Refinement (majority vote of neighbors)
adj_2d = spg.calculate_adj_matrix(x=x_array, y=y_array, histology=False)
refined_pred = spg.refine(sample_id=adata.obs.index.tolist(),
                          pred=y_pred.tolist(), dis=adj_2d)
adata.obs["spaGCN_domains"] = refined_pred
```

- **What it demonstrates**: SpaGCN uses histology + spatial + expression. `beta=55` for Visium spot size; `alpha=1` means histology pixel intensity has same scale as coordinates. Refinement cleans up stray spots when domains have clear boundaries.

### Moran's I + SpatialDE for SVGs

```python
# Moran's I — fast
sq.gr.spatial_neighbors(adata)
sq.gr.spatial_autocorr(adata, mode="moran", genes=adata.var_names)
adata.uns["moranI"].head()  # I, pval_norm, var_norm, pval_norm_corr
sq.pl.spatial_scatter(adata, color=["Nrgn", "Ttr"])

# SpatialDE — slower, GP-based
import NaiveDE, SpatialDE
adata.var_names_make_unique()
counts = sc.get.obs_df(adata, keys=list(adata.var_names), use_raw=True)
total_counts = sc.get.obs_df(adata, keys=["total_counts"])
norm_expr = NaiveDE.stabilize(counts.T).T  # Anscombe transform
resid_expr = NaiveDE.regress_out(total_counts, norm_expr.T,
                                  "np.log(total_counts)").T
results = SpatialDE.run(adata.obsm["spatial"], resid_expr)
top10 = results.sort_values("qval").head(10)[["g", "l", "qval"]]
```

- **What it demonstrates**: two SVG methods. Moran's I is fast and integrated into Squidpy; SpatialDE decomposes spatial vs non-spatial variance and gives a distance scale `l` per gene.

### Cell2location deconvolution (Kuppe human heart)

```python
import cell2location as c2l

# Prep reference (snRNA-seq)
adata_sc.X = adata_sc.layers["counts"]
adata_st.var["feature_name"] = adata_st.var_names
adata_st.var.set_index("gene_ids", drop=True, inplace=True)
# remove MT genes
adata_st.var["MT_gene"] = [g.startswith("MT-") for g in adata_st.var["feature_name"]]
adata_st.obsm["MT"] = adata_st[:, adata_st.var["MT_gene"].values].X.toarray()
adata_st = adata_st[:, ~adata_st.var["MT_gene"].values]
shared_features = [f for f in adata_st.var_names if f in adata_sc.var_names]
adata_sc = adata_sc[:, shared_features].copy()
adata_st = adata_st[:, shared_features].copy()

# Permissive gene filter
selected = c2l.utils.filtering.filter_genes(
    adata_sc, cell_count_cutoff=5, cell_percentage_cutoff2=0.03, nonz_mean_cutoff=1.12)
adata_sc = adata_sc[:, selected].copy()
adata_st = adata_st[:, selected].copy()

# Reference model (NB regression for cell-type signatures)
c2l.models.RegressionModel.setup_anndata(
    adata=adata_sc, batch_key="donor_id",
    labels_key="cell_type_original",
    categorical_covariate_keys=["assay"], layer="counts")
model = c2l.models.RegressionModel(adata_sc)
model.train(max_epochs=250, batch_size=2500, train_size=1, lr=0.002, use_gpu=True)
model.export_posterior(adata_sc, sample_kwargs={"num_samples": 1000, "batch_size": 2500, "use_gpu": True})
inf_aver = adata_sc.varm["means_per_cluster_mu_fg"][
    [f"means_per_cluster_mu_fg_{i}" for i in adata_sc.uns["mod"]["factor_names"]]].copy()

# Spatial model (Cell2location)
c2l.models.Cell2location.setup_anndata(adata=adata_st, batch_key="patient")
model = c2l.models.Cell2location(adata_st, cell_state_df=inf_aver, N_cells_per_location=8)
model.train(max_epochs=30000, batch_size=None, train_size=1, use_gpu=True)
adata_st = model.export_posterior(adata_st, sample_kwargs={
    "num_samples": 1000, "batch_size": adata_st.n_obs, "use_gpu": True})
adata_st.obs[adata_st.uns["mod"]["factor_names"]] = adata_st.obsm["q05_cell_abundance_w_sf"]

# Downstream: spatial regions from cell abundance
sc.pp.neighbors(adata_st, use_rep="q05_cell_abundance_w_sf")
sc.tl.leiden(adata_st, resolution=0.5)
adata_st.obs["region_cluster"] = adata_st.obs["leiden"].astype("category")
```

- **What it demonstrates**: full Cell2location chain. `N_cells_per_location=8` derived from histology; `detection_alpha=20` default handles technical variability. Use 5% quantile (`q05_cell_abundance_w_sf`) for visualization — high-confidence lower bound on abundance.

### Tangram imputation (Lu MERFISH fetal liver)

```python
import tangram as tg

markers = list(set.intersection(set(adata_sc.var_names), set(adata_st.var_names)))
tg.pp_adatas(adata_sc, adata_st, genes=markers)

ad_map = tg.map_cells_to_space(
    adata_sc, adata_st, mode="cells",
    density_prior="rna_count_based", num_epochs=500, device="cuda")

# Project cell annotations + impute genes
tg.project_cell_annotations(ad_map, adata_st, annotation="leiden")
ad_ge = tg.project_genes(adata_map=ad_map, adata_sc=adata_sc)

# Validate on training genes
tg.plot_training_scores(ad_map, bins=20, alpha=0.5)
tg.plot_genes_sc(
    genes=["tek", "stab2", "hc"],
    adata_measured=adata_st[adata_st.obs.FOV == 0],
    adata_predicted=ad_ge[ad_ge.obs.FOV == 0], perc=0.02, spot_size=50)
```

- **What it demonstrates**: Tangram's `mode="cells"` maps individual cells to voxels (slow, GPU recommended); `mode="clusters"` is faster but loses single-cell resolution. Validate by comparing measured vs predicted expression on shared marker genes.

## Reference Tables

### Spatial method category × task

| Task | Method (recommended first) | Alternatives |
|---|---|---|
| Neighborhood statistics | Squidpy `nhood_enrichment` | `interaction_matrix`, `co_occurrence` |
| Spatial domains (no histology) | Squidpy joint graph | HMRF, BayesSpace |
| Spatial domains (with histology) | SpaGCN | stLearn |
| SVGs (fast) | Moran's I (Squidpy) | SPARK-X |
| SVGs (model-based) | SpatialDE | Sepal, scGCO |
| Deconvolution (best benchmark) | Cell2location | Stereoscope, RCTD, DestVI, SPOTlight |
| Imputation | Tangram | gimVI, SpaGE |

### Cell2location key hyperparameters

| Hyperparameter | Meaning | Typical |
|---|---|---|
| `N_cells_per_location` | Expected cell abundance per spot | ~8 for Visium; from histology |
| `detection_alpha` | Technical variability in RNA detection | 20 (default) |
| `max_epochs` (reference) | NB regression training | 250 |
| `max_epochs` (spatial) | Cell2location training | 30000 |
| `batch_size` (reference) | Mini-batch size | 2500 |
| `train_size` | Train/test split | 1 (full training) |

### Deconvolution method comparison

| Method | Distribution | Cell-type profile assumption | Strengths |
|---|---|---|---|
| Stereoscope | NB (success prob) | Constant per cell type globally | Fast; dummy cell type for tech shifts |
| Cell2location | NB (mean) | NB regression with covariates | Hierarchical priors; tissue prototypes |
| RCTD | Poisson | Mean per cell type | Robust on doublets |
| DestVI | NB | Neural network (latent variable) | Captures intra-cell-type variation |
| SPOTlight | NMF | Topics matrix | Non-probabilistic; topic interpretation |

## Worked Example

**Visium mouse brain (V1_Adult_Mouse_Brain, 10x Genomics)** — used for neighborhood, domains, and SVG tutorials. Pipeline: load with `sq.datasets.visium_hne_adata()`; compute spatial neighbors; `nhood_enrichment` reveals `Pyramidal_layer` + `Dentate_gyrus` enrichment; joint-graph domains (α=0.2) smooth cluster annotations spatially; SpaGCN with histology (β=55, α=1, p=0.5, target=15 clusters) produces coherent domains; refinement step cleans stray spots. Moran's I identifies `Nrgn` and `Ttr` as SVGs (corrected p=0). SpatialDE confirms with GP-based p-values; top genes include `Esrra` (cortex layers + thalamus), `Fbxo31` and `Jph3` (pyramidal layer).

**Kuppe human heart (Visium + snRNA-seq, 4 control patients)** — Cell2location deconvolution. Reference: 41,663 nuclei across 11 cell types (adipocytes, cardiomyocyte, cycling cells, endothelial, fibroblast, lymphoid, mast, myeloid, neuronal, pericyte, vSMCs). Filter genes (`cell_count_cutoff=5`, `cell_percentage_cutoff2=0.03`, `nonz_mean_cutoff=1.12`); train reference NB regression (250 epochs); train Cell2location (30000 epochs, GPU). Cardiomyocytes dominate most spots; mast and endothelial enriched in specific regions. Downstream: Leiden on `q05_cell_abundance_w_sf` identifies discrete tissue regions visible across slides.

**Lu MERFISH fetal liver (140 images, 4 livers, 132 genes × 40,864 cells)** — Tangram imputation. Reference: 10x scRNA-seq of E14.5 fetal liver. Intersect gene panels → training genes (~132). `tg.map_cells_to_space(mode="cells", density_prior="rna_count_based", num_epochs=500)` learns mapping. `tg.project_genes` produces full-transcriptome spatial data. Validate: compare measured vs predicted expression on shared markers (`tek`, `stab2`, `hc`) — cosine similarity high on training genes confirms reliable mapping. Inspect imputed genes (`rp1`, `sox17`, `mrpl15`) not measured by MERFISH. Cross-validate cell-type expression levels of control markers (`tfrc`, `cd34`, `kdr`) match between measured and predicted.

## Key Takeaways
1. **Pick analysis by spatial resolution tier** — multi-cell → deconvolve; single-cell → annotate + neighborhood; sub-cellular → segment then proceed
2. **Squidpy is the scanpy equivalent for spatial** — start with `sq.gr.spatial_neighbors` then `nhood_enrichment` / `interaction_matrix` / `co_occurrence`
3. **Spatial domains = joint graph Leiden** — combine expression + spatial graphs with α weighting (~0.2 default); use SpaGCN when histology is informative
4. **SVG ≠ HVG** — Moran's I (fast) and SpatialDE (model-based) identify spatially patterned genes; use both to cross-validate
5. **Cell2location is the deconvolution default** — best benchmark accuracy; needs GPU + scRNA-seq reference; check `N_cells_per_location` from histology
6. **Tangram imputes targeted in-situ data** — needs ~200 shared training genes; validate on training scores before trusting imputed genes
7. **Always validate imputation/deconvolution on shared genes** — low training scores = unreliable results
8. **Cell-type abundance → downstream spatial regions** — Leiden on `q05_cell_abundance_w_sf` identifies tissue regions with similar composition
9. **Spatial CCC is hypothesis-generating** — neighborhood enrichment identifies candidates; combine with ligand-receptor methods (Ch 6) for functional validation

## Connects To
- **Ch 1 (Introduction)**: AnnData stores spatial coords in `.obsm["spatial"]`; histology image in `.uns["spatial"]`
- **Ch 2 (Preprocessing)**: spatial QC includes filtering empty spots; same normalization pipeline applies
- **Ch 3 (Annotation)**: cell-type labels feed into all neighborhood/domain analyses
- **Ch 6 (Cell-cell communication)**: spatial co-localization validates ligand-receptor hypotheses; CCC + spatial is the gold standard for inference
- **Ch 7 (Bulk deconvolution)**: same `y = bX` framework; spatial deconvolution adds NB noise model + per-spot output
- **Ch 12 (Multimodal integration)**: spatial + scRNA-seq integration via Tangram/Cell2location is a multimodal task
