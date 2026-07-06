# Chapter 3: Cellular Structure — Clustering, Annotation & Integration

## Core Idea
Three sequential layers turn a normalized expression matrix into a labeled atlas: (1) **clustering** groups cells by transcriptomic similarity on a kNN graph, (2) **annotation** assigns biological identity to each cluster using marker genes (manual), DGE (rank_genes_groups), automated models (CellTypist), or reference-based label transfer (scArches), and (3) **integration** removes batch effects so that cells cluster by biology rather than technical batch. Each layer depends on the previous one — clustering needs good preprocessing, annotation needs clustering (or a reference), and integration is required before clustering when batch effects dominate.

## Frameworks Introduced

- **Leiden clustering on a kNN graph**
  - When to use: any unsupervised cell discovery after PCA + neighbors
  - How: build kNN graph in PCA space → `sc.tl.leiden(resolution=...)` → community detection optimizes modularity; resolution controls granularity (lower = fewer larger clusters, higher = more smaller clusters)
  - Why: Leiden generalizes Louvain with guaranteed well-connected communities; resolution is the single biggest knob for cluster granularity

- **Sub-clustering for hidden heterogeneity**
  - When to use: known cell type contains sub-structure (e.g., T cells → CD4/CD8/cytotoxic)
  - How: subset to cluster → re-run PCA → re-compute neighbors → re-Leiden at higher resolution → assign sub-labels → merge back into main `obs`
  - Why: global clustering resolution is a compromise; rare subtypes need focal re-analysis

- **Manual marker-based annotation**
  - When to use: well-studied system with known markers (immune, brain, etc.)
  - How: build `marker_genes` dict mapping cell type → list of markers; intersect with `adata.var_names` (handles naming mismatches); `sc.pl.dotplot`/`matrixplot` of marker expression per cluster; assign cluster → cell type via dotplot patterns
  - Why: most robust; gold standard for novel cell type discovery; marker dictionaries are reusable assets

- **DGE-based cluster annotation (rank_genes_groups)**
  - When to use: clusters lack known markers; want data-driven markers per cluster
  - How: `sc.tl.rank_genes_groups(adata, groupby="leiden", method="wilcoxon")` → `sc.pl.rank_genes_groups_dotplot(n_genes=5)` → look up top genes in literature / CellMarker / Panglao
  - Why: unbiased; complements manual annotation; t-test overestimates significance on sparse single-cell — Wilcoxon is the safe default

- **Automated annotation with CellTypist**
  - When to use: align to a published reference; high-throughput annotation
  - How: `celltypist.annotate(adata, model="Immune_All_Low.pkl")` → returns `predicted_labels` + `conf_score` + `majority_voting` (over-clusters to smooth noise)
  - Why: pre-trained models encode reference annotations; fast and reproducible; majority voting corrects spurious single-cell predictions

- **Reference-based label transfer with scArches / scVI**
  - When to use: query dataset has no labels; reference has labels + trained scVI/scANVI model
  - How:
    1. Download trained `model.pt` from scvi-tools hub
    2. `sca.models.SCVI.load_query_data(adata_query, model_dir)` → `model.train(max_epochs=200)` → `get_latent_representation()`
    3. Concatenate ref+query embedding → neighbors → UMAP → check reference/query mixing
    4. KNN-based label transfer: `sca.utils.knn.weighted_knn_transfer(query, ref, n_neighbors=15)` → `labels` + `uncert` (per-cell uncertainty)
    5. Flag high-uncertainty cells (>0.2) as "Unknown" for manual review
  - Why: leverages large published atlases (e.g., lung, brain); uncertainty score flags novel cell states that the reference doesn't cover

- **Data integration: 4 method categories**
  - When to use: batch effects obscure biological signal (visible as batch-driven UMAP separation)
  - How (categories):
    1. **Global models** (ComBat) — additive/multiplicative effect across all cells; from bulk transcriptomics
    2. **Linear embedding models** (MNN, Seurat, Scanorama, FastMNN, Harmony) — SVD-based embedding, locally adaptive correction via mutual nearest neighbors
    3. **Graph-based methods** (BBKNN) — force inter-batch edges in kNN graph; fastest
    4. **Deep learning** (scVI, scANVI, scGen) — conditional VAE; conditions on batch covariate; scales best to large datasets
  - Why: method choice depends on (a) batch complexity, (b) labels available?, (c) required output (embedding vs corrected expression)

