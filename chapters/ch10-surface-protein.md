# Chapter 10: Surface Protein (CITE-seq)

## Core Idea
ADT (Antibody-Derived Tag) data from CITE-seq measures ~100-200 surface proteins per cell alongside RNA, capturing protein-level identity that RNA cannot (e.g., ICOS protein changes without mRNA change). ADT data is bimodal (negative peak = non-specific binding, positive peak = true signal), not negative binomial like UMI counts, so it needs different normalization (dsb or CLR), different QC (filter on number of proteins detected, not just total counts), and orthogonal doublet detection (mutually-exclusive markers like CD3/CD19).

## Frameworks Introduced

- **ADT-specific QC: filter on `n_genes_by_counts` not just `total_counts`**
  - When to use: any CITE-seq QC
  - How: `sc.pp.calculate_qc_metrics(mdata["prot"])`; plot distribution of `n_genes_by_counts`; find the "valley" (~55 ADTs in NeurIPS); apply hard upper bound on `total_counts` (~100k) to remove doublets; then sample-wise 5-MAD filter on `log1p_total_counts` + `log1p_n_genes_by_counts`
  - Why: ADT counts are dominated by a few abundant proteins; total counts can increase disproportionately when targeted proteins succeed. Number of detected ADTs is a more robust quality signal

- **Two normalization methods: dsb vs CLR**
  - When to use: dsb if you have empty droplets + isotype controls; CLR otherwise
  - How:
    1. **dsb** (`mu.prot.pp.dsb(mdata, mdata_raw, isotype_controls=[...])`) — uses empty droplets to estimate ambient noise per antibody (subtracted); uses isotype controls (Mouse-IgG1, etc.) to estimate non-specific binding per cell (corrected); output is denoised + scaled by background
    2. **CLR** (`mu.prot.pp.clr(mdata["prot"])`) — Centered Log-Ratio: each protein count divided by geometric mean of all proteins in the cell, then log-transformed; accounts for sequencing depth only
  - Why: CLR doesn't remove ambient noise or unspecific binding; dsb is the modern best practice when the additional data is available

- **Marker-based heterotypic doublet detection**
  - When to use: after QC, before clustering; complements RNA-based scDblFinder
  - How: identify mutually-exclusive marker pairs (CD3 vs CD19 for T/B doublets, CD3 vs CD14 for T/monocyte doublets); scatter plot; cells with both markers above threshold (~2.5 dsb-normalized) are doublets; remove
  - Why: ADTs detect doublets that RNA cannot — protein-level co-expression of mutually exclusive markers is unambiguous

- **Dimensionality reduction + batch correction**
  - When to use: after normalization and doublet removal
  - How: PCA → elbow plot (~20 PCs for 130 ADTs) → kNN → UMAP; if donors/batches form separate clusters (CD4 T cells fragmenting by donor = batch effect), run Harmony (`sc.external.pp.harmony_integrate(adata, key="donor")`) → re-compute neighbors on `X_pca_harmony` → re-UMAP
  - Why: ADT batch effects are pronounced; Harmony (originally for RNA) works well empirically. ADTnorm and CytoVI are newer ADT-specific alternatives

- **Manual annotation via marker dotplots**
  - When to use: after batch correction; ADT annotation mirrors RNA workflow
  - How: `sc.tl.leiden(resolution=0.1)` → `sc.tl.rank_genes_groups(groupby="leiden")` → `sc.pl.rank_genes_groups_dotplot(n_genes=3, values_to_plot="logfoldchanges")`; assign clusters by canonical markers (CD3=T, CD19=B, CD14=Monocyte, CD56=NK, CD11c/CD303=DC, CD16=CD16 Mono/NK)
  - Why: ADT markers are less prone to dropouts than RNA (antibody-based detection); CD45 is broadly expressed but often dropped in RNA, detectable in ADT

