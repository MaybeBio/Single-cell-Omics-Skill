# Chapter 1: Introduction — Sequencing, Data Structures & Interoperability

## Core Idea
Single-cell analysis rests on three foundations: (1) **the wet-lab protocol** (droplet vs plate-based; full-length vs tag-based; scRNA-seq vs scATAC-seq vs CITE-seq) which determines what biases live in the data, (2) **the data structure** (AnnData for unimodal, MuData for multimodal, SpatialData for spatial) which provides a unified in-memory + on-disk representation, and (3) **the framework** (scanpy for RNA, muon for multimodal, squidpy for spatial, scirpy for AIRR) which provides modular `pp`/`tl`/`pl` APIs. Underlying it all is raw data processing (FASTQ → count matrix via alignment, barcode correction, UMI counting) and interoperability (R↔Python via h5ad/h5mu files or rpy2/anndata2ri in-memory).

## Frameworks Introduced

- **Sequencing generations**
  - When to use: understanding protocol biases
  - How: 1st-gen (Sanger, ~1kb, 99.99% accuracy) → 2nd-gen (Illumina NGS, 75-300bp, >99.9%) → 3rd-gen (PacBio/Nanopore, 8kb-2Mb, 87-98%)
  - Why: long reads resolve repeats/isoforms; short reads give high accuracy at low cost

- **scRNA-seq protocol selection**
  - When to use: experimental design
  - How (two classes):
    1. **Droplet-based** (10x Chromium, Drop-seq, inDrop) — microfluidic encapsulation; high throughput (10k+ cells); ~10% transcript recovery; 3' or 5' tag-based; UMIs
    2. **Plate-based** (SMART-seq2, MARS-seq, QUARTZ-seq) — FACS-sorted into wells; 100s-1000s cells; full-length transcripts; higher per-cell depth
  - Why: droplet = broad tissue characterization + rare cell types; plate = deep characterization of specific populations; protocol choice biases observed cell types

- **Single-cell vs single-nuclei**
  - When to use: tissue-dependent choice
  - How: cells = fresh tissue + dissociation (vulnerable cell types lost); nuclei = frozen tissue (biobank-compatible) + no dissociation enzymes
  - Why: dissociation biases cell type proportions (e.g., neurons underrepresented in single-cell brain); nuclei recoverable from frozen

- **Raw data processing: FASTQ → count matrix**
  - When to use: starting point of any scRNA-seq analysis
  - How: FastQC → align reads (STARsolo, salmon-alevin, kb-python, cellranger) → identify + correct cell barcodes (Hamming/Levenshtein distance whitelist) → collapse UMIs (directional adjacency method)
  - Why: errors here propagate downstream; UMI deduplication removes PCR duplicates; barcode correction fixes sequencing errors

- **AnnData: the scverse unimodal container**
  - When to use: any unimodal scRNA-seq analysis (or ATAC, ADT per modality)
  - How: `adata.X` = count matrix (cells × genes); `adata.obs` = cell metadata DataFrame; `adata.var` = gene metadata; `adata.obsm`/`varm` = multi-dim annotations (PCA, UMAP); `adata.obsp`/`varp` = pairwise graphs; `adata.layers` = alternative representations (counts, logcounts); `adata.uns` = unstructured
  - Why: couples matrix + annotations; sparse matrix support; backed mode for large files; view-based subsetting (no copy until `.copy()`)

- **MuData: multimodal container**
  - When to use: paired multi-omics (CITE-seq RNA+ADT; 10x Multiome RNA+ATAC; DOGMA-seq RNA+ADT+ATAC)
  - How: `mdata = md.MuData({"rna": adata_rna, "prot": adata_prot})`; `mdata.mod` = dict of AnnData objects; `mdata.obs`/`var` = global (with modality:column prefix); `mdata.obsm` = joint embeddings; `mdata.update()` syncs changes from modality AnnData
  - Why: shares cell index across modalities; modality-specific + joint analyses; `.h5mu` on-disk format mirrors `.h5ad`

- **SpatialData: spatial omics container**
  - When to use: Visium, Slide-seq, MERFISH, CosMx, Xenium
  - How: integrates images + coordinates + geometries + molecular measurements; built on AnnData/Zarr; unified schema across technologies
  - Why: spatial context couples with expression; separate tables for multi-modal spatial (RNA + protein)

- **scanpy API design**
  - When to use: any RNA analysis
  - How: `sc.pp.*` = preprocessing (filter_cells/genes, normalize, log1p, highly_variable_genes, pca, neighbors); `sc.tl.*` = tools (leiden, umap, rank_genes_groups, diffmap); `sc.pl.*` = plots (umap, dotplot, violin, heatmap); `sc.datasets.*` = example datasets
  - Why: modular; scales to millions of cells; consistent AnnData in/out interface

