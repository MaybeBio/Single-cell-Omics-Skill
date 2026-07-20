# Chapter 14: scATAC-seq Tools Ecosystem

## Core Idea
scATAC-seq analysis diverges from scRNA-seq at every stage: features are genomic regions (peaks or bins) not genes, counts are ultra-sparse (1-10% of regions detected per cell, max count 2 per diploid position), and the dominant signal is binary (open vs closed). The tool ecosystem reflects this - ArchR and Signac dominate end-to-end analysis, SnapATAC2 leads on scalability, cisTopic provides LDA-based topic modeling, and integration with scRNA-seq is a first-class concern (label transfer, joint visualization).

## Frameworks Introduced

- **End-to-end suite choice: ArchR vs Signac vs SnapATAC2**
  - When to use: starting scATAC analysis
  - How: ArchR (R, HDF5-backed, >1M cells, fastest, most features); Signac (R, Seurat extension, easiest if already using Seurat); SnapATAC2 (Rust+Python, fastest for >1M cells, matrix-free spectral clustering)
  - Why: choose based on dataset size and existing ecosystem; ArchR is the default recommendation

- **Feature strategy: peaks vs bins vs topics**
  - When to use: after QC, before clustering
  - How:
    1. **Bins** (SnapATAC, ArchR default): 500bp or 5kb uniform windows; unbiased but noisy
    2. **Peaks** (MACS2 on aggregated, Signac): biologically meaningful but may miss cell-type-specific peaks
    3. **Topics** (cisTopic LDA): probabilistic, captures co-accessibility patterns; best for rare cell types
  - Why: bins let you cluster first then call cluster-specific peaks; topics model uncertainty

- **TF activity inference: chromVAR vs motif enrichment**
  - When to use: after clustering, to identify regulatory drivers
  - How: chromVAR computes deviation scores per cell per motif (background = all peaks with motif); motif enrichment (HOMER, MEME) is cluster-level; for paired RNA+ATAC use FigR or SCENIC+
  - Why: chromVAR gives per-cell TF activity (good for clustering); motif enrichment is post-hoc interpretation

- **scATAC + scRNA integration: label transfer**
  - When to use: paired (10x Multiome) or unpaired (separate ATAC + RNA on similar sample)
  - How: ArchR/Signac both implement label transfer via shared latent space (LSI for ATAC, PCA for RNA, mutual nearest neighbors); for unpaired, scAI or GLUE
  - Why: ATAC clusters are hard to annotate from motifs alone; RNA labels transfer well

## Key Concepts
- **Tn5 transposase**: inserts adapters into open chromatin; two insertions = one fragment; bias toward TSS, promoters, enhancers
- **Fragment vs read**: fragment = paired-end read pair; ATAC counts use fragments (or read ends), not reads
- **Peak**: genomic region with significantly enriched fragment counts vs background; called via MACS2 on aggregated data
- **Bin**: uniform genomic window (500bp, 5kb, 10kb); alternative to peaks for unbiased clustering
- **Topic (cisTopic)**: LDA latent dimension summarizing co-accessibility; analog of "gene module" for ATAC
- **TSS enrichment**: TSS-centered vs TSS-flanking fragment ratio; canonical ATAC signal-to-noise metric (cutoff >2)
- **Nucleosome signal**: mono-nucleosomal (~200bp) / nucleosome-free (<100bp) fragment ratio; <4 = good
- **FRiP (Fraction of Reads in Peaks)**: % reads in called peaks; >0.15 = good
- **LSI (Latent Semantic Indexing)**: TF-IDF + SVD on peak × cell matrix; analog of PCA for ATAC; skip component 1 (depth-correlated)
- **DORC (Domain of Regulatory Chromatin)**: gene + correlated accessible peaks within ~200kb; used by FigR for GRN
- **chromVAR deviation score**: per-cell TF activity = observed/expected motif accessibility; controls for locus-specific bias
- **Doublet detection**: AMULET (read-pair distance, catches homotypic) + scDblFinder (simulated, catches heterotypic)

