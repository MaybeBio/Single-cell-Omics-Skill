# Chapter 6: Modeling Mechanisms

## Core Idea
Two complementary inference layers over an annotated dataset: **GRN inference** (which TFs regulate which target genes) and **cell-cell communication** (which ligands from source cells trigger which receptor responses in target cells). Both rely on prior knowledge + co-expression, both lack ground truth, and both should be cross-validated against orthogonal modalities (motif databases for GRNs; spatial/protein data for CCC).

## Frameworks Introduced

- **SCENIC three-step GRN pipeline**
  - When to use: scRNA-seq with cluster annotations, want TF→target hypotheses
  - How:
    1. **Co-expression** (`pyscenic grn` / GRNBoost) — gradient-boosted trees infer TF→target importance weights; output `adj.csv` with TF, target, importance
    2. **Motif enrichment** (`pyscenic ctx` / cisTarget) — prune adjacencies by TF motif presence in target promoter region (ranked feather databases, ~10kb up/down TSS); output `reg.csv` with cis-regulatory evidence
    3. **Regulon activity** (`pyscenic aucell`) — AUCell scores each cell for each regulon (TF + its targets); output cell × regulon AUC matrix
  - Why: co-expression alone has too many false positives; motif enrichment keeps only cis-regulated targets; AUCell converts sparse regulon expression into a continuous, denoised activity per cell

- **Ligand-receptor inference (CellPhoneDB / LIANA)**
  - When to use: steady-state or single-condition scRNA-seq with cluster annotations; want ligand-receptor hypotheses
  - How:
    1. Group cells by type; compute mean expression of each ligand and receptor subunit per cluster
    2. For heteromeric complexes: take minimum subunit expression (most-limiting)
    3. Permutation test: shuffle cluster labels → null distribution; `cellphone_pvals` = specificity
    4. `lr_means` = mean ligand-receptor expression = magnitude
    5. Filter: drop interactions where any entity expressed in <10% of cells (false-positive guard)
  - LIANA's `rank_aggregate`: runs multiple methods (CellPhoneDB, NATMI, Connectome, etc.) and ranks consensus via `magnitude_rank` + `specificity_rank`

- **NicheNet intracellular signaling (ligand → target)**
  - When to use: differential condition data (treatment vs control); want to know which ligands drive the observed DE response
  - How:
    1. Define sender/receiver cell types
    2. Filter expressed ligands/receptors (≥10% per cell type)
    3. Define **geneset of interest** = DE genes in receiver (positive, signif lFC ≥1, p ≤0.05) — pseudobulk per sample+cell type → t-test or DESeq2
    4. **Ligand activity** = AUPR of regulatory-potential-ranked genes vs geneset of interest (analogous to TF activity in decoupler)
    5. Top ligands → predict target genes via regulatory potential matrix; visualize as ligand × target heatmap
  - Why: ligand-receptor methods only ask "is the ligand expressed?"; NicheNet asks "does the ligand explain the receiver's transcriptional response?"

## Key Concepts
- **Transcription Factor (TF)**: ~1,600 in human, ~1,500 in mouse; genes that regulate transcription of other genes
- **Regulon**: TF + its set of target genes; activity scored per cell by AUCell
- **AUCell**: Area Under Recovery Curve — how highly regulon genes rank in each cell's expression profile
- **cisTarget**: motif-enrichment database ranking genes by TF motif presence near TSS
- **Heteromeric complex**: ligand or receptor composed of multiple subunits; min subunit expression = complex expression
- **Ligand activity (NicheNet)**: AUPR of ligand-target regulatory potential vs observed DE geneset; higher = ligand better predicts response
- **Regulatory potential**: Personalized PageRank-derived score from ligand through signaling network to target gene; prior knowledge
- **Pseudobulk**: aggregate cells per sample+cell type before DE; avoids pseudo-replication
- **Futility theorem**: most TF binding motifs in the genome are unbound; motif presence alone doesn't guarantee regulation