- **scVI / scANVI integration**
  - When to use: complex multi-batch; need corrected embedding for downstream
  - How:
    1. `scvi.model.SCVI.setup_anndata(adata, layer="counts", batch_key="batch")` — must use raw counts, not normalized
    2. `model_scvi = scvi.model.SCVI(adata); model_scvi.train()` — epochs scale with cells (~400 for <20k, fewer for larger)
    3. `adata.obsm["X_scVI"] = model_scvi.get_latent_representation()` → neighbors → UMAP on `X_scVI`
    4. **scANVI** = scVI + labels: `scvi.model.SCANVI.from_scvi_model(model_scvi, labels_key="cell_type", unlabeled_category="unlabelled")`; refine scVI with supervised signal; preserves more biology
  - Why: top-performing in benchmarks; models count distribution properly (ZINB); scANVI > scVI for biological conservation when labels exist

- **BBKNN graph-based integration**
  - When to use: fast first-pass; strong batch effects; only need graph (not corrected expression)
  - How: `bbknn.bbknn(adata, batch_key="batch", neighbors_within_batch=3)` — 25 if >100k cells else 3
  - Why: replaces `sc.pp.neighbors` directly; output is graph (no embedding); limited downstream tools accept it

- **Seurat integration via MNN anchors (R)**
  - When to use: need corrected expression matrix (not just embedding); downstream tools require gene-space
  - How (via rpy2): `SplitObject(seurat, split.by=batch)` → `FindIntegrationAnchors(batch_list)` → `IntegrateData(anchors)` → extract `integrated` assay → transfer back to Python
  - Why: produces batch-corrected expression; pairwise MNN anchors work well but scales poorly with many batches

- **scIB benchmarking metrics**
  - When to use: comparing integration methods on a labeled dataset
  - How: `scib.metrics.metrics_fast(adata_unint, adata_int, batch_key, label_key, embed="X_scVI")` → per-method scores; group into Batch (ASW_label/batch, PCR_batch, graph_conn) and Bio (ASW_label, isolated_label_silhouette) categories; scale per-metric (0=worst, 1=best); overall = 0.4·Batch + 0.6·Bio
  - Why: UMAP inspection is misleading; quantitative comparison is mandatory; no single metric tells the whole story

## Key Concepts
- **Leiden algorithm**: generalization of Louvain with guaranteed well-connected communities and faster convergence
- **Resolution parameter (Leiden)**: controls cluster granularity — lower = coarser, higher = finer
- **kNN graph**: cells connected to k most similar cells (in PCA space); basis for Leiden and UMAP
- **Marker gene**: gene with high expression specific to a cell type (e.g., CD3 for T cells)
- **DGE (Differential Gene Expression)**: per-cluster test comparing one cluster vs rest; Wilcoxon is default
- **CellTypist**: logistic regression models pre-trained on reference atlases; provides predicted label + confidence + majority voting
- **scArches**: scvi-tools framework for reference-based query mapping; preserves reference model weights
- **Batch effect**: technical/biological variation correlated with sample handling rather than biology
- **Batch covariate choice**: sample = strongest correction; donor = preserves between-individual variation; dataset = preserves between-cohort variation
- **Mutual nearest neighbors (MNN)**: pairs of cells from two batches that are each in the other's neighborhood; basis for anchor-based correction
- **scVI**: conditional VAE that models raw counts with ZINB distribution; conditions on batch
- **scANVI**: scVI + semi-supervised label information; preserves biological variation better
- **scIB metric**: batch removal vs biological conservation — 14 metrics across both categories
- **kBET**: k-Nearest Neighbor Batch Effect Test — first quantitative batch correction metric
- **ASW (Average Silhouette Width)**: cluster cohesion vs separation; ASW_label for biology, ASW_label/batch for batch mixing
- **PCR_batch**: Principal Component Regression batch metric — variance explained by batch in PCs
- **graph_conn**: graph connectivity — whether batches are connected in kNN graph

