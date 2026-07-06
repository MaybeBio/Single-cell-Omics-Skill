# Chapter 11: Adaptive Immune Receptor (AIR) Profiling

## Core Idea
AIRR sequencing (TCR/BCR) captures the cellular identity of adaptive immune cells through V(D)J recombination — each cell's receptor sequence is a unique, heritable barcode that reveals clonal lineage, expansion history, and antigen specificity. scirpy extends AnnData to store AIR data alongside transcriptomics, enabling integrated analysis of immune repertoire (clonotype definition, expansion, gene usage, spectratype, specificity) and gene expression. AIR sequence determines epitope specificity; cells with similar receptors bind the same antigen.

## Frameworks Introduced

- **scirpy AIR data loading + chain_qc**
  - When to use: any AIRR analysis starting from Cell Ranger `filtered_contig_annotations.csv`
  - How: `ir.io.read_10x_vdj("contigs.csv")` → `ir.pp.merge_with_ir(adata, adata_air)` (left join: only cells with GEX kept); `ir.tl.chain_qc(adata)` annotates `chain_pairing`: `single pair` / `orphan VJ` / `orphan VDJ` / `two full chains` / `extra VJ` / `extra VDJ` / `multichain`; filter `single pair` + `orphan VDJ` (T) or `orphan VJ` (B) for clean analysis; `multichain` = doublets/clonal expansion artifacts
  - Why: scirpy stores AIR in `adata.obs` (columns `IR_VJ_1_*`, `IR_VDJ_1_*`, `IR_VJ_2_*`, `IR_VDJ_2_*`) rather than a separate modality — left join keeps GEX+AIR cells, drops cells with AIR but no GEX (no transcriptome to analyze)

- **Clonotype definition via sequence distance**
  - When to use: after chain_qc filtering, before expansion/diversity analysis
  - How:
    1. `ir.pp.ir_dist(adata, metric="identity", sequence="aa")` — compute CDR3 distance matrix (also supports `"alignment"` for BLOSUM-based, `"tcrdist"` for TCRdist)
    2. `ir.tl.define_clonotype_clusters(adata, sequence="aa", metric="identity", receptor_arms="all", dual_ir="primary_only", key_added="cc_aa_identity")` — cluster by CDR3 identity
    3. `ir.tl.clonotype_network(adata, min_cells=3, min_nodes=3, sequence="aa", metric="identity")` — build network layout
    4. `ir.pl.clonotype_network(adata, color="cell_type")` — visualize
  - Parameters: `receptor_arms` = `VJ` (α only) / `VDJ` (β only) / `all` (both must match) / `any` (either); `dual_ir` = `primary_only` / `any` / `all` (controls secondary receptor)
  - Why: clonotype = cells descending from a common recombination event; identical CDR3α+CDR3β is the strictest definition (identity); distance metrics (alignment/tcrdist) catch near-matches from shared specificity but introduce false positives

- **BCR clonotype via Dandelion (somatic hypermutation-aware)**
  - When to use: BCR analysis where somatic hypermutation (SHM) means identical CDR3 is too strict
  - How: `vdjx = ddl.from_scirpy(adata)` → clean NA in v_call/j_call/junction_aa → `ddl.pp.calculate_threshold(vdjx, model="hh_s5f")` (finds bimodal distance breakpoint) → `ddl.tl.define_clones(vdjx, key_added="changeo_clone_id", model="hh_s5f")` → `ddl.tl.generate_network(vdjx, key="sequence_alignment", layout_method="sfdp", clone_key="changeo_clone_id")` → `ddl.tl.transfer(adata, vdjx, clone_key="changeo_clone_id")`
  - Why: BCRs undergo ~10,000× higher mutation rate than germline via somatic hypermutation; identical-CDR3 misses clonally related cells that have diverged; the hh_s5f substitution model (5-mer context) computes realistic distances; threshold = first-mode/second-mode breakpoint separates clonally related from singletons

