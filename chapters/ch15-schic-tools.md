# Chapter 15: scHi-C Tools Ecosystem

## Core Idea
scHi-C measures 3D chromatin contacts per cell - the sparsest single-cell modality (often <50k contacts per cell vs ~10k genes detected in scRNA). Tools organize by pipeline stage like other modalities: normalization (BandNorm, scHiCNorm), imputation (Higashi, scHiCluster), clustering (Fast-Higashi, scHiCTools), compartment detection (scGHOST), loop calling (SnapHiC, scGSLoop), TAD detection (DeDoc2, scKTLD), and 3D structure reconstruction (DPDchrom, hickit). The ecosystem is younger than scRNA/scATAC and dominated by the Ma lab (Higashi, Fast-Higashi, scGHOST) and Zhang lab (BandNorm, 3DVI).

## Frameworks Introduced

- **Distance-centric normalization (BandNorm)**
  - When to use: before clustering or downstream analysis
  - How: normalize contacts by their genomic distance band (contact frequency decays with distance); regresses out the dominant distance-decay signal
  - Why: bulk Hi-C normalization (ICE, KR) underperforms on sparse scHi-C; band normalization captures the per-cell variation in distance-decay

- **Imputation before analysis (Higashi, scHiCluster)**
  - When to use: always - scHi-C is too sparse for direct clustering
  - How: Higasha (hypergraph representation learning, borrows from kNN cells); scHiCluster (linear convolution + random walk); both impute missing contacts
  - Why: >5k contacts per cell is "sufficient" but raw contacts are too sparse for clustering; imputation reveals structure

- **Tensor decomposition for clustering (Fast-Higashi)**
  - When to use: rare cell type identification or trajectory inference
  - How: PARAFAC2 tensor decomposition on chromosome-specific 3-way tensors (cell × bin × bin); partial random walk imputation
  - Why: tensor structure preserves multi-way contact information; faster than Higashi with similar accuracy

- **Subcompartment detection (scGHOST)**
  - When to use: after clustering, for regulatory state interpretation
  - How: Higashi embeddings + random walks + graph clustering → scA1, scA2, scB1, scB2, scB3 subcompartments per cell
  - Why: bulk Hi-C subcompartments (Ren et al. 2017) are population averages; scGHOST reveals cell-to-cell subcompartment variation

## Key Concepts
- **3C/Hi-C/scHi-C**: chromosome conformation capture; Hi-C = genome-wide; scHi-C = single-cell
- **Contact matrix**: bins × bins per cell; extremely sparse (most entries 0)
- **Distance decay**: contact frequency decreases with genomic distance; the dominant signal to normalize out
- **A/B compartment**: active (A) vs inactive (B) chromatin; identified via eigenvector decomposition
- **TAD (Topologically Associating Domain)**: ~1Mb region with elevated intra-domain contacts; dynamic in single cells
- **Loop**: focal long-range contact between two loci (e.g., enhancer-promoter); CTCF-mediated
- **Subcompartment**: finer than A/B (A1, A2, B1, B2, B3); associated with different epigenomic signatures
- **Imputation**: smoothing the sparse contact matrix by borrowing from similar cells (kNN) or neighbors (convolution)
- **Haplotype-resolved scHi-C**: Dip-C, snHi-C; separates maternal/paternal contacts
- **Multi-way interaction**: scSPRITE, ChIA-Drop; captures >2-locus contacts (not just pairwise)
- **Dip-C**: diploid chromatin conformation capture; haplotype-resolved; ~5x more contacts than bulk scHi-C

## Mental Models
- **scHi-C is sparser than scATAC** - <50k contacts per cell, max 1 contact per bin pair; imputation is mandatory
- **Distance decay dominates** - normalize it out (BandNorm) before any clustering or differential analysis
- **TADs are dynamic in single cells** - bulk TADs are averages; per-cell TAD-like structures exist but are heterogeneous
- **Bulk methods fail on single cells** - ICE/KR normalization, HiCCUPS loop calling need depth; use single-cell-specific tools
- **Multi-omics scHi-C is mature** - sn-m3C-seq (methylation+3C), HiRES (RNA+3C), scHiCAR (ATAC+3C) all published
- **3D reconstruction is illustrative, not quantitative** - DPDchrom/hickit give nice visuals; ground truth is hard without FISH

## Anti-patterns
- **Using bulk Hi-C normalization (ICE, KR)** - decreases performance on scHi-C; use BandNorm or 3DVI
- **Skipping imputation** - raw scHi-C contacts are too sparse for clustering; always impute (Higashi or scHiCluster)
- **Calling loops per cell** - too sparse; aggregate per cluster first, then call loops (SnapHiC, scGSLoop)
- **Treating TADs as fixed** - per-cell TADs are highly variable; report population-level TADs + per-cell insulation scores
- **Forgetting haplotype** - diploid cells have maternal/paternal contacts; hickit handles phasing
- **3D structure as ground truth** - 3D-FISH validation needed; reconstruction is model-dependent
- **Filtering cells with <100k reads** - 5k contacts is enough for clustering after imputation (per scHiCluster paper)

## Reference Tables

### scHi-C Pipeline Stage Tools