## Key Concepts
- **ADT (Antibody-Derived Tag)**: nucleotide-barcode-tagged antibody that binds a surface protein; sequenced as a proxy for protein abundance
- **CITE-seq**: Cellular Indexing of Transcriptomes and Epitopes by Sequencing; uses streptavidin-biotin DNA barcodes; measures RNA + ADT
- **REAP-seq**: similar to CITE-seq but covalent antibody-oligo conjugation
- **DOGMA-seq**: CITE-seq + ATAC (ASAP-seq bridge); measures RNA + ADT + chromatin
- **ASAP-seq**: scATAC + ADT via bridge oligo; can measure surface + intracellular proteins
- **Isotype control**: non-specific antibody (e.g., Mouse-IgG1) used to estimate non-specific binding background
- **CLR (Centered Log-Ratio)**: count / geometric_mean(cell) → log; depth normalization only
- **dsb (denoised and scaled by background)**: empty-droplet ambient correction + isotype-control non-specific correction
- **Mutually exclusive markers**: cell-type-specific proteins (CD3, CD19, CD14) that should never co-express in a singlet
- **Bimodal ADT distribution**: negative peak (non-specific binding) + positive peak (true signal); distinct from NB-distributed UMI counts

## Mental Models
- **ADT complements RNA, doesn't replace it** — protein and mRNA only partially correlate; some biology (ICOS treatment response) is protein-only
- **Filter on detected protein count, not total counts** — total counts reward cells with abundant non-specific binding
- **dsb > CLR when possible** — dsb removes two noise sources; CLR removes one
- **Mutually exclusive markers = unambiguous doublets** — CD3+CD19+ cells cannot be singlets
- **ADT is less sparse than RNA** — antibody-based detection has fewer dropouts; CD45 detection is robust in ADT where RNA often drops
- **Batch effects are pronounced in ADT** — donor-driven CD4 T cell fragmentation is a typical sign Harmony is needed
- **Resolution 0.1 is a starting point** — coarse cell-type-level; raise for sub-type annotation

## Anti-patterns
- **Reusing RNA QC thresholds on ADT** — ADT count distributions are different; cells with few detected proteins need different cutoffs
- **Filtering on total ADT counts alone** — high counts may reflect successful target capture, not low quality
- **Using CLR when you have isotype controls + empty droplets** — dsb is strictly better; CLR is the fallback
- **Skipping isotype controls in dsb** — without them you only get ambient correction, not non-specific-binding correction
- **Relying on RNA scDblFinder only** — misses doublets that ADT markers catch (CD3+CD19+)
- **Trusting ADT clusters without batch correction** — donor-driven clusters are batch effects, not biology
- **Forgetting `var_names_make_unique`** — ADT names can collide with RNA gene names (CD3, CD8); prefix or rename
- **Including isotype controls in downstream analysis** — they have no biological signal; remove after dsb normalization
- **Removing doublets only by count threshold** — many doublets have normal counts; need marker-based detection

## Code Examples

### ADT QC (muon)

```python
import muon as mu
import scanpy as sc
import seaborn as sns
from scipy.stats import median_abs_deviation
import numpy as np

sc.pp.calculate_qc_metrics(mdata["prot"], inplace=True, percent_top=None)

# Hard upper bound on total counts (doublets / aggregates)
sc.pp.filter_cells(mdata["prot"], max_counts=100000)
mu.pp.filter_obs(mdata, mdata["prot"].obs_names)

# Sample-wise 5-MAD filter
def is_outlier(adata, metric: str, nmads: int):
    M = adata.obs[metric]
    return (M < np.median(M) - nmads * median_abs_deviation(M)) | (
        np.median(M) + nmads * median_abs_deviation(M) < M)

outliers = []
for sample in np.unique(mdata["prot"].obs["donor"]):
    adata_temp = mdata["prot"][mdata["prot"].obs["donor"] == sample].copy()
    adata_temp.obs["outlier"] = (
        is_outlier(adata_temp, "log1p_total_counts", 5) |
        is_outlier(adata_temp, "log1p_n_genes_by_counts", 5))
    outliers.append(adata_temp.obs["outlier"])
mdata["prot"].obs["outliers"] = pd.concat(outliers)
mdata = mdata[~mdata["prot"].obs["outliers"]].copy()
```

- **What it demonstrates**: the "valley" in `n_genes_by_counts` distribution (~55 ADTs) is the natural cutoff for low-quality cells; sample-wise 5-MAD removes ~3% per sample. Total counts upper bound removes aggregates.

### dsb normalization

```python
isotype_controls = ["Mouse-IgG1", "Mouse-IgG2a", "Mouse-IgG2b", "Rat-IgG2b"]
mdata["prot"].layers["counts"] = mdata["prot"].X.copy()

# Requires raw mudata with empty droplets
mu.prot.pp.dsb(mdata, mdata_raw, isotype_controls=isotype_controls, random_state=0)
```

