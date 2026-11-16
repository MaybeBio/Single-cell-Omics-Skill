# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A standalone knowledge base combining three layers:

1. **Source material** (`sc-best-practices/`) — a read-only snapshot of the "Single-cell best practices" book (theislab/single-cell-best-practices, v0.2.0, 2026-06-11), the published companion to Heumos, Schaar, Lance et al., *Nat Rev Genet* (2023), https://doi.org/10.1038/s41576-023-00586-w.
2. **Tool documentation** (`manuals/`) — curated tool catalogs (seandavi/awesome-single-cell, mdozmorov/sc{RNA,ATAC,HiC}-notes, crazyhottommy/awesome_spatial_omics) plus the official **anndata** and **Scanpy** API documentation.
3. **Distilled skill** (root-level `SKILL.md` + `chapters/` + `glossary.md` + `patterns.md` + `cheatsheet.md`) — a structured agent skill generated from the sources via the `book-to-skill` converter, designed for fast lookup during single-cell multi-omics analysis.

This is **not** a git repo, has no build system, no tests, no dependency files, and no environment yml. The notebooks in `sc-best-practices/` reference datasets loaded via `lamindb` that are not in this snapshot, so they cannot be executed here. The distilled skill files are reference content to be consulted, not executed.

## Layout

### Distilled skill (root level)