| Stage | Recommended | Alternatives | Notes |
|---|---|---|---|
| FASTQ → contacts | hickit, NucProcess, scHiCExplorer | dip-c (Dip-C), SnapHiC | hickit for diploid/phasing |
| Normalization | BandNorm, 3DVI | scHiCNorm, scHiCluster | BandNorm = distance-centric |
| Imputation | Higashi (hypergraph) | scHiCluster (convolution+RW), scOpen | Higashi best for clustering |
| Clustering | Fast-Higashi (tensor) | Higashi, scHiCTools, scHiCluster, BandNorm | Fast-Higashi = scalable |
| Embedding | Fast-Higashi, Higashi | scHiCTools (MDS/t-SNE/PHATE), HiCRep+MDS | Higashi embeddings interpretable |
| Compartment | scGHOST (subcompartments) | eigenvector method, scHiCluster | scGHOST = finer subcompartments |
| Loop calling | SnapHiC, scGSLoop | HiCCUPS (bulk, not recommended) | SnapHiC2 = faster, 5kb res |
| TAD detection | DeDoc2, scKTLD | scHiCluster (TopDom), Higashi | DeDoc2 = no imputation needed |
| 3D reconstruction | DPDchrom, hickit | NucDynamics, Dip-C tools | hickit = diploid-aware |
| Differential contacts | SimpleDiff, diffHic | edgeR on pseudobulk | SimpleDiff = Wilcoxon on Z-scored |
| Simulation | scHi-CSim | (limited options) | For benchmarking |
| Multi-omics | scHiCAR (ATAC+Hi-C), HiRES (RNA+Hi-C) | sn-m3C-seq (methylation+Hi-C), Methyl-HiC | All experimental |
| Visualization | scHiCExplorer, HiGlass | juicebox, cooltools | HiGlass = interactive |

### scHi-C Suite Comparison

| Suite | Language | Strength | Best For |
|---|---|---|---|
| scHiCExplorer | Python | Full pipeline, documentation | New users |
| Higashi/Fast-Higashi | Python | Tensor decomposition, rare cell types | Clustering, trajectory |
| BandNorm/3DVI | R/Python | Normalization + denoising | Preprocessing |
| SnapHiC/SnapHiC2 | Python | Loop calling at 5-10kb | Regulatory inference |
| hickit | C | Diploid, 3D modeling | Phasing, structure |
| dip-c | C | Dip-C analysis | Haplotype-resolved |

### QC Thresholds (scHi-C)

| Metric | Threshold | Action |
|---|---|---|
| Uniquely mapped reads per cell | >5,000 (clustering) | Drop lower; sufficient after imputation |
| Uniquely mapped reads per cell | >50,000 (TADs) | Drop lower for TAD detection |
| Uniquely mapped reads per cell | >100,000 (loops) | Drop lower for loop calling |
| Cis/total ratio | >0.8 | Lower = junk contacts |
| Contact distance distribution | smooth decay | Sharp peaks = artifacts |

### scHi-C Technologies

| Technology | Key Feature | Cells | Throughput |
|---|---|---|---|
| snHi-C (Nagano 2013) | First scHi-C | ~10 cells | Low |
| sci-Hi-C (Ramani 2017) | Combinatorial indexing | >10k cells | High |
| Dip-C (Tan 2018) | Haplotype-resolved | ~50 cells | Medium |
| sn-m3C-seq | + DNA methylation | thousands | Medium |
| HiRES | + RNA-seq | thousands | Medium |
| scHiCAR | + ATAC-seq | thousands | Medium |
| scSPRITE | Multi-way interactions | >1k | Medium |
| MERFISH (imaging) | Direct visualization | hundreds | Low |

## Worked Example

**Scenario**: Mouse brain scHi-C (Lee 2019, sn-m3C-seq), ~3000 cells from prefrontal cortex, want to identify cell-type-specific 3D genome differences.

**Tool selection walkthrough**:
1. **Preprocessing**: `scHiCExplorer` for FASTQ → contact matrices (1Mb for clustering, 10kb for loops); filter cells with <5k uniquely mapped reads
2. **Normalization**: `BandNorm` (distance-centric) on 1Mb matrices
3. **Imputation**: `Higasha` (hypergraph) - imputes missing contacts using cell similarity
4. **Clustering**: `Fast-Higashi` (tensor decomposition) - identifies cell types (Astro, ODC, OPC, neurons by subtype)
5. **Subcompartment**: `scGHOST` on Higashi embeddings - per-cell scA1/scA2/scB1/scB2/scB3
6. **Differential contacts**: aggregate per cell type → `SimpleDiff` (Wilcoxon on Z-scored contacts) between e.g., neurons vs glia
7. **Loop calling**: `SnapHiC2` at 5kb on pseudobulk per cell type - identifies cell-type-specific loops
8. **TAD-like structures**: `DeDoc2` per cell (no imputation needed); report insulation score distributions per cell type
9. **Integration with methylation** (sn-m3C-seq): correlate methylation with contact frequency; differential boundaries ↔ differential methylation
10. **Optional 3D reconstruction**: `hickit` for diploid structure; validate key findings with MERFISH if available

## Key Takeaways
1. Always impute - scHi-C contacts are too sparse (<50k per cell) for direct analysis
2. BandNorm > ICE/KR for normalization - distance-centric is right for single cells
3. Fast-Higashi (tensor) for clustering and rare cell type detection
4. Bulk loop callers (HiCCUPS) fail on scHi-C - use SnapHiC2 or scGSLoop
5. TADs are dynamic per cell - report distributions, not single calls
6. Multi-omics scHi-C (sn-m3C-seq, HiRES, scHiCAR) is mature - integrate methylation/RNA/ATAC
7. 3D reconstruction is illustrative; validate with imaging (MERFISH, FISH)

## Connects To
- **Ch 14**: scATAC tools (similar pipeline structure; chromatin state)
- **Ch 12**: Multimodal integration (scHi-C + RNA, scHi-C + ATAC, scHi-C + methylation)
- **Ch 08**: Chromatin accessibility (chromatin state, TADs overlap with regulatory domains)
- **Ch 13**: scRNA-seq tools (HiRES = joint RNA+Hi-C)
- **Ch 06**: Modeling mechanisms (3D genome ↔ gene regulation)
