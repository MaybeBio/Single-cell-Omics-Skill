# Chapter 4: Trajectories

## Core Idea
Reconstruct dynamic biological processes from snapshot scRNA-seq by ordering cells along a pseudotime, inferring direction from unspliced/spliced mRNA kinetics (RNA velocity), or rebuilding cell-division history from heritable mutations (lineage tracing). Each approach has distinct assumptions: pseudotime needs a known root, RNA velocity needs the process timescale to match mRNA half-life, and lineage tracing needs heritable variation (engineered or natural).

## Frameworks Introduced

- **Four pseudotime construction principles**
  - When to use: any trajectory inference (TI) task — pick by topology of the process (linear / cyclic / branching / tree)
  - How:
    1. **Cluster approach** — cluster cells first, then connect clusters via similarity or MST
    2. **Graph approach** (PAGA) — partition graph into Leiden clusters, estimate inter-cluster connectivity; preserves global topology
    3. **Manifold-learning** (Slingshot) — fit principal curves through cells in high-dim space
    4. **Probabilistic** (DPT, Palantir) — assign cell-cell transition probabilities; DPT = random-walk distance, Palantir = Markov chain
  - Why: the right principle depends on whether the process is linear, cyclic, branching, or multifurcating; `dynguidelines` helps select

- **RNA velocity: steady-state vs EM model**
  - When to use: directed dynamics when unspliced/spliced counts are available (10x loom, velocyto, kb-python)
  - How:
    1. **Steady-state model** (`scvelo.tl.velocity(mode="deterministic")`) — linear regression on extreme quantiles of the phase portrait; assumes constant rates, common splicing rate across genes, observed steady states
    2. **EM model** (`scvelo.tl.recover_dynamics` → `mode="dynamical"`) — Expectation-Maximization infers gene-specific rates and latent cell time; no equilibrium or shared-splicing-rate assumption
  - Why: steady-state is fast but breaks when kinetics aren't at equilibrium (e.g. erythroid maturation); EM is broadly applicable but still assumes constant rates per gene

- **Lineage tracing with Cassiopeia (CRISPR/Cas9 evolving tracers)**
  - When to use: engineered target sites that accumulate indels over cell divisions; needs scRNA-seq + amplicon sequencing of target sites
  - How:
    1. Preprocess amplicon reads → **allele table** (cell × target-site indels as CIGAR strings)
    2. Convert allele table → **character matrix** (rows=cells, cols=target sites, values=indel categories; 0=uncut, -1=missing)
    3. Infer phylogeny: character-based (Max Parsimony, Max Likelihood, Bayesian) or distance-based (Neighbor-Joining, UPGMA); Cassiopeia also has VanillaGreedy / ILPSolver / HybridSolver
    4. Downstream: `compute_expansion_pvalues` (clades growing non-neutrally), `score_small_parsimony` (effective plasticity = state changes / tree size)
  - Why: lineage tracing gives ground-truth ancestry that snapshot RNA cannot; CRISPR/Cas9 systems scale to 10⁴+ cells with up to 100 characters

## Key Concepts
- **Pseudotime**: continuous ranking of cells by developmental stage; requires a root cell
- **Diffusion map**: non-linear dimensionality reduction capturing transition probabilities; DPT operates on diffusion components
- **Phase portrait**: scatter of unspliced vs spliced counts per gene; almond shape = expected kinetics; multiple kinetics per gene = caution
- **Unspliced vs spliced mRNA**: pre-mRNA (introns retained) vs mature mRNA; ratio informs transcriptional dynamics
- **Lineage tracing**: ancestry reconstruction from heritable marks
- **Prospective vs retrospective tracing**: prospective = introduce a mark and follow descendants (clone-level); retrospective = infer history from natural variation (subclonal resolution)
- **Clone**: all descendants of a single progenitor
- **Character matrix**: cells × target-sites table of indel identities (the input to phylogeny inference)
- **intBC**: integration barcode identifying each target site
- **Effective plasticity**: Fitch-Hartigan parsimony score / tree size; 0 = stable states, 1 = fully mixed

## Mental Models
- **Pseudotime orders, but doesn't direct** — it ranks cells by progress but says nothing about direction; root cell must be specified from biological knowledge
- **RNA velocity requires matching timescales** — works for pancreatic endocrinogenesis (~hours, matches mRNA half-life); fails for Alzheimer's/Parkinson's (~years) or steady-state blood
- **Always check phase portraits** — if they don't look almond-shaped, the model is wrong for that gene; subset to individual lineages if a gene has multiple pronounced kinetics
- **Don't trust projected velocity streams** — the 2D arrows depend heavily on (1) number of genes included and (2) plotting parameters; boundaries are unreliable
- **Lineage tracing ≠ trajectory inference** — TI orders snapshot cells; lineage tracing rebuilds the actual division tree
- **Apply ≥2 phylogeny algorithms** — character-based and distance-based methods highlight different parts of the lineage; compare
- **Effective plasticity captures state instability** — high plasticity = cells frequently switch transcriptional state across generations; often enriched in "mid-stage" populations

