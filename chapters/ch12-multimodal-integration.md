# Chapter 12: Multimodal Integration

## Core Idea
Paired multi-omics (CITE-seq = RNA+ADT; 10x Multiome = RNA+ATAC) measure multiple modalities per cell. Integration must handle different dimensionality (genes ~20k, ADTs ~100-200, peaks ~200k) and different distributions (NB for RNA, bimodal for ADT, Bernoulli/binarized for ATAC). Three integration families: **linear factor models** (MOFA+), **graph-based** (WNN), and **variational** (totalVI for CITE-seq, multiVI for Multiome). Use scIB metrics to pick the best for your task; for unpaired data, use GLUE (graph-linked) or bridge integration.

## Frameworks Introduced

- **Three integration challenges**
  - When to use: any paired multimodal integration
  - How: address each separately
    1. **Dimensionality mismatch** — RNA 20-30k genes, ADT 100-200, ATAC 200k+ peaks; reduce each to a common latent space
    2. **Distribution mismatch** — RNA: NB; ADT: bimodal (NB mixture for totalVI); ATAC: Bernoulli (binarized) or Poisson
    3. **Batch effects** — donor/site effects within each modality; need batch-corrected latent space
  - Why: methods differ in how they handle these; totalVI/multiVI model each modality with its correct distribution; MOFA+ assumes one distribution; WNN avoids the question by working on graphs

- **Four integration methods (paired CITE-seq)**
  - When to use: CITE-seq data with batch effects
  - How:
    1. **WNN (Seurat v4)** — graph-based; compute per-modality kNN graphs, weighted combination → joint WNN graph; supervised PCA (sPCA) gives embedding; via R `FindMultiModalNeighbors` + `RunSPCA`
    2. **MOFA+** — linear factor model; decomposes each modality into low-rank matrices; factors interpretable as sources of variation; `mu.tl.mofa(mdata, groups_label="batch")`
    3. **totalVI** — variational inference; NB for RNA + NB mixture (foreground/background) for ADT; models protein background noise; `scvi.model.TOTALVI` with `setup_anndata(layer="counts", protein_expression_obsm_key=..., batch_key=...)`
    4. **multiVI** (Multiome only) — NB for RNA + Bernoulli for binarized ATAC; `scvi.model.MULTIVI` requires concatenated features axis (genes + peaks)
  - Why: each has strengths — WNN preserves graph structure, MOFA+ is interpretable, totalVI/multiVI handle distributions correctly and win scIB benchmarks (totalVI scored best in NeurIPS 2021 demo)

- **scIB integration metrics**
  - When to use: comparing integration methods
  - How: `scib.metrics.metrics(adata, adata, batch_key="batch", label_key="cell_type", embed="X_...", ari_=True, nmi_=True, silhouette_=True, graph_conn_=True, isolated_labels_asw_=True)`; overall = `0.4 * batch_correction + 0.6 * bio_conservation`
  - Bio conservation: NMI, ARI, ASW_label, isolated_label_silhouette
  - Batch correction: ASW_label/batch, graph_conn
  - Why: no single method wins all metrics; pick based on your priority (bio preservation vs batch mixing)

- **Unpaired integration with GLUE**
  - When to use: RNA and ATAC (or RNA and ADT) from different experiments, no shared cells
  - How: construct prior-knowledge graph linking features across modalities (e.g., gene → its protein); train conditional VAE with graph guidance; `scglue.models.fit_SCGLUE({"rna": ..., "adt": ...}, graph)`; outputs shared latent space
  - Why: without paired cells, prior knowledge graph is the only bridge between modalities

- **Query-to-reference mapping (scArches)**
  - When to use: have a pre-built reference, want to map new query data (possibly missing modalities)
  - How: train reference model (totalVI/MultiVI/Multigrate) → `load_query_data(query, ref_model)` → fine-tune with `weight_decay=0, scale_adversarial_loss=0`; query can be unimodal (RNA only) or multimodal; imputes missing modalities (e.g., predicts ADT from RNA)
  - Why: leverage large reference atlases for new small studies; enables cross-modality imputation