- **muon API design**
  - When to use: ATAC, CITE-seq preprocessing; multimodal integration
  - How: `mu.pp.*`/`mu.tl.*`/`mu.pl.*` = general multimodal; `mu.atac.pp.*` = ATAC-specific (TSS enrichment, nucleosome signal); `mu.prot.pp.*` = protein-specific (dsb, CLR)
  - Why: extends scanpy pattern to non-RNA modalities; modality-specific operations live in submodules

- **squidpy API design**
  - When to use: spatial transcriptomics
  - How: `sq.gr.*` = graphs (spatial_neighbors); `sq.pl.*` = spatial plots (spatial_scatter, interaction); `sq.tl.*` = tools (nhood_enrichment, co_occurrence); built on AnnData + SpatialData
  - Why: spatial graph + image aware; reuses scanpy patterns

- **Interoperability: disk-based vs in-memory**
  - When to use: disk = pipelines + sharing; in-memory = interactive R↔Python
  - How:
    - **Disk**: H5AD (AnnData) ↔ Bioconductor via `zellkonverter::readH5AD()` ↔ Seurat via `SeuratDisk::Convert()`; H5MU (MuData) ↔ MultiAssayExperiment via `MuData::readH5MU()` ↔ Seurat via `MuDataSeurat::ReadH5MU()`
    - **In-memory**: `rpy2` + `anndata2ri` (Python→R, AnnData auto-converts to SingleCellExperiment); `reticulate` (R→Python); `%%R -i adata -o result` IPython magic
  - Why: R has stronger stats/Seurat-specific tools (WNN, FindIntegrationAnchors); Python has scvi-tools/PyTorch; use best of both

## Key Concepts
- **NGS (Next-Generation Sequencing)**: 2nd-gen short-read sequencing (Illumina); dominates scRNA-seq
- **Long-read sequencing**: PacBio (HiFi 8-15kb, CLR >200kb); Nanopore (MinION portable, >2Mb reads)
- **UMI (Unique Molecular Identifier)**: 4-8bp random barcode added pre-PCR; deduplicates PCR copies
- **Full-length protocol**: sequences whole transcript (SMART-seq2); detects splice variants
- **Tag-based protocol**: sequences only 3' or 5' end (10x); UMIs possible; isoform detection hard
- **Droplet protocol**: cells encapsulated in nanoliter oil droplets (10x, Drop-seq, inDrop)
- **Plate-based protocol**: cells sorted into microwell plates (SMART-seq2, MARS-seq, QUARTZ-seq)
- **STAMPs**: Single-cell Transcriptomes Attached to Microparticles (droplet protocol intermediate)
- **Poisson loading**: droplet bead occupancy follows Poisson; 10x deforms beads for >80% occupancy
- **mRNA abundance**: 100,000-1,000,000 mRNA per mammalian cell; ~50% of genes transcribed
- **Transcriptional bursts**: stochastic mRNA production; modeled by negative binomial distribution
- **AnnData**: unimodal scverse data structure (`X` + `obs` + `var` + `obsm` + `varm` + `obsp` + `varp` + `layers` + `uns`)
- **MuData**: multimodal extension; dict of AnnData in `.mod` + global `obs`/`var`/`obsm`
- **SpatialData**: spatial container integrating images, coordinates, geometries, expression
- **scverse**: Python single-cell ecosystem (scanpy, anndata, muon, mudata, squidpy, scvi-tools, scirpy)
- **Bioconductor**: R ecosystem (SingleCellExperiment, OSCA book)
- **Seurat**: R package (Seurat object, common in multimodal WNN)
- **H5AD**: HDF5-based disk format for AnnData
- **H5MU**: HDF5-based disk format for MuData
- **Loom**: older HDF5 format; used by velocyto for spliced/unspliced
- **rpy2**: Python→R interface; `%%R` IPython magic
- **anndata2ri**: rpy2 extension; AnnData ↔ SingleCellExperiment auto-conversion
- **reticulate**: R→Python interface
- **zellkonverter**: Bioconductor R package for H5AD ↔ SingleCellExperiment
- **SeuratDisk**: R package for H5AD ↔ Seurat via H5Seurat intermediate

