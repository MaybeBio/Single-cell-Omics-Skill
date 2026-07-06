---
name: single-cell-best-practices
description: Distilled best-practice workflows for single-cell multi-omics analysis — covers scRNA-seq, scATAC-seq, CITE-seq, spatial, AIRR, and multimodal integration. Use when planning or executing single-cell analysis pipelines, choosing QC/normalization/integration methods, or interpreting results from scanpy/muon/scirpy/squidpy/scvi-tools.
---

# Single-cell Best Practices

A practical reference for single-cell multi-omics analysis distilled from theislab/single-cell-best-practices (Heumos et al., Nat Rev Genet 2023, v0.2.0 2026-06-11). Covers 12 chapters from raw scRNA-seq processing through multimodal integration, with decision rules, code patterns, and worked examples using the scverse ecosystem (scanpy, muon, scirpy, squidpy, scvi-tools, decoupler, pertpy).

## When to Use

- Designing a single-cell analysis pipeline (QC → normalization → clustering → annotation → downstream)
- Choosing between methods (dsb vs CLR, scVI vs Harmony, pseudobulk vs single-cell DE, Leiden resolution)
- Working with scRNA-seq, scATAC-seq, CITE-seq, spatial, AIRR, or multimodal data
- Interpreting cluster annotations, integration results, or differential analyses
- Debugging pipelines (batch effects, doublets, ambient RNA, sparse data)

**Not for**: bulk RNA-seq, single-cell proteomics (SCP), imaging-only analysis. For those, see the original book's outlook chapter.

## Core Frameworks

The 12 chapters teach a coherent end-to-end pipeline. Each chapter file (`chapters/chNN-*.md`) follows the same structure: Core Idea → Frameworks Introduced → Key Concepts → Mental Models → Anti-patterns → Code Examples → Reference Tables → Worked Example → Key Takeaways → Connects To.

1. **Per-modality QC specialization** — RNA (mt%, gene count, MAD), ATAC (TSS enrichment, nucleosome signal, FRiP), ADT (n_proteins detected, valley filter), AIRR (chain_qc). Each modality has different quality signals; reusing RNA thresholds is a common anti-pattern.
2. **Normalization matches the data distribution** — RNA: log1p + scran + Pearson residuals (3 layers); ATAC: TF-IDF + LSI; ADT: dsb (with empty droplets + isotype controls) > CLR (fallback); AIRR: scirpy stores in `adata.obs`.
3. **Clustering is iterative** — Leiden resolution 0.1 (coarse) → 0.4-0.8 (sub-types) → sub-cluster mixed identities. Annotation via CellTypist (auto), marker dotplot (manual), or scArches (reference mapping) with uncertainty.
4. **Integration via scVI/scANVI for raw counts** — conditional VAE with ZINB distribution; batch_key as condition. Harmony for fast PCA correction. BBKNN for graph-only. Benchmark with scIB (0.6·Bio + 0.4·Batch).
5. **Pseudobulk for DGE** — aggregate per sample+cell type, then PyDESeq2/limma-voom. Avoids pseudoreplication that inflates false positives in single-cell DE. scCODA for compositional analysis (Dirichlet-multinomial with reference cell type).
6. **Pseudotime is hypothesis, not ground truth** — DPT (linear), Palantir (multi-fate with terminal states), PAGA (graph), scVelo (RNA velocity, dynamical mode preferred over steady-state). Cassiopeia for CRISPR lineage tracing.
7. **GRN + CCC are complementary** — SCENIC 3-step (co-expression → motif → AUCell); CellPhoneDB/LIANA consensus for ligand-receptor plausibility; NicheNet for ligand activity (AUPR vs DE geneset). Run both CCC approaches, intersect for high-confidence hits.
8. **Multimodal integration matches the data** — WNN (Seurat) for RNA+ADT; totalVI for end-to-end CITE-seq; MultiVI for RNA+ATAC; GLUE for unpaired RNA+ATAC (needs GRN prior); Multigrate for trimodal. All experimental — benchmark with scIB.
9. **Spatial analysis is resolution-tiered** — subcellular (Xenium) vs single-cell (Visium HD) vs spot-based (Visium, needs cell2location deconvolution). Moran's I + SpatialDE for SVGs; SpaGCN/STAGATE for spatial domains.
10. **AIR is a heritable barcode** — scirpy stores in `adata.obs`; chain_qc filters doublets; clonotype via identity (strict) or distance (TCRdist, alignment); BCR needs Dandelion (SHM-aware, hh_s5f model). Specificity inference has 3 approaches (DB query, distance, prediction) — run multiple, intersect, validate.

