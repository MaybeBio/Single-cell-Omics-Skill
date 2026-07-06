# Chapter 8: Chromatin Accessibility (scATAC-seq)

## Core Idea
scATAC-seq measures chromatin accessibility per cell — an orthogonal layer to gene expression that captures the *regulatory state* (DNA methylation, histone mods, TF binding). The data is far sparser than scRNA-seq (1–10% of open regions detected per cell) and the maximum count per diploid position is 2, so the analysis pipeline diverges: feature definition (peaks vs bins), specialized QC metrics (TSS enrichment, nucleosome signal), and orthogonal doublet detection (simulated + coverage-based).

## Frameworks Introduced

- **Feature definition: peaks vs bins**
  - When to use: starting point of any scATAC analysis, before QC
  - How: either (a) call peaks on aggregated or cluster-specific regions, or (b) divide genome into uniform bins (SnapATAC, ArchR default) for clustering-first analysis
  - Why: gene-level features don't exist for ATAC (most reads are non-coding); bin strategy avoids hiding cell-type-specific peaks behind global peak calls

- **Three counting strategies**
  1. **Reads overlapping feature** (Cell Ranger ATAC) — produces parity artifacts (many even counts)
  2. **Fragments overlapping feature** (Signac) — no parity artifact
  3. **Binarized read ends** (ArchR default) — maximally robust to sparsity, loses quantitative info
  - Choice still under debate; match to research question

- **Four-metric QC panel**
  1. `total_fragment_counts` — sequencing depth (analog of `total_counts` in scRNA)
  2. `tss_enrichment` — TSS-centered vs TSS-flanking fragment ratio; signal-to-noise
  3. `n_features_per_cell` — peaks with non-zero counts (analog of `n_genes_by_counts`)
  4. `nucleosome_signal` — mono-nucleosomal / nucleosome-free fragment ratio; quality of transposition
  - Filter on upper AND lower bounds; lower bounds typically define the empty-droplet / low-quality cluster

- **Two orthogonal doublet-detection methods**
  1. **scDblFinder with `aggregateFeatures=TRUE`** — simulated doublets on aggregated meta-features (25 meta-features recommended); detects heterotypic doublets
  2. **AMULET** — counts genomic positions with >2 overlapping fragments (impossible for singlet diploid cell); detects both heterotypic AND homotypic doublets; requires ≥10–15k reads per cell
  - Use both — top-right quadrant of scatter (high both scores) = confident doublets; AMULET-only hits = homotypic doublets

- **DORC + FigR GRN inference** (paired RNA+ATAC)
  - When to use: 10x Multiome or other paired RNA+ATAC data, to infer TF-driven gene regulation
  - How:
    1. Run cisTopic LDA on ATAC to get cell-topic matrix → kNN graph
    2. `runGenePeakcorr` — correlate every peak with every gene within 200 kbp; filter by p-value
    3. Group peaks around each gene into a DORC; compute DORC score per cell
    4. Smooth DORC scores and RNA via cell kNN (k=30)
    5. ChromVAR maps TF motifs to peaks; `runFigRGRN` correlates TF expression with DORC scores
  - Output: TF × target-gene regulatory scores; TF activator = high motif enrichment + positive TF-DORC correlation; TF repressor = high motif + negative correlation

## Key Concepts
- **Chromatin accessibility**: open chromatin regions where TFs and machinery can bind; proxy for active regulatory state
- **Tn5 transposase**: enzyme that inserts adapters into open chromatin; two insertions = one sequenced fragment
- **Peak**: genomic region with significantly elevated fragment counts vs background
- **Topic (cisTopic)**: latent Dirichlet allocation topic summarizing co-accessibility patterns across cells
- **DORC (Domain of Regulatory Chromatin)**: gene + its collection of correlated accessible peaks within ~200 kbp
- **Nucleosome signal**: ratio of mono-nucleosomal (~200 bp) to nucleosome-free (<100 bp) fragments; high signal = degraded/low-quality
- **TSS enrichment**: normalized fragment density at TSS vs flanks; the canonical ATAC signal-to-noise metric
- **Blacklist regions**: ENCODE-defined regions with artifactual signal; usually not problematic at single-cell level
- **Heterotypic vs homotypic doublets**: heterotypic = different cell types (detectable from counts); homotypic = same cell type (only AMULET can catch)