- **Clonal expansion + diversity quantification**
  - When to use: after clonotype definition, comparing conditions (e.g. disease vs healthy)
  - How: `ir.tl.clonal_expansion(adata, target_col="clone_id", breakpoint=1, key_added="clonal_expansion")` → categories `1` / `2` / `>= 4` / `>= 8`; `ir.pl.clonal_expansion(adata, groupby="patient_id", target_col="clone_id")`; `ir.pl.alpha_diversity(adata, groupby="cell_type", target_col="clone_id")` — Shannon/normalized Shannon entropy per group; `ir.pl.group_abundance(adata, groupby="clone_id", target_col="cell_type", normalize=True, max_cols=10)`
  - Why: expansion size proxies antigen-driven response; diversity (Shannon entropy) inversely correlates with oligoclonal expansion; per-condition/per-cell-type stratification is essential

- **Gene segment usage + spectratype + motif analysis**
  - When to use: characterizing V(D)J segment preferences and CDR3 length distributions
  - How:
    1. Gene usage: `ir.pl.group_abundance(adata, groupby="IR_VJ_1_v_call", target_col="cell_type", normalize=True)`; `ir.pl.vdj_usage(adata, full_combination=False, max_ribbons=30)` — Sankey-style V→(D)→J ribbon plot
    2. Spectratype: `ir.pl.spectratype(adata, cdr3_col="IR_VDJ_1_junction_aa", color="cell_type", viztype="bar"|"curve")` — CDR3 length distribution; immunodominance = peak length
    3. Motif: `from palmotif import compute_motif, svg_logo`; `motif = compute_motif(seqs_list)`; `svg_logo(motif, "logo.svg", color_scheme="taylor")` — position-weighted amino acid logo
  - Why: selective V-gene usage is a hallmark of antigen-driven response; spectratype peaks reveal immunodominant CDR3 lengths (often 15 aa for TCRβ); motif logos reveal conserved positions (C-A-S start, F/W end for TCRβ)

- **Repertoire overlap (sample-sample comparison)**
  - When to use: comparing repertoires across patients/samples
  - How: `df, dst, lk = ir.tl.repertoire_overlap(adata, "patient_id", target_col="cc_aa_identity", inplace=False)` — `df` = clonotype×sample abundance, `dst` = Jaccard distance matrix, `lk` = linkage; `ir.pl.repertoire_overlap(adata, "patient_id", target_col="cc_aa_identity", heatmap_cats=["Centre"])` — clustered heatmap; `ir.pl.repertoire_overlap(adata, "patient_id", pair_to_plot=["s1","s2"])` — scatter of clonotype sizes
  - Why: shared clonotypes across patients suggest common antigen exposure (e.g. SARS-CoV-2); Jaccard distance + hierarchical clustering groups similar repertoires

- **Specificity inference (3 approaches)**
  - When to use: predicting which antigen each TCR/BCR binds
  - How (3 paths, decreasing reliability):
    1. **Database query**: `vdjdb = ir.datasets.vdjdb()`; `ir.pp.ir_dist(adata, vdjdb, metric="identity", sequence="aa")`; `ir.tl.ir_query(adata, vdjdb, metric="identity", sequence="aa", receptor_arms="VDJ", dual_ir="primary_only")`; `ir.tl.ir_query_annotate(adata, vdjdb, include_ref_cols=["antigen.species", "antigen.epitope"], suffix="_VDJ")` — strictness: `VDJ` < `VJ` (more matches, ambiguous) < `all` (fewest matches, highest confidence)
    2. **Distance-based clustering**: TCRdist (BLOSUM-based, all CDRs, requires V-gene + CDR3) — `from tcrdist.repertoire import TCRrep`; `tr = TCRrep(cell_df=df, organism="human", chains=["alpha","beta"])`; `dist = tr.pw_alpha + tr.pw_beta`; cutoff 120 = related; TCRmatch (k-mer CDR3β only, less strict) — `./tcrmatch -i input.csv -t 1 -d db.csv -s 0.9`
    3. **Epitope prediction**: ERGO-II (ML model, requires CDR3β, optional α+V/J genes+MHC+peptide); categorical (fixed epitope set) vs epitope-sequence models (arbitrary epitopes, lower accuracy on unknowns)
  - Information importance ranking (Springer et al.): **CDR3β > V/J-gene > CDR3α > MHC > cell type**
  - Why: databases are biased towards well-studied diseases (CMV/EBV/SARS-CoV-2); distance metrics catch similar-but-not-identical TCRs (more hits, more false positives); epitope predictors are unbenchmarked and biased — use with caution, validate orthogonally

