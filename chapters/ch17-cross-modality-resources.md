# Chapter 17: Cross-Modality Resources & Databases

## Core Idea
Beyond pipeline-specific tools, the single-cell ecosystem has a layer of cross-cutting resources: curated tool catalogs (seandavi/awesome-single-cell, scRNA-tools.org), data portals (CELLxGENE, Human Cell Atlas, CZ CELLxGENE Discover), reference atlases (Tabula Muris/Sapiens, CellTypist models), benchmarking suites (scIB, pipeComp, CellBench), and learning resources (scverse tutorials, Bioconductor OSCA/OSTA, single-cell-tutorial by Theis lab). Knowing these resources shortcuts tool discovery, dataset finding, and method selection.

## Frameworks Introduced

- **Tool discovery via curated catalogs**
  - When to use: starting a new analysis type or looking for alternatives
  - How: seandavi/awesome-single-cell (comprehensive, software packages by category); scRNA-tools.org (database of published tools with paper links); scverse ecosystem page (curated, scverse-compatible only)
  - Why: pubmed search misses tools; these catalogs are community-maintained and updated

- **Atlas as reference for annotation**
  - When to use: cell type annotation, reference mapping
  - How: Human Cell Atlas (HCA) for human; Tabula Muris/Sapiens for cross-tissue; CellTypist models for immune/brain; CZ CELLxGENE Discover for census (33M+ cells)
  - Why: pretrained references save months; scArches + scVI enables query → reference mapping

- **Benchmarking before method choice**
  - When to use: choosing between methods (integration, DE, clustering)
  - How: scIB (integration benchmark, 16 methods, 13 metrics); pipeComp (preprocessing benchmark); CellBench (mixology); "Confronting false discoveries" (DE benchmark); scATAQ-benchmarking (10 ATAC methods)
  - Why: method choice has outsized impact; benchmarks reveal which methods are robust

- **Learning resources by ecosystem**
  - When to use: onboarding new team members or learning new analysis type
  - How: Bioconductor OSCA/OSTA (R, book format); scverse tutorials (Python, official); Theis lab single-cell-tutorial (Python, best practices); Seurat vignettes (R, Satija lab); coursera/edX courses
  - Why: structured tutorials > reading papers; pick the ecosystem you'll use

## Key Concepts
- **CELLxGENE Discover**: CZI's census of standardized scRNA-seq data (>33M cells, >60K genes); API in Python/R; `cellxgene-census` package
- **Human Cell Atlas (HCA)**: global consortium; tissue-specific atlases; downloadable via CZ CELLxGENE
- **Tabula Muris / Tabula Sapiens**: mouse / human multi-tissue atlas references
- **scRNA-tools.org**: database of single-cell tools, tracks ~1000+ tools with paper links
- **awesome-single-cell (seandavi)**: GitHub curated list, software packages by category
- **scverse**: foundational tools consortium (scanpy, anndata, muon, scirpy, squidpy, scvi-tools); guarantees interoperability
- **Bioconductor OSCA/OSTA**: Orchestrating Single-Cell Analysis / Spatial Transcriptomics Analysis; book-format tutorials
- **scIB metrics**: integration benchmarking suite; Bio conservation + Batch correction; Overall = 0.6·Bio + 0.4·Batch
- **pipeComp**: pipeline comparison framework; scRNA preprocessing benchmark
- **CellBench**: benchmarking scRNA-seq protocols and analysis methods
- **Confronting false discoveries**: Crowell et al. 2020; pseudobulk DE benchmark
- **CuratedAtlasQueryR**: R interface to query harmonized CELLxGENE atlas

## Mental Models
- **Start with catalogs, not PubMed** - awesome-single-cell and scRNA-tools.org are faster than literature search
- **Atlas-first annotation** - check if a reference atlas exists before training your own; CellTypist + scArches cover most use cases
- **Benchmark before adopting** - novel preprint methods often fail to outperform established ones; check scIB/pipeComp
- **scverse = interoperability** - tools in scverse (scanpy, muon, scirpy, squidpy, scvi-tools) share AnnData/MuData and work together
- **Census > single-dataset** - CZ CELLxGENE census (33M cells) enables cross-study questions without downloading individual datasets
- **Pseudobulk > per-cell DE** - benchmarked in Crowell 2020; pseudobulk with DESeq2/edgeR is the gold standard

## Anti-patterns
- **Googling for tools instead of checking catalogs** - awesome-single-cell is more complete and curated
- **Training your own reference atlas** - use CellTypist models or HCA first; only train custom if no reference exists
- **Trusting novel preprint methods** - check scIB/pipeComp first; new ≠ better
- **Skipping benchmarking data** - if a method isn't in scIB or similar, run your own mini-benchmark before adopting
- **Mixing ecosystems unnecessarily** - Python (scverse) ↔ R (Seurat/Bioconductor) conversion works but adds friction; pick one for a project
- **Downloading individual datasets when census exists** - CZ CELLxGENE census (33M cells) is standardized; use the API

