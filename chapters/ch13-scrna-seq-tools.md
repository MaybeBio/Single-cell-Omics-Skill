# Chapter 13: scRNA-seq Tools Ecosystem

## Core Idea
The scRNA-seq tool ecosystem spans every pipeline stage from raw FASTQ to biological interpretation. Tools cluster around the scverse ecosystem (scanpy, anndata, muon) in Python and Bioconductor/Seurat in R, with stage-specific specialists: SoupX/CellBender for ambient RNA, scDblFinder/Scrublet for doublets, scVI/Harmony for integration, CellTypist/SingleR for annotation, pseudobulk+DESeq2/muscat for DGE. Most pipelines assemble multiple tools rather than use one end-to-end suite - the skill is knowing which tool for which decision.

## Frameworks Introduced

- **Pipeline-stage tool selection**
  - When to use: building any scRNA-seq pipeline
  - How: identify the stage (preprocessing → QC → normalization → integration → dimensionality reduction → clustering → annotation → DGE → downstream), pick the best-in-class tool per stage, glue together with AnnData
  - Why: no single tool excels at every stage; the ecosystem has converged on stage-specific specialists

- **scverse vs Bioconductor vs Seurat ecosystem choice**
  - When to use: starting a project
  - How: scverse (Python, scanpy) for scalability and integration with ML; Seurat (R) for mature methods and CITE-seq/WNN; Bioconductor for tutorials/OSCA compatibility
  - Why: pick the ecosystem that matches your team's expertise and the methods you need - cross-ecosystem data exchange works via anndata2ri/rpy2 but adds friction

- **Per-stage multi-tool benchmarking**
  - When to use: any production pipeline
  - How: for each stage (integration, clustering, DGE), check published benchmarks (scIB for integration, pipeComp for preprocessing, Confronting false discoveries for DGE); prefer methods with public benchmarks over novel preprints
  - Why: tool choice has outsized impact on results; benchmarks reveal which methods are robust vs brittle

## Key Concepts
- **Ambient RNA correction**: SoupX (R, estimation from empty droplets), CellBender (Python, deep learning); 2-50% contamination typical, 10% common for 10X PBMC
- **Doublet detection**: scDblFinder (R, simulation-based, recommended), Scrublet (Python), DoubletFinder; orthogonal to marker-based for ADT
- **Normalization alternatives**: log1p+normalize_total (visualization), scran (DE), Pearson residuals (HVG), sctransform (Seurat v3, variance-stabilizing)
- **Batch correction hierarchy**: scVI/scANVI (raw counts, deep generative) > Harmony (PCA, fast) > BBKNN (graph only) > Seurat anchors (CCA+MNN) > Scanorama (panorama stitching) > Liger (NMF)
- **Cell type annotation**: CellTypist (logistic regression, pretrained atlases), SingleR (correlation-based), scPred, CellAssign (probabilistic), scArches (reference mapping with scVI), Garnett (marker-based)
- **DGE methods**: pseudobulk + DESeq2/edgeR/limma-voom (recommended), muscat (multi-sample), MAST (Hurdle model), NEBULA (negative binomial mixed), scVI-DE (deep generative)
- **Clustering validity**: SC3 (consensus), sc-SHC (significance testing), CHOIR (statistical distinctness), clustermole (marker database)
- **Trajectory**: Monocle3 (DDRTree), Palantir (multi-fate), scVelo (RNA velocity), CellRank (fate mapping), STREAM (interactive)
- **Cell-cell communication**: CellPhoneDB, LIANA (consensus), NicheNet (ligand activity), CellChat, NATMI, SingleCellSignalR
- **CNV from RNA**: inferCNV (damage pattern), HoneyBADGER (allele-aware), Numbat (allele+phasing), SCEVAN (tumor subclones)
- **Co-expression/modules**: SCENIC (regulons), cNMF (gene programs), WGCNA/scWGCNA (modules), Hotspot (spatial-aware)

## Mental Models
- **Tools are stage-specialized, not pipeline-monolithic** - the best integration method isn't the best clustering method; assemble per stage
- **Raw counts are sacred** - always preserve `layers["counts"]` before any normalization; downstream methods (scVI, DESeq2) need raw
- **Benchmarking > novelty bias** - new preprint methods often fail to outperform established ones; check scIB/pipeComp first
- **Pseudobulk is the default for DGE** - per-cell DE inflates false positives via pseudoreplication; aggregate to sample × cell type, then run bulk DE tools
- **Annotation is multi-tool** - run CellTypist + marker dotplot + scArches; agreement = high confidence, disagreement = investigate
- **Imputation is optional and dangerous** - SAVER/MAGIC/scOpen can introduce artifacts; only impute for visualization, never for DE or clustering

## Anti-patterns
- **Trusting per-cell DE for condition comparison** - pseudoreplication; always pseudobulk
- **Skipping ambient RNA correction** - 10% contamination creates spurious "expressed everywhere" genes
- **Using one doublet detector only** - scDblFinder + marker-based (ADT) or scDblFinder + AMULET (ATAC) for orthogonal coverage
- **Choosing UMAP over t-SNE for global structure** - UMAP doesn't preserve global distances better than t-SNE with same initialization; both are visualization tools, not analysis
- **Running SCENIC across batches without integration** - co-expression confounded by batch; run per-batch or on integrated data
- **Trusting cell type annotation without marker validation** - always check CellTypist output against canonical markers (CD3=T, CD19=B)
- **Using CLR for ADT when dsb is possible** - dsb strictly better with empty droplets + isotype controls

## Reference Tables

### scRNA-seq Pipeline Tools by Stage