- **Multimodal AIR+GEX integration**
  - When to use: paired scRNA+AIRR data; want shared embedding/clusters
  - How (uni-modal first, multimodal second):
    1. **Uni-modal with multimodal conditions**: merge via `ir.pp.merge_with_ir(adata_tc, adata_tcr)` → use GEX clusters (Leiden, marker genes like IFNG, cell types) as grouping for AIR analysis (spectratype, diversity); or use AIR clusters (clonotypes, disease specificity from database query) as grouping for GEX DGE (`sc.tl.rank_genes_groups`)
    2. **Multimodal models** (novel, not standard practice):
       - **TESSA** (TCR only): Bayesian — pretrained TCR autoencoder (30-dim) → upweight dimensions to correlate with GEX → clusters of similar TCR+GEX
       - **CoNGA** (TCR only): graph-theoretic — TCRDist graph + GEX graph; "CoNGA clusters" share neighbors in both
       - **mvTCR** (TCR): VAE — Transformer (TCR) + MLP (GEX) → joint embedding; hyperparameter `pseudo_metric` with `prediction_labels={"clonotype": 1, "full_clustering": 1}`
       - **Benisse** (BCR only): pretrained BCR encoder + sparse graph from GEX+BCR similarity
  - Why: AIR (sequence) and GEX (count matrix) have incompatible structures — can't directly concatenate; shared embedding captures both specificity (AIR) and state (GEX); models are experimental, not independently benchmarked

## Key Concepts
- **AIR (Adaptive Immune Receptor)**: TCR (αβ or γδ) or BCR (heavy+light chain); generated by V(D)J recombination
- **V(D)J recombination**: somatic assembly of Variable-(Diversity)-Joining gene segments; creates combinatorial + junctional diversity (P/N nucleotides, deletions)
- **CDR1/2/3**: complementarity-determining regions; CDR3 (junction) is most variable and antigen-contacting
- **Productive AIR**: in-frame, no premature stop codons; only productive chains encode functional receptors
- **chain_pairing (scirpy)**: classification of cell's AIR status — `single pair`, `orphan VJ` (α only), `orphan VDJ` (β only), `extra VJ/VDJ` (secondary receptor), `multichain` (doublet/artifact)
- **Clonotype**: cells descending from one recombined precursor; defined by identical CDR3 (strict) or sequence distance (relaxed)
- **Clonal expansion**: number of cells per clonotype; proxies antigen-driven proliferation
- **Spectratype**: distribution of CDR3 amino-acid lengths; peaks = immunodominant lengths
- **Somatic hypermutation (SHM)**: BCR-only; ~10,000× germline mutation rate in V-gene during affinity maturation
- **Affinity maturation**: B-cell process where SHM + selection produces high-affinity BCRs
- **TCRdist**: BLOSUM-based distance metric using all CDRs; cutoff 120 = same specificity cluster
- **TCRmatch**: k-mer-based CDR3β-only distance; threshold 0.9 (medium) / 0.97 (high confidence)
- **Isotype**: BCR constant region class (IgM/IgG/IgA/IgE); indicates maturation stage
- **repertoire_overlap**: Jaccard distance between sample clonotype sets; hierarchical clustering groups similar repertoires
- **ERGO-II**: ML epitope-binding predictor; categorical (fixed epitopes) vs epitope-sequence (arbitrary, lower accuracy on unknowns)