## Mental Models
- **ATAC is binary at single-cell level** - max count 2 per diploid position; binarization loses little information but stabilizes clustering
- **LSI replaces PCA** - TF-IDF normalization + SVD on the binarized matrix; first component is technical (sequencing depth), drop it
- **Bins for clustering, peaks for interpretation** - cluster on bins (unbiased), then call cluster-specific peaks for biological interpretation
- **Integration > annotation from motifs** - ATAC clusters are hard to label from motif enrichment alone; transfer labels from paired/unpaired RNA
- **Topic modeling handles sparsity** - LDA on binarized ATAC = soft clustering of cells by "topics" (co-accessibility programs)
- **Imputation is the sparsity solution but use with care** - scOpen, scImpute smooth the matrix; improves clustering but can introduce artifacts; never impute before differential accessibility

## Anti-patterns
- **Using scRNA-seq normalization on ATAC** - log1p(normalize_total) collapses the dynamic range; use TF-IDF
- **Keeping LSI component 1** - it's sequencing depth, not biology; start from component 2
- **Filtering on total_counts alone** - high counts can be doublets; use TSS enrichment + nucleosome signal jointly
- **Calling peaks per cell** - too sparse; aggregate first (per cluster or all cells), then call peaks
- **Skipping AMULET for doublet detection** - scDblFinder misses homotypic doublets; AMULET catches both
- **Trusting motif enrichment as TF activity** - motif presence ≠ TF activity; use chromVAR for per-cell deviation scores
- **Computing peak-gene correlation without smoothing** - per-cell peak × gene correlation is too noisy; smooth via kNN (FigR) or aggregate per pseudobulk

## Reference Tables

### scATAC-seq Suite Comparison

| Suite | Language | Scalability | Strength | Limitation |
|---|---|---|---|---|
| ArchR | R | >1M cells (HDF5) | Most features, fastest, doublet detection | R-only; steep learning curve |
| Signac | R (Seurat) | ~500k cells | Easy if Seurat user; CITE-seq integration | Slower than ArchR |
| SnapATAC2 | Rust+Python | >1M cells, fastest | Matrix-free spectral clustering | Newer; smaller community |
| cisTopic | R | <500k cells | LDA topic modeling; great for rare types | Slower; topic selection finicky |
| EpiScanpy | Python | <500k cells | scverse-compatible | Less mature than ArchR/Signac |
| ChromSCape | R/Shiny | <500k cells | Interactive UI; good for exploration | Not scriptable for production |

### Pipeline Stage Tools

| Stage | Recommended | Alternatives |
|---|---|---|
| FASTQ → fragments | cellranger-atac, bwa+samtools | SnapTools, sinto |
| QC metrics | ArchR (`addQCParams`), Signac (`CreateChromatinAssay`) | scater, custom |
| Doublet detection | AMULET + scDblFinder | ArchR doublet (simulated only) |
| Feature matrix | ArchR (bins/peaks), Signac (peaks), SnapATAC2 (bins) | cicero, cisTopic |
| Normalization | TF-IDF + LSI | scOpen (imputation), scran |
| Dimensionality | LSI (skip comp 1), UMAP | cisTopic LDA, scHICluster |
| Clustering | Leiden on LSI | k-means, hierarchical |
| TF activity | chromVAR | motif enrichment (HOMER, MEME) |
| Peak-gene linkage | FigR (paired), Cicero (covariance) | ArchR `getPeak2GeneLinkage` |
| GRN inference | FigR (paired RNA+ATAC), SCENIC+ | Pando, celloracle |
| Integration with RNA | ArchR/Signac label transfer | scAI, GLUE (unpaired) |
| Differential accessibility | Signac `FindMarkers`, ArchR `getMarkerFeatures` | edgeR, DESeq2 on pseudobulk |