- **What it demonstrates**: one-line dsb call; `mdata_raw` must contain empty droplets (total RNA counts 10–10^2.1). Output `.X` is denoised + scaled; range shifts from 0–20000 to ~-200 to 300.

### CLR fallback

```python
mdata_clr = mdata.copy()
mdata_clr["prot"].X = mdata_clr["prot"].layers["counts"].copy()
mu.prot.pp.clr(mdata_clr["prot"])
```

### Marker-based doublet detection

```python
sc.pl.scatter(mdata["prot"], x="CD3", y="CD19-1", color="log1p_total_counts")
sc.pl.scatter(mdata["prot"], x="CD3", y="CD14-1", color="log1p_total_counts")

# Flag doublets: CD3+CD19+ or CD3+CD14+ (threshold 2.5 dsb-normalized)
genes2filter = ["CD3", "CD19-1", "CD14-1"]
temp = mdata["prot"][:, genes2filter].X.T.tolist()
mdata["prot"].obs["doublets_markers"] = [
    (temp[0][i] > 2.5 and temp[1][i] > 2.5) or
    (temp[0][i] > 2.5 and temp[2][i] > 2.5)
    for i in range(mdata.shape[0])]
mdata["prot"].obs["doublets_markers"] = mdata["prot"].obs["doublets_markers"].astype(str)
mdata = mdata[mdata["prot"].obs["doublets_markers"] == "False"].copy()
```

### PCA + Harmony + UMAP + annotation

```python
# Remove isotype controls (no biological signal)
isotype_controls = ["Mouse-IgG1", "Mouse-IgG2a", "Mouse-IgG2b", "Rat-IgG2b"]
temp = mdata["prot"].var.loc[~mdata["prot"].var.index.isin(isotype_controls), :].index
mu.pp.filter_var(data=mdata["prot"], var=temp.tolist())

sc.pp.pca(mdata["prot"], svd_solver="arpack", random_state=0)
sc.pl.pca_variance_ratio(mdata["prot"], n_pcs=50)
sc.pp.neighbors(mdata["prot"], n_pcs=20, random_state=0)
sc.tl.umap(mdata["prot"], random_state=0)

# Harmony batch correction
sc.external.pp.harmony_integrate(adata=mdata["prot"], key="donor", random_state=0)
sc.pp.neighbors(mdata["prot"], n_pcs=20, use_rep="X_pca_harmony", random_state=0)
sc.tl.umap(mdata["prot"], random_state=0)

# Cluster + annotate
sc.tl.leiden(mdata["prot"], resolution=0.1, flavor="igraph",
             n_iterations=2, directed=False, random_state=0)
sc.tl.rank_genes_groups(mdata["prot"], groupby="leiden")
sc.pl.rank_genes_groups_dotplot(mdata["prot"], n_genes=3, values_to_plot="logfoldchanges")

mdata["prot"].obs["celltype"] = mdata["prot"].obs.leiden.map({
    "0": "CD4 T", "1": "Cytotoxic T", "2": "CD8 T",
    "3": "CD14 Mono", "4": "Proliferating cells", "5": "DC",
    "6": "CD16 Mono", "7": "B", "8": "NK"})
```

- **What it demonstrates**: complete ADT pipeline — QC → dsb → doublet removal → PCA → Harmony → UMAP → leiden → marker dotplot → celltype label. Resolution 0.1 gives coarse populations; raise for sub-types.

## Reference Tables

### ADT vs RNA: data properties

| Property | RNA | ADT |
|---|---|---|
| Distribution | Negative binomial | Bimodal (neg peak + pos peak) |
| Sparsity | High (most genes 0 in most cells) | Low (most proteins detected) |
| Features | 20-30k genes | 100-200 proteins |
| Noise sources | Library size, ambient RNA | Library size, ambient antibody, non-specific binding |
| Normalization | log1p, scran, Pearson | dsb (preferred) or CLR |

### CITE-seq QC thresholds (NeurIPS 2021, 140 ADTs)

| Metric | Lower | Upper | Rationale |
|---|---|---|---|
| `n_genes_by_counts` | ~55 (valley) | (none) | Below: failed capture |
| `total_counts` | (none) | 100,000 | Above: doublet / aggregate |
| `log1p_total_counts` (per-sample 5-MAD) | dynamic | dynamic | Sample-wise outlier removal |
| `log1p_n_genes_by_counts` (per-sample 5-MAD) | dynamic | dynamic | Sample-wise outlier removal |