## Mental Models
- **AIR is a heritable barcode** — clonally related cells share identical CDR3; expansion size proxies antigen-driven response
- **CDR3β dominates specificity** — β-chain CDR3 contacts pMHC most; querying by VDJ-only catches most specificity but increases ambiguity
- **Stricter query = fewer hits, higher confidence** — `receptor_arms="all"` < `"VDJ"` < `"VJ"` in match count; choose based on precision vs recall needs
- **BCRs need distance-based clonotyping** — somatic hypermutation makes identical-CDR3 too strict; Dandelion's hh_s5f model + bimodal threshold separates clonally related from singletons
- **Specificity inference has no ground truth** — databases are biased (CMV/EBV/HIV/SARS-CoV-2 overrepresented); distance metrics aren't independently benchmarked; epitope predictors are unvalidated — use 3 approaches, intersect, validate experimentally
- **Left join keeps GEX+AIR cells** — cells with AIR but no GEX are dropped (no transcriptome to analyze); cells with GEX but no AIR are kept (often non-lymphocytes)
- **Uni-modal-with-conditions is the practical default** — multimodal models (TESSA/CoNGA/mvTCR/Benisse) are experimental; standard practice is GEX-defined groups for AIR analysis or vice versa
- **AIR complements GEX for immune cells** — GEX tells cell state; AIR tells specificity/fate; both interlinked (same AIR → similar phenotype)

## Anti-patterns
- **Skipping chain_qc before clonotype definition** — `multichain` cells are doublets/artifacts and contaminate clonotype networks
- **Defining BCR clonotypes by identical CDR3** — SHM makes this too strict; use Dandelion distance-based definition
- **Querying only `receptor_arms="all"`** — strictest, misses real matches; start with `VDJ` for TCR (highest specificity signal), then verify with `all`
- **Trusting database query annotations without ambiguity check** — cells matching multiple diseases return `ambiguous`; don't drop these, label them and investigate
- **Using epitope predictors without checking training data** — if your epitope isn't in training, categorical models can't predict it; epitope-sequence models have low accuracy on unknowns
- **Treating AIR clusters as ancestral trees** — distance-based clusters share specificity, not ancestry; only identical-CDR3 clonotypes are ancestrally related
- **Multimodal integration without validation** — TESSA/CoNGA/mvTCR/Benisse are novel, not benchmarked; treat outputs as hypotheses
- **Forgetting to trim C/F from CDR3β for TCRmatch** — Cell Ranger includes leading C and trailing F/W; TCRmatch expects them removed
- **Substituting V-gene subtype as `*01` blindly** — TCRdist requires subtype; if you don't have it, `*01` is a heuristic but may bias distances
- **Confusing clonal expansion with specificity** — large clones are antigen-driven but you don't know which antigen without database query or prediction
- **Comparing spectratypes across cell types without normalization** — use `normalize="cell_type"` to compare distributions, not raw counts

## Code Examples

### scirpy loading + chain_qc + clonotype definition

```python
import scirpy as ir
import scanpy as sc

# Load AIR data and merge with GEX (left join: keep GEX cells)
adata_air = ir.io.read_10x_vdj("filtered_contig_annotations.csv")
ir.pp.merge_with_ir(adata, adata_air)

# QC: classify chain pairing
ir.tl.chain_qc(adata)
adata.obs["chain_pairing"].value_counts()
# Filter to clean T cells: single pair or orphan VDJ (β-only)
adata = adata[adata.obs["chain_pairing"].isin(["single pair", "orphan VDJ"])].copy()

# Define clonotypes by identical CDR3α+CDR3β
ir.pp.ir_dist(adata, metric="identity", sequence="aa")
ir.tl.define_clonotype_clusters(
    adata, sequence="aa", metric="identity",
    receptor_arms="all", dual_ir="primary_only",
    key_added="cc_aa_identity",
)
ir.tl.clonotype_network(adata, min_cells=3, min_nodes=3,
                       sequence="aa", metric="identity")
ir.pl.clonotype_network(adata, color="cell_type", base_size=10)
```

- **What it demonstrates**: the canonical scirpy chain — load → merge → chain_qc filter → distance → clonotype clusters → network visualization. `min_cells=3, min_nodes=3` keeps only expanded clonotype clusters worth plotting.

### Expansion, diversity, gene usage, spectratype, motif