- **Bridge integration (Seurat v5)**
  - When to use: map a new modality (e.g., ADT) onto an RNA reference using a "bridge" dataset with paired RNA+ADT
  - How: 3 datasets (RNA ref, paired RNA+ADT bridge, ADT query); `PrepareBridgeReference` → `FindBridgeTransferAnchors` → `MapQuery`; predicts cell types and imputes RNA for the ADT-only query
  - Why: enables cross-technology mapping when no shared cells exist between query and reference

- **Multigrate (trimodal integration)**
  - When to use: RNA + ATAC + ADT from separate CITE-seq + Multiome experiments; want a unified trimodal reference
  - How: organize data with `mtg.data.organize_multiome_anndatas`; train MultiVAE with PoE (Product of Experts); missing modalities set to non-informative marginals; `load_query_data` for query mapping
  - Why: PoE gracefully handles missing modalities; single integrated latent space across three modalities

## Key Concepts
- **Modality**: a measurement type (RNA, ADT, ATAC); paired = measured in same cells; unpaired = different cells
- **WNN (Weighted Nearest Neighbors)**: per-modality graphs combined with modality weights per cell
- **sPCA (supervised PCA)**: PCA guided by WNN graph; embeds cells in a latent space preserving WNN structure
- **MOFA+ factor**: latent dimension; interpretable as a source of variation (e.g., cell cycle, batch)
- **totalVI**: VAE for CITE-seq; NB for RNA, NB mixture (foreground+background) for ADT
- **multiVI**: VAE for Multiome; NB for RNA, Bernoulli for binarized ATAC
- **GLUE**: graph-linked unified embedding; uses prior knowledge graph to connect features
- **Bridge dataset**: paired multi-modal dataset used to map a new modality onto a reference
- **Product of Experts (PoE)**: combines marginal distributions into joint; treats missing modalities as non-informative
- **scArches**: framework for query-to-reference mapping with pre-trained deep generative models
- **Mosaic integration**: paired + unpaired data combined (some cells have 2 modalities, others have 1)

## Mental Models
- **Paired ≠ unpaired** — paired integration uses cell-level correspondence; unpaired needs a feature-level bridge (graph or paired dataset)
- **Distribution matters** — totalVI/multiVI win because they model NB for RNA, NB-mixture for ADT, Bernoulli for ATAC; methods that assume one distribution leave signal on the table
- **WNN is graph-native** — doesn't produce a single embedding; sPCA gives one but the WNN graph is the primary output
- **MOFA+ factors are interpretable** — each factor is a source of variation; can inspect factor weights per gene/protein/peak
- **scIB overall = 0.4 batch + 0.6 bio** — bio conservation weighted higher; reflects that losing cell-type signal is worse than imperfect batch mixing
- **Query-to-reference needs scArches-compatible models** — totalVI/MultiVI/Multigrate support `load_query_data`; MOFA+ and WNN don't
- **Bridge integration needs a bridge dataset** — a small paired dataset opens the door to mapping a new modality onto an established atlas
- **Multigrate handles missing modalities via PoE** — unimodal queries (RNA only) can be mapped onto a trimodal reference

## Anti-patterns
- **Concatenating modalities without distribution awareness** — raw features from different distributions don't combine well; use a method that models each correctly
- **Using WNN when you need an embedding for downstream tasks** — WNN graph is primary; sPCA is the embedding
- **Forgetting to rename ADT features to avoid collision with RNA genes** — `adt.var_names = ["PROT_" + n for n in adt.var_names]`
- **Comparing methods without scIB metrics** — visual UMAP inspection is misleading; use quantitative metrics
- **Using MOFA+ on raw counts** — needs normalized data in `.X`; pass `groups_label="batch"` for batch correction
- **Passing non-binarized ATAC to multiVI** — multiVI requires binary ATAC (0/1 open/closed)
- **Skipping `batch_key` in scvi models** — without it, batch effects aren't corrected
- **Training totalVI/MultiVI on HVG subset only** — these methods expect raw counts in `layer="counts"`, not HVG-filtered `.X`
- **Query-to-reference mapping without fine-tuning** — `load_query_data` then `train(weight_decay=0)` is mandatory
- **Bridge integration without a paired bridge dataset** — bridge must have both reference modality (RNA) and query modality (ADT)