## Mental Models
- **GRN inference is hypothesis generation, not ground truth** — benchmarks disagree; use SCENIC output as a starting point for downstream validation (ChIP-seq, perturbation)
- **Three-step pipeline trades off false positives vs sensitivity** — co-expression permissive → motif enrichment strict → AUCell denoising
- **TF expression ≠ TF activity** — AUCell on regulon targets is a better proxy for activity than TF's own expression
- **Ligand-receptor methods answer "is the interaction plausible?"; NicheNet answers "is it functional?"** — run both, intersect
- **Use LIANA consensus, not one method** — limited agreement between methods; consensus rank > single-method output
- **CCC needs a condition contrast for NicheNet** — without a DE geneset, ligand activity can't be computed; for steady-state use ligand-receptor methods alone
- **Filter expressed genes per cell type first** — 10% threshold removes false-positive interactions where a ligand or receptor is barely detectable

## Anti-patterns
- **Running SCENIC on full multi-batch dataset** — confounds batch effects with co-expression; subset to one donor or batch-correct first
- **Using all genes instead of HVGs for GRNBoost** — slow and noisy; use HVGs ∪ all TFs
- **Skipping TF symbol QC** — check that ≥50% of expected TFs are in `adata.var_names`; case mismatches (human `UPPER` vs mouse `Title`) silently produce empty TF lists
- **Trusting one ligand-receptor method** — benchmarked methods show low agreement; use `rank_aggregate` for consensus
- **Running ligand-receptor methods across conditions without stratifying** — designed for steady-state; for condition comparisons use NicheNet or dedicated tools (CellChat, Tensor-cell2cell)
- **Skipping the 10% expression filter** — produces huge lists of false-positive interactions from low-expressed genes
- **Defining NicheNet geneset of interest from single-cell DE** — pseudobulk first; single-cell DE inflates false positives via pseudo-replication
- **Interpreting CCC as one-to-one events** — real signaling is many-to-many; treat inferred pairs as hypotheses about a community, not as a wiring diagram

## Code Examples

### SCENIC three-step pipeline (NeurIPS donor s1d1)

```python
import loompy as lp
import scanpy as sc
import numpy as np

# subset to one donor + HVG ∪ TFs
adata_batch = rna[rna.obs.batch == "s1d1", :]
mask = adata_batch.var["highly_variable"] | adata_batch.var.index.isin(tfs)
adata_batch = adata_batch[:, mask]

# export to loom
row_attrs = {"Gene": np.array(adata_batch.var.index)}
col_attrs = {
    "CellID": np.array(adata_batch.obs.index),
    "nGene": np.array(np.sum(adata_batch.X.T > 0, axis=0)).flatten(),
    "nUMI": np.array(np.sum(adata_batch.X.T, axis=0)).flatten(),
}
lp.create(loom_path, adata_batch.X.T, row_attrs, col_attrs)
```

```bash
# Step 1: co-expression (GRNBoost)
pyscenic grn {loom_path} {tfs_path} -o adj.csv --num_workers 3

# Step 2: motif enrichment (cisTarget) — prunes adj.csv by motif presence
pyscenic ctx adj.csv {feather_dbs} \
    --annotations_fname motifs-v9-nr.hgnc-m0.001-o0.0.tbl \
    --expression_mtx_fname {loom_path} \
    --output reg.csv --mask_dropouts --num_workers 3

# Step 3: regulon activity per cell (AUCell)
pyscenic aucell {loom_path} reg.csv --output {loom_path_output} --num_workers 3
```

```python
# Load AUC matrix and embed regulon activities
lf = lp.connect(loom_path_output, mode="r+", validate=False)
auc_mtx = pd.DataFrame(lf.ca.RegulonsAUC, index=lf.ca.CellID)
lf.close()

ad_auc_mtx = ad.AnnData(auc_mtx)
sc.pp.neighbors(ad_auc_mtx, n_neighbors=10, metric="correlation")
sc.tl.umap(ad_auc_mtx)
adata_batch.obsm["X_umap_aucell"] = ad_auc_mtx.obsm["X_umap"]

# Heatmap of cell-type × top-50 regulons
mean_auc_by_cell_type = auc_mtx.groupby(adata_batch.obs["cell_type"]).mean()
top_tfs = mean_auc_by_cell_type.max(axis=0).sort_values(ascending=False).head(50)
sns.clustermap(mean_auc_by_cell_type[list(top_tfs.index)],
               figsize=[15, 6.5], cmap="Blues", xticklabels=True, yticklabels=True)
```