```python
# Clonal expansion (categories: 1, 2, >= 4, >= 8)
ir.tl.clonal_expansion(adata, target_col="cc_aa_identity", key_added="expansion")
ir.pl.clonal_expansion(adata, groupby="patient_id", target_col="cc_aa_identity")

# Alpha diversity (Shannon entropy per group)
ir.pl.alpha_diversity(adata, groupby="cell_type", target_col="cc_aa_identity")

# Clonotype abundance per cell type
ir.pl.group_abundance(adata, groupby="cc_aa_identity",
                     target_col="cell_type", normalize=True, max_cols=10)

# V-gene usage
ir.pl.group_abundance(adata, groupby="IR_VJ_1_v_call",
                     target_col="cell_type", normalize=True, max_cols=10)
ir.pl.vdj_usage(adata, full_combination=False, max_ribbons=30)

# Spectratype (CDR3 length distribution)
ir.pl.spectratype(adata, cdr3_col="IR_VDJ_1_junction_aa",
                 color="cell_type", viztype="curve",
                 curve_layout="shifted", kde_kws={"kde_norm": False})

# Motif logo for top V-gene + length-15 CDR3β
from palmotif import compute_motif, svg_logo
seqs = adata[
    (adata.obs["IR_VDJ_1_v_call"].isin(["TRBV5-1", "TRBV11-2"]))
    & (adata.obs["IR_VDJ_1_junction_aa"].str.len() == 15)
].obs["IR_VDJ_1_junction_aa"].tolist()
motif = compute_motif(seqs)
svg_logo(motif, "logo.svg", color_scheme="taylor")

# Repertoire overlap across patients
df, dst, lk = ir.tl.repertoire_overlap(
    adata, "patient_id", target_col="cc_aa_identity", inplace=False)
ir.pl.repertoire_overlap(adata, "patient_id",
                        target_col="cc_aa_identity", heatmap_cats=["Centre"])
```

- **What it demonstrates**: the full clonotype analysis toolkit — expansion (size), diversity (entropy), abundance (top clonotypes per cell type), V(D)J usage (segment preference), spectratype (length distribution), motif (sequence logo), repertoire overlap (sample similarity). All plot functions accept `groupby`/`target_col`/`normalize` for stratification.

### BCR analysis with Dandelion (SHM-aware)

```python
import dandelion as ddl

vdjx = ddl.from_scirpy(adata_bcr)

# Clean NA in required columns
for col in ["v_call", "j_call", "junction_aa"]:
    vdjx.data[col].replace("", np.nan, inplace=True)
    vdjx.data.dropna(subset=[col], inplace=True)
vdjx.data["junction_length"] = [len(a) for a in vdjx.data["junction_aa"]]

# Find distance threshold (bimodal: clonally related vs singletons)
ddl.pp.calculate_threshold(vdjx, model="hh_s5f", plot=False)
# Define clones by distance
ddl.tl.define_clones(vdjx, key_added="changeo_clone_id", model="hh_s5f")
ddl.tl.generate_network(vdjx, key="sequence_alignment",
                       layout_method="sfdp", clone_key="changeo_clone_id")
ddl.tl.transfer(adata_bcr, vdjx, clone_key="changeo_clone_id")

# Visualize
edgeweights = [1/(e+1) for e in ddl.tl.extract_edge_weights(vdjx)]
ddl.pl.clone_network(adata_bcr, color=["isotype_status"],
                    edges_width=edgeweights, size=50)
```

- **What it demonstrates**: BCR-specific clonotype definition via Dandelion — the `hh_s5f` 5-mer substitution model computes realistic distances that account for somatic hypermutation context. Bimodal threshold separates clonally related sequences (mode 1) from singletons (mode 2).

### Specificity: database query + TCRdist + ERGO-II