## Mental Models
- **Cluster first, annotate second** — clustering is unsupervised; annotation maps clusters to known biology
- **Resolution is a knob, not a truth-claim** — try multiple resolutions; sub-cluster when needed
- **Manual annotation is gold standard** — CellTypist/scArches predictions need marker-gene validation
- **Uncertainty is informative** — high scArches uncertainty flags novel cell states, not just failure
- **Batch covariate choice is design-driven** — sample vs donor vs dataset preserves/removes different biology
- **Integration ≠ batch correction** — batch correction = simple same-protocol; integration = complex cross-protocol
- **Labels help scANVI** — semi-supervised preserves more biology than unsupervised scVI when labels exist
- **UMAP is suggestive, scIB is decisive** — don't pick a method by eye; run metrics
- **Bio vs Batch is a tradeoff** — most methods sacrifice some of each; pick the tradeoff that fits your question

## Anti-patterns
- **Clustering on raw counts or normalized expression** — always cluster on PCA (or other embedding) of HVGs
- **Single resolution clustering** — explore 0.2–1.5; rare populations need sub-clustering
- **Skipping marker gene QC** — `marker_genes` dict often has wrong names (CD8A vs CD8); intersect with `var_names` first
- **Trusting t-test for DGE** — t-test inflates significance on sparse single-cell; use Wilcoxon
- **Annotating without dotplots** — labels without marker validation are untrustworthy
- **Treating CellTypist predictions as ground truth** — always cross-validate with markers; majority voting corrects noise
- **Using raw counts for scVI** — scVI requires raw counts in `layer="counts"` (it models ZINB)
- **Picking integration by UMAP** — UMAP mixing looks good even when biological signal is removed; use scIB metrics
- **Forgetting batch-aware HVG selection** — `batch_key` in `sc.pp.highly_variable_genes` prevents selecting batch-marker genes
- **Selecting too few HVGs** — 1000–10000 recommended; too few risks losing rare-cell markers
- **Confounding batch with biology** — sample-as-batch removes donor variation; if you want donor variation, use donor-as-batch
- **Skipping label harmonization** — scANVI assumes consistent labels across datasets; harmonize first

## Code Examples

### Leiden clustering + sub-clustering

```python
sc.tl.leiden(adata, resolution=0.5, flavor="igraph", n_iterations=2,
             directed=False, random_state=0)
sc.pl.umap(adata, color="leiden", legend_loc="on data")

# Sub-cluster a specific cluster (e.g., T cells = "3")
adata_sub = adata[adata.obs.leiden == "3"].copy()
sc.pp.pca(adata_sub)
sc.pp.neighbors(adata_sub)
sc.tl.leiden(adata_sub, resolution=0.8, key_added="leiden_sub")
adata.obs["leiden_sub"] = "NA"
adata.obs.loc[adata_sub.obs_names, "leiden_sub"] = adata_sub.obs["leiden_sub"]
```

### Manual annotation via marker dotplot

```python
marker_genes = {
    "CD14+ Mono": ["FCN1", "CD14"],
    "CD16+ Mono": ["FCGR3A", "MS4A7"],
    "T cell": ["CD3D", "CD3E"],
    "B cell": ["CD79A", "MS4A1"],
    "NK": ["GNLY", "NKG7"],
    "DC": ["CST3", "FCER1A"],
    "pDC": ["LILRA4", "IL3RA"],
}

# Intersect with var_names (handles case/naming mismatches)
marker_genes_in_data = {}
for ct, markers in marker_genes.items():
    found = [m for m in markers if m in adata.var_names]
    if found:
        marker_genes_in_data[ct] = found

sc.pl.dotplot(adata, var_names=marker_genes_in_data, groupby="leiden",
              standard_scale="var")
```

### DGE-based annotation

```python
sc.tl.rank_genes_groups(adata, groupby="leiden", method="wilcoxon")
sc.pl.rank_genes_groups_dotplot(adata, n_genes=5, standard_scale="var")

# Look up top markers for cluster "5"
top_markers = sc.get.rank_genes_groups_df(adata, group="5").head(10)
```

### CellTypist automated annotation

```python
import celltypist
model = celltypist.models.get("Immune_All_Low.pkl")
predictions = celltypist.annotate(adata, model=model, majority_voting=True)
adata.obs["celltypist_cell_type"] = predictions.predicted_labels["majority_voting"]
adata.obs["celltypist_conf_score"] = predictions.conf_score
```