## Mental Models
- **Protocol determines biases** — 10x prefers short high-GC genes; Drop-seq prefers low-GC; know your protocol
- **Droplet = breadth, plate = depth** — droplet for tissue composition; plate for cell-type-specific characterization
- **UMIs remove amplification bias** — design experiments with UMIs whenever possible
- **Cells vs nuclei is a tissue-driven choice** — frozen tissue = nuclei; dissociation-vulnerable cell types = nuclei
- **AnnData couples data + metadata** — never separate the count matrix from cell/gene annotations
- **Views save memory** — subsetting returns views; `.copy()` materializes
- **MuData shares cells, not features** — `obs_names` global; `var_names` per-modality
- **scverse is the future for scale** — R ecosystems struggle with 500k+ cells
- **Interoperability is best via disk** — h5ad/h5mu are the lingua franca
- **Use rpy2 for interactive cross-language** — anndata2ri makes AnnData auto-convert to SCE
- **Pick best tool, not best ecosystem** — fluency in R + Python > loyalty to one

## Anti-patterns
- **Treating all protocols the same** — 10x, Drop-seq, SMART-seq2 have different biases; don't apply one-size-fits-all preprocessing
- **Skipping FastQC** — base quality, adapter contamination, N content affect all downstream steps
- **Ignoring UMI deduplication** — PCR duplicates inflate counts; collapse via UMI-tools directional method
- **Using raw counts for scVI/scANVI** — these models need raw counts; log-normalized breaks the ZINB assumption
- **Mixing `.X` and `.layers["counts"]`** — be explicit which layer is which; many pipelines silently use `.X`
- **Forgetting `.copy()` on view** — modifying a view auto-copies, but explicit is safer
- **Not running `mdata.update()` after modality changes** — global `obs`/`var` won't sync
- **Using RDS for sharing** — RDS is R-only; use h5ad for cross-ecosystem
- **Skipping zellkonverter** — manual conversion of AnnData → SingleCellExperiment is error-prone
- **Re-implementing scanpy in custom loops** — `sc.pp` is highly optimized; use it
- **Trusting 10x as ground truth** — 10x captures ~14% of mRNA; many genes are dropout artifacts

## Code Examples

### AnnData fundamentals

```python
import anndata as ad
import numpy as np
import pandas as pd
from scipy.sparse import csr_matrix

# Initialize with sparse count matrix
counts = csr_matrix(np.random.default_rng().poisson(1, size=(100, 2000)),
                    dtype=np.float32)
adata = ad.AnnData(counts)
adata.obs_names = [f"Cell_{i}" for i in range(adata.n_obs)]
adata.var_names = [f"Gene_{i}" for i in range(adata.n_vars)]

# Add cell metadata
adata.obs["cell_type"] = pd.Categorical(
    np.random.default_rng().choice(["B", "T", "Monocyte"], size=adata.n_obs))

# Add multi-dim annotations (PCA, UMAP)
adata.obsm["X_umap"] = np.random.default_rng().normal(0, 1, size=(adata.n_obs, 2))

# Add layers (counts vs logcounts)
adata.layers["counts"] = adata.X.copy()
adata.layers["log_transformed"] = np.log1p(adata.X)

# Subset (returns view)
bdata = adata[adata.obs.cell_type == "B"].copy()  # explicit copy

# Read/write
adata.write("my_results.h5ad", compression="gzip")
adata_new = ad.read_h5ad("my_results.h5ad")

# Backed mode for large files
adata_backed = ad.read_h5ad("my_results.h5ad", backed="r")
adata_backed.file.close()  # explicit close
```

### MuData fundamentals

```python
import mudata as md

adata_rna = ad.AnnData(rna_counts)
adata_prot = ad.AnnData(adt_counts)

mdata = md.MuData({"rna": adata_rna, "prot": adata_prot})

# Access modalities
mdata.mod["rna"]
mdata["rna"]  # shorthand

# Global obs/var with modality prefixes
mdata.obs.columns  # ['rna:n_genes', 'prot:n_genes', ...]

# Update after modality-level changes
adata_rna.obs["new_col"] = ...
mdata.update()

# Read/write
mdata.write("my_mudata.h5mu")
mdata = md.read("my_mudata.h5mu")
```

### scanpy quick demo

```python
import scanpy as sc

sc.settings.set_figure_params(dpi=80, facecolor="white")
sc.logging.print_header()  # version info for reproducibility

adata = sc.datasets.pbmc3k()

# QC
sc.pp.filter_cells(adata, min_genes=200)
sc.pp.filter_genes(adata, min_cells=3)

# Normalize + HVG + PCA
sc.pp.normalize_total(adata, target_sum=1e4)
sc.pp.log1p(adata)
sc.pp.highly_variable_genes(adata)
sc.tl.pca(adata, svd_solver="arpack")

# Neighbors + UMAP + Leiden
sc.pp.neighbors(adata, n_neighbors=10, n_pcs=40)
sc.tl.umap(adata)
sc.tl.leiden(adata, resolution=0.5)

# Plot
sc.pl.umap(adata, color=["leiden", "CST3", "NKG7"])
```