```python
# Database query (VDJdb) with various strictness
vdjdb = ir.datasets.vdjdb()
metric, sequence = "identity", "aa"
ir.pp.ir_dist(adata, vdjdb, metric=metric, sequence=sequence)

# Strict: CDR3β only
ir.tl.ir_query(adata, vdjdb, metric=metric, sequence=sequence,
              receptor_arms="VDJ", dual_ir="primary_only")
ir.tl.ir_query_annotate(adata, vdjdb, metric=metric, sequence=sequence,
                       include_ref_cols=["antigen.species", "antigen.epitope"],
                       suffix="_VDJ")
adata.obs["antigen.species_VDJ"].value_counts()
# Returns: CMV, EBV, SARS-CoV-2, ... or "ambiguous" if multiple diseases match

# Stricter: both chains
ir.tl.ir_query(adata, vdjdb, metric=metric, sequence=sequence,
              receptor_arms="all", dual_ir="primary_only")
ir.tl.ir_query_annotate(adata, vdjdb, ..., suffix="_fullIR")

# TCRdist for clustering by similarity (not identity)
from tcrdist.repertoire import TCRrep
df_tcrdist = adata.obs[["IR_VJ_1_junction_aa", "IR_VJ_1_v_call",
                       "IR_VDJ_1_junction_aa", "IR_VDJ_1_v_call"]].copy()
df_tcrdist = df_tcrdist.rename(columns={
    "IR_VJ_1_junction_aa": "cdr3_a_aa", "IR_VJ_1_v_call": "v_a_gene",
    "IR_VDJ_1_junction_aa": "cdr3_b_aa", "IR_VDJ_1_v_call": "v_b_gene",
})
df_tcrdist["v_a_gene"] = df_tcrdist["v_a_gene"].astype(str) + "*01"
df_tcrdist["v_b_gene"] = df_tcrdist["v_b_gene"].astype(str) + "*01"
tr = TCRrep(cell_df=df_tcrdist, organism="human", chains=["alpha", "beta"])
dist_total = tr.pw_alpha + tr.pw_beta  # pairwise distances
# Cutoff 120 = same specificity cluster (60 per chain)
```

- **What it demonstrates**: 3-tier specificity inference — strict identity DB query (highest confidence, fewest hits), relaxed DB query via `alignment` metric (more hits, more false positives), and TCRdist clustering (groups by similarity, not identity). Always check for `ambiguous` annotations and intersect results across methods.

### Multimodal: GEX-defined groups for AIR analysis

```python
# Merge GEX + AIR (left join: keep GEX cells, drop AIR-only)
ir.pp.merge_with_ir(adata_tc, adata_tcr)

# Standard GEX clustering
sc.pp.neighbors(adata_tc); sc.tl.leiden(adata_tc); sc.tl.umap(adata_tc)
sc.pl.umap(adata_tc, color=["full_clustering", "has_ir", "leiden"], ncols=1)

# AIR analysis on GEX-defined cluster (e.g. spectratype by Leiden)
ir.pl.spectratype(adata_tc, color="leiden", viztype="curve",
                 curve_layout="shifted")

# Marker-gene-defined group: IFNG-high T cells
adata_tc.obs["elevated_IFNG"] = (adata_tc[:, "IFNG"].X.todense() > 3).astype(str)

# Motif comparison: IFNG-high vs background
from palmotif import compute_pal_motif, svg_logo
seqs_high = adata_tc[adata_tc.obs["elevated_IFNG"] == "True"].obs["IR_VDJ_1_junction_aa"]
seqs_back = adata_tc[adata_tc.obs["elevated_IFNG"] == "False"].obs["IR_VDJ_1_junction_aa"]
centroid = adata_tc.obs["IR_VDJ_1_junction_aa"].value_counts().index[0]
motif_high, _ = compute_pal_motif(seqs_high, centroid=centroid)
motif_back, _ = compute_pal_motif(seqs_back, centroid=centroid)

# Reverse: DGE on AIR-defined cluster
sc.tl.rank_genes_groups(adata_tc, groupby="elevated_IFNG")
```

- **What it demonstrates**: the practical multimodal workflow — merge, cluster on GEX, then run AIR analysis (spectratype, motif) on GEX-defined groups; or cluster on AIR (clonotype, disease annotation) and run GEX DGE. Avoids experimental multimodal models (TESSA/CoNGA/mvTCR) which aren't standard practice yet.

## Reference Tables

### scirpy chain_pairing categories

