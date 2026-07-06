# Patterns — Techniques and Workflows

## QC Patterns

### MAD-based cell filtering (per-sample)
**When to use**: scRNA-seq, scATAC-seq, CITE-seq QC
**How**: For each sample independently, compute `is_outlier(metric, nmads)` on `log1p_total_counts`, `log1p_n_genes_by_counts`, `pct_counts_mt`, `pct_counts_ribo`. Use 5 MADs for counts/genes, 3 MADs + 8% hard cap for mitochondrial. Concatenate per-sample results.
**Trade-offs**: Per-sample is robust to donor-specific distributions but slower than global cutoffs. MAD is robust to outliers but assumes unimodal distribution.

### Per-modality QC specialization
**When to use**: any multimodal dataset
**How**: RNA → filter cells on mt%, gene count, total counts; ATAC → filter on TSS enrichment, nucleosome signal, FRiP; ADT → filter on `n_genes_by_counts` (valley ~55), `total_counts` upper bound (100k).
**Trade-offs**: Different modalities have different quality signals; reusing RNA thresholds on ADT/ATAC is a common anti-pattern.

### Orthogonal doublet detection
**When to use**: after QC, before clustering
**How**: RNA → scDblFinder (simulated doublets + PCA + classification); ATAC → AMULET (read-pair distance) + scDblFinder; ADT → mutually-exclusive marker pairs (CD3+CD19+, CD3+CD14+).
**Trade-offs**: Each method catches different doublet types; orthogonal methods complement each other but may disagree on borderline cases.

## Normalization Patterns

### Three-layer RNA normalization
**When to use**: scRNA-seq, store alternatives for downstream flexibility
**How**: `adata.layers["counts"]` = raw counts (preserved); `adata.X` = log1p(normalize_total(target_sum=1e4)) (default analysis); `adata.layers["scran"]` = size_factors from scran. Shifted log for visualization, scran for DE, Pearson residuals for HVG selection.
**Trade-offs**: Different methods suit different tasks; storage cost is 3× the count matrix but enables per-task selection.

### dsb vs CLR for ADT
**When to use**: dsb if empty droplets + isotype controls available; CLR otherwise
**How**: dsb (`mu.prot.pp.dsb(mdata, mdata_raw, isotype_controls=[...])`) — empty droplets estimate ambient noise (subtracted); isotype controls estimate non-specific binding (corrected); output is denoised + scaled. CLR (`mu.prot.pp.clr(mdata["prot"])`) — count / geometric_mean(cell) → log; depth-only.
**Trade-offs**: dsb strictly better when input data exists; CLR is fallback for pre-collected data without raw empty droplets.

### TF-IDF + LSI for scATAC
**When to use**: scATAC-seq dimensionality reduction
**How**: TF-IDF normalize peak×cell matrix → SVD (LSI) → top 2:30 components used as "PCA". LSI components 2:30 (skip LSI-1 which correlates with sequencing depth).
**Trade-offs**: Log-normalization + PCA produces too dense clusters; LSI separates biological from technical variation better but component 1 is depth-confounded.

## Clustering & Annotation Patterns

### Leiden resolution selection
**When to use**: cell-type annotation
**How**: Start at resolution=0.1 (coarse); inspect clusters via marker dotplot (CD3=T, CD19=B, CD14=Mono, CD56=NK); raise to 0.4-0.8 for sub-types (CD4 vs CD8, naive vs memory); sub-cluster specific clusters if they show mixed markers.
**Trade-offs**: Lower resolution = stable cell types but merges sub-types; higher resolution = sub-types but risk over-clustering noise.

### Marker-based manual annotation
**When to use**: gold-standard annotation; CellTypist fails or no reference exists
**How**: `sc.tl.rank_genes_groups(adata, groupby="leiden", method="wilcoxon")` → `sc.pl.rank_genes_groups_dotplot(n_genes=5, values_to_plot="logfoldchanges")` → assign clusters by canonical markers.
**Trade-offs**: Most reliable but labor-intensive and subjective; requires domain knowledge of markers.

### CellTypist automated annotation
**When to use**: when reference exists (immune cells, brain, etc.)
**How**: `CellTypist.annotate(adata, model="Immune_All_Low.pkl", majority_voting=True)`; returns `predicted_labels` + `conf_score` + `majority_voting`.
**Trade-offs**: Fast, reproducible; fails on rare/unseen cell types; always check confidence scores.

### scArches label transfer
**When to use**: query dataset has reference scVI/scANVI model
**How**: Train scVI/scANVI on reference → save → `sca.models.SCVI.load_query_data(query, ref)` → `train(max_epochs=200)` → KNN classifier on latent → uncertainty via `get_latent_representation` + KNN distance.
**Trade-offs**: Best for cross-study integration; needs reference model; uncertainty quantification distinguishes novel vs known cell types.

