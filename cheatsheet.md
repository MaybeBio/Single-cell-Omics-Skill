# Cheatsheet — Decision Rules & Thresholds

## QC Thresholds Quick Lookup

### scRNA-seq (NeurIPS 2021 BMMC defaults)
| Metric | Lower | Upper | Method |
|---|---|---|---|
| `log1p_total_counts` | 5 MAD | 5 MAD | per-sample |
| `log1p_n_genes_by_counts` | 5 MAD | 5 MAD | per-sample |
| `pct_counts_mt` | (none) | 3 MAD + 8% hard cap | per-sample |
| `pct_counts_ribo` | (none) | 5 MAD + 8% | per-sample |
| `pct_counts_hb` | (none) | 3 MAD | per-sample |

### scATAC-seq
| Metric | Threshold | Action |
|---|---|---|
| `n_fragment_per_cell` | > 1000 | Drop lower |
| `n_fragment_per_cell` | < 100,000 | Drop higher (doublets) |
| `tss_enrichment_score` | > 2 | Drop lower |
| `nucleosome_signal` | < 4 | Drop higher |
| `reads_in_peaks_frac` | > 0.15 | Drop lower |
| `reads_in_blacklist_frac` | < 0.05 | Drop higher |

### CITE-seq ADT
| Metric | Threshold | Action |
|---|---|---|
| `n_genes_by_counts` (ADT) | valley (~55) | Hard lower |
| `total_counts` (ADT) | < 100,000 | Hard upper (doublets) |
| `log1p_total_counts` | 5 MAD per-sample | Soft |
| `log1p_n_genes_by_counts` | 5 MAD per-sample | Soft |

### AIRR (scirpy)
| Filter | Drop | Keep |
|---|---|---|
| `chain_pairing` | `multichain` (doublet) | `single pair`, `orphan VJ`, `orphan VDJ`, `extra VJ/VDJ` |

## Normalization Decision Tree

```
Has raw counts in .layers["counts"]?
├── RNA
│   ├── Have batch_key + raw counts → scVI/scANVI (raw counts input)
│   ├── Want fast batch correction → Harmony (on PCA)
│   ├── Just visualization → log1p(normalize_total(1e4))
│   └── HVG selection → Pearson residuals
├── ATAC → TF-IDF + LSI (use components 2:30)
├── ADT
│   ├── Has empty droplets + isotype controls → dsb (preferred)
│   └── Otherwise → CLR (fallback)
└── Spatial → same as RNA per modality
```

## Clustering Resolution Guide

| Resolution | Granularity | Use Case |
|---|---|---|
| 0.1 | Coarse cell types | Initial annotation (T, B, Mono, NK) |
| 0.4-0.5 | Sub-types | CD4 vs CD8, naive vs memory |
| 0.8-1.0 | Fine sub-states | Activated vs resting, tissue-resident |
| > 1.0 | Over-clustering | Sanity check, novel subtype discovery |

**Always**: sub-cluster specific clusters if marker dotplot shows mixed identities.

## Annotation Method Selection

| Scenario | Recommended Method |
|---|---|
| Reference atlas exists (immune, brain) | CellTypist (logistic regression) |
| Reference scVI/scANVI model exists | scArches label transfer |
| No reference, know markers | Manual marker-based (`rank_genes_groups`) |
| Multi-modality (RNA+ADT) | WNN + manual annotation |
| Spatial data | CellTypist + spatial coherence check |

## Integration Method Selection

| Modality | Paired? | Recommended |
|---|---|---|
| RNA only (multi-batch) | n/a | scVI (raw counts) or Harmony (PCA) |
| RNA + ADT | Yes | totalVI or WNN (Seurat) |
| RNA + ADT | No | MultiVI (multigrate) or MOFA+ |
| RNA + ATAC | Yes | MultiVI |
| RNA + ATAC | No | GLUE (needs GRN prior) |
| RNA + ADT + ATAC | Yes | Multigrate (trimodal) |
| Spatial + RNA reference | n/a | Cell2location (deconvolution) |
| Spatial query → spatial ref | n/a | Tangram (alignment) |

## scIB Metrics (Benchmarking Integration)

| Metric | Type | Range | Goal |
|---||---|---|
| ASW_label | Bio | [0, 1] | Higher |
| NMI | Bio | [0, 1] | Higher |
| ARI | Bio | [-1, 1] | Higher |
| Isolated label silhouette | Bio | [-1, 1] | Higher |
| cLISI | Bio | [1, n_cell_types] | Lower |
| ASW_label/batch | Batch | [0, 1] | Higher |
| PCR_batch | Batch | [0, 1] | Lower |
| Graph connectivity | Batch | [0, 1] | Higher |
| iLISI | Batch | [1, n_batches] | Higher |
| kBET | Batch | [0, 1] | Lower |
| **Overall** | Combined | 0.6·Bio + 0.4·Batch | Higher |

## DGE Method Selection

| Comparison | Sample Replicates | Method |
|---|---|---|
| Condition (within cell type) | ≥3 per condition | Pseudobulk + PyDESeq2 / limma-voom |
| Cell type * condition | ≥3 per cell type × condition | PyDESeq2 interaction design |
| Cell type proportions | n/a | scCODA (with reference cell type) |
| Transitional states | n/a | MILO (KNN-based) or DA-Seq |
| Per-cell DE (single-cell) | n/a | **Don't** — pseudoreplication inflates FP |