## Code Examples

### WNN (Seurat v4 via rpy2)

```python
%%R -i adata_
cite <- as.Seurat(adata_, data='X', counts=NULL)
cite[["ADT"]] <- CreateAssayObject(data = adt@assays$ADT@data)
cite <- RenameAssays(object = cite, originalexp = "RNA", verbose=FALSE)

# Per-modality PCA
DefaultAssay(cite) <- "RNA"
VariableFeatures(cite) <- rownames(cite)
cite <- ScaleData(cite, verbose=FALSE) %>% RunPCA(verbose=FALSE)
cite@reductions$harmony_pca <- adt@reductions$harmony_pca

# WNN
cite <- FindMultiModalNeighbors(
    cite, reduction.list = list("pca", "harmony_pca"),
    dims.list = list(1:50, 1:30), modality.weight.name = "RNA.weight", verbose=FALSE)

# sPCA + UMAP (return.model=TRUE for query mapping later)
cite <- RunSPCA(cite, assay = "RNA", graph = "wsnn", npcs = 20)
cite <- RunUMAP(cite, nn.name = "weighted.nn", reduction.name = "wnn.umap",
                reduction.key = "wnnUMAP_", return.model=TRUE)
```

```python
# Move sPCA + WNN graph back to Python
mdata.obsm["X_spca"] = spca
mdata.obsp["wnn_connectivities"] = scipy.sparse.coo_matrix((wnn["x"], (wnn["i"]-1, wnn["j"]-1)))
sc.pp.neighbors(mdata, use_rep="X_spca")
mdata.obsp["connectivities"] = mdata.obsp["wnn_connectivities"].copy()
sc.tl.umap(mdata)
mdata.obsm["X_umap_wnn"] = mdata.obsm["X_umap"].copy()
```

### totalVI (CITE-seq, scvi-tools)

```python
import scvi
adata = mdata["rna"].copy()
adata.obsm["protein_expression"] = mdata["adt"].layers["counts"].A.copy()

scvi.model.TOTALVI.setup_anndata(
    adata, protein_expression_obsm_key="protein_expression",
    layer="counts", batch_key="batch")

vae = scvi.model.TOTALVI(adata)
vae.train()

mdata.obsm["X_totalVI"] = vae.get_latent_representation()
sc.pp.neighbors(mdata, use_rep="X_totalVI")
sc.tl.umap(mdata)
mdata.obsm["X_umap_totalVI"] = mdata.obsm["X_umap"].copy()
```

### multiVI (Multiome, scvi-tools)

```python
# Concatenate RNA + ATAC along feature axis (paired cells)
adata_paired = ad.concat([rna.copy().T, atac.copy().T]).T
adata_paired.obs["modality"] = "paired"
adata_mvi = scvi.data.organize_multiome_anndatas(adata_paired)

scvi.model.MULTIVI.setup_anndata(
    adata_mvi, batch_key="modality",
    categorical_covariate_keys=["batch"], layer="counts")

mvi = scvi.model.MULTIVI(adata_mvi, n_genes=n_genes, n_regions=n_regions)
mvi.train()

mdata_multiome.obsm["X_multiVI"] = mvi.get_latent_representation()
```

### MOFA+

```python
mu.tl.mofa(mdata, groups_label="batch", gpu_mode=True)
sc.pp.neighbors(mdata, use_rep="X_mofa")
sc.tl.umap(mdata)
mdata.obsm["X_umap_mofa"] = mdata.obsm["X_umap"].copy()
```