- **What it demonstrates**: the canonical SCENIC chain — co-expression → motif pruning → AUCell. Note `--mask_dropouts` in ctx (only score cells expressing the TF) and `metric="correlation"` for the regulon-based neighbors graph (better than Euclidean for AUC values).

### CellPhoneDB + LIANA consensus (PBMC IFN-β)

```python
import liana as li
from liana.method import cellphonedb, rank_aggregate

adata_stim = adata[adata.obs["condition"] == "stim"].copy()

# Single method
cellphonedb(adata_stim, groupby="cell_type", use_raw=False,
            return_all_lrs=True, verbose=True)

# Consensus across methods
rank_aggregate(adata_stim, groupby="cell_type", return_all_lrs=True,
               use_raw=False, verbose=True)

# Visualize top-20 consensus interactions
li.pl.dotplot(
    adata=adata_stim,
    colour="magnitude_rank", size="specificity_rank",
    inverse_colour=True, inverse_size=True,
    source_labels=["CD4 T cells", "B cells", "FCGR3A+ Monocytes"],
    target_labels=["CD8 T cells", "CD14+ Monocytes", "NK cells"],
    filter_fun=lambda x: x["specificity_rank"] <= 0.05,
    orderby="magnitude_rank", orderby_ascending=True,
    top_n=20, figure_size=(9, 5), size_range=(1, 6),
)
```

- **What it demonstrates**: CellPhoneDB writes results to `adata.uns['liana_res']` with `lr_means` (magnitude) + `cellphone_pvals` (specificity). `rank_aggregate` adds `magnitude_rank` + `specificity_rank` across methods. Filter by specificity_rank ≤0.05 for consensus hits.

### NicheNet ligand activity (PBMC IFN-β, CD8 T cells as receiver)

```python
# 1. Define sender/receiver cell types
sender_celltypes = ["CD4 T cells", "B cells", "FCGR3A+ Monocytes"]
receiver_celltypes = ["CD8 T cells"]

# 2. Expressed ligands/receptors (≥10% per cell type)
sender_expressed = reduce(np.union1d, [
    get_expressed_genes(adata, ct, 0.1) for ct in sender_celltypes])
receiver_expressed = reduce(np.union1d, [
    get_expressed_genes(adata, ct, 0.1) for ct in receiver_celltypes])

# 3. Geneset of interest = DE genes in receiver (pseudobulk)
pdata = dc.get_pseudobulk(adata, sample_col="sample", groups_col="cell_type",
                          min_prop=0.1, min_smpls=3, layer="counts")
sc.pp.normalize_total(pdata, target_sum=1e4); sc.pp.log1p(pdata)
logFCs, pvals = dc.get_contrast(pdata, group_col="cell_type",
    condition_col="condition", condition="stim", reference="ctrl", method="t-test")
deg = dc.format_contrast_results(logFCs, pvals)
deg = deg[np.isin(deg["contrast"], receiver_celltypes)]
geneset_oi = deg[(deg["pvals"] <= 0.05) & (deg["logFCs"] > 1)]["name"].values
background_genes = deg["name"].values
```

```R
%%R -i geneset_oi -i background_genes -o ligand_activities
ligand_activities <- predict_ligand_activities(
    geneset = geneset_oi,
    background_expressed_genes = background_genes,
    ligand_target_matrix = ligand_target_matrix,
    potential_ligands = potential_ligands) %>%
    arrange(-aupr) %>% mutate(rank = rank(desc(aupr)))
```

- **What it demonstrates**: NicheNet's critical step is the geneset of interest — it must come from a condition contrast in the receiver, not from ligand-receptor expression. Pseudobulk before DE. Ligand activity = AUPR of regulatory potential vs geneset; rank ligands by AUPR.

## Reference Tables

### SCENIC pipeline at a glance

| Step | Tool | Input → Output | Purpose |
|---|---|---|---|
| Co-expression | `pyscenic grn` (GRNBoost) | loom → `adj.csv` (TF, target, importance) | Permissive TF→target candidates |
| Motif pruning | `pyscenic ctx` (cisTarget) | `adj.csv` + feather DB + motif2TF → `reg.csv` | Keep cis-regulated targets only |
| Activity scoring | `pyscenic aucell` | `reg.csv` + loom → cell × regulon AUC | Denoised TF activity per cell |