| Stage | Python (scverse) | R (Seurat/Bioconductor) | When to pick |
|---|---|---|---|
| FASTQ → counts | kb-python, alevin-fry, STARsolo | cellranger, rsem | kallisto/bustools fastest; cellranger standard |
| Ambient RNA | CellBender | SoupX, DecontX | CellBender if GPU; SoupX if quick |
| Doublet detection | Scrublet, scDblFinder (via rpy2) | scDblFinder, DoubletFinder | scDblFinder preferred |
| QC metrics | scanpy.pp.calculate_qc_metrics | scater::perCellQCMetrics | equivalent |
| Normalization | scanpy (log1p), scran, scvi-tools | scran, scater, sctransform | match to distribution |
| HVG | scanpy (seurat_v3, cellranger, deviance) | Seurat (vst), M3Drop | seurat_v3 default |
| Batch correction | scVI, scANVI, Harmony, BBKNN | Harmony, Seurat anchors, RSEC | scVI for raw counts; Harmony for speed |
| Dimensionality | scanpy PCA, UMAP, PAGA | Seurat PCA, UMAP, Monocle3 | standard |
| Clustering | Leiden (igraph) | Louvain (Seurat v3), Leiden (Seurat v4) | Leiden preferred |
| Annotation | CellTypist, scArches | SingleR, scPred, clustermole | CellTypist for immune |
| DGE | pseudobulk + PyDESeq2, decoupler | muscat + edgeR/DESeq2, MAST | pseudobulk always |
| Trajectory | scVelo, Palantir, CellRank | Monocle3, Slingshot | match biology |
| CCC | LIANA, CellPhoneDB, NicheNet | CellChat, NATMI | run consensus |
| GRN | pySCENIC, celloracle | SCENIC (R), SCINET | SCENIC standard |
| CNV | infercnvpy, Numbat | inferCNV, HoneyBADGER | Numbat allele-aware |
| Visualization | scanpy.pl, squidpy | Seurat DimPlot, scater | match ecosystem |

### DGE Method Trade-offs

| Method | Input | Pros | Cons |
|---|---|---|---|
| Pseudobulk + DESeq2 | sample × cell type aggregated counts | Handles biological replicates properly; gold standard | Loses single-cell resolution |
| MAST | per-cell counts + hurdle model | Single-cell aware; handles dropout | Slower; pseudoreplication risk |
| NEBULA | per-cell, mixed model | Random effects for samples | Complex setup |
| muscat | pseudobulk, multi-condition | Designed for SC; methods dispatch | R-only |
| scVI-DE | per-cell counts + latent | Conditions in latent space | Slow; black box |

### Integration Method Selection (scRNA-seq)

| Scenario | Recommended | Why |
|---|---|---|
| Multi-batch raw counts | scVI/scANVI | Conditional VAE handles ZINB |
| Need fast PCA correction | Harmony | Iterative soft clustering + linear |
| Graph-only correction | BBKNN | Simple, fast |
| Reference mapping | scArches | Train scVI on ref, query transfer |
| Cross-species / cross-modality | Seurat anchors (CCA+MNN) | Mature, well-benchmarked |
| Many datasets, atlas-scale | Scanorama, Liger | Panorama / NMF scaling |

## Worked Example

**Scenario**: 5-donor PBMC scRNA-seq, 10x Chromium v3, want to compare IFN-β stimulation response.

**Tool selection walkthrough**:
1. **Preprocessing**: `kb-python` for FASTQ → counts (faster than cellranger); preserve raw in `layers["counts"]`
2. **Ambient RNA**: `CellBender` (GPU available) or `SoupX` (faster); 10% typical for PBMC
3. **Doublet detection**: `scDblFinder` (per-sample, then concat); ~5-10% doublet rate expected
4. **QC**: per-sample 5-MAD filter on `log1p_total_counts`, `log1p_n_genes_by_counts`, `pct_counts_mt` (3 MAD + 8% cap)
5. **Normalization**: `X` = log1p(normalize_total(1e4)) for visualization; `layers["scran"]` for DE; Pearson residuals for HVG
6. **HVG**: seurat_v3 on raw counts, top 2000
7. **Integration**: `scVI` (raw counts, `batch_key="donor"`); `Harmony` as fallback if no GPU
8. **Clustering**: Leiden resolution 0.1 → 0.5 for sub-types
9. **Annotation**: `CellTypist` (Immune_All_Low model) + marker dotplot (CD3, CD19, CD14, CD56) + scArches if reference exists
10. **DGE**: `decoupler.pp.pseudobulk` per donor × cell type → `PyDESeq2` with design `~ condition` per cell type
11. **Compositional**: `scCODA` with reference cell type (smallest log-fold change)
12. **Optional**: `LIANA` for CCC, `scGEN` for perturbation prediction

## Key Takeaways
1. Pick tools per pipeline stage, not one monolithic suite - the ecosystem rewards specialization
2. Preserve `layers["counts"]` from the start - downstream methods (scVI, DESeq2) need raw counts
3. Pseudobulk is the default for DGE - per-cell DE inflates false positives via pseudoreplication
4. Benchmark before adopting novel methods - scIB, pipeComp, "Confronting false discoveries" papers are your friends
5. Cell type annotation is multi-tool - CellTypist + markers + scArches; agreement = confidence
6. Ambient RNA + doublet correction are non-negotiable - 10% contamination and 5% doublets are typical

## Connects To
- **Ch 02**: Preprocessing fundamentals (this chapter lists the tools; Ch 02 explains the why)
- **Ch 03**: Cellular structure (clustering + annotation methods rationale)
- **Ch 05**: Conditions (DGE method trade-offs in depth)
- **Ch 06**: Mechanism modeling (SCENIC, CCC tool selection)
- **Ch 11**: AIRR (scirpy extends scverse for immune receptors)
- **Ch 17**: Cross-modality resources (seandavi's curated list overlaps)