### muon ATAC preprocessing

```python
import muon as mu

atac = mdata.mod["atac"]

# Reuse scanpy QC (peaks ≈ genes conceptually)
sc.pp.calculate_qc_metrics(atac, percent_top=None, log1p=False, inplace=True)

# Filter with muon (in-place, no copy)
mu.pp.filter_var(atac, "n_cells_by_counts", lambda x: x >= 10)
mu.pp.filter_obs(atac, "n_genes_by_counts",
                 lambda x: (x >= 2000) & (x <= 15000))
mu.pp.filter_obs(atac, "total_counts",
                 lambda x: (x >= 4000) & (x <= 40000))

# ATAC-specific (TSS enrichment, nucleosome signal)
mu.atac.pp.tss_enrichment(atac)
mu.atac.pp.nucleosome_signal(atac)
```

### Disk-based R↔Python (H5AD)

```python
# Python: write
adata.write_h5ad("example.h5ad")
```

```R
# R Bioconductor: read
library(zellkonverter)
sce <- readH5AD("example.h5ad")

# R Seurat: read (via H5Seurat intermediate)
library(SeuratDisk)
Convert("example.h5ad", dest="h5seurat", overwrite=TRUE)
seurat <- LoadH5Seurat("example.h5seurat")
```

### In-memory R↔Python (rpy2 + anndata2ri)

```python
import anndata2ri
%load_ext rpy2.ipython
anndata2ri.set_ipython_converter()

# Pass AnnData to R as SingleCellExperiment
%%R -i adata -o qc
library(scuttle)
qc <- perCellQCMetrics(adata)
head(qc)
```

```python
# Run R function from Python
import rpy2.robjects as ro
from rpy2.robjects import pandas2ri

with ro.conversion.localconverter(anndata2ri.converter):
    sce_in_r = ro.conversion.py2rpy(adata)
```

### Interop for multimodal (H5MU)

```python
# Python: write MuData
mdata.write("multimodal.h5mu")
```

```R
# Bioconductor
library(MuData)
mae <- readH5MU("multimodal.h5mu")  # MultiAssayExperiment

# Seurat
library(MuDataSeurat)
seurat <- ReadH5MU("multimodal.h5mu")
```

## Reference Tables

### Sequencing generations comparison

| Generation | Method | Read length | Accuracy | Cost ($/Gb) |
|---|---|---|---|---|
| 1st | Sanger | ~1 kb | 99.99% | 500,000 |
| 2nd | Illumina NovaSeq | 0.25 kb | >99.9% | 10-35 |
| 3rd | PacBio HiFi | >20 kb | >99% | 43-86 |
| 3rd | Nanopore PromethION | >1,000 kb | 87-98% | 21-42 |

### scRNA-seq protocol comparison

| Protocol | Isolation | Throughput | Per-cell depth | Transcript coverage | UMIs |
|---|---|---|---|---|---|
| 10x Chromium | Droplet | 10k+ cells | Low (~1.5k genes) | 3' tag | Yes |
| Drop-seq | Droplet | 10k+ cells | Low | 3' tag | Yes |
| inDrop | Droplet | 10k+ cells | Lowest | 3' tag | Yes |
| SMART-seq2 | Plate (FACS) | 100s-1k | High (~10k genes) | Full-length | No |
| MARS-seq | Plate | 100s-1k | Medium | 3' tag | Yes |
| QUARTZ-seq2 | Plate | 100s-1k | Highest | Full-length | Yes |

### AnnData slots at a glance

| Slot | Type | Shape | Use |
|---|---|---|---|
| `X` | sparse/dense matrix | n_obs × n_vars | Main data (counts, logcounts) |
| `obs` | DataFrame | n_obs | Cell metadata (batch, donor, QC) |
| `var` | DataFrame | n_vars | Gene metadata (highly_variable, mean) |
| `obsm` | dict of arrays | n_obs × k | Multi-dim cell annotations (PCA, UMAP) |
| `varm` | dict of arrays | n_vars × k | Multi-dim gene annotations (PCs) |
| `obsp` | dict of matrices | n_obs × n_obs | Cell-cell graphs (connectivities) |
| `varp` | dict of matrices | n_vars × n_vars | Gene-gene graphs |
| `layers` | dict of matrices | n_obs × n_vars | Alternative X (counts, logcounts) |
| `uns` | dict | any | Unstructured (color palettes, params) |