## Chapter Index

| Chapter | Title | Key Question |
|---|---|---|
| 01 | [Introduction](chapters/ch01-introduction.md) | What are the data structures (AnnData/MuData/SpatialData) and tools (scanpy/muon/squidpy)? |
| 02 | [Preprocessing & Visualization](chapters/ch02-preprocessing-visualization.md) | How to QC, normalize, select HVGs, embed cells? |
| 03 | [Cellular Structure](chapters/ch03-cellular-structure.md) | How to cluster, annotate, integrate multi-batch data? |
| 04 | [Trajectories](chapters/ch04-trajectories.md) | How to order cells along developmental/transition processes? |
| 05 | [Conditions](chapters/ch05-conditions.md) | How to compare conditions (DGE, compositional, perturbation)? |
| 06 | [Modeling Mechanisms](chapters/ch06-modeling-mechanisms.md) | How to infer GRNs (SCENIC) and cell-cell communication (CCC)? |
| 07 | [Bulk Deconvolution](chapters/ch07-bulk-deconvolution.md) | How to estimate cell-type proportions from bulk RNA-seq? |
| 08 | [Chromatin Accessibility](chapters/ch08-chromatin-accessibility.md) | How to analyze scATAC-seq (peaks, QC, FigR GRN)? |
| 09 | [Spatial Omics](chapters/ch09-spatial-omics.md) | How to analyze spatial transcriptomics (SVGs, domains, deconvolution)? |
| 10 | [Surface Protein](chapters/ch10-surface-protein.md) | How to analyze CITE-seq ADT data (dsb, marker doublets)? |
| 11 | [AIRR Profiling](chapters/ch11-airr.md) | How to analyze TCR/BCR (clonotypes, specificity, multimodal)? |
| 12 | [Multimodal Integration](chapters/ch12-multimodal-integration.md) | How to integrate paired multi-omics (WNN, totalVI, MultiVI, GLUE)? |

## Topic Index (Cross-Chapter Lookup)

| Topic | Primary Chapter | Also In |
|---|---|---|
| AnnData structure | 01 | all chapters |
- Batch correction
  - scVI/scANVI — Ch 03, 12
  - Harmony — Ch 03, 10
  - BBKNN — Ch 03
  - Seurat anchors — Ch 03
- Benchmarking
  - scIB metrics — Ch 03, 12
- Cell-cell communication
  - CellPhoneDB, LIANA — Ch 06
  - NicheNet — Ch 06
  - Spatial CCC — Ch 09
- Clustering
  - Leiden — Ch 03
  - Sub-clustering — Ch 03
- Compositional analysis
  - scCODA — Ch 05
  - MILO, DA-Seq — Ch 05
- Deconvolution
  - Bulk (AutoGeneS, MuSiC) — Ch 07
  - Spatial (Cell2location, Tangram) — Ch 09
- Differential expression
  - Pseudobulk + PyDESeq2 — Ch 05
  - rank_genes_groups (Wilcoxon) — Ch 03
  - Interaction contrasts — Ch 05
- Doublet detection
  - scDblFinder (RNA) — Ch 02
  - AMULET (ATAC) — Ch 08
  - Marker-based (ADT) — Ch 10
- Gene set testing
  - GSEA — Ch 05
  - AUCell — Ch 05, 06
  - limma-fry — Ch 05
- GRN inference
  - SCENIC — Ch 06
  - FigR (ATAC+RNA) — Ch 08
  - SCENIC+ — Ch 08
- Immune receptor (AIRR)
  - scirpy — Ch 11
  - Dandelion (BCR) — Ch 11
  - TCRdist, TCRmatch — Ch 11
- Lineage tracing
  - Cassiopeia (CRISPR) — Ch 04
- Multimodal integration
  - WNN — Ch 12
  - totalVI — Ch 12
  - MultiVI — Ch 12
  - GLUE — Ch 12
  - Multigrate — Ch 12
  - TESSA, CoNGA, mvTCR, Benisse — Ch 11