## Integration Patterns

### scVI/scANVI for batch correction
**When to use**: multi-batch scRNA-seq with raw counts
**How**: `scvi.model.SCVI.setup_anndata(adata, layer="counts", batch_key="donor")` → `model = scvi.model.SCVI(adata)` → `model.train()` → `adata.obsm["X_scVI"] = model.get_latent_representation()`.
**Trade-offs**: Conditional VAE handles raw counts with ZINB; needs GPU for large data; batch must be discrete (not continuous).

### Harmony for fast batch correction
**When to use**: PCA embedding correction; RNA + ADT
**How**: `sc.external.pp.harmony_integrate(adata, key="donor")` → re-compute neighbors on `X_pca_harmony`.
**Trade-offs**: Faster than scVI; works on existing PCA; assumes linear batch effects.

### BBKNN for graph-based integration
**When to use**: per-batch kNN construction
**How**: `sc.external.pp.bbknn(adata, batch_key="donor")` → directly builds kNN without modifying PCA.
**Trade-offs**: Simple, fast; less effective than scVI on strong batch effects; only modifies graph.

## Differential Analysis Patterns

### Pseudobulk DGE with PyDESeq2
**When to use**: condition comparison within cell types
**How**: `decoupler.pp.pseudobulk(adata, sample_col="sample", groups_col="cell_type", layer="counts")` → `filter_by_expr` + `filter_by_prop` → `PyDESeq2(dds)` → `test_contrasts([("cell_type", "stim", "ctrl")])` → volcano plot.
**Trade-offs**: Sample-level view avoids pseudoreplication; needs ≥3 biological replicates per condition; loses single-cell resolution.

### Complex designs with interactions
**When to use**: cell_type * condition; or condition * time
**How**: `design = "~ cell_type + condition + cell_type:condition"` → `dds.design = design` → `test_contrasts([("cell_type", "CD14 Mono", "stim", "ctrl")])` → contrast isolates cell-type-specific response.
**Trade-offs**: Detects cell-type-specific responses; needs more samples for power; harder to interpret main effects.

### scCODA compositional analysis
**When to use**: cell-type proportion changes across conditions
**How**: Choose reference cell type (smallest log-fold change) → `sccoda_model.load(adata, feature_key="cell_type", sample_key="sample")` → `sccoda_model.run_nuts(...)` → credible effects if `Final Parameter` ≠ 0.
**Trade-offs**: Correctly handles compositional constraint; requires reference cell type (assumed unchanged); only detects log-fold changes, not absolute abundance.

## Trajectory Patterns

### Pseudotime construction principles
**When to use**: developmental/transition processes
**How**: (1) select root cell via marker (hematopoietic stem cell marker); (2) branch point at intermediate cluster (multi-potent progenitors); (3) terminal states at leaves (differentiated cells); (4) bias toward known biology (HSC → CMP → MegaE rather than reverse).
**Trade-offs**: Pseudotime is hypothesis, not ground truth; root cell choice biases entire trajectory.

### Palantir for branching trajectories
**When to use**: known terminal states, dense progenitor population
**How**: Mark terminal states → Palantir runs diffusion pseudotime + MSC markov chain → per-cell probability of each terminal fate → branches are auto-detected.
**Trade-offs**: Best for multi-fate trajectories; requires explicit terminal state annotation; fails on cyclic processes.

### RNA velocity with EM (scVelo dynamical)
**When to use**: steady-state assumption violated (developmental systems)
**How**: `scv.pp.moments(adata)` → `scv.tl.recover_dynamics(adata)` → `scv.tl.velocity(adata, mode="dynamical")` → `scv.tl.velocity_graph(adata)` → `scv.pl.velocity_embedding_stream()`.
**Trade-offs**: More accurate than steady-state (La Manno 2018) but slow; needs spliced/unspliced counts from alevin-fry or velocyr; fails on sparse data.

## Mechanism Patterns

### SCENIC 3-step GRN
**When to use**: TF-target hypotheses from scRNA-seq
**How**: (1) `pyscenic grn` (GRNBoost) — co-expression candidates; (2) `pyscenic ctx` (cisTarget) — motif enrichment prunes non-cis-regulated; (3) `pyscenic aucell` — per-cell regulon activity. Run per-donor (avoid batch confounding).
**Trade-offs**: Co-expression permissive → motif strict → AUCell denoising; outputs are hypotheses for orthogonal validation (ChIP-seq, perturbation).