### scIB metrics comparison

```python
import scib
metrics = []
for method, embed in [("WNN", "X_spca"), ("MOFA+", "X_mofa"), ("totalVI", "X_totalVI")]:
    scib_anndata = sc.AnnData(mdata.obsm[embed]).copy()
    scib_anndata.obs = mdata.obs.copy()
    scib_anndata.obsp["connectivities"] = mdata.obsp["connectivities"].copy()
    scib_anndata.obsm[embed] = mdata.obsm[embed].copy()
    m_ = scib.metrics.metrics(scib_anndata, scib_anndata,
        batch_key="batch", label_key="cell_type", embed=embed,
        ari_=True, nmi_=True, silhouette_=True, graph_conn_=True,
        isolated_labels_asw_=True)[0]
    m_["method"] = method
    metrics.append(m_)
metrics = pd.DataFrame(metrics).set_index("method")

# Overall score: 0.4 batch + 0.6 bio
metrics["overall"] = (
    0.4 * (metrics["ASW_label/batch"] + metrics["graph_conn"]) / 2 +
    0.6 * (metrics["NMI_cluster/label"] + metrics["ARI_cluster/label"] +
           metrics["ASW_label"] + metrics["isolated_label_silhouette"]) / 4)
```

### GLUE unpaired RNA + ADT

```python
import scglue, networkx as nx

# Prior knowledge graph: connect proteins to their encoding genes
mask = np.array(np.repeat(p.reshape(-1, 1), r.shape[0], axis=1) == r)
graph = nx.Graph()
graph.add_nodes_from(rna_vars); graph.add_nodes_from(prot_vars)
graph.add_edges_from([(n1, n2, {"weight": 1.0, "sign": 1})
                      for n1, n2 in adj[adj > 0].stack().index.tolist()])
graph.add_edges_from([(g, g, {"weight": 1.0, "sign": 1}) for g in rna_vars + prot_vars])

# Configure: NB for RNA, Normal for ADT (CLR-normalized)
scglue.models.configure_dataset(rna_multiome, "NB", use_rep="X_pca",
                                 use_batch="Samplename", use_layer="counts")
scglue.models.configure_dataset(adt_cite, "Normal", use_rep="X_pca",
                                 use_batch="Samplename")

glue = scglue.models.fit_SCGLUE({"rna": rna_multiome, "adt": adt_cite}, graph)
rna_multiome.obsm["X_glue"] = glue.encode_data("rna", rna_multiome)
adt_cite.obsm["X_glue"] = glue.encode_data("adt", adt_cite)
```

### totalVI query-to-reference (scArches)

```python
# Train reference (with one batch's protein set to 0 = RNA-only cells)
adata.obsm["protein_counts"] = adt_cite.layers["counts"].A.copy()
adata.obsm["protein_counts"][adata.obs["batch"] == "s1d3"] = 0.0  # mimic RNA-only

TOTALVI.setup_anndata(adata, layer="counts", batch_key="batch",
                     protein_expression_obsm_key="protein_counts")
vae = TOTALVI(adata, **arches_params)
vae.train()

# Map query
vae_q = TOTALVI.load_query_data(query, vae)
vae_q.train(plan_kwargs={"weight_decay": 0.0, "scale_adversarial_loss": 0.0})
query.obsm["X_totalvi_scarches"] = vae_q.get_latent_representation(query)

# Combine ref + query
full_data = adata.concatenate(query, batch_key="concat_batch")
sc.pp.neighbors(full_data, use_rep="X_totalvi_scarches")
sc.tl.umap(full_data)
```

### Multigrate trimodal reference

