# Chapter 16: Spatial Omics Tools Ecosystem

## Core Idea
Spatial transcriptomics adds tissue coordinates to single-cell measurements, requiring new tools for spatially-aware analysis: SVG detection (Moran's I, nnSVG, SpatialDE), spatial domains (SpaGCN, STAGATE, BANKSY), deconvolution (Cell2location, RCTD, CARD), cell-cell communication in tissue context (NicheNet spatial, NICHES, stLearn), and 3D reconstruction. The field is fragmented across platforms (Visium, Xenium, MERFISH, Stereo-seq, Curio Seeker) each with platform-specific preprocessing, but downstream analysis converges on SpatialExperiment (Bioconductor) or SpatialData (scverse) data structures.

## Frameworks Introduced

- **Resolution tier determines tool choice**
  - When to use: starting spatial analysis
  - How:
    1. **Subcellular** (Xenium, CosMx, MERFISH, Stereo-seq, Visium HD): molecule coordinates → cell segmentation → per-cell counts
    2. **Single-cell** (Visium HD, Stereo-seq at cellular resolution): cells in tissue, no deconvolution needed
    3. **Spot-based** (Visium, Slide-seq, Curio Seeker): multi-cell spots → deconvolution required
  - Why: spot-based needs Cell2location/RCTD; subcellular needs segmentation (Baysor, Cellpose); single-cell tier uses scRNA-seq tools directly

- **Spatial vs non-spatial SVG detection**
  - When to use: identifying spatially variable genes
  - How: `squidpy.gr.spatial_autocorr(mode="moran")` (fast, Moran's I) for exploratory; `nnSVG` (nearest-neighbor GP, scalable) or `SpatialDE` (GP, handles covariates) for production
  - Why: Moran's I ignores covariates; SpatialDE/nnSVG model spatial structure properly

- **Spatial domain detection by graph type**
  - When to use: identifying tissue regions
  - How: SpaGCN (graph + CNN), STAGATE (graph + autoencoder), BANKSY (cell × microenvironment product space), BASS (multi-scale), PRECAST (probabilistic embedding)
  - Why: pure Leiden on expression misses spatial coherence; domain methods incorporate neighbor graph

- **Deconvolution for spot-based spatial**
  - When to use: Visium, Slide-seq, or any spot with multiple cells
  - How: Cell2location (best, Bayesian, needs scRNA reference), RCTD (fast, R), CARD (spatially-aware), SpatialDWLS, or DestVI (continuum)
  - Why: spot expression is a mixture; deconvolution assigns cell-type proportions per spot

## Key Concepts
- **SpatialExperiment / SpatialData**: Bioconductor / scverse data structures extending SingleCellExperiment / AnnData with spatial coordinates and images
- **Spot vs cell vs subcellular**: resolution tiers; determines analysis pipeline
- **SVG (Spatially Variable Gene)**: gene with expression pattern correlated with spatial position
- **Spatial domain**: spatially-coherent tissue region; identified via graph-based clustering
- **Moran's I**: spatial autocorrelation metric; [-1,1], 0=random; high positive = clustered
- **Geary's C**: local spatial autocorrelation; complements Moran's I
- **Neighborhood enrichment**: permutation test for cell-type co-localization (`squidpy.gr.nhood_enrichment`)
- **Deconvolution**: estimating cell-type proportions per spot from mixture expression
- **Cell2location**: Bayesian model, scRNA reference + spatial, gives per-spot per-cell-type abundance
- **Niche**: local microenvironment; niche differential expression (NicheDE) identifies context-dependent interactions
- **Cell-cell communication in space**: NICHES, stLearn, CellChat spatial - filter L-R pairs by co-localization
- **3D reconstruction**: aligning serial sections (PASTE, PRECAST, Gaussian process alignment)
- **Spatial neighborhood graph**: kNN based on spatial coordinates, not expression; `squidpy.gr.spatial_neighbors`

## Mental Models
- **Resolution tier dominates tool choice** - subcellular needs segmentation, spot-based needs deconvolution, single-cell tier uses scRNA-seq tools
- **Spatial ≠ scRNA-seq + coordinates** - need spatial-aware methods for SVGs, domains, CCC; standard scRNA-seq tools miss spatial structure
- **Deconvolution requires matched reference** - Cell2location/RCTD need scRNA-seq from similar tissue; mismatched reference = bad deconvolution
- **Spot swapping is a quality issue** - reads from neighboring spots contaminate each other; SpotClean adjusts
- **Imputation can recover single-cell resolution** - Tangram, gimVI, SpaGE map scRNA-seq to spatial; iSpatial infers genome-wide spatial expression
- **3D needs alignment** - serial sections need PASTE/PRECAST alignment before joint analysis
- **Subcellular needs segmentation** - Baysor (stain-free), Cellpose (deep learning), StarFusion; segmentation quality dominates downstream

## Anti-patterns
- **Running standard scRNA-seq pipeline on Visium** - spots aren't cells; need deconvolution
- **Using Leiden on expression alone for domains** - misses spatial coherence; use SpaGCN/STAGATE/BANKSY
- **Moran's I without covariates** - confounded by cell-type composition; use SpatialDE or nnSVG for production
- **Deconvolution without matched reference** - brain scRNA reference on tumor = wrong cell types
- **Skipping spot swapping correction** - 10-30% reads contaminate neighbors; SpotClean or cell2location handles
- **Cell-cell communication without spatial filter** - L-R pairs in non-adjacent cells are spurious; filter by co-localization
- **log1p on raw counts (not normalized) for spatial** - same as scRNA-seq; normalize first, then log1p
- **Treating Xenium/MERFISH as complete** - panel-based; impute missing genes with Tangram/iSpatial

## Reference Tables

### Spatial Platform Comparison

| Platform | Resolution | Throughput | Panel | Preprocessing |
|---|---|---|---|---|
| Visium | 55μm spots (~10 cells) | Standard slides | Whole transcriptome | Space Ranger (10x) |
| Visium HD | 2μm bins | Standard slides | Whole transcriptome | Space Ranger v3+ |
| Xenium | Subcellular | Standard slide | ~300-5000 genes | Xenium Onboard Analysis |
| MERFISH | Subcellular | Custom slide | 100-1000 genes | Custom (Zhuang lab) |
| CosMx | Subcellular | Standard slide | ~1000 genes | CosMx software |
| Stereo-seq (BGI) | Subcellular | Large tissue | Whole transcriptome | STOmics pipeline |
| Slide-seq v2 | 10μm beads | Custom slide | Whole transcriptome | Custom (Curio Seeker) |
| Curio Seeker | ~10μm | Commercial Slide-seq | Whole transcriptome | Curio pipeline |
| HDST | 2μm | Custom | Whole transcriptome | Custom |
| STARmap/STARmap PLUS | Subcellular | Custom | ~200-1000 genes | Custom |
| DBiT-seq | 25μm | Custom | Whole transcriptome | Custom |

### Pipeline Stage Tools

| Stage | Recommended | Alternatives | Notes |
|---|---|---|---|
| Data structure | SpatialData (scverse), SpatialExperiment (Bioconductor) | Seurat object | Match ecosystem |
| Preprocessing (Visium) | Space Ranger | squidy, scanpy.read_visium | Standard 10x |
| Segmentation (subcellular) | Baysor, Cellpose | sopa, StarFusion | Baysor = stain-free |
| Normalization | same as scRNA (log1p, scran) | tf-idf for some platforms | Per-modality |
| Spot swapping | SpotClean | cell2location ambient | 10-30% contamination |
| SVG detection | squidpy (Moran's I) for fast; nnSVG, SpatialDE for production | SpatialDE2, HVF | Covariates matter |
| Spatial domain | SpaGCN, STAGATE, BANKSY | BASS, PRECAST, DeepST | Bank by graph type |
| Deconvolution | Cell2location (best), RCTD | CARD, SpatialDWLS, DestVI | Needs scRNA reference |
| Imputation to single-cell | Tangram, gimVI, SpaGE | iSpatial, MISTy | For panel data |
| Cell-cell communication | NICHES, stLearn, CellChat spatial | NicheNet spatial, Squidpy lr | Spatial filter |
| Niche DE | NicheDE | C-SIDE | Context-dependent interactions |
| 3D alignment | PASTE, PRECAST | GP alignment | Serial sections |
| Visualization | squidpy.pl, Voyager | Giotto, Seurat spatial | Squidpy = scverse |

### SVG Detection Method Selection

| Method | Scalability | Handles Covariates | Output | Use Case |
|---|---|---|---|---|
| Moran's I (squidpy) | Fast, scalable | No | p-value per gene | Exploratory, large data |
| Geary's C | Fast | No | p-value per gene | Local autocorrelation |
| SpatialDE | Slow (~hours) | Yes | p-value + lengthscale | Production, small data |
| SpatialDE2 | Faster | Yes | p-value + lengthscale | Production |
| nnSVG | Scalable (GP) | Yes | p-value + lengthscale | Production, large data |
| HVG (non-spatial) | Fast | No | Variance ranking | Wrong for spatial |

### Deconvolution Method Comparison

| Method | Approach | Pros | Cons | Best For |
|---|---|---|---|---|
| Cell2location | Bayesian NN | Best accuracy; cell-type abundance + uncertainty | Slow; GPU helpful | Production, Visium |
| RCTD | Maximum likelihood | Fast; R/Bioconductor | Less accurate than Cell2location | Quick Visium |
| CARD | Conditional autoregressive | Spatially-aware | Needs reference | Visium with tissue structure |
| SpatialDWLS | Weighted least squares | Fast | Less accurate | Quick Visium |
| DestVI | Bayesian + scVI | Continuum of cell types | Slow | Cell state variation |
| Tangram | Optimal transport | Also does imputation | Less accurate for deconv | Joint tasks |
| SpotClean | Adjusts for spot swapping | Pre-deconvolution | Doesn't deconvolve | Quality correction |

## Worked Example

**Scenario**: Visium fresh-frozen mouse brain (coronal section), want to identify spatial domains and cell-type composition.

**Tool selection walkthrough**:
1. **Preprocessing**: `Space Ranger` for FASTQ → spot × gene matrix + tissue image; load into `SpatialData` (scverse) or `Seurat` object
2. **QC**: filter spots on `total_counts` and `n_genes_by_counts` (MAD-based); remove low-quality edge spots
3. **Normalization**: same as scRNA (`X` = log1p(normalize_total(1e4))); preserve raw in `layers["counts"]`
4. **Spot swapping**: `SpotClean` if Visium; or rely on cell2location ambient correction
5. **SVG detection**: `squidpy.gr.spatial_neighbors` → `squidpy.gr.spatial_autocorr(mode="moran")` for exploratory; `nnSVG` for production
6. **Spatial domain**: `SpaGCN` (default) or `STAGATE` (if graph + autoencoder needed); compare with `BANKSY` for cell × microenvironment approach
7. **Deconvolution**: `Cell2location` with matched scRNA reference (mouse brain atlas); per-spot per-cell-type abundance; cross-check with `RCTD` (faster)
8. **Cell-cell communication**: `NicheNet spatial` (ligand activity) + `Squidpy.lr` (L-R co-expression); filter by spatial co-localization
9. **Niche DE** (optional): `NicheDE` for context-dependent gene expression changes
10. **Visualization**: `squidpy.pl.spatial_scatter` for domains; overlay cell-type proportions; `cell2location.plt.spatial_fixed_effect` for cell-type maps

## Key Takeaways
1. Resolution tier (subcellular / single-cell / spot) determines the entire pipeline - identify first
2. Spot-based platforms (Visium, Slide-seq) require deconvolution - Cell2location is best, RCTD is faster
3. Spatial domains need graph-aware methods (SpaGCN, STAGATE, BANKSY) - not just Leiden on expression
4. SVG detection: Moran's I for fast exploration, nnSVG/SpatialDE for production (handles covariates)
5. Cell-cell communication must be spatially filtered - L-R pairs in non-adjacent cells are spurious
6. Subcellular platforms (Xenium, MERFISH) need segmentation - Baysor (stain-free) or Cellpose
7. Panel platforms (Xenium, MERFISH) need imputation for whole-transcriptome analysis - Tangram, iSpatial

## Connects To
- **Ch 09**: Spatial omics (theoretical foundations; this chapter lists tools)
- **Ch 13**: scRNA-seq tools (deconvolution references, integration)
- **Ch 06**: Modeling mechanisms (spatial CCC, NicheNet)
- **Ch 02**: Preprocessing (normalization, HVG - shared with scRNA)
- **Ch 17**: Cross-modality resources (seandavi has spatial section)