| Category | Meaning | Action |
|---|---|---|
| `single pair` | One VJ + one VDJ (productive) | Keep — clean singlet |
| `orphan VJ` | Only α-chain (TCR) or light-chain (BCR) | Keep for B; review for T |
| `orphan VDJ` | Only β-chain (TCR) or heavy-chain (BCR) | Keep for T; review for B |
| `extra VJ` / `extra VDJ` | Secondary receptor | Keep — biologically real |
| `two full chains` | Two αβ pairs | Review — allelic inclusion |
| `multichain` | Many chains | Drop — doublet/artifact |

### Clonotype definition strictness

| `receptor_arms` | `dual_ir` | Match criterion | Strictness |
|---|---|---|---|
| `VDJ` | `primary_only` | CDR3β only | Lowest (most matches, most ambiguous) |
| `VJ` | `primary_only` | CDR3α only | Low (α less variable → more matches) |
| `all` | `primary_only` | CDR3α AND CDR3β | Highest (fewest matches, highest confidence) |
| `any` | `primary_only` | Either chain | Lowest |

### Specificity inference: 3 approaches compared

| Approach | Input | Output | Pros | Cons |
|---|---|---|---|---|
| Database query (identity) | CDR3 (any chain) | Antigen.species, antigen.epitope | Fast, reproducible | Biased to well-studied diseases; misses unknowns |
| Distance-based (TCRdist/TCRmatch) | CDR3 + V-gene | Clonotype clusters by similarity | Catches near-matches; no DB bias | No epitope annotation; cutoffs arbitrary |
| Epitope prediction (ERGO-II) | CDR3 (+V/J/MHC/peptide) | Binding score per epitope | Works for any epitope (sequence model) | Unbenchmarked; biased training data; high FP |

### Information importance for TCR specificity (Springer et al.)

| Rank | Element | Notes |
|---|---|---|
| 1 | CDR3β | Most antigen contact; primary identifier |
| 2 | V- and J-gene | Determine CDR1/2 + framework |
| 3 | CDR3α | Secondary contact |
| 4 | MHC class | Restricts epitope presentation |
| 5 | Cell type | Minimal added info |

### Multimodal AIR+GEX integration methods

| Method | Modality | Approach | Status |
|---|---|---|---|
| Uni-modal with conditions | TCR/BCR + GEX | GEX clusters for AIR analysis or vice versa | Standard practice |
| TESSA | TCR + GEX | Bayesian; TCR autoencoder + GEX weighting | Experimental (TCR only) |
| CoNGA | TCR + GEX | Graph-theoretic; TCRDist + GEX graphs | Experimental (TCR only) |
| mvTCR | TCR + GEX | VAE; Transformer + MLP | Experimental (TCR only) |
| Benisse | BCR + GEX | Pretrained BCR encoder + sparse graph | Experimental (BCR only) |

### BCR-specific: Dandelion substitution models

| Model | Organism | Use |
|---|---|---|
| `hh_s5f` | Human | 5-mer context, S5F model (Yaari 2013) |
| `hs1f` | Human | Single-nucleotide substitution |
| `m1n` | Mouse | Single-nucleotide substitution (Cui 2016) |

## Worked Example

**Haniffa Lab SARS-CoV-2 PBMCs (Weizmann/Broad/Cambridge/Newcastle)** — 750k cells, 130 patients, COVID-19 + healthy controls; scRNA-seq + scTCR/scBCR (10x Genomics). TCR analysis sub-sampled to patients `CV0902` + `AP6`.