### CCC method categories

| Category | Question | Example tools | Output |
|---|---|---|---|
| Ligand-receptor | Is the interaction plausible? | CellPhoneDB, LIANA, NATMI, Connectome, CellChat | lr_means (magnitude) + p-values (specificity) |
| Intracellular | Does the ligand explain the response? | NicheNet, CellCall, Cellinker | ligand activity (AUPR) + target predictions |

### CCC assumptions & limitations

| Assumption | Limitation |
|---|---|
| Gene expression = protein abundance | Post-transcriptional regulation ignored |
| Local cell-cell signaling | Long-range/endocrine not captured |
| One-to-one interactions | Real signaling is many-to-many |
| Cell-type averaged expression | Single-cell resolution lost (most methods) |
| Steady-state for ligand-receptor | Cross-condition needs NicheNet or Tensor-cell2cell |

## Worked Example

**SCENIC on NeurIPS 2021 BMMC, donor s1d1 (6,224 cells, 22 cell types)** — subset to one donor to avoid batch confounding; use HVG ∪ all TFs; export to loom; run `pyscenic grn` → `ctx` → `aucell`. The resulting UMAP on AUCell coordinates separates cell types as well as the original expression UMAP. Cell-type-specific TFs identified: RUNX2 in pDCs, TCF7L2 in CD16+ Mono, LEF1 in CD4+ T activated. Cross-validate by checking TF expression (Reds matrixplot) matches regulon AUC (Blues heatmap).

**CCC on PBMC IFN-β stimulation (Kang et al., ~25k cells, 8 patients × 2 conditions)** — sub-sample to `stim` condition only for ligand-receptor; senders = CD4 T, B, FCGR3A+ Mono; receivers = CD8 T, CD14+ Mono, NK. CellPhoneDB finds HLA-B → CD8A/B (CD8 T specific) among top interactions; LIANA `rank_aggregate` consensus narrows to top-20 interactions. For NicheNet: pseudobulk per sample+cell type, t-test stim vs ctrl on CD8 T cells, take positive signif DE as geneset_oi. Predict top ligands; intersect with CellPhoneDB results to identify HLA-A → CD3G as a candidate functional interaction.

The complementarity is the key insight: ligand-receptor methods give you a list of plausible interactions but cannot tell you which ones actually drive the response. NicheNet tells you which ligands best explain the receiver's transcription, but doesn't itself infer ligand-receptor pairs. Run both, intersect the top candidates.

## Key Takeaways
1. **GRN inference is three steps: co-expression → motif pruning → activity scoring** — each layer removes false positives from the previous
2. **Run SCENIC per donor/batch** — co-expression across batches is confounded by batch effects
3. **AUCell is a better TF activity proxy than TF expression** — regulon target expression denoises
4. **Ligand-receptor methods answer plausibility; NicheNet answers functionality** — run both, intersect
5. **Use LIANA consensus rank, not single method** — benchmarked methods show low agreement
6. **Filter by 10% expression per cell type** — removes false positives from low-expressed ligands/receptors
7. **NicheNet needs a condition contrast** — pseudobulk per sample, then DE in receiver; single-cell DE inflates false positives
8. **CCC is hypothesis generation** — orthogonal validation (spatial co-localization, protein data, perturbation) is essential before treating an interaction as confirmed
9. **Ligand activity ≠ ligand expression** — high-AUPR ligands are those whose regulatory potential best predicts the geneset; they may not be the most highly expressed

## Connects To
- **Ch 2 (Preprocessing)**: SCENIC requires log1p-normalized data; CCC methods typically use raw or normalized counts depending on the method
- **Ch 5 (Conditions / DGE)**: NicheNet's geneset of interest comes from pseudobulk DGE — same preprocessing
- **Ch 8 (scATAC)**: SCENIC+ extends SCENIC with chromatin accessibility (peak-TF motif enrichment); DORC concept (Ch 8) complements regulon activity
- **Ch 9 (Spatial omics)**: spatial CCC validates predicted interactions by physical co-localization
- **Ch 11 (AIRR)**: immune cell CCC tools (CellChat, LIANA+) integrate receptor-ligand with AIRR data