### scArches query-to-reference label transfer

```python
import scarches as sca

# Load trained reference model
ref_model_dir = "/path/to/reference_scvi_model"
model = sca.models.SCVI.load_query_data(adata_query, ref_model_dir)
model.train(max_epochs=200)

adata_query.obsm["X_scANVI"] = model.get_latent_representation()

# Concatenate ref + query embedding
import anndata as ad
emb_ref_query = ad.concat([ref_emb, adata_query], axis=0,
                          label="reference_or_query",
                          keys=["reference", "query"])
sc.pp.neighbors(emb_ref_query)
sc.tl.umap(emb_ref_query)

# KNN-based label transfer with uncertainty
knn_transformer = sca.utils.knn.weighted_knn_trainer(
    train_adata=ref_emb, train_adata_emb="X", n_neighbors=15)
labels, uncert = sca.utils.knn.weighted_knn_transfer(
    query_adata=adata_query, query_adata_emb="X",
    label_keys="cell_type", knn_model=knn_transformer,
    ref_adata_obs=ref_emb.obs)

adata_query.obs["transf_cell_type"] = labels.loc[adata_query.obs_names, "cell_type"]
adata_query.obs["transf_cell_type_unc"] = uncert.loc[adata_query.obs_names, "cell_type"]

# Flag high-uncertainty cells
adata_query.obs.loc[
    adata_query.obs.transf_cell_type_unc > 0.2, "transf_cell_type"
] = "Unknown"
```

### scVI integration

```python
import scvi

sc.pp.highly_variable_genes(adata, n_top_genes=2000, flavor="cell_ranger",
                           batch_key="batch")
adata_hvg = adata[:, adata.var.highly_variable].copy()

scvi.model.SCVI.setup_anndata(adata_hvg, layer="counts", batch_key="batch")
model_scvi = scvi.model.SCVI(adata_hvg)
model_scvi.train()  # ~400 epochs for <20k cells; fewer for larger

adata_hvg.obsm["X_scVI"] = model_scvi.get_latent_representation()
sc.pp.neighbors(adata_hvg, use_rep="X_scVI")
sc.tl.umap(adata_hvg)
```

### scANVI (semi-supervised)

```python
model_scanvi = scvi.model.SCANVI.from_scvi_model(
    model_scvi, labels_key="cell_type", unlabeled_category="unlabelled")
model_scanvi.train(max_epochs=10)  # ~1/3 of scVI epochs

adata_hvg.obsm["X_scANVI"] = model_scanvi.get_latent_representation()
sc.pp.neighbors(adata_hvg, use_rep="X_scANVI")
sc.tl.umap(adata_hvg)
```

### BBKNN graph-based integration

```python
import bbknn
adata_bbknn = adata_hvg.copy()
adata_bbknn.X = adata_bbknn.layers["logcounts"].copy()
sc.pp.pca(adata_bbknn)
bbknn.bbknn(adata_bbknn, batch_key="batch",
            neighbors_within_batch=25 if adata_bbknn.n_obs > 100000 else 3)
sc.tl.umap(adata_bbknn)
```

### Seurat integration via rpy2

```python
import anndata2ri
%load_ext rpy2.ipython
anndata2ri.set_ipython_converter()

adata_seurat = adata_hvg.copy()
adata_seurat.obs["batch"] = adata_seurat.obs["batch"].astype(str)
adata_seurat.obs["cell_type"] = adata_seurat.obs["cell_type"].astype(str)
del adata_seurat.uns  # avoid conversion issues
```

```R
%%R -i adata_seurat -i batch_key
seurat <- as.Seurat(adata_seurat, counts="counts", data="logcounts")
batch_list <- SplitObject(seurat, split.by=batch_key)
anchors <- FindIntegrationAnchors(batch_list, anchor.features=rownames(seurat))
integrated <- IntegrateData(anchors)
integrated_expr <- GetAssayData(integrated)
integrated_expr <- integrated_expr[rownames(seurat), colnames(seurat)]
integrated_expr <- t(integrated_expr)
```