```python
import multigrate as mtg

# Organize RNA (paired CITE+Multiome) + ADT + ATAC
adata = mtg.data.organize_multiome_anndatas(
    adatas=[[rna1_ref, rna2_ref], [None, atac_multiome], [adt_cite, None]],
    layers=[["counts", "counts"], [None, "cpm"], [None, None]])

mtg.model.MultiVAE.setup_anndata(
    adata, categorical_covariate_keys=["Modality", "Samplename"],
    rna_indices_end=4000)

model = mtg.model.MultiVAE(adata, losses=["nb", "mse", "mse"],
    loss_coefs={"kl": 1e-3, "integ": 10000},
    integrate_on="Modality", mmd="marginal")
model.train()
model.get_latent_representation()
adata.obsm["latent_ref"] = adata.obsm["latent"].copy()

# Query mapping (supports unimodal queries via zero-masking)
q_model = mtg.model.MultiVAE.load_query_data(query, model)
q_model.train(weight_decay=0)
q_model.get_latent_representation(adata=query)
q_model.get_latent_representation(adata=adata)
```

- **What it demonstrates**: full trimodal integration. PoE allows unimodal queries (ATAC-only or RNA-only) to map onto the trimodal reference. `integrate_on="Modality"` with `mmd="marginal"` aligns modalities in latent space.

## Reference Tables

### Integration method selection

| Data type | Recommended | Alternatives |
|---|---|---|
| Paired CITE-seq (RNA+ADT) | totalVI | WNN, MOFA+ |
| Paired Multiome (RNA+ATAC) | multiVI | WNN, MOFA+ |
| Unpaired RNA + ADT | GLUE | — |
| Unpaired RNA + ATAC | GLUE | — |
| Trimodal (RNA+ADT+ATAC) | Multigrate | — |
| RNA ref + ADT query (no shared cells) | Bridge (Seurat v5) | — |
| Query → multimodal reference | scArches (totalVI/MultiVI/Multigrate) | Seurat v4 (WNN) |

### Distribution assumptions by method

| Method | RNA | ADT | ATAC |
|---|---|---|---|
| WNN | (graph-only, no distribution) | (graph-only) | (graph-only) |
| MOFA+ | Linear Gaussian | Linear Gaussian | Linear Gaussian |
| totalVI | NB | NB mixture (fg+bg) | — |
| multiVI | NB | — | Bernoulli (binarized) |
| Multigrate | NB | MSE (Gaussian) | MSE (Gaussian) |
| GLUE | NB | Normal (CLR) | — |

### scIB metrics — what they measure

| Metric | Type | What it captures |
|---|---|---|
| NMI_cluster/label | Bio | Cluster labels match cell types |
| ARI_cluster/label | Bio | Same as NMI but more stringent |
| ASW_label | Bio | Cell-type separation in embedding |
| isolated_label_silhouette | Bio | Rare cell types preserved |
| ASW_label/batch | Batch | Same cell type across batches mixed |
| graph_conn | Batch | Connectivity across batches |

## Worked Example

**NeurIPS 2021 BMMC — CITE-seq integration** — 3 donors from site 1 (s1d1, s1d2, s1d3) of the CITE-seq dataset. WNN, MOFA+, totalVI all run on the same preprocessed data (RNA HVG + ADT CLR/Harmony).

Pipeline:
1. Subset RNA + ADT to common cells (intersect `obs_names`); prefix ADT var names with `PROT_`
2. Build MuData: `mu.MuData({"rna": rna, "adt": adt})`
3. WNN: move to R → Seurat → `FindMultiModalNeighbors` (PCA 1:50 for RNA, harmony_pca 1:30 for ADT) → `RunSPCA` (npcs=20) → `RunUMAP` (return.model=TRUE for query mapping later); extract sPCA + WNN graph to Python
4. MOFA+: `mu.tl.mofa(mdata, groups_label="batch", gpu_mode=True)`; 20 factors by default
5. totalVI: setup_anndata with `protein_expression_obsm_key`, `layer="counts"`, `batch_key="batch"`; train → `get_latent_representation()` → `X_totalVI`
6. Compute scIB metrics for each; totalVI typically wins overall (0.4 batch + 0.6 bio)