### Marker pairs for doublet detection

| Pair | Cell types | Threshold (dsb) |
|---|---|---|
| CD3 + CD19 | T + B | both > 2.5 |
| CD3 + CD14 | T + Monocyte | both > 2.5 |
| CD3 + CD56 | (not a doublet — NKT) | co-expression is real |

### Cell-type markers (canonical)

| Cell type | Markers |
|---|---|
| T cells | CD3, CD4, CD8 |
| B cells | CD19, CD20 |
| NK cells | CD56 (CD3-) |
| Monocytes | CD14 (CD16-), CD16 (CD14-) |
| DC | CD11c, CD123, CD303 |
| Proliferating | (high RNA, low ADT signal) |

## Worked Example

**NeurIPS 2021 CITE-seq BMMC** — 122,016 droplets × 140 ADTs × 36,601 genes from 12 donors × 4 sites. Pipeline:
1. Load `cite_filtered.h5mu` MuData with RNA + ADT modalities
2. ADT QC: `n_genes_by_counts` valley ~55 → lower cutoff; `total_counts` upper 100k → aggregates; per-sample 5-MAD → ~3% cells removed
3. dsb normalization: load `cite_raw.h5mu` (24M droplets incl. empty); identify empty droplets (RNA total_counts 10-10^2.1); pass isotype controls (4 Mouse/Rat IgG); call `mu.prot.pp.dsb(mdata, mdata_raw, isotype_controls=...)`; total count range shifts 0–20000 → -200 to 300
4. Marker doublet detection: CD3 vs CD19 scatter → CD3+CD19+ cells removed; CD3 vs CD14 → CD3+CD14+ removed (~612 doublets)
5. Remove isotype controls; PCA → 20 PCs → UMAP; CD4 T cells fragment by donor → batch effect
6. Harmony on `donor` → re-UMAP; CD4 T cells now intermixed
7. Leiden (resolution=0.1) → 9 clusters; rank_genes_groups dotplot identifies: CD4 T (0), Cytotoxic T (1), CD8 T (2), CD14 Mono (3), Proliferating (4), DC (5), CD16 Mono (6), B (7), NK (8)
8. Save as `cite_annotation.h5mu` for joint analysis in Ch 12

The "valley" in `n_genes_by_counts` distribution is the key insight — it's a natural break between viable cells and empty droplets, more robust than total counts which can be inflated by abundant ambient proteins.

## Key Takeaways
1. **Filter ADT QC on `n_genes_by_counts` (number of proteins detected), not just total counts** — total counts are dominated by abundant ambient proteins
2. **Apply per-sample MAD filtering** — ADT distributions vary by sample; global cutoffs are inappropriate
3. **Use dsb when possible (empty droplets + isotype controls)** — removes ambient + non-specific-binding noise; CLR is the fallback
4. **Detect doublets via mutually exclusive markers** — CD3+CD19+ and CD3+CD14+ are unambiguous doublets that RNA-based methods miss
5. **ADT batch effects are pronounced** — donor-driven CD4 T cell fragmentation is typical; Harmony works well empirically
6. **ADT annotation is more robust than RNA for immune cells** — fewer dropouts; CD45 detection is reliable in ADT
7. **Resolution 0.1 for coarse cell types** — raise for sub-type annotation; canonical markers (CD3/CD19/CD14/CD56/CD11c) drive assignment
8. **Remove isotype controls after dsb normalization** — they carry no biological signal
9. **ADT is a complement to RNA, not a replacement** — joint analysis (Ch 12) leverages both modalities

## Connects To
- **Ch 1 (MuData)**: CITE-seq data lives in `MuData` with `mod["rna"]` + `mod["prot"]` slots
- **Ch 2 (Preprocessing)**: parallel QC logic; sample-wise MAD filtering carries over
- **Ch 8 (scATAC)**: DOGMA-seq measures RNA+ADT+ATAC; ASAP-seq measures ATAC+ADT; same ADT QC applies
- **Ch 12 (Multimodal integration)**: paired CITE-seq integration via WNN, MOFA+, totalVI uses this preprocessed data
- **Ch 6 (CCC)**: ADT surface proteins are direct ligands/receptors; can validate RNA-based CCC hypotheses