## Anti-patterns
- **Running TI without knowing the process topology** — linear methods fail on branching trajectories and vice versa; check `dynguidelines`
- **Trusting DPT root selection blindly** — root cell must come from biology (e.g. HSC cluster for hematopoiesis); pick by extreme diffusion component within the biologically-known root cluster
- **Applying RNA velocity to steady-state systems** — peripheral blood without transitions, or long-term diseases where timescale ≫ mRNA half-life
- **Skipping phase portrait QC** — multiple-kinetics genes silently produce wrong velocities; subset by lineage
- **Interpreting velocity stream projections quantitatively** — they are suggestive only; use CellRank for rigorous downstream analysis
- **Reconstructing low-quality tumors/clones** — filter on percent-unique, percent-cut, percent-exhausted, and clone size before tree inference
- **Using only one phylogeny algorithm** — different algorithms expose different lineages; greedy is fast, ILP/Hybrid is precise, NJ is a good distance-based comparison
- **Forgetting to compute indel priors** — high-frequency indels occur independently; without priors, unrelated cells may be wrongly grouped

## Code Examples

### DPT pseudotime on bone marrow

```python
import scanpy as sc

sc.pp.filter_genes(adata, min_counts=20)
sc.pp.normalize_total(adata)
sc.pp.log1p(adata)
sc.pp.highly_variable_genes(adata)
sc.tl.pca(adata)
sc.pp.neighbors(adata, n_pcs=10)

sc.tl.diffmap(adata)
# identify root cell: extreme value in diffusion component 3 within HSC cluster
root_ixs = adata.obsm["X_diffmap"][:, 3].argmin()
adata.uns["iroot"] = root_ixs
sc.tl.dpt(adata)

# compare DPT vs Palantir pseudotime on t-SNE
sc.pl.scatter(adata, basis="tsne",
              color=["dpt_pseudotime", "palantir_pseudotime"],
              color_map="gnuplot2")
```

- **What it demonstrates**: classic DPT workflow — diffusion map → biologically-informed root → dpt pseudotime. Always compare multiple pseudotimes; here DPT mis-ranks CLP cells (extreme high value) while Palantir increases monotonically with maturity.

### scVelo RNA velocity (EM model)

```python
import scvelo as scv

scv.pp.filter_and_normalize(adata, min_shared_counts=20, n_top_genes=2000)
sc.tl.pca(adata)
sc.pp.neighbors(adata)
scv.pp.moments(adata, n_pcs=None, n_neighbors=None)  # kNN smoothing

# EM model: recover dynamics → velocity → projection
scv.tl.recover_dynamics(adata, n_jobs=8)
scv.tl.velocity(adata, mode="dynamical")
scv.tl.velocity_graph(adata, n_jobs=8)
scv.pl.velocity_embedding_stream(adata, basis="umap")

# QC: rank genes by fit likelihood, inspect phase portraits
top_genes = adata.var["fit_likelihood"].sort_values(ascending=False).index
scv.pl.scatter(adata, basis=top_genes[:5], color="clusters", frameon=False)
```

- **What it demonstrates**: full EM pipeline. Phase portrait inspection is mandatory — `Pcsk2`, `Top2a`, `Ppp1r1a` show almond shapes; `Nfib` (two steady-state populations) and `Ghrl` (sparse) are low-confidence and would be downweighted by automated confidence scoring.

### Cassiopeia lineage reconstruction

```python
import cassiopeia as cas

# allele table → character matrix (with indel priors)
character_matrix, priors, state_to_indel = cas.pp.convert_alleletable_to_character_matrix(
    tumor_allele_table, allele_rep_thresh=0.9, mutation_priors=indel_priors)

tree = cas.data.CassiopeiaTree(character_matrix=character_matrix, priors=priors)
greedy_solver = cas.solver.VanillaGreedySolver()
greedy_solver.solve(tree)
cas.pl.plot_matplotlib(tree, orient="right", allele_table=tumor_allele_table)

# expansion detection (non-neutral subclones)
cas.tl.compute_expansion_pvalues(tree,
    min_clade_size=0.15 * tree.n_cell, min_depth=1)

# effective plasticity (Fitch-Hartigan parsimony / tree size)
parsimony = cas.tl.score_small_parsimony(tree, meta_item="Cluster-Name")
plasticity = parsimony / len(tree.nodes)
```

- **What it demonstrates**: the Cassiopeia pipeline. `allele_rep_thresh=0.9` filters cut sites where one allele dominates >90% (uninformative). Expansion p-values identify aggressive subclones; plasticity scores quantify transcriptional-state instability per cell.

## Reference Tables

### Pseudotime method selection

| Topology | Recommended method | Principle |
|---|---|---|
| Linear | Monocle / DPT | Probabilistic |
| Branching tree | PAGA, Slingshot, Palantir | Graph / manifold |
| Cyclic | Cycle stand-alone tools | (none of the above) |
| Multifurcating | Palantir | Markov chain |

### RNA velocity model comparison