### LIANA consensus + NicheNet
**When to use**: cell-cell communication inference
**How**: `li.mt.rank_aggregate(adata, groupby="cell_type")` for ligand-receptor plausibility → pseudobulk DE in receiver per sample+cell type → `predict_ligand_activities(geneset_oi, ...)` for ligand activity (AUPR) → intersect top candidates from both.
**Trade-offs**: Ligand-receptor = plausibility (CellPhoneDB, NATMI, Connectome — low agreement); NicheNet = functionality (needs condition contrast). Run both, intersect for high-confidence CCC.

## Spatial Patterns

### Three resolution tiers
**When to use**: choosing spatial analysis strategy
**How**: (1) Subcellular (Xenium, CosMx, MERFISH) — molecule coordinates; (2) Single-cell (Visium HD, Stereo-seq) — single cells in tissue; (3) Spot-based (Visium, Slide-seq) — multi-cell spots needing deconvolution.
**Trade-offs**: Higher resolution = lower throughput; spot-based needs cell2location or RCTD for cell-type deconvolution.

### Moran's I + SpatialDE for SVGs
**When to use**: identifying spatially-variable genes
**How**: `sq.gr.spatial_neighbors(adata, coord_type="generic")` → `sq.gr.spatial_autocorr(adata, mode="moran", genes=adata.var_names, n_jobs=4)` → filter Moran's I > 0.3 + FDR < 0.05. SpatialDE for GP-based SVG detection (handles covariates).
**Trade-offs**: Moran's I is fast but ignores covariates; SpatialDE is more powerful but slow on large datasets.

### Cell2location deconvolution
**When to use**: spot-based spatial with mixed cell types
**How**: Train on scRNA reference (`cell2location.models.RegressionModel`) → estimate cell-type signatures → apply to spatial (`cell2location.models.Cell2location`) → `adata.obsm["q05_cell_abundance_w_sf"]` = per-cell-type abundance per spot.
**Trade-offs**: Best-in-class for spot deconvolution; needs matched scRNA reference; computationally expensive.

## Multimodal Integration Patterns

### WNN (Seurat v4)
**When to use**: paired RNA+ADT, want joint clusters
**How**: Per-modality PCA → per-cell modality weights via within-modality kNN + cross-modality prediction → joint WNN graph → UMAP/clustering on WNN.
**Trade-offs**: Captures cell-type-specific modality importance (e.g. ADT dominates for T cell subtypes); requires Seurat/R; no explicit batch correction.

### totalVI for RNA+ADT
**When to use**: paired CITE-seq, end-to-end integration
**How**: `scvi.model.TOTALVI.setup_anndata(mdata, protein_layer=None, batch_key="donor")` → `model = scvi.model.TOTALVI(mdata)` → `model.train()` → `mdata.obsm["X_totalVI"] = model.get_latent_representation()` → denoised RNA + ADT via `model.get_normalized_expression()`.
**Trade-offs**: End-to-end VAE handles RNA (ZINB) + ADT (NB) jointly; slower than WNN; needs GPU.

### GLUE for RNA+ATAC
**When to use**: paired unpaired RNA+ATAC integration (no shared cells)
**How**: Train RNA VAE + ATAC VAE + graph VAE (using prior GRN as edges) → triple loss aligns embeddings → `mdata.obsm["X_glue"]` joint latent.
**Trade-offs**: Works without paired cells (using prior GRN); requires GRN prior; novel but benchmarked well.

### Multigrate for trimodal
**When to use**: DOGMA-seq or TEA-seq (RNA+ADT+ATAC paired)
**How**: `multigrate.models.Multigrate.setup_anndata(mdata, rna_layer="counts", atac_layer="counts", prot_layer="counts")` → train → joint latent for trimodal integration.
**Trade-offs**: Extends totalVI to ATAC; experimental but handles trimodal; needs paired data.

## Perturbation Patterns

### Augur cell-type prioritization
**When to use**: which cell types respond most to perturbation
**How**: `augur.run(adata, label_col="cell_type", condition_col="condition")` — train classifier (LR or RF) on each cell type's expression to predict condition; AUC = prioritization score (1 = perfectly separable, 0.5 = no response).
**Trade-offs**: Robust to cell-type-specific baseline; doesn't identify responding genes; compare AUC delta across conditions.

### scGen perturbation prediction
**When to use**: predict perturbation response in unseen cell types
**How**: Train VAE on control + perturbed → vector arithmetic (perturbation = mean(perturbed) - mean(control) in latent) → apply vector to unseen cell type latent → decode to predict response.
**Trade-offs**: Assumes linear perturbation effect; works for similar cell types; fails for drastically different biology.