## Reference Tables

### Cross-Modality Resource Catalog

| Category | Resource | Type | Link / Package |
|---|---|---|---|
| Tool catalog (general) | seandavi/awesome-single-cell | GitHub list | github.com/seandavi/awesome-single-cell |
| Tool catalog (database) | scRNA-tools.org | Database | scrna-tools.org |
| Tool catalog (scverse) | scverse.org | Curated | scverse.org |
| Tool catalog (spatial) | crazyhottommy/awesome_spatial_omics | GitHub list | github.com/crazyhottommy/awesome_spatial_omics |
| Tool catalog (scHi-C) | mdozmorov/scHiC_notes | GitHub list | github.com/mdozmorov/scHiC_notes |
| Tool catalog (scATAC) | mdozmorov/scATAC-seq_notes | GitHub list | github.com/mdozmorov/scATAC-seq_notes |
| Tool catalog (scRNA) | mdozmorov/scRNA-seq_notes | GitHub list | github.com/mdozmorov/scRNA-seq_notes |
| Data portal (human) | CZ CELLxGENE Discover | Census | cellxgene.cziscience.com |
| Data portal (human) | Human Cell Atlas | Atlas | data.humancellatlas.org |
| Data portal (human) | EMBL-EBI Single Cell | Atlas | ebi.ac.uk/gxa/sc/home |
| Data portal (cancer) | TISCH | Tumor TME | tisch.comp-genomics.org |
| Data portal (cancer) | CancerSCEM | Cancer | ngdc.cncb.ac.cn/cancerscem |
| Data portal (cancer) | DISCO | Integrated | immunesinglecell.org |
| Data portal (mouse) | Tabula Muris Senis | Atlas | tabula-muris-senis.ds.czbiohub.org |
| Data portal (interactive) | cellxgene (viewer) | Web viewer | github.com/chanzuckerberg/cellxgene |
| Data portal (interactive) | SCope | Web viewer | github.com/aertslab/SCope |
| Data portal (interactive) | iSEE | Bioconductor | bioconductor.org/packages/iSEE |
| Reference atlas (immune) | CellTypist models | Pretrained | celltypist.org |
| Reference atlas (T cells) | projecTILs | CD4 atlas | spica.unil.ch |
| Reference atlas (brain) | Allen Brain Cell Atlas | Brain | portal.brain-map.org |
| Reference atlas (spatial) | SODB | Spatial DB | gene.ai.tencent.com/SpatialOmics |
| Benchmark (integration) | scIB | Suite | github.com/theislab/scib |
| Benchmark (preprocessing) | pipeComp | Suite | github.com/plger/pipeComp |
| Benchmark (DE) | Crowell 2020 | Paper | nature.com/articles/s41467-021-25960-2 |
| Benchmark (scATAC) | scATAC-benchmarking | Suite | github.com/pinellolab/scATAC-benchmarking |
| Benchmark (clustering) | SC3, sc-SHC, CHOIR | Various | github.com/igrabski/sc-SHC |
| Learning (Python) | scverse tutorials | Official | scverse.org/tutorials |
| Learning (Python) | Theis lab tutorial | Best practices | github.com/theislab/single-cell-tutorial |
| Learning (R) | Bioconductor OSCA | Book | bioconductor.org/books/OSCA |
| Learning (R spatial) | Bioconductor OSTA | Book | lmweber.org/OSTA |
| Learning (R) | Seurat vignettes | Official | satijalab.org/seurat/articles |
| Learning (courses) | Hemberg lab course | Cambridge | github.com/hemberg-lab/scRNA.seq.course |
| Learning (courses) | YeoLab single-cell-bioinformatics | UCSD | github.com/YeoLab/single-cell-bioinformatics |
| Learning (lectures) | Ming Tang YouTube | scATAC, etc. | youtube.com/@MingTang |

### scverse Ecosystem (Core Tools)

| Tool | Purpose | Replaces |
|---|---|---|
| AnnData | Data structure | SingleCellExperiment (R) |
| MuData | Multimodal container | MultiAssayExperiment (R) |
| SpatialData | Spatial container | SpatialExperiment (R) |
| scanpy | scRNA-seq pipeline | Seurat (R) |
| muon | Multimodal pipeline | MOFA+ (R) |
| scirpy | AIRR (TCR/BCR) | Dandelion (R) |
| squidpy | Spatial omics | Seurat spatial (R) |
| scvi-tools | Deep generative models | (no direct R equivalent) |
| decoupler | GSEA / pathway | limma-fry (R) |
| pertpy | Perturbation modeling | (no direct equivalent) |

### Key Reference Atlases by Tissue