For Multiome (s1d1, s1d2, s1d3):
1. Concatenate RNA + ATAC along feature axis via `ad.concat([rna.T, atac.T]).T`
2. `scvi.data.organize_multiome_anndatas`
3. multiVI: `setup_anndata(batch_key="modality", categorical_covariate_keys=["batch"], layer="counts")`; train → `X_multiVI`

For unpaired RNA + ADT (Multiome RNA + CITE-seq ADT, no shared cells):
1. Rename ADT features to gene symbols (CD103→ITGAE, etc.)
2. Construct prior-knowledge graph: protein → encoding gene (mask matrix), self-loops
3. Configure datasets: NB for RNA (counts), Normal for ADT (CLR)
4. `scglue.models.fit_SCGLUE({"rna": ..., "adt": ...}, graph)` → shared latent space

For query-to-reference mapping (totalVI):
1. Train reference with one batch's protein set to 0 (mimics RNA-only cells in reference)
2. `TOTALVI.load_query_data(query, vae)` → `train(plan_kwargs={"weight_decay": 0.0, "scale_adversarial_loss": 0.0})`
3. Concatenate reference + query; recalculate UMAP

For trimodal reference (Multigrate):
1. `mtg.data.organize_multiome_anndatas` with paired CITE RNA + paired Multiome RNA + ADT + ATAC
2. `MultiVAE.setup_anndata(categorical_covariate_keys=["Modality", "Samplename"], rna_indices_end=4000)`
3. Train with `losses=["nb", "mse", "mse"]`, `loss_coefs={"kl": 1e-3, "integ": 10000}`, `integrate_on="Modality"`, `mmd="marginal"`
4. Query: zero-mask missing modalities (RNA=0 for ATAC-only, ATAC=0 for RNA-only); `load_query_data` → `train(weight_decay=0)`

The key architectural decision is `integrate_on="Modality"` with `mmd="marginal"` — this aligns the marginal distributions of each modality in latent space via Maximum Mean Discrepancy, while PoE gracefully handles missing modalities by treating them as non-informative.

## Key Takeaways
1. **Pick method by data type** — paired CITE-seq: totalVI; paired Multiome: multiVI; unpaired: GLUE; trimodal: Multigrate
2. **Distribution-aware methods (totalVI/multiVI) typically win scIB benchmarks** — model NB for RNA, NB-mixture for ADT, Bernoulli for ATAC
3. **WNN is graph-native** — primary output is the WNN graph; sPCA provides an embedding if needed
4. **MOFA+ factors are interpretable** — each factor is a source of variation; useful for hypothesis generation
5. **Compare methods with scIB metrics** — overall = 0.4 batch + 0.6 bio; visual UMAP inspection is misleading
6. **Rename features to avoid modality collisions** — prefix ADT names with `PROT_`
7. **GLUE for unpaired integration needs a prior-knowledge graph** — connect features (gene → protein, gene → peak)
8. **scArches for query-to-reference** — train reference with `load_query_data` support (totalVI/MultiVI/Multigrate); fine-tune with `weight_decay=0`
9. **Bridge integration needs a paired bridge dataset** — opens cross-modality mapping when query and reference share no cells
10. **Multigrate handles missing modalities via PoE** — train trimodal reference; query can be unimodal (zero-mask missing)

## Connects To
- **Ch 1 (MuData)**: paired data lives in MuData with modality slots; integration methods consume this structure
- **Ch 2 (Preprocessing)**: per-modality normalization (log1p for RNA, dsb/CLR for ADT, TSS for ATAC) happens first
- **Ch 8 (scATAC)**: Multiome uses the same ATAC QC + binarization before multiVI
- **Ch 10 (Surface protein)**: CITE-seq ADT preprocessing (dsb, doublet removal) feeds into totalVI
- **Ch 6 (GRNs)**: integrated latent space enables SCENIC+ / FigR for paired RNA+ATAC regulatory inference
- **Ch 9 (Spatial)**: spatial transcriptomics can be treated as another modality; spatial RNA + scRNA-seq integration via Tangram (Ch 9) is unpaired integration
