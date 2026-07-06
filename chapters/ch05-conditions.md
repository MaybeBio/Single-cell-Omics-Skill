# Chapter 5: Conditions — DGE, Compositional Analysis, Pathways & Perturbations

## Core Idea
Four layers turn an annotated dataset into condition-level biology: (1) **Differential Gene Expression (DGE)** via pseudobulk + DESeq2/edgeR (sample-level view outperforms cell-level for scRNA-seq), (2) **Compositional analysis** via scCODA/tascCODA (Dirichlet-multinomial with reference cell type) or MILO/DA-Seq (KNN-based for transitional states), (3) **Pathway enrichment** via GSEA/fgsea (cluster-level) or AUCell/GSVA (cell-level activity), and (4) **Perturbation modeling** via Augur (cell-type prioritization), scGen (response prediction), Mixscape (perturbation separation). Each layer addresses a different question: which genes? which cell types change in proportion? which pathways? which cell type is most affected?

## Frameworks Introduced

- **Pseudobulk DGE (sample-level view)**
  - When to use: any condition comparison with biological replicates
  - How:
    1. Aggregate cells per (sample, cell_type) via `decoupler.pp.pseudobulk(adata, sample_col, groups_col, layer="counts", mode="sum")`
    2. Filter low-quality pseudobulks (min 10 cells, min 1000 counts)
    3. PCA on pseudobulks → identify covariates (cell type, batch, sex) → design matrix
    4. Filter lowly-expressed genes per cell type: `decoupler.pp.filter_by_expr()` + `filter_by_prop()`
    5. Fit model: `pertpy.tl.PyDESeq2(adata, design="~ label")` → `fit()` → `test_contrasts()`
    6. Output: `variable`, `baseMean`, `log_fc`, `lfcSE`, `stat`, `p_value`, `adj_p_value`
  - Why: avoids pseudoreplication (cells from same patient aren't independent); bulk DESeq2/edgeR methods outperform single-cell-specific methods (MAST, Wilcoxon) which inflate FDR

- **Complex experimental designs with interactions**
  - When to use: cell-type-specific treatment effects; multiple factors (sex, responder, age)
  - How: `design="~ cell_type * label"` (shorthand for `cell_type + label + cell_type:label`); use `pds2.cond()` to define custom contrasts; interaction contrast = (cell_type_A stim vs ctrl) - (cell_type_B stim vs ctrl)
  - Why: tests whether treatment effect differs between cell types; interaction terms are the only way to ask "is the response cell-type-specific?"

- **Compositional data analysis: scCODA**
  - When to use: known cell types with distinct clusters; sample-level replicates
  - How:
    1. Find a **reference cell type** (assumed unchanged) — via `scCODA.model.model_selection.find_coder_gene()` or domain knowledge
    2. Fit Dirichlet-multinomial: `sccoda_model.fit(Y=count_matrix, covariate_matrix=design)` → effect sizes + inference
    3. Interpret: only "credible" effects (not crossing 0 in 95% CI) are significant
  - Why: scRNA-seq counts are compositional (sum-to-sample-size constraint); standard tests (Wilcoxon, scDC) falsely perceive negative correlations induced by sum constraint

- **KNN-based compositional analysis: MILO / DA-Seq**
  - When to use: developmental trajectories; transitional states; no discrete clusters
  - How: build KNN graph on cells (not clusters) → assign cells to neighborhoods → test neighborhoods for condition enrichment → neighborhoods can span cell-type boundaries
  - Why: scCODA requires discrete clusters; MILO captures abundance shifts in continuous cell-state space

- **Gene set enrichment: GSEA vs activity scoring**
  - When to use: GSEA for cluster/cell-type-level enrichment; activity scoring for per-cell pathway scores
  - How:
    - **GSEA**: rank genes by t-statistic → `decoupler.run_gsea(ranked_df, pathways)` → enrichment score + p-value; competitive null hypothesis (genes in set vs not in set)
    - **AUCell**: per-cell ranking → AUC of gene set in top-expressed genes → `adata.obsm["aucell_estimate"]`
    - **GSVA**: per-sample pathway activity (similar to AUCell but sample-level)
    - **fgsea**: fast GSEA implementation in R; uses precomputed ranks
  - Why: GSEA tests "is the pathway enriched?"; AUCell/GSVA score "how active is the pathway in this cell?"

- **limma-fry for complex designs**
  - When to use: pseudobulk + complex experimental design (interactions, batches)
  - How: pseudobulk per (sample, cell_type, condition) → `limma::fry(log_norm_X, index=pathway_idx, design, contrast)` → FDR + FDR.Mixed (mixed = genes changing in inconsistent directions)
  - Why: linear model framework handles arbitrary designs; accounts for inter-gene correlations; outperforms GSEA for tightly related gene sets (REACTOME_INTERFERON_SIGNALING vs ALPHA_BETA)

- **Augur: cell-type prioritization**
  - When to use: identify which cell type is most affected by perturbation
  - How: train classifier (random_forest or logistic_regression) to predict condition labels per cell type → AUC = perturbation response magnitude; `pertpy.tl.Augur("random_forest_classifier")` → `load(adata, label_col, cell_type_col)` → `predict()`
  - Why: cells highly affected by perturbation are easier to classify; AUC quantifies separability; works without ground-truth DE genes
  - Limitations: requires distinct cell types; aggregates cell-type-level response (ignores intra-type heterogeneity); subsampling discards abundance changes

- **scGen: perturbation response prediction**
  - When to use: predict response for unseen cell type/condition; autoencoder + vector arithmetic
  - How: train VAE on (control + treated) cells → compute latent difference vector (treated - control) per cell type → add to held-out control cells → reconstruct perturbed expression
  - Why: enables in-silico prediction for missing populations; works well for highly expressed genes; struggles for lowly expressed genes

- **Mixscape: perturbation separation**
  - When to use: pooled CRISPR screens; separate perturbed from non-perturbed cells
  - How: per-guide model → identify perturbed cells via perturbation signature → linear discriminant analysis for visualization → UMAP on LDA coordinates
  - Why: not all cells with a guide are perturbed (e.g.,heterogeneous knockout efficiency); Mixscape separates true responders
  - Limitation: LDA+UMAP is for visualization; don't infer distances between knockouts

## Key Concepts
- **Pseudobulk**: aggregate single-cell counts per (sample, cell_type) by summing/averaging; converts scRNA-seq to bulk-like for DESeq2/edgeR
- **Pseudoreplication**: treating cells from same patient as independent samples; inflates FDR
- **DESeq2 / edgeR / limma-voom**: bulk RNA-seq DE tools; robust to small sample sizes via Empirical Bayes shrinkage
- **PyDESeq2**: Python port of DESeq2; used in pertpy
- **Design matrix**: encodes group membership for linear model; `~ label`, `~ label + batch`, `~ cell_type * label`
- **Contrast**: linear combination of design matrix coefficients representing the comparison of interest
- **Interaction term (`a:b`)**: tests whether effect of `a` depends on `b`; `a * b` = `a + b + a:b`
- **Compositional data**: data where features sum to a constant (cell counts per sample); induces spurious negative correlations
- **Reference cell type (scCODA)**: cell type assumed unchanged; needed for unique compositional inference
- **GSEA (Gene Set Enrichment Analysis)**: ranks genes by stat → enrichment score = running sum over gene set
- **fgsea**: fast GSEA implementation; uses precomputed ranks
- **AUCell**: per-cell pathway activity scoring via AUC of gene set in top-ranked genes
- **GSVA (Gene Set Variation Analysis)**: sample-level pathway activity
- **MSigDB**: gene set database (C5=GO, C2=curated/KEGG/REACTOME, C7=immune, Hallmark=cancer)
- **Competitive null**: genes in set vs genes not in set (GSEA, fgsea)
- **Self-contained null**: genes in set are DE, regardless of others (fry, roast)
- **Augur**: ML classifier AUC = cell-type perturbation response
- **scGen**: VAE + vector arithmetic for response prediction
- **Mixscape**: separates perturbed from non-perturbed cells in pooled CRISPR screens
- **pertpy**: Python package wrapping Augur, scGen, Mixscape, PyDESeq2

## Mental Models
- **Pseudobulk first, always** — sample-level view beats cell-level for DGE; avoids pseudoreplication
- **Cell counts are compositional** — one cell type up means another appears down; use scCODA with a reference
- **GSEA tests, AUCell scores** — different questions need different methods
- **Filter gene sets before testing** — drop sets with <10-15 genes; small sets have unstable statistics
- **Complex designs need interactions** — `cell_type * label` is the only way to ask cell-type-specific effects
- **Augur prioritizes, doesn't explain** — AUC tells you which cell type responds; follow up with DGE
- **scGen works for highly expressed genes** — low-expression predictions are unreliable
- **Mixscape LDA+UMAP is visualization only** — don't infer knockout distances from UMAP
- **fry > GSEA for related pathways** — fry handles overlapping/redundant gene sets better
- **Validate Augur with DA + DGE** — abundance changes need separate test; genes need DGE

## Anti-patterns
- **Treating cells as independent samples** — pseudoreplication; inflates FDR; use pseudobulk
- **Wilcoxon for condition DGE** — single-cell methods underperform bulk-derived; only for cluster-level marker discovery
- **Single-cell t-test for DGE** — same issue; not designed for complex designs
- **Skipping PCA on pseudobulks** — fails to identify covariates that should be in design matrix
- **Filtering genes globally** — different cell types express different gene sets; filter per cell type
- **Using scCODA without a reference** — results are non-unique without anchoring
- **Wilcoxon for compositional shifts** — induces false abundance shifts from sum constraint
- **Skipping gene set size filter** — small gene sets (<10 genes) have unstable statistics
- **Using GSEA when fry fits better** — GSEA can't handle complex designs; can't model interactions
- **Interpreting AUCell as differential** — activity score ≠ differential activity; need DGE first
- **Trusting scGen for low-expression genes** — VAE predictions are unreliable below ~10 counts
- **Inferring knockout distances from Mixscape UMAP** — UMAP distances are not metric
- **Skipping Augur limitations** — requires distinct cell types; misses intra-type heterogeneity
- **Augur without DA test** — abundance changes are confounded with expression changes; both need testing

## Code Examples

### Pseudobulk DGE with PyDESeq2

```python
import decoupler as dc
import pertpy as pt
import scanpy as sc

# Create sample column (patient + condition)
adata.obs["sample"] = pd.Categorical(
    f"{r}_{l}" for r, l in zip(adata.obs["replicate"], adata.obs["label"]))

# Pseudobulk: sum counts per (sample, cell_type)
adata_pb = dc.pp.pseudobulk(adata, sample_col="sample", groups_col="cell_type",
                           layer="counts", mode="sum")

# Filter low-quality pseudobulks
dc.pp.filter_samples(adata_pb, min_cells=10, min_counts=1000)

# PCA to identify covariates
adata_pb.layers["counts"] = adata_pb.X.copy()
sc.pp.normalize_total(adata_pb, target_sum=1e6)
sc.pp.log1p(adata_pb); sc.pp.scale(adata_pb, max_value=10)
sc.tl.pca(adata_pb)
dc.tl.rankby_obsm(adata_pb, key="X_pca")  # which covariates correlate with PCs?
dc.pp.swap_layer(adata_pb, "counts", inplace=True)  # restore counts

# Subset to one cell type + filter genes
adata_mono = adata_pb[adata_pb.obs.cell_type == "CD14+ Monocytes"].copy()
dc.pp.filter_by_expr(adata_mono, group="label", min_count=10, min_total_count=15)
dc.pp.filter_by_prop(adata_mono, min_prop=0.1, min_smpls=2)

# Fit DESeq2
pds2 = pt.tl.PyDESeq2(adata=adata_mono, design="~ label")
pds2.fit()
res_df = pds2.test_contrasts(pds2.contrast(column="label", baseline="ctrl",
                                          group_to_compare="stim"))
pds2.plot_volcano(res_df, log2fc_thresh=0)
```

### Interaction contrast (cell-type-specific treatment effect)

```python
pds2 = pt.tl.PyDESeq2(adata=adata_pb, design="~ cell_type * label")
pds2.fit()

interaction_contrast = (
    pds2.cond(cell_type="CD14+ Monocytes", label="stim") -
    pds2.cond(cell_type="CD14+ Monocytes", label="ctrl")
) - (
    pds2.cond(cell_type="CD4 T cells", label="stim") -
    pds2.cond(cell_type="CD4 T cells", label="ctrl")
)
res_df = pds2.test_contrasts(interaction_contrast)
```

### scCODA compositional analysis

```python
import pertpy as pt

sccoda_model = pt.tl.SCODA()
sccoda_data = sccoda_model.load(adata, sample_col="sample",
                               covariate_col="condition", feature_col="cell_type")
# Find reference cell type
sccoda_data = sccoda_model.find_coder_gene(sccoda_data)  # suggests reference
sccoda_model.fit(sccoda_data)
sccoda_model.summary()  # credible effects only
```

### GSEA with decoupler

```python
import decoupler as dc

# Rank genes by t-statistic
sc.tl.rank_genes_groups(adata, "group", method="t-test", key="t-test")
t_stats = (sc.get.rank_genes_groups_df(adata, "stim_FCGR3A+ Monocytes", key="t-test")
           .set_index("names").loc[adata.var["highly_variable"]]
           .sort_values("scores", key=np.abs, ascending=False)[["scores"]])

# Filter gene sets by size
geneset_size = reactome.groupby("geneset").size()
gsea_genesets = geneset_size.index[(geneset_size > 15) & (geneset_size < 500)]

scores, norm, pvals = dc.run_gsea(
    t_stats.T, reactome[reactome["geneset"].isin(gsea_genesets)],
    source="geneset", target="genesymbol")
```

### AUCell per-cell activity scoring

```python
decoupler.run_aucell(adata, reactome, source="geneset", target="genesymbol",
                    use_raw=False)
adata.obs[ifn_pathways] = adata.obsm["aucell_estimate"][ifn_pathways]
sc.pl.umap(adata, color=["condition", "cell_type"] + ifn_pathways)
```

### limma-fry (R via rpy2)

```python
import anndata2ri
%load_ext rpy2.ipython
anndata2ri.activate()

groups = pb_data.obs.condition.astype("string") + "_" + pb_data.obs.cell_type
log_norm_X = pb_data.to_df().T
```

```R
%%R -i groups -i log_norm_X -i reactome -o fry_results
group <- as.factor(gsub(" |\\+", "_", groups))
design <- model.matrix(~ 0 + group)
contrasts <- limma::makeContrasts(
    groupstim_FCGR3A__Monocytes - groupctrl_FCGR3A__Monocytes,
    levels = design)
pathways <- split(reactome$genesymbol, reactome$geneset)
idx <- limma::ids2indices(pathways, rownames(log_norm_X))
idx <- idx[sapply(idx, length) >= 15]  # filter small sets
fry_results <- limma::fry(log_norm_X, index=idx, design=design,
                         contrast=contrasts[,1])
```

### Augur cell-type prioritization

```python
import pertpy as pt

adata = pt.dt.kang_2018()
adata.obs.rename({"label": "condition"}, axis=1, inplace=True)

ag = pt.tl.Augur("random_forest_classifier")
loaded = ag.load(adata, label_col="condition", cell_type_col="cell_type")
v_adata, v_results = ag.predict(loaded, subsample_size=20, n_threads=4,
                                select_variance_features=True)
ag.plot_lollipop(v_results)
# CD14+ Monocytes most affected; Megakaryocytes least

# Differential prioritization (compare two treatments)
pvals = ag.predict_differential_prioritization(
    augur_results1=results_15d, augur_results2=results_48h,
    permuted_results1=permuted_15d, permuted_results2=permuted_48h)
```

### scGen perturbation prediction

```python
import pertpy as pt

adata = pt.dt.kang_2018()
sc.pp.log1p(adata); sc.pp.highly_variable_genes(adata)

# Hold out CD4 T cells (stimulated) for prediction
adata_t = adata[~((adata.obs.cell_type == "CD4 T cells") &
                  (adata.obs.condition == "stimulated"))].copy()
cd4t_stim = adata[((adata.obs.cell_type == "CD4 T cells") &
                   (adata.obs.condition == "stimulated"))].copy()

pt.tl.SCGEN.setup_anndata(adata_t, batch_key="condition",
                         labels_key="cell_type")
model = pt.tl.SCGEN(adata_t)
model.train(max_epochs=100)

# Predict stimulated CD4 T cells from control
pred_cd4t_stim = model.predict(batch_key="condition",
                              cell_type="CD4 T cells")
```

## Reference Tables

### DGE method comparison

| Method | View | Handles replicates? | False positive rate |
|---|---|---|---|
| Pseudobulk DESeq2/edgeR | Sample | Yes | Low (best) |
| MAST with random effect | Cell | Yes | Low |
| Wilcoxon rank-sum | Cell | No | High (pseudoreplication) |
| Seurat t-test | Cell | No | High |
| scDC bootstrap | Cell | Weak | Medium |

### Compositional method selection

| Method | Data type | Use case |
|---|---|---|
| scCODA | Discrete clusters + reference | Distinct cell types, known reference |
| tascCODA | Multi-resolution (clusters + subclusters) | Hierarchical cell types |
| MILO | KNN graph neighborhoods | Transitional states, no clear clusters |
| DA-Seq | KNN classifier | Similar to MILO; different classifier |

### Gene set test comparison

| Test | Null hypothesis | Input | Complex designs? |
|---|---|---|---|
| Hypergeometric / Fisher | Competitive | DE gene counts | No |
| GSEA / fgsea | Competitive | Ranked genes | No (preranked) |
| GSVA | Competitive | Gene ranks per sample | No |
| fry (limma) | Self-contained | Expression matrix | Yes (linear model) |
| camera (limma) | Competitive | Expression matrix | Yes (handles inter-gene correlation) |
| roast (limma) | Self-contained | Expression matrix | Yes |

### MSigDB collections

| Collection | Content | Use case |
|---|---|---|
| Hallmark | 50 robust co-expression programs | Cancer, broad biology |
| C2 (curated) | KEGG, REACTOME, published signatures | Pathway analysis |
| C5 (GO) | Gene Ontology (BP, MF, CC) | Functional annotation |
| C7 (immune) | Immune signatures from immunologic studies | Immunology |

### Perturbation modeling tools

| Tool | Question | Output |
|---|---|---|
| Augur | Which cell type is most affected? | AUC per cell type (priority ranking) |
| scGen | Predict response for unseen cell type | Predicted expression matrix |
| Mixscape | Separate perturbed from non-perturbed cells | Cell labels (perturbed/non-perturbed) |
| CellRank | Predict fate probabilities | Fate probabilities per cell |
| scVI/scANVI | Latent space with batch/perturbation | Embedding for downstream |

### PyDESeq2 results columns

| Column | Meaning |
|---|---|
| `variable` | Gene name |
| `baseMean` | Mean normalized counts across all samples |
| `log_fc` | log2 fold change (group vs baseline) |
| `lfcSE` | Standard error of log_fc |
| `stat` | Wald statistic |
| `p_value` | Wald test p-value |
| `adj_p_value` | Benjamini-Hochberg adjusted p-value |

## Worked Example

**Kang et al. PBMC IFN-β stimulation** — 8 Lupus patients × 2 conditions (ctrl/stim) × 7 cell types, ~25k cells:

**DGE workflow** (CD14+ Monocytes):
1. Pseudobulk: 8 patients × 2 conditions × 7 cell types = 112 pseudobulks (some filtered by min_cells=10)
2. PCA on pseudobulks → PC1 = cell type, PC2 = condition; `psbulk_counts` correlates with CD4 T/CD14 Mono
3. Subset CD14+ Monocytes → filter genes (`filter_by_expr` + `filter_by_prop`) → ~6,000 genes retained
4. PyDESeq2 `~ label` → IFITM2 log_fc=+4.7, VCAN log_fc=-4.4 (top DE genes)
5. Volcano plot → IFIT family upregulated (IFN response)

**Complex design** (`~ cell_type * label`):
1. Interaction contrast: (CD14+ Mono stim vs ctrl) - (CD4 T stim vs ctrl)
2. RABGAP1L: higher change in CD14+ Mono; ENO1: lower change
3. Fold-change plot of top 15 genes (adj_p < 0.01)

**Compositional analysis** (Haber et al. mouse intestine, Salmonella/Hpoly infection):
- 4 conditions (Control, Salmonella, Hpoly.Day3, Hpoly.Day10) × 10 batches
- scCODA with reference = Enterocytes (assumed stable)
- Findings: Salmonella → loss of mature enterocytes; Hpoly.Day10 → expansion of tuft/goblet cells

**Pathway analysis** (Kang CD16 Monocytes stim):
1. rank_genes_groups t-test → t-statistic per gene
2. GSEA on REACTOME pathways (filter 15 < size < 500)
3. Top: REACTOME_INTERFERON_SIGNALING, INTERFERON_GAMMA_SIGNALING, INTERFERON_ALPHA_BETA_SIGNALING
4. AUCell per-cell scores → UMAP shows high IFN scores in stim CD16 Mono

**Perturbation modeling**:
1. Augur on Kang IFN-β → CD14+ Monocytes most affected (highest AUC), Megakaryocytes least
2. Differential prioritization (Bhattacherjee cocaine withdrawal 15d vs 48h) → Inhibitory cells not different (permanent damage)
3. scGen hold-out CD4 T cells (stimulated) → train on rest → predict CD4 T stim response → high-expression genes correlate well with true stim

## Key Takeaways
1. **Pseudobulk + DESeq2/edgeR is the gold standard** — sample-level view beats cell-level; avoids pseudoreplication
2. **PCA on pseudobulks before DGE** — identify covariates (cell type, batch, sex) for design matrix
3. **Filter genes per cell type** — different cell types express different gene sets
4. **Interaction terms test cell-type-specific effects** — `~ cell_type * label` is the only way
5. **Cell counts are compositional** — scRNA-seq sums to sample size; use scCODA with a reference
6. **MILO/DA-Seq for transitional states** — KNN-based methods handle continuous cell states
7. **GSEA tests enrichment; AUCell scores activity** — different questions, different methods
8. **Filter small gene sets (<10 genes)** — small sets have unstable statistics
9. **limma-fry > GSEA for complex designs** — handles interactions, inter-gene correlations, redundancy
10. **Augur prioritizes cell types by AUC** — follow up with DA + DGE for full picture
11. **scGen predicts response for highly expressed genes only** — VAE struggles with low-expression
12. **Mixscape LDA+UMAP is visualization only** — don't infer knockout distances from UMAP
13. **Validate perturbation results with multiple methods** — Augur + DA + DGE + pathway together

## Connects To
- **Ch 3 (Cellular structure)**: cell-type annotations required for pseudobulk per cell type
- **Ch 4 (Trajectories)**: MILO works on continuous cell states; pseudotime orders perturbation response
- **Ch 6 (GRNs/CCC)**: DGE genesets feed into NicheNet ligand activity; SCENIC regulons = AUCell on TF targets
- **Ch 8 (scATAC)**: pseudobulk DGE extends to peaks; chromatin accessibility changes mirror expression
- **Ch 11 (AIRR)**: perturbation modeling applies to BCR/TCR stimulation experiments
- **Ch 12 (Multimodal)**: DGE per modality (RNA + ATAC + ADT) after multimodal integration