| File | Purpose |
|---|---|
| `SKILL.md` | Master skill file with frontmatter, 18 Core Frameworks, Chapter Index, Topic Index, Scope & Limits |
| `chapters/ch01-…` to `ch24-…` | 24 chapter summaries following a fixed structure: Core Idea → Frameworks → Key Concepts → Mental Models → Anti-patterns → Code Examples → Reference Tables → Worked Example → Key Takeaways → Connects To |
| `glossary.md` | 50+ key terms with chapter pointers (ADT, AnnData, AUCell, batch effect, CDR3, CLR, clonotype, dsb, HVG, Harmony, Leiden, MAD, Moran's I, MuData, NicheNet, pseudobulk, scIB, scirpy, scVI, SHM, TCRdist, totalVI, UMI, VDJ, WNN) |
| `patterns.md` | Concrete techniques by task: QC, normalization, clustering, integration, DGE, trajectory, mechanism, spatial, multimodal, perturbation |
| `cheatsheet.md` | Decision rules and lookup tables: QC thresholds per modality, normalization decision tree, integration method selection, scIB metrics, anti-patterns, standard pipeline order |

### Source material (`sc-best-practices/`)

16 numbered chapter directories following a typical single-cell analysis workflow — raw input first, multimodal downstream analyses later:

| # | Directory | Scope |
|---|---|---|
| 1 | `1-Introduction` | prior art, scRNA-seq overview, raw data processing (FASTQ→count matrix), AnnData/MuData, interoperability, RAPIDS-singlecell (GPU) |
| 2 | `2-Preprocessing_and_visualization` | QC, normalization, feature selection, dimensionality reduction |
| 3 | `3-Identifying_cellular_structure` | clustering, annotation, integration |
| 4 | `4-Inferring_trajectories` | pseudotime, RNA velocity, lineage tracing |
| 5 | `5-Dealing_with_conditions` | DGE, compositional analysis, GSEA, perturbation modeling |
| 6 | `6-Modeling_mechanisms` | gene regulatory networks, cell-cell communication |
| 7 | `7-Deconvolution` | bulk deconvolution |
| 8 | `8-Chromatin_Accessibility` | scATAC-seq intro, QC, GRNs |
| 9 | `9-Spatial_omics` | neighborhoods, domains, spatially variable genes, deconvolution, imputation |
| 10 | `10-Surface_protein` | CITE-seq: QC, normalization, doublets, dim red, batch correction, annotation |
| 11 | `11-Adaptive_immune_receptor_repertoire` | AIRR profiling, clonotypes, specificity, multimodal |
| 12 | `12-Multimodal_integration` | paired and advanced integration |
| 13 | `13-Outlook` | forward-looking notes |
| 14 | `14-Acknowledgements` | credits |
| 15 | `15-Glossary` | term definitions (good entry point for concept→chapter lookup) |
| 16 | `16-Changelog` | v0.1.0 (2023-11-14) → v0.2.0 (2026-06-11) diff |

Files within each source chapter are named `<NN>_<topic>.ipynb` or `.md`. The two-digit prefix is **global across the book** (01–49), so a file number uniquely identifies a file regardless of which chapter it lives in.

The distilled skill chapter files in `chapters/` use a separate numbering `ch01-…` to `ch24-…` and are independent markdown documents (not Jupyter Book notebooks). Chapters 01–12 distill the book, 13–17 distill the tool catalogs, and 18–24 distill the anndata/Scanpy official documentation.

## What is missing from this snapshot

Only chapter content files are present. The following upstream-book assets are **not** here, so directives that reference them will look broken — treat this as expected, not a bug:

- `_static/` — images, figures, and `default_text_env_setup.md` that chapters `{include}`. All `:::{figure-md}` blocks and `{include} ../_static/...` directives will not render.
- `references.bib` — every `{cite}`key`` has no resolvable target here.
- Per-chapter `*.yml` environment files (referenced via `{literalinclude}` inside env-setup dropdowns).
- Build config (`_config.yml`, `_toc.yml`, `Makefile`) — this cannot be built with `jupyter-book` as-is.

## Content format

Files use **MyST markdown** with Jupyter Book directives. Notebooks (`.ipynb`) hold the same prose in markdown cells plus executable Python in code cells. Common directives you will encounter:

- ` ```{dropdown}` / ` ```{admonition}` / ` ```{tab-set}` — collapsible callouts and tabbed content
- ` ```{glossary}` — term definitions rendered into `15-Glossary/48_glossary.md`
- ` ```{bibliography}` — per-chapter reference list (filtered by `docname in docnames`)
- ` {term}`Term`` / ` {ref}` / ` {numref}` / ` {cite}`key`` — inline cross-references
- `(label-id)=` — anchor that other files link to via `{ref}`label-id``
- ` :::figure-md` / ` :::card` — block-level containers

Each chapter opens with a "Key takeaways" dropdown and an "Environment setup" dropdown; the takeaways dropdown is the fastest way to extract a chapter's headline recommendations.

When editing `.md` files, preserve these directives verbatim — they are not arbitrary text. When editing `.ipynb` files, edit existing markdown/source cells only; do not add or remove cells unless asked.

## Working in this repo

There is no build, lint, test, or run step. Verify edits by reading the file back, not by executing it. Do not attempt to install scanpy/anndata/pertpy/etc. to "run" the notebooks — they reference datasets loaded via `lamindb` that are not in this snapshot.

### Two lookup paths

When a user asks a single-cell question, choose the right layer:

- **Fast lookup / decision rules / code patterns** → check distilled skill files first: `cheatsheet.md` (thresholds, method selection), `patterns.md` (techniques by task), `glossary.md` (term definitions), `chapters/chNN-*.md` (chapter-level frameworks with worked examples). `SKILL.md` provides the master index.
- **Deep reasoning / verbatim quotes / full context** → go to source: `sc-best-practices/<chapter>/<NN>_<topic>.ipynb` or `.md`. Each source chapter opens with a "Key takeaways" dropdown — the fastest way to extract headline recommendations.

The glossary at `sc-best-practices/15-Glossary/48_glossary.md` is a useful index for the source book: look up a term there to find which chapter discusses it in depth. The prior-art chapter at `sc-best-practices/1-Introduction/01_prior_art.md` contrasts this book with Bioconductor OSCA/OSTA and the Lücken & Theis 2019 tutorial — useful when a user asks how this book's recommendations differ from other guides.

When a user asks for "the best practice for X", find the matching distilled chapter in `chapters/`, then cross-reference the source chapter's Key Takeaways dropdown plus the closing recommendations section — those are the explicitly-endorsed practices, with the body text providing the reasoning and the alternatives that were considered and rejected.