```python
adata_seurat.X = integrated_expr
adata_seurat.layers["seurat"] = integrated_expr
sc.tl.pca(adata_seurat)
sc.pp.neighbors(adata_seurat)
sc.tl.umap(adata_seurat)
```

### scIB benchmarking

```python
import scib

metrics_scvi = scib.metrics.metrics_fast(adata, adata_scvi, batch_key, label_key,
                                         embed="X_scVI")
metrics_scanvi = scib.metrics.metrics_fast(adata, adata_scanvi, batch_key, label_key,
                                          embed="X_scANVI")
metrics_bbknn = scib.metrics.metrics_fast(adata, adata_bbknn, batch_key, label_key)
metrics_seurat = scib.metrics.metrics_fast(adata, adata_seurat, batch_key, label_key)
metrics_hvg = scib.metrics.metrics_fast(adata, adata_hvg, batch_key, label_key)

metrics = pd.concat([metrics_scvi, metrics_scanvi, metrics_bbknn,
                     metrics_seurat, metrics_hvg], axis="columns")
metrics = metrics.set_axis(["scVI", "scANVI", "BBKNN", "Seurat", "Unintegrated"],
                           axis="columns")
metrics = metrics.loc[["ASW_label", "ASW_label/batch", "PCR_batch",
                       "isolated_label_silhouette", "graph_conn"]].T
metrics_scaled = (metrics - metrics.min()) / (metrics.max() - metrics.min())

# Bio vs Batch summary
metrics_scaled["Batch"] = metrics_scaled[["ASW_label/batch", "PCR_batch",
                                          "graph_conn"]].mean(axis=1)
metrics_scaled["Bio"] = metrics_scaled[["ASW_label",
                                        "isolated_label_silhouette"]].mean(axis=1)
metrics_scaled["Overall"] = 0.4 * metrics_scaled["Batch"] + 0.6 * metrics_scaled["Bio"]
```

## Reference Tables

### Clustering resolution guide

| Resolution | Use case | Typical cluster count (10k cells) |
|---|---|---|
| 0.1–0.3 | Coarse cell-type level | 5–10 |
| 0.4–0.8 | Standard analysis | 10–20 |
| 1.0–1.5 | Sub-type level | 20–40 |
| >2.0 | Rare population discovery | 40+ |

### Annotation method comparison

| Method | Input | Output | When to use |
|---|---|---|---|
| Manual markers | marker dict + clusters | cell type per cluster | Gold standard; novel cell types |
| rank_genes_groups DGE | clusters | top markers per cluster | Unknown clusters; data-driven markers |
| CellTypist | raw counts | predicted label + confidence | Align to reference; high-throughput |
| scArches (scVI/scANVI) | query + trained model | embedding + transferred labels | Query-to-reference mapping; novel state flagging |

### Integration method categories

| Category | Example | Output format | Strengths |
|---|---|---|---|
| Global | ComBat | corrected expression | Simple; from bulk |
| Linear embedding | Harmony, Seurat, Scanorama, MNN | embedding or expression | Locally adaptive; widely used |
| Graph-based | BBKNN | kNN graph | Fastest; minimal compute |
| Deep learning | scVI, scANVI, scGen | embedding | Best for complex/large; semi-supervised option |

### scIB metrics at a glance

| Category | Metric | Measures |
|---|---|---|
| Batch removal | ASW_label/batch | Lower silhouette within batch (higher = better batch mixing) |
| Batch removal | PCR_batch | Variance explained by batch in PCs (lower = better) |
| Batch removal | graph_conn | kNN graph connectivity across batches |
| Bio conservation | ASW_label | Higher silhouette within cell type |
| Bio conservation | isolated_label_silhouette | Quality for rare cell types |
| Bio conservation | hvg_overlap | HVG set overlap across batches |

### Integration method selection

| Scenario | Recommended |
|---|---|
| Simple batch correction (same protocol) | Harmony, ComBat |
| Complex multi-batch, no labels | scVI, Scanorama |
| Complex multi-batch, with labels | scANVI |
| Fast first-pass / large dataset | BBKNN |
| Need corrected expression matrix | Seurat, ComBat |
| Query-to-reference mapping | scArches (scVI/scANVI) |

## Worked Example