| Tissue | Atlas | Coverage | Use Case |
|---|---|---|---|
| Immune (human) | CellTypist Immune_All_Low/High | ~500k cells | PBMC, tissue immune |
| Brain (human) | Allen Brain Cell Atlas | ~3M cells | Cortex, subcortex |
| Brain (mouse) | Allen Mouse Brain Atlas | ~1.6M cells | Mouse brain reference |
| Whole organism (mouse) | Tabula Muris Senis | ~500k cells, 23 tissues | Cross-tissue |
| Whole organism (human) | Tabula Sapiens | ~500k cells, 24 tissues | Cross-tissue |
| Tumor (TME) | TISCH | scRNA-seq database | Tumor microenvironment |
| Human cell census | CZ CELLxGENE Census | 33M+ cells | Cross-study queries |
| CD4 T cells | projecTILs reference | CD4 atlas | T cell state mapping |

### Benchmarking Suite Quick Reference

| Benchmark | What it Tests | Key Finding |
|---|---|---|
| scIB (2022) | 16 integration methods, 13 metrics | scVI/scANVI top tier; Harmony competitive; BBKNN weaker |
| pipeComp (2020) | Preprocessing pipelines | Seurat V3 HVG + lognorm robust; doublet removal method matters |
| CellBench (2019) | scRNA-seq protocols + methods | Methods developed for bulk often outperform scRNA-specific |
| Crowell 2020 (DE) | DE methods | Pseudobulk + edgeR/DESeq2 = gold standard; per-cell methods inflate FP |
| scATAC-benchmarking (2019) | 10 scATAC methods | SnapATAC, Cusanovich2018, cisTopic perform best |
| Arc Weiler 2024 (clustering) | Clustering validity | sc-SHC for significance testing; CHOIR for distinctness |

## Worked Example

**Scenario**: New postdoc joins lab, needs to analyze scRNA-seq data for the first time.

**Resource onboarding walkthrough**:
1. **Learn the ecosystem** (1 day):
   - Read `theislab/single-cell-tutorial` (Python) or `Bioconductor OSCA` (R) - pick one ecosystem
   - Skim `scverse.org/tutorials` for scverse tool overview
   - Watch Ming Tang's YouTube lectures on scRNA-seq basics
2. **Find tools for each pipeline stage** (1 hour):
   - Bookmark `seandavi/awesome-single-cell` and `scRNA-tools.org`
   - For Python: use scverse toolkit (scanpy, muon, scirpy, squidpy, scvi-tools, decoupler, pertpy)
   - For R: use Seurat + Bioconductor (scran, scater, SingleR)
3. **Check benchmarks before choosing methods**:
   - Integration: `scIB` paper → scVI (raw counts) or Harmony (PCA)
   - DE: `Crowell 2020` → pseudobulk + DESeq2/edgeR
   - Clustering: Leiden (default); sc-SHC for significance testing
4. **Find reference atlas for annotation**:
   - Immune cells: `CellTypist` models
   - Brain: `Allen Brain Cell Atlas`
   - Cross-tissue: `Tabula Sapiens` (human) or `Tabula Muris` (mouse)
   - Query census via `cellxgene-census` Python/R package
5. **For a new modality** (e.g., scATAC, spatial):
   - scATAC: read `mdozmorov/scATAC-seq_notes` → pick ArchR or Signac
   - Spatial: read `crazyhottommy/awesome_spatial_omics` → pick squidpy (scverse) or Seurat spatial
   - scHi-C: read `mdozmorov/scHiC_notes` → pick scHiCExplorer or Higashi
6. **For specific questions**:
   - "What's the best integration method?" → scIB benchmark + scIB metrics
   - "How do I annotate T cells?" → CellTypist + projecTILs
   - "Where can I find matched scRNA reference for Visium brain?" → CZ CELLxGENE Census API
   - "Is my clustering real?" → sc-SHC (significance test) or CHOIR (distinctness)

## Key Takeaways
1. Tool discovery starts with curated catalogs (awesome-single-cell, scRNA-tools.org), not PubMed
2. CZ CELLxGENE Census (33M+ cells) is the standard for cross-study queries; use the API, don't download individual datasets
3. Reference atlases (CellTypist, HCA, Tabula) save months - check before training your own
4. Benchmark before adopting: scIB (integration), Crowell 2020 (DE), pipeComp (preprocessing)
5. scverse ecosystem guarantees interoperability - prefer scverse tools (scanpy, muon, scirpy, squidpy, scvi-tools) for Python projects
6. For new modalities, start with the awesome-list for that modality (scATAC, scHi-C, spatial, etc.)
7. Learning resources: scverse tutorials (Python), Bioconductor OSCA/OSTA (R), Seurat vignettes (R), Theis lab tutorial (best practices)

## Connects To
- **All chapters (Ch 01-12)**: Theoretical foundations; this chapter is the resource layer on top
- **Ch 13-16**: Specific modality tool ecosystems; this chapter is the cross-cutting index
- **Ch 02-12**: Each chapter's tools are linked from the modality-specific awesome-lists here