## Mental Models
- **ATAC is orthogonal to RNA, not redundant** — same cell type can have similar RNA but different regulatory state; accessibility reveals *what could be expressed*, not what is
- **Bin first, peak later** — uniform bins let you cluster cells without peak-calling bias, then call cluster-specific peaks
- **Binarization trades sensitivity for stability** — appropriate for clustering, not for differential accessibility
- **Filter cells on multiple metrics jointly** — a low-fragment cell with high TSS score may be a real rare cell; a low-fragment cell with low TSS is junk
- **DORC aggregation denoises peak-gene correlations** — single peak-gene correlations are noisy; summing across a DORC reveals the regulatory relationship
- **TF activator vs repressor: motif + correlation sign** — motif enrichment alone is not regulation; need both motif presence AND TF-expression ↔ DORC-score correlation direction

## Anti-patterns
- **Applying scRNA-seq doublet methods directly to scATAC** — sparsity is too high; aggregate features first or use AMULET
- **Running AMULET on shallow data (<10–15k reads/cell)** — coverage-based test lacks power; false negatives
- **Excluding chrM/chrX/chrY/repeats from AMULET** — violates diploid assumption (mtDNA has many copies, sex chromosomes don't fit the 2-copy model, repeats cause mapping ambiguity)
- **Calling peaks on all cells before clustering** — hides cell-type-specific peaks and rare-population peaks
- **Filtering cells on `total_fragment_counts` alone** — depth and quality are different axes; a deep low-quality cell still fails TSS
- **Using `reads_in_peaks_frac` as primary QC** — confounded by peak-calling quality; TSS enrichment is more independent
- **Trusting a single counting strategy without checking parity** — read counting produces artificial even-count peaks; verify with fragment counting
- **Forgetting `var_names_make_unique`** — peak identifiers often collide; silent downstream errors

## Code Examples

### QC metric calculation (muon.atac)

```python
import muon as mu
from muon import atac as ac

mdata = mu.read_10x_h5("cellranger_out/filtered_feature_bc_matrix.h5")
mdata.var_names_make_unique()
atac = mdata.mod["atac"]

# Generic metrics via scanpy (renamed for ATAC context)
sc.pp.calculate_qc_metrics(atac, percent_top=None, log1p=False, inplace=True)
atac.obs.rename(columns={
    "n_genes_by_counts": "n_features_per_cell",
    "total_counts": "total_fragment_counts",
}, inplace=True)
atac.obs["log_total_fragment_counts"] = np.log10(atac.obs["total_fragment_counts"])

# ATAC-specific metrics
ac.tl.nucleosome_signal(atac, n=10e3 * atac.n_obs)  # 10x downsample for speed
tss = ac.tl.tss_enrichment(mdata, n_tss=3000, random_state=666)
```

- **What it demonstrates**: the four-metric QC panel. `n=10e3*n_obs` is a 10× downsample of the default `10e4*n_obs` for speed; restore default in production.

### Threshold-based filtering (muon)

```python
mu.pp.filter_obs(atac, "total_fragment_counts", lambda x: (x >= 1500) & (x <= 100000))
mu.pp.filter_obs(atac, "n_features_per_cell", lambda x: x >= 750)
mu.pp.filter_obs(atac, "tss_score", lambda x: (x >= 1.5) & (x <= 50))
mu.pp.filter_obs(atac, "nucleosome_signal", lambda x: x <= 2)
# Rare-cell-aware feature filter (2% population × 5% detection ≈ 15 cells in 15k dataset)
mu.pp.filter_var(atac, "n_cells_by_counts", lambda x: x >= 15)
atac.layers["counts"] = atac.X
```

### AMULET coverage-based doublet detection (rpy2)

```R
%%R
suppressPackageStartupMessages(library(GenomicRanges); library(rtracklayer))
repeats <- import('resources/blacklist_repeats_segdups_rmsk_hg38.bed')
otherChroms <- GRanges(c("chrM","chrX","chrY","MT"), IRanges(1L, width=10^8))
toExclude <- suppressWarnings(c(repeats, otherChroms))
```
```python
frag_path = atac.uns["files"]["fragments"]
%R -i frag_path -o amulet_result amulet_result <- amulet(frag_path, regionsToExclude=toExclude)
atac.obs["AMULET_negLog10qVal"] = -1 * np.log10(amulet_result["q.value"])
```

- **What it demonstrates**: AMULET requires explicit exclusion of mtDNA, sex chromosomes, and repeats. Runtime ~4 hours for 16k cells on a laptop — plan accordingly. Output `q.value` tests the null hypothesis that a cell is a singlet.

### FigR DORC + GRN pipeline

```R
library(FigR); library(cisTopic); library(BSgenome.Hsapiens.UCSC.hg38)
# 1. Peak-gene correlations within 200 kbp
cisCorr <- FigR::runGenePeakcorr(ATAC.se = ATAC.se, RNAmat = RNAmat,
    genome = "hg38", nCores = 2, p.cut = NULL, n_bg = 250)
cisCorr.filt <- cisCorr %>% filter(pvalZ <= 0.05)

# 2. DORC scores (aggregate peaks per gene)
dorcGenes <- cisCorr.filt %>% dorcJPlot(cutoff=2, returnGeneList=TRUE)
dorcMat <- getDORCScores(ATAC.se, dorcTab=cisCorr.filt,
                          geneList=dorcGenes, nCores=2)
dorcMat.s <- smoothScoresNN(NNmat = cellkNN[,1:30], mat = dorcMat, nCores=2)
RNAmat.s  <- smoothScoresNN(NNmat = cellkNN[,1:30], mat = RNAmat,  nCores=2)

# 3. TF-DORC regulatory network
figR.d <- runFigRGRN(ATAC.se=ATAC.se, dorcTab=cisCorr.filt, genome="hg38",
                     dorcMat=dorcMat.s, rnaMat=RNAmat.s, nCores=2)
```

- **What it demonstrates**: the full FigR chain. Smoothing via cell kNN (k=30 from cisTopic) is essential — both DORC scores and RNA are too sparse per-cell for direct correlation. `dorcJPlot(cutoff=2)` requires ≥30 DORC genes to proceed; lower the cutoff if needed.

## Reference Tables

### scATAC QC thresholds (worked example, NeurIPS 2021 BMMC s4d8)

| Metric | Lower bound | Upper bound | Rationale |
|---|---|---|---|
| `total_fragment_counts` | 1,500 | 100,000 | Below: empty droplet / low-quality cluster; above: doublet suspect |
| `n_features_per_cell` | 750 | (none) | Linearly correlated with fragment count; 750 ≈ fragment-count 1500 |
| `tss_score` | 1.5 | 50 | Below: noise; above: artifact (often low-fragment outliers) |
| `nucleosome_signal` | (none) | 2 | Above: degraded chromatin |
| Feature `n_cells_by_counts` | 15 (≈1% of 15k) | (none) | Rare-state-aware: 2% population × 5% detection ≈ 15 cells |

### Doublet-detection method comparison

| Method | Detects | Min depth | Runtime (16k cells) | Independent of count method |
|---|---|---|---|---|
| scDblFinder (simulated, aggregated features) | Heterotypic | low | minutes | yes (post-aggregation) |
| AMULET (coverage-based) | Heterotypic + homotypic | 10–15k reads/cell | ~4 hours | yes |

### FigR outputs

| Output | What it tells you |
|---|---|
| `cisCorr` table | Per peak-gene pair: correlation, p-value, distance |
| `dorcGenes` | Genes with ≥2 correlated peaks (DORC cutoff) |
| `dorcMat.s` | Smoothed DORC score per cell × gene |
| `figR.d` (TF × target) | `Corr.log10P` (TF RNA ↔ DORC), `Enrichment.log10P` (motif enrichment), `Score` (combined) — upper-right = activator, upper-left = repressor |

## Worked Example

**NeurIPS 2021 10x Multiome BMMC** — paired RNA + ATAC from bone marrow mononuclear cells, 12 donors × 4 sites. The chapter demonstrates per-sample processing on donor 8 sample s4d8 (16,934 cells × 154,027 ATAC peaks).

Pipeline:
1. `mu.read_10x_h5` loads both modalities into a `MuData` object; `atac.uns` auto-populates with peak annotation + fragment file path
2. Calculate 4 QC metrics; identify upper/lower thresholds via jointplot of `log_total_fragment_counts` vs `tss_score` (the bottom-left cluster = empty droplets)
3. Apply 4-metric filter; expect to lose ~10–20% of barcodes
4. Filter features with rare-state-aware cutoff (15 cells)
5. Run scDblFinder (simulated doublets, 25 aggregated meta-features, cluster-based) → ~18% doublet rate (reasonable for nuclei)
6. Run AMULET (exclude chrM/chrX/chrY/repeats; ~4 hours) → q-values per barcode
7. Scatter `scDblFinder_score` vs `AMULET_negLog10qVal`; top-right = confident doublets; AMULET-only = homotypic
8. Save filtered ATAC object with `layers["counts"]`

For GRN inference (donor s1d1):
1. Subset to one donor; split RNA and ATAC into separate SCE objects
2. Select 10,000 RNA features (all TFs + sampled non-TFs) and 10,000 ATAC peaks (coefficient-of-variation criterion)
3. Run cisTopic LDA (1–25 topics); pick model with maximum log-likelihood; UMAP on cell-topic matrix → kNN graph (k=30)
4. `runGenePeakcorr`: correlate peaks with genes within 200 kbp; filter `pvalZ ≤ 0.05`
5. `dorcJPlot(cutoff=2)` → list of DORC genes (require ≥30 to proceed)
6. `getDORCScores` → per-cell DORC matrix; smooth via kNN
7. `runFigRGRN` → TF × target regulatory scores
8. Visualize: `plotMarker2D` (RNA vs DORC UMAP), `rankDrivers` (top TFs by mean score or nTargets), `plotfigRHeatmap` (TF-target matrix), `plotfigRNetwork` (interactive D3)

The DORC concept is the key innovation — instead of trusting individual peak-gene correlations (noisy at single-cell resolution), aggregate signal across the regulatory domain around each gene. The smoothed DORC score then has enough signal-to-noise to correlate with TF RNA expression.

## Key Takeaways
1. **Define features before QC** — peaks (cluster-specific) or bins (uniform); avoid global peak calls that hide cell-type-specific accessibility
2. **Use four QC metrics jointly** — `total_fragment_counts`, `tss_enrichment`, `n_features_per_cell`, `nucleosome_signal`; filter on upper AND lower bounds
3. **Two orthogonal doublet methods, always** — scDblFinder (simulated) + AMULET (coverage); AMULET needs ≥10–15k reads/cell and excludes mt/sex/repeats
4. **AMULET catches homotypic doublets that count-based methods miss** — but only with sufficient sequencing depth
5. **For paired RNA+ATAC, use FigR + cisTopic** — DORC aggregation + kNN smoothing converts sparse peak-gene correlations into robust TF-target regulatory scores
6. **TF activator vs repressor: motif enrichment + correlation sign** — upper-right of the FigR scatter = activator; upper-left = repressor
7. **Smooth before correlating** — both DORC scores and RNA need kNN smoothing (k=30) before per-cell correlation; raw single-cell values are too sparse
8. **Rare cell types require rare-state-aware feature filter** — 1% of cells is a reasonable floor if you care about rare populations; otherwise filter features to ≥1% of cells

## Connects To
- **Ch 1 (MuData)**: scATAC + scRNA paired data lives in a `MuData` object with `mod["atac"]` and `mod["rna"]` slots
- **Ch 2 (Preprocessing)**: parallel QC logic — MAD-based filtering philosophy carries over, but with ATAC-specific metrics
- **Ch 6 (GRNs from RNA-only)**: SCENIC+ and FigR extend RNA-only GRN inference by adding chromatin accessibility constraints
- **Ch 12 (Multimodal integration)**: paired RNA+ATAC integration builds on the per-modality preprocessing established here
- **Ch 9 (Spatial omics)**: spatial ATAC uses similar peak/bin feature definitions and TSS enrichment QC