### QC Thresholds (10x scATAC)

| Metric | Lower | Upper | Reason |
|---|---|---|---|
| `n_fragment_per_cell` | 1,000 | 100,000 | Below = empty; above = doublet |
| `tss_enrichment_score` | 2 | (none) | Signal-to-noise |
| `nucleosome_signal` | (none) | 4 | Mono-nucleosome quality |
| `reads_in_peaks_frac` | 0.15 | (none) | Signal enrichment |
| `reads_in_blacklist_frac` | (none) | 0.05 | Artifactual signal |

### scATAC + scRNA Integration Methods

| Method | Paired? | Approach | Use Case |
|---|---|---|---|
| ArchR label transfer | Either | LSI ↔ PCA, mutual NN | Standard for 10x Multiome |
| Signac WNN | Yes | Weighted nearest neighbors | Joint clustering |
| scAI | Either | Matrix factorization | Same-cell multi-omics |
| GLUE | No | Graph-linked VAE, needs GRN prior | Unpaired RNA+ATAC |
| MAESTRO | Either | CCA-based | End-to-end pipeline |
| UnionCom | No | Topological alignment | Unmatched features |
| MOFA+ | Yes | Factor analysis | Multi-omics factors |

## Worked Example

**Scenario**: 10x Multiome PBMC (paired RNA + ATAC, 10 donors), want to identify TFs driving cell-type-specific regulatory states.

**Tool selection walkthrough**:
1. **Preprocessing**: `cellranger-arc` for FASTQ → fragments + counts; load into `ArchR` (faster) or `Signac` (if Seurat user)
2. **QC (ATAC)**: filter on `n_fragment` 1k-100k, `tss_enrichment` >2, `nucleosome_signal` <4, `reads_in_peaks_frac` >0.15
3. **QC (RNA)**: standard scRNA-seq QC (Ch 02)
4. **Doublet detection**: AMULET (ATAC, catches homotypic) + scDblFinder (RNA, simulated); remove cells flagged by either
5. **Feature matrix (ATAC)**: bins 500bp for clustering, then call peaks per cluster via MACS2
6. **Normalization (ATAC)**: TF-IDF + LSI, keep components 2:30
7. **Clustering**: Leiden on LSI components; annotate via RNA label transfer (paired data)
8. **TF activity**: `chromVAR` for per-cell deviation scores; aggregate per cluster
9. **GRN inference**: `FigR` (paired) - cisTopic LDA → DORC scores → ChromVAR motifs → TF-DORC correlations; output = TF × target gene regulatory scores
10. **Differential accessibility**: pseudobulk per donor × cell type → edgeR or DESeq2
11. **Optional**: SCENIC+ for end-to-end GRN; ArchR `getPeak2GeneLinkage` for cis-regulatory mapping

## Key Takeaways
1. Pick ArchR for production pipelines (>1M cells, most features), Signac if Seurat user, SnapATAC2 for >5M cells
2. TF-IDF + LSI replaces log1p + PCA; drop LSI component 1 (it's sequencing depth)
3. Bin first, peak later - cluster on unbiased bins, call cluster-specific peaks for interpretation
4. Always integrate with RNA - ATAC clusters are hard to annotate from motifs alone
5. chromVAR for per-cell TF activity, motif enrichment for cluster interpretation
6. AMULET + scDblFinder for orthogonal doublet detection (homotypic + heterotypic)
7. For paired RNA+ATAC, FigR is the standard GRN method; SCENIC+ for end-to-end

## Connects To
- **Ch 08**: Chromatin accessibility (theoretical foundations; this chapter lists tools)
- **Ch 13**: scRNA-seq tools (label transfer, integration methods)
- **Ch 12**: Multimodal integration (WNN, scAI, GLUE in depth)
- **Ch 15**: scHi-C tools (similar pipeline structure for 3D genomics)
- **Ch 06**: Modeling mechanisms (GRN inference, FigR vs SCENIC)