**NeurIPS 2021 BMMC** — 10x Multiome bone marrow dataset, 69,249 cells × 129,921 features across 13 batches. For demonstration: subset to 3 batches (s1d3, s2d1, s3d7) → 10,270 cells.

**Clustering + annotation workflow**:
1. HVG selection → PCA → neighbors → Leiden (resolution 0.5) → UMAP
2. Build bone marrow marker dict (CD14+ Mono, CD16+ Mono, T cell, B cell, NK, DC, pDC, progenitors)
3. `rank_genes_groups` per cluster → dotplot top 5 markers per cluster → cross-validate with marker dict
4. Assign clusters to cell types; sub-cluster T cells (cluster 3) at resolution 0.8 to identify CD4/CD8/cytotoxic subtypes
5. Save as `s4d8_annotated.h5ad`

**scArches label transfer workflow** (reference = NeurIPS annotated atlas, query = s4d8):
1. Load trained scANVI reference model
2. `sca.models.SCVI.load_query_data` → train 200 epochs → `get_latent_representation()`
3. Concatenate ref + query embeddings → UMAP → check partial mixing (full separation = mapping failed)
4. KNN label transfer (n_neighbors=15) → per-cell label + uncertainty
5. Cells with uncertainty >0.2 set to "Unknown" → manual review with marker dotplots
6. Findings: progenitor cells and "Other T" have highest uncertainty (transcriptionally ambiguous); pDC has low uncertainty (transcriptionally distinct)
7. Uncertainty also flags disease-specific or novel cell states not in reference

**Data integration workflow** (3 batches):
1. Inspect unintegrated UMAP → batch-driven shift visible
2. Batch-aware HVG selection (`batch_key="batch"`) → 2000 HVGs
3. Run scVI (raw counts) → `X_scVI` → UMAP → batches mix, labels separate
4. Run scANVI (refined scVI with labels) → `X_scANVI` → preserves more biology
5. Run BBKNN (graph only) → UMAP → moderate mixing
6. Run Seurat (via rpy2) → corrected expression matrix → UMAP
7. Run scIB metrics on all four + unintegrated
8. Scale per-metric → Batch (mean of ASW_label/batch, PCR_batch, graph_conn) + Bio (mean of ASW_label, isolated_label_silhouette)
9. Overall = 0.4·Batch + 0.6·Bio → scANVI > scVI > Seurat > BBKNN > Unintegrated

The integration tradeoff is the key insight: no method dominates both Batch and Bio. scANVI wins by preserving more biology (uses labels), but at the cost of slightly weaker batch mixing than scVI. BBKNN is fast but underperforms on both. For new datasets, run 2–3 methods and pick by scIB metrics, not UMAP.

## Key Takeaways
1. **Leiden on kNN graph is the standard** — resolution is the main knob; sub-cluster for hidden subtypes
2. **Manual marker annotation is the gold standard** — always validate automated predictions with dotplots
3. **Wilcoxon for DGE, not t-test** — t-test overestimates significance on sparse single-cell
4. **CellTypist for high-throughput** — pre-trained reference models + majority voting smooths noise
5. **scArches for query-to-reference** — uncertainty score flags novel cell states, not just failure
6. **Batch covariate choice is design-driven** — sample removes most; donor preserves between-individual
7. **scVI/scANVI are top benchmarks** — use raw counts; scANVI preserves biology when labels exist
8. **UMAP is suggestive, scIB is decisive** — quantitative metrics (Bio + Batch) drive method choice
9. **Integration ≠ batch correction** — complexity scales with protocol diversity and batch nesting
10. **Overall scIB = 0.4·Batch + 0.6·Bio** — weighting reflects biological signal priority; adjust per question

## Connects To
- **Ch 2 (Preprocessing)**: HVG selection, PCA, neighbors feed directly into clustering
- **Ch 4 (Trajectories)**: cluster labels + pseudotime order reveal developmental progression
- **Ch 5 (Conditions)**: integration is prerequisite for cross-condition DGE
- **Ch 6 (GRNs/CCC)**: cell type annotations drive SCENIC, LIANA, NicheNet analyses
- **Ch 8 (scATAC)**: same Leiden + scIB framework applies to peak-level clustering
- **Ch 12 (Multimodal integration)**: extends scVI to totalVI/multiVI for paired multi-omics