## Gene Set Test Selection

| Goal | Method |
|---|---|
| Cluster-level enrichment (one cluster vs rest) | GSEA (`decoupler`) |
| Per-cell activity scoring | AUCell |
| Complex design (interactions, covariates) | limma-fry (handles inter-gene correlation) |
| Competitive vs self-contained | Use limma-fry `competitive=FALSE` for self-contained |

## CCC Method Selection

| Question | Method |
|---|---|
| Is the L-R interaction plausible? | CellPhoneDB / LIANA consensus |
| Does the ligand explain the response? | NicheNet (needs condition contrast) |
| Want consensus across methods? | `li.mt.rank_aggregate` |
| Cross-condition comparison | Tensor-cell2cell or CellChat |

**Always**: filter expressed genes per cell type (≥10%) before CCC; intersect L-R + NicheNet for high-confidence hits.

## Trajectory Method Selection

| Biology | Method |
|---|---|
| Linear progression | DPT (diffusion pseudotime) |
| Multi-fate branching | Palantir (mark terminal states) |
| Cyclic processes | PAGA (graph abstraction) |
| RNA velocity needed | scVelo (dynamical mode) |
| Lineage tracing (CRISPR) | Cassiopeia (greedy, mpBoot, ilp) |

## Spatial Method Selection

| Task | Method |
|---|---|
| Neighborhood enrichment | `sq.gr.nhood_enrichment` (permutation test) |
| Spatial co-occurrence | `sq.gr.co_occurrence` (quadrat-based) |
| SVGs (fast) | `sq.gr.spatial_autocorr(mode="moran")` |
| SVGs (handles covariates) | SpatialDE |
| Spatial domains | SpaGCN or STAGATE |
| Spot deconvolution | Cell2location (best) or RCTD |
| Imputation to single-cell | Tangram |

## AIRR Method Selection

| Task | Method |
|---|---|
| TCR clonotype (strict) | scirpy `define_clonotype_clusters(receptor_arms="all")` |
| BCR clonotype (SHM-aware) | Dandelion `define_clones(model="hh_s5f")` |
| TCR specificity (DB query) | scirpy + VDJdb / IEDB |
| TCR specificity (distance) | TCRdist (cutoff 120) or TCRmatch (0.9) |
| Epitope prediction | ERGO-II (check training data!) |
| Multimodal AIR+GEX | Uni-modal with conditions (default); TESSA/CoNGA/mvTCR (experimental) |

## Perturbation Method Selection

| Goal | Method |
|---|---|
| Cell-type prioritization | Augur (AUC = separability) |
| Predict response in unseen cell type | scGen (VAE + vector arithmetic) |
| Separate perturbed from control | Mixscape (LDA + UMAP) |
| Gene-level response | Pseudobulk DGE (PyDESeq2) |

## Common Anti-Patterns to Avoid

1. **Reusing RNA thresholds on ADT/ATAC** — different distributions need different cutoffs
2. **Filtering ADT on total counts alone** — total counts reward abundant ambient proteins
3. **Using CLR when dsb is possible** — dsb strictly better with empty droplets + isotypes
4. **Skipping chain_qc before clonotype definition** — `multichain` cells contaminate networks
5. **Defining BCR clonotypes by identical CDR3** — SHM makes this too strict; use Dandelion
6. **Single-cell DE for condition comparison** — pseudoreplication; use pseudobulk
7. **Running SCENIC across batches** — co-expression is confounded by batch effects
8. **Trusting one ligand-receptor method** — `rank_aggregate` consensus; methods disagree
9. **Substituting ADT clusters without batch correction** — donor-driven clusters = batch effect
10. **Treating AIR distance clusters as ancestral** — distance-based clusters share specificity, not ancestry
11. **Forgetting `var_names_make_unique`** — ADT names collide with RNA gene names (CD3, CD8)
12. **Including isotype controls downstream** — no biological signal; remove after dsb
13. **Comparing spectratypes without normalization** — use `normalize=True` to compare distributions
14. **Trusting epitope predictors without checking training data** — biased to well-studied epitopes

## Standard Pipeline Order (RNA)

1. Load `raw` → `counts` matrix
2. Calculate QC metrics (`calculate_qc_metrics`)
3. Per-sample MAD filter on counts, genes, mt%
4. SoupX ambient RNA correction
5. scDblFinder doublet removal
6. Store `layers["counts"]` = filtered raw counts
7. Normalize: `X` = log1p(normalize_total(1e4)); `layers["scran"]` = scran size factors
8. HVG selection (deviance or seurat_v3)
9. PCA → elbow plot → choose PCs
10. kNN → UMAP → Leiden (resolution=0.1)
11. Annotate (CellTypist or markers or scArches)
12. Save as `.h5ad` for downstream

## Standard Pipeline Order (CITE-seq)

1. RNA pipeline (above) on `mdata["rna"]`
2. ADT QC: `n_genes_by_counts` valley filter, `total_counts` < 100k, per-sample 5-MAD
3. dsb normalization (with empty droplets + isotype controls)
4. Marker doublet removal (CD3+CD19+, CD3+CD14+)
5. Remove isotype controls
6. PCA on ADT → 20 PCs → UMAP → Leiden
7. Harmony batch correction if donor-driven fragmentation
8. Annotate via marker dotplot
9. Joint RNA+ADT analysis: WNN (Seurat) or totalVI