### Interoperability matrix

| From → To | Disk format | In-memory bridge |
|---|---|---|
| AnnData → SingleCellExperiment | H5AD → `zellkonverter::readH5AD()` | `anndata2ri` (auto-converts) |
| AnnData → Seurat | H5AD → `SeuratDisk::Convert()` → H5Seurat | `sceasy::convertFormat()` |
| SingleCellExperiment → AnnData | `zellkonverter::writeH5AD()` | `zellkonverter::SCE2AnnData()` |
| MuData → MultiAssayExperiment | H5MU → `MuData::readH5MU()` | — |
| MuData → Seurat | H5MU → `MuDataSeurat::ReadH5MU()` | — |

### scverse ecosystem at a glance

| Library | Purpose |
|---|---|
| anndata | Unimodal data structure |
| mudata | Multimodal container |
| scanpy | RNA analysis |
| muon | Multimodal + ATAC + protein tools |
| squidpy | Spatial transcriptomics |
| scvi-tools | Deep generative models (scVI, scANVI, totalVI, multiVI) |
| scirpy | AIRR (TCR/BCR) analysis |
| spatialdata | Spatial data structure |
| scarches | Reference-based query mapping |

## Worked Example

**PBMC 3k dataset** (2,700 cells × 32,738 genes, Illumina NextSeq 500) — scanpy demo:
1. `adata = sc.datasets.pbmc3k()`
2. QC: `sc.pp.filter_cells(min_genes=200)`, `sc.pp.filter_genes(min_cells=3)`
3. MALAT1 is highest-expressed (poly-A capture artifact, inversely correlated with cell health)
4. Normalize + log1p + HVG + PCA
5. `sc.pp.neighbors(n_neighbors=10, n_pcs=40)` → `sc.tl.umap()` → `sc.tl.leiden()`
6. Plot: `sc.pl.umap(color=["CST3", "NKG7", "PPBP"])` reveals cluster-specific markers

**Interoperability scenario** — running Bioconductor's `scuttle::perCellQCMetrics()` on a Python AnnData:
1. Disk approach: `adata.write_h5ad("data.h5ad")` → R: `sce <- zellkonverter::readH5AD("data.h5ad")` → `qc <- scuttle::perCellQCMetrics(sce)` → write back
2. In-memory approach: `anndata2ri.set_ipython_converter()` → `%%R -i adata` → `qc <- scuttle::perCellQCMetrics(adata)` → `%%R -o qc`

The in-memory approach is preferred for interactive analysis (no disk I/O, immediate feedback); disk approach is preferred for pipelines (Nextflow/snakemake) and sharing with collaborators.

## Key Takeaways
1. **Protocol determines biases** — 10x (droplet) captures breadth; SMART-seq2 (plate) captures depth; choose based on question
2. **UMIs deduplicate PCR copies** — design experiments with UMIs; collapse computationally
3. **Single-cell vs single-nuclei is tissue-driven** — frozen tissue requires nuclei; dissociation biases cell-type proportions
4. **AnnData is the foundation** — `.X` + `.obs` + `.var` + `.layers` + `.obsm` + `.uns` covers 95% of analyses
5. **MuData extends to multimodal** — `mdata.mod["rna"]` + `mdata.mod["prot"]` shares cells across modalities
6. **scanpy API: `pp`/`tl`/`pl`** — preprocessing, tools, plotting; consistent across scverse
7. **Disk-based interop via H5AD/H5MU** — `zellkonverter` (Bioconductor) and `SeuratDisk` (Seurat) read/write
8. **In-memory interop via rpy2 + anndata2ri** — AnnData auto-converts to SingleCellExperiment
9. **Pick best tool, not best ecosystem** — R for Seurat WNN; Python for scvi-tools; fluent in both
10. **scverse scales to millions of cells** — R ecosystems struggle past 500k; Python is the future for atlas-scale

## Connects To
- **Ch 2 (Preprocessing)**: AnnData `.X`/`.layers["counts"]` is the input to QC + normalization
- **Ch 3 (Cellular structure)**: scanpy `sc.tl.leiden` + `sc.tl.umap` build on the AnnData neighbors graph
- **Ch 7 (Bulk deconvolution)**: rpy2 + MuSiC for R-based deconvolution of bulk RNA-seq
- **Ch 10 (Surface protein)**: MuData `mod["prot"]` for ADT data; muon `mu.prot.pp.dsb`
- **Ch 12 (Multimodal integration)**: MuData + scvi-tools (totalVI, multiVI) for paired multi-omics