- Normalization
  - log1p, scran, Pearson — Ch 02
  - dsb, CLR (ADT) — Ch 10
  - TF-IDF + LSI (ATAC) — Ch 08
- Perturbation
  - Augur — Ch 05
  - scGen — Ch 05
  - Mixscape — Ch 05
- Pseudotime / trajectory
  - DPT, Palantir, PAGA — Ch 04
  - scVelo (RNA velocity) — Ch 04
- QC
  - MAD-based filtering — Ch 02
  - Per-modality thresholds — Ch 02, 08, 10, 11
- Spatial
  - Moran's I, SpatialDE — Ch 09
  - SpaGCN, STAGATE — Ch 09
  - Squidpy neighborhood stats — Ch 09
- Visualization
  - UMAP, PCA, dotplot — Ch 02, 03
  - Spatial embedding — Ch 09

## Supporting Files

- **[glossary.md](glossary.md)** — every significant term with chapter pointer (ADT, AnnData, AUCell, batch effect, BCR, CDR3, CLR, clonotype, dsb, doublet, FASTQ, HVG, Harmony, Leiden, MAD, marker gene, Moran's I, MuData, NicheNet, pseudobulk, pseudotime, regulon, scIB, scirpy, scVI, spectratype, SHM, TCRdist, totalVI, UMI, VDJ, WNN).
- **[patterns.md](patterns.md)** — concrete techniques organized by task: QC patterns, normalization patterns, clustering & annotation, integration, differential analysis, trajectory, mechanism, spatial, multimodal, perturbation.
- **[cheatsheet.md](cheatsheet.md)** — decision rules, thresholds, and lookup tables: QC thresholds per modality, normalization decision tree, clustering resolution guide, integration method selection, scIB metrics, DGE/CCC/trajectory/spatial/AIRR method selection, common anti-patterns, standard pipeline order (RNA + CITE-seq).

## Worked Examples

Each chapter includes a worked example on real data, primarily using:
- **NeurIPS 2021 BMMC** (122k cells, 12 donors × 4 sites) — RNA, ATAC, ADT, multimodal
- **Kang et al. PBMC IFN-β** (~25k cells, 8 patients × 2 conditions) — DGE, perturbation
- **Haniffa Lab SARS-CoV-2 PBMCs** (750k cells, 130 patients) — AIRR
- **Haber et al. mouse intestine** — compositional analysis
- **10x Genomics PBMC 3k** — introductory scanpy demo
- **COVID-19 whole blood** — bulk deconvolution

## Scope & Limits

**In scope**: scRNA-seq (droplet + plate-based), scATAC-seq (10x Multiome), CITE-seq/REAP-seq, spatial transcriptomics (Visium, Visium HD, Xenium, MERFISH, Stereo-seq), TCR/BCR sequencing, multimodal integration (paired and unpaired).

**Out of scope**: single-cell proteomics (SCP), imaging-based cytometry (without sequencing), long-read scRNA-seq assembly, single-cell methylation (scBS-seq). See Outlook chapter of the original book for these emerging areas.

**Software assumptions**: scverse ecosystem (scanpy ≥1.10, muon, scirpy ≥0.13, squidpy, scvi-tools ≥1.0, decoupler, pertpy). R interoperability via rpy2 + anndata2ri + reticulate; Seurat v4/v5 for WNN and anchor-based integration. Python 3.10+ recommended.

**Citation**: Heumos L, Schaar AC, Lance C, et al. (2023). "Best practices for single-cell analysis across modalities." *Nat Rev Genet* 24:550–572. DOI: 10.1038/s41576-023-00586-w. Book: https://www.sc-best-practices.org

## Conventions

- All code assumes `import scanpy as sc; import muon as mu; import scirpy as ir; import squidpy as sq; import decoupler as dc` at minimum
- `adata` for AnnData (single modality), `mdata` for MuData (multimodal)
- Chapter references use `Ch N` format (e.g. "Ch 03" for cellular structure)
- Code examples prioritize clarity over performance; real pipelines need GPU for scVI/totalVI on >100k cells
- Anti-patterns marked with ❌ in code; recommended patterns marked with ✓