Pipeline:
1. Load `TCR_filtered.h5ad` (post-QC) → chain_qc → filter `single pair` + `orphan VDJ` (T cells)
2. Clonotype definition: `ir.pp.ir_dist(metric="identity", sequence="aa")` → `ir.tl.define_clonotype_clusters(receptor_arms="all", dual_ir="primary_only", key_added="cc_aa_identity")` → `ir.tl.clonotype_network(min_cells=3, min_nodes=3)` → network shows clonal expansion hotspots
3. Clonal expansion: `ir.tl.clonal_expansion` → CD8+ effector T cells have largest clones (>= 8 cells); `ir.pl.alpha_diversity(groupby="full_clustering")` → proliferating/effector CD8+ have lowest diversity (expected — oligoclonal expansion)
4. Gene usage: `ir.pl.vdj_usage` → TRBD2 only D segment detected (donor-specific); `ir.pl.group_abundance(groupby="IR_VJ_1_v_call")` → TRAV19 in CD8+, TRAV29/DV5 in CD4+ (selective V-gene usage)
5. Spectratype: `ir.pl.spectratype(cdr3_col="IR_VDJ_1_junction_aa", viztype="bar")` → CDR3β length 15 dominant (CD4+ + CD8+); gdT cells show heterogeneous lengths
6. Motif: `compute_motif` on TRBV5-1/TRBV11-2 + length-15 → C-A-S start, F end conserved; position 14 = Y/F/H/T variable
7. Repertoire overlap: `ir.tl.repertoire_overlap("patient_id")` → COVID-066 + CV0902 most similar (Jaccard) despite different centers
8. Specificity (DB query): `vdjdb = ir.datasets.vdjdb()`; query `receptor_arms="VDJ"` → CMV + EBV dominant (latent infections common); some HIV-1 (donor-specific); query `receptor_arms="all"` → fewer hits, higher confidence
9. BCR via Dandelion: `ddl.from_scirpy(adata_bcr)` → `calculate_threshold(model="hh_s5f")` → `define_clones` → `generate_network(layout_method="sfdp")` → IGHV3-48 + IGHV1-18 dominant in IgG isotype (affinity-matured)

The complementarity between GEX and AIR is the key insight: GEX-defined CD8+ effector cluster + largest clonal expansion + CMV-annotated specificity = strong evidence of antigen-driven CMV-response CD8+ T cells. Neither modality alone gives this complete picture.

## Key Takeaways
1. **AIR = heritable barcode for adaptive immune cells** — identical CDR3 = clonally related; expansion size proxies antigen response
2. **chain_qc is the first step** — `multichain` cells are doublets/artifacts; filter before clonotype definition
3. **BCR needs distance-based clonotyping** — somatic hypermutation makes identical-CDR3 too strict; use Dandelion's hh_s5f model + bimodal threshold
4. **CDR3β dominates specificity** — query by `receptor_arms="VDJ"` first (best signal-to-noise); verify with `receptor_arms="all"` for highest confidence
5. **Three specificity approaches have orthogonal biases** — DB query (disease bias), distance (no annotation), prediction (unbenchmarked) — run multiple, intersect, validate
6. **Information importance: CDR3β > V/J > CDR3α > MHC > cell type** — choose query/clustering features accordingly
7. **Uni-modal-with-conditions is the practical default** — TESSA/CoNGA/mvTCR/Benisse are experimental, not standard practice
8. **AIR complements GEX** — GEX = cell state; AIR = specificity/fate; both interlinked (same AIR → similar phenotype)
9. **Left join when merging GEX+AIR** — keep GEX cells (some are non-lymphocytes); drop AIR-only cells (no transcriptome)
10. **Validate specificity experimentally** — pMHC multimers (TCR) or antigen-tagged feature barcodes (BCR); computational predictions are hypotheses

## Connects To
- **Ch 1 (AnnData)**: scirpy stores AIR data in `adata.obs` (not a separate modality); MuData for paired multi-modal
- **Ch 2 (Preprocessing)**: GEX QC + AIR QC are independent; merge after QC on both
- **Ch 5 (Conditions)**: DGE on AIR-defined groups (clonotypes, disease-specific cells) uses pseudobulk per sample+cell type
- **Ch 6 (CCC)**: AIR-defined cell types (clonotypes) can refine CCC analysis; ligand-receptor methods don't use AIR directly
- **Ch 10 (Surface Protein)**: ADT + AIR combined for immune cell annotation; ADT markers (CD3/CD19) corroborate AIR-based cell type
- **Ch 12 (Multimodal Integration)**: TESSA/CoNGA/mvTCR/Benisse are AIR-specific multimodal models; totalVI/MOFA+ handle ADT+RNA but not AIR sequences