| Model | Key assumption | When it fails |
|---|---|---|
| Steady-state (`mode="deterministic"`) | Constant rates, common splicing rate, observed equilibria | Erythroid maturation; non-equilibrium systems; genes with multiple kinetics |
| EM (`mode="dynamical"`) | Constant rates per gene (but gene-specific) | Same violation as above; non-almond phase portraits |

### Cassiopeia solver comparison

| Solver | Type | Scales to | Use case |
|---|---|---|---|
| VanillaGreedy | Character (greedy) | 10⁴+ cells | First-pass reconstruction |
| Neighbor-Joining | Distance | 10⁴+ cells | Independent comparison vs character-based |
| UPGMA | Distance | 10⁴+ cells | Faster than NJ; stricter assumptions |
| ILPSolver | Character (Steiner tree) | ~1.5k cells | Most precise; slow |
| HybridSolver | Hybrid | 10³–10⁴ cells | Balance of scalability + precision (recommended production choice) |

### Lineage tracing QC thresholds (KPTracer lung cancer)

| Statistic | Filter | Rationale |
|---|---|---|
| `NumCells` | ≥100 | Below: too few cells for tree inference |
| `PercentUnique` | >0.05 | Below: insufficient informative states |
| `PercentUnsaturatedTargets` | >0.20 | Below: too many exhausted target sites |
| intBCs per tumor | 5–30 (15–90 characters) | Information capacity for the clone |

## Worked Example

**Adult human bone marrow (pseudotime)** — compare DPT vs Palantir on the same data. DPT requires: filter genes (≥20 cells) → normalize → log1p → HVG → PCA(10 PCs) → neighbors → diffmap → set `iroot` to extreme of diffusion component 3 within HSC cluster → `dpt`. Visualize on t-SNE and as violin plots per cluster. DPT fails on CLP cluster (artifactual high values); Palantir pseudotime increases monotonically with developmental maturity. Conclusion: continue with Palantir pseudotime.

**Pancreatic endocrinogenesis (RNA velocity)** — Bastidas-Ponce et al. data, scVelo. Pre-endocrine cells (*Ductal*, *Ngn3 low/high EP*, *Pre-endocrine*) → four endocrine types (*Alpha*, *Beta*, *Delta*, *Epsilon*). Pipeline: filter_and_normalize → PCA → neighbors → moments → recover_dynamics → velocity(mode="dynamical") → velocity_graph → velocity_embedding_stream. Compare steady-state vs EM: EM captures Ductal cell cycle; steady-state shows "backflow" from Alpha to Pre-endocrine. Use CellRank for downstream fate analysis.

**KPTracer lung cancer (lineage tracing)** — KP (Kras/Tp53) mouse model with CRISPR/Cas9 evolving tracer, ~70k cells across 100+ tumors. Filter tumors by NumCells ≥100, PercentUnique >0.05, PercentUnsaturatedTargets >0.20. Reconstruct tumor `3726_NT_T1` (~10 intBCs, ~30 characters) with VanillaGreedySolver. Compute expansion p-values to identify aggressive subclades (threshold p<0.01, min_clade_size=15% of cells, min_depth=1). Compute effective plasticity via Fitch-Hartigan parsimony normalized by tree size; per-cell plasticity = mean of ancestor subtrees' plasticities. Finding: AT1-like and High-Plasticity states show elevated scPlasticity, suggesting they are intermediate states in tumor progression.

## Key Takeaways
1. **Match TI method to process topology** — linear, branching, cyclic, multifurcating need different algorithms; consult `dynguidelines`
2. **Pseudotime needs a known root** — specify from biology; pick within the expected root cluster using diffusion components
3. **Always compare pseudotimes** — different methods expose different artifacts; visualize on embedding + per-cluster violin plots
4. **RNA velocity needs matching timescales** — works for developmental processes spanning hours; fails for chronic disease or steady-state
5. **Phase portrait QC is non-negotiable** — if a gene has multiple kinetics or non-almond shape, subset to individual lineages or exclude
6. **Don't over-interpret projected velocity streams** — boundary effects and plotting parameter sensitivity; use CellRank for downstream
7. **Filter lineage tracing clones before reconstruction** — too few cells, too few unique states, or saturated targets produce unreliable trees
8. **Apply ≥2 phylogeny algorithms** — character-based and distance-based highlight different parts of the lineage
9. **Effective plasticity quantifies state instability** — parsimony score per cell over ancestor subtrees; high plasticity often marks transitional states

## Connects To
- **Ch 3 (Cellular structure)**: clustering + annotation precedes trajectory inference; cluster labels feed into lineage-tracing metadata
- **Ch 6 (GRNs)**: trajectory-inferred cell order is input to SCENIC+ / dynamical GRN inference
- **Ch 8 (scATAC)**: paired RNA+ATAC lineage analysis uses pseudotime to order chromatin accessibility changes
- **Ch 11 (AIRR)**: B/T cell clonal lineage tracing uses V(D)J recombination rather than CRISPR editing; same Cassiopeia-style character matrix concept applies
