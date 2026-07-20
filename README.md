<div align="center">

# 单细胞多组学分析.skill

> *「你想分析的下一个数据集，何必从零开始」*

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-blueviolet)](https://claude.ai/code)
[![Skills](https://img.shields.io/badge/skills.sh-Compatible-green)](https://skills.sh)
[![scverse](https://img.shields.io/badge/scverse-Ecosystem-blue)](https://scverse.org)
[![Book v0.2.0](https://img.shields.io/badge/Source%20Book-v0.2.0-lightgrey)](https://sc-best-practices.org)
<br>

**蒸馏教程 theislab/single-cell-best-practices v0.2.0 (2026-06-11) + 6 个社区 awesome-lists，**

**让单细胞最佳实践直接落地你的下一轮分析，适配单细胞多组学、空间转录组分析等各类场景**

<br>

<img src="image.png" alt="alt text" width="500">

<br>
原教程 16 章 50+ notebook + 6 个 awesome-lists → 17 个章节文件 + 4 个支撑索引

覆盖从 FASTQ 到多模态整合的完整工作流 + 按模态的工具生态

[看效果](#效果示例) · [安装](#安装) · [它蒸馏了什么](#它蒸馏了什么) · [工作原理](#工作原理)

<br>

</div>


---

## 目录

- [效果示例](#效果示例)
- [安装与使用](#安装与使用)
- [仓库结构](#仓库结构)
- [安装与使用](#安装与使用)
- [它蒸馏了什么](#它蒸馏了什么)
- [背后的故事](#背后的故事)

---



## 效果示例

```
用户      ❯ 我的 CITE-seq ADT 数据该怎么 QC？

Skill     ❯ 别直接套 RNA 的 QC 阈值。ADT 的分布是双峰的
            （阴性峰 = 非特异结合，阳性峰 = 真信号），不是负二项。

            三步走：
            1. 在 n_genes_by_counts 分布里找"谷底"（NeurIPS 数据约 55），
               这是低质量细胞的自然分界点——比 total_counts 稳健，后者
               会被高丰度的 ambient protein 抬高
            2. total_counts 上限 100k 硬截断 → 去除双胞/聚集体
            3. 每个 sample 独立做 5-MAD 过滤（log1p_total_counts +
               log1p_n_genes_by_counts），跨样本合并

            然后 dsb normalization（如果有 empty droplets + isotype
            controls）。CLR 只是 fallback。

            详见 chapters/ch10-surface-protein.md
```

```
用户      ❯ 多批次 scRNA-seq，scVI 和 Harmony 该选哪个？

Skill     ❯ 看你手上是什么数据：

            - 有 raw counts + 想端到端整合 → scVI（条件 VAE + ZINB，
              batch_key 作为 condition；scANVI 是半监督版本，需要部分
              cell type label）
            - 已算好 PCA，想快速 batch correction → Harmony（迭代
              soft clustering + 线性修正，比 scVI 快得多）
            - 只想改 graph 不动 PCA → BBKNN（per-batch kNN）

            scVI 需要 GPU（>100k cells 时），Harmony 几秒就跑完。
            评估用 scIB metrics：Overall = 0.6·Bio + 0.4·Batch，
            Bio 看 ASW_label / NMI / ARI / isolated_label_silhouette，
            Batch 看 PCR_batch / graph_conn / iLISI / kBET。

            详见 chapters/ch03-cellular-structure.md 和
            chapters/ch12-multimodal-integration.md
```

```
用户      ❯ BCR clonotype 该怎么定义？

Skill     ❯ 别用 identical CDR3。BCR 会经历 somatic hypermutation
            （SHM），突变率比 germline 高约 10000 倍，identical-CDR3
            太严格，会漏掉同源但已分歧的 clone。

            用 Dandelion：
            1. ddl.from_scirpy(adata) 转格式
            2. ddl.pp.calculate_threshold(vdjx, model="hh_s5f")
               —— hh_s5f 是 5-mer 上下文感知的替换模型，距离分布
               双峰，断点 = 同源 vs singleton 的分界
            3. ddl.tl.define_clones(vdjx, model="hh_s5f",
               key_added="changeo_clone_id")
            4. ddl.tl.generate_network(vdjx, layout_method="sfdp")

            TCR 没这个问题，scirpy 的 define_clonotype_clusters
            （receptor_arms="all"）就够了。

            详见 chapters/ch11-airr.md
```

```
用户      ❯ 单细胞做 DGE，为什么不能直接用 rank_genes_groups？

Skill     ❯ 因为 pseudoreplication。把每个细胞当独立样本会严重
            低估方差，p-value 一堆显著但都是假阳性。

            正确做法是 pseudobulk：
            1. 按 sample × cell_type 聚合 counts
               （decoupler.pp.pseudobulk）
            2. filter_by_expr + filter_by_prop 过滤低表达基因
            3. PyDESeq2 跑 DESeq2 标准流程
            4. test_contrasts 提取 cell-type-specific 响应

            只有想要 per-cell activity score 时才用 AUCell。
            想要 cluster-level enrichment 用 GSEA。
            复杂设计（cell_type × condition）用 limma-fry。

            详见 chapters/ch05-conditions.md
```

```
用户      ❯ 我有一批 10x Multiome（RNA + ATAC 配对），ATAC 部分该用什么工具？

Skill     ❯ 看你数据规模和已有生态：

            - 默认选 ArchR（R，HDF5-backed，>1M cells，功能最全 + 最快）
            - 已经在用 Seurat -> Signac（Seurat 扩展，CITE-seq 联动方便）
            - >5M cells 或要 matrix-free spectral clustering -> SnapATAC2（Rust+Python）

            关键决策点（不是工具选型，是方法学）：
            1. Normalization 用 TF-IDF + LSI，**不要**用 log1p(normalize_total)
               -- ATAC 是二值的，log-normalize 会压缩动态范围
            2. LSI 第 1 主成分是测序深度，**丢掉**，从 comp 2 开始用
            3. 先 bins（500bp，无偏聚类）-> 再 MACS2 call cluster-specific peaks
               -- 不要直接 call per-cell peaks（太稀疏）
            4. 双胞检测要 AMULET（同源）+ scDblFinder（异源）双跑
            5. TF activity 用 chromVAR（per-cell deviation score）
               -- 不要信 motif enrichment（cluster-level，≠ TF activity）
            6. 配对 RNA+ATAC 跑 FigR 做 GRN（DORC + chromVAR + 相关性）

            详见 chapters/ch14-scatac-seq-tools.md
```

这不是查询文档。Skill 用的是「per-modality QC 专业化」「normalization 匹配分布」「scVI for raw counts」「BCR 需要 SHM-aware 距离」「pseudobulk 避免 pseudoreplication」「ATAC 用 TF-IDF+LSI 丢 comp 1」这些心智模型在帮你做决策，不是在复读章节标题。

---

## 安装与使用

```bash
# 方式 1: 手动克隆（推荐）
# sc-omics即skill名
git clone https://github.com/MaybeBio/Single-cell-Omics-Skill ~/.claude/skills/sc-omics

# 方式 2: 使用 npx
npx skills add MaybeBio/Single-cell-Omics-Skill -a claude-code
```

然后在 Claude Code 里自动触发或手动触发`/sc-omics`：

```
> 我有一批 CITE-seq 数据，该怎么开始分析？
> 多批次整合用 scVI 还是 Harmony？
> scATAC-seq 的 QC 阈值跟 RNA 有什么不同？
> AIRR 数据的 clonotype 怎么定义？
> 帮我用 marker 检测 ADT 双胞
> pseudobulk DGE 还是单细胞 DE？
> scATAC 用 ArchR 还是 Signac？
> scHi-C 怎么 normalize 和聚类？
> Visium 数据怎么选 deconvolution 方法？
> 哪里能找到匹配的 scRNA-seq 参考图谱？
```

---

## 它蒸馏了什么

蒸馏一本 16 章的单细胞领域圣经 + 6 个社区 awesome-lists，需要提取比章节标题更深的东西。这个 Skill 提取两层、五层结构：

- **理论层（Ch 01-12）**--来自 theislab/single-cell-best-practices，每模态的 QC / normalization / 整合 / 下游分析 best practice
- **工具生态层（Ch 13-17）**--来自 6 个 awesome-lists，按 pipeline stage 组装的工具目录 + 决策规则

每章提取五层：

| 层次 | 说明 |
|---|---|
| **核心思想** | 每章 Core Idea——1-2 句话讲清这章为什么存在 |
| **心智模型** | "用 X 当 Y 用、当 Z 时用 X"——决策级框架，不是知识点列表 |
| **代码模式** | 可直接复用的 scanpy / muon / scirpy / scvi-tools 代码片段 |
| **反模式** | 哪些做法会失败 + 为什么——比 best practice 更值钱 |
| **诚实边界** | 什么时候该用别的方法，什么时候数据不够 |

章节内容可以靠读原教程获取，但让分析师面对新数据集时做出正确判断的，是心智模型。这个 Skill 提取的是单细胞分析的认知操作系统。

### 诚实边界

每个章节文件都明确标注了不适用场景：

- **scVI** 需要 GPU + raw counts——已 log-normalize 的数据不能用
- **dsb** 需要 empty droplets + isotype controls——预筛过的数据只能用 CLR
- **NicheNet** 需要条件对比——稳态数据只能跑 ligand-receptor
- **TESSA / CoNGA / mvTCR / Benisse** 是实验性的多模态模型，不是标准流程
- **多模态整合** 都是实验性的——benchmark 用 scIB，别盲信

**一个不告诉你局限在哪的 Skill，不值得信任。**

---

## 已蒸馏章节

17 章 + 4 个支撑文件，每个都是独立的、可按需加载的 markdown：

### 章节文件

| # | 章节 | 核心问题 |
|---|---|---|
| 01 | Introduction | AnnData / MuData / SpatialData 数据结构，scanpy / muon / squidpy 工具栈，R↔Python 互操作 |
| 02 | Preprocessing & Visualization | MAD-based QC，SoupX ambient RNA，scDblFinder，3 层 normalization，deviance HVG |
| 03 | Cellular Structure | Leiden 聚类，marker 注释，scArches label transfer，scVI / scANVI / BBKNN / Seurat 整合，scIB 评估 |
| 04 | Trajectories | DPT / Palantir / PAGA pseudotime，scVelo RNA velocity（dynamical mode），Cassiopeia CRISPR lineage tracing |
| 05 | Conditions | Pseudobulk DGE + PyDESeq2（避免 pseudoreplication），scCODA 组成分析，GSEA / AUCell / limma-fry，Augur / scGen / Mixscape 扰动建模 |
| 06 | Modeling Mechanisms | SCENIC 三步 GRN（co-expression -> motif -> AUCell），CellPhoneDB / LIANA consensus，NicheNet intracellular signaling |
| 07 | Bulk Deconvolution | AutoGeneS + MuSiC via rpy2，Cobos 5 推荐，CPM 标准化 |
| 08 | Chromatin Accessibility | scATAC-seq peaks vs bins，4-metric QC，scDblFinder + AMULET 双胞检测，FigR + cisTopic DORC / GRN |
| 09 | Spatial Omics | 三层分辨率，Squidpy neighborhood stats，Moran's I + SpatialDE SVGs，Cell2location deconvolution，Tangram imputation |
| 10 | Surface Protein | CITE-seq ADT QC（valley filter），dsb vs CLR normalization，marker 双胞检测（CD3+CD19+），PCA -> Harmony -> UMAP |
| 11 | AIRR Profiling | scirpy chain_qc，clonotype 定义，Dandelion BCR SHM-aware，TCRdist / TCRmatch / ERGO-II 特异性推断，TESSA / CoNGA / mvTCR 多模态 |
| 12 | Multimodal Integration | WNN / MOFA+ / totalVI / MultiVI / GLUE / Multigrate，scIB benchmarking，scArches query-to-reference，Bridge integration |
| 13 | scRNA-seq Tools Ecosystem | 按阶段组装 pipeline（FASTQ -> ambient -> doublet -> QC -> normalize -> HVG -> integrate -> cluster -> annotate -> DGE），scverse vs Seurat+Bioconductor 二选一 |
| 14 | scATAC-seq Tools Ecosystem | ArchR vs Signac vs SnapATAC2，TF-IDF + LSI（drop comp 1），chromVAR TF activity，FigR / SCENIC+ GRN，AMULET + scDblFinder 双胞 |
| 15 | scHi-C Tools Ecosystem | BandNorm distance-centric normalization，Higashi / Fast-Higashi imputation + tensor clustering，scGHOST subcompartments，SnapHiC2 loop calling |
| 16 | Spatial Omics Tools Ecosystem | Visium / Xenium / MERFISH 分辨率分层，Cell2location / RCTD deconvolution，SpaGCN / STAGATE / BANKSY domains，Baysor / Cellpose segmentation |
| 17 | Cross-Modality Resources | seandavi/awesome-single-cell + scRNA-tools.org 工具目录，CZ CELLxGENE Census 33M+ cells，scIB / pipeComp / Crowell 2020 benchmark，CellTypist / HCA / Tabula 参考图谱 |

### 支撑文件

| 文件 | 用途 |
|---|---|
| `SKILL.md` | skill 主文件--13 个 Core Frameworks（含工具生态）+ 章节 / 主题索引 + Scope & Limits |
| `glossary.md` | 80+ 术语表（带章节定位）--ADT、AnnData、AUCell、CDR3、CLR、dsb、HVG、Moran's I、scIB、scVI、TCRdist、WNN；以及 ArchR、BandNorm、Cell2location、chromVAR、FigR、Higashi、SnapATAC2、SpaGCN、Tangram、Visium、Xenium 等工具生态术语 |
| `patterns.md` | 按任务分类的具体技术--QC、normalization、clustering、integration、DGE、trajectory、mechanism、spatial、multimodal、perturbation；以及按阶段组装工具、scHi-C 预处理、空间分辨率分层、SVG 检测、参考优先注释、工具发现、基准测试、Census 查询 |
| `cheatsheet.md` | 决策规则速查--每模态 QC 阈值、normalization 决策树、整合方法选择、scIB metrics、反模式清单、标准 pipeline 顺序；以及 scRNA/scATAC/spatial/scHi-C 工具套件选择、DGE/整合/空间工具快速挑选、参考图谱快速选择、ambient RNA/doublet/CNV/CCC 决策表 |

---

## 工作原理


**1. 文本提取**——两路来源：

- 原教程：43 个 `.ipynb` + 7 个 `.md` 文件（含代码 cell）→ nbformat 提取 → 合并成 208K token 的 `full_text.txt`
- 工具目录：6 个 awesome-lists README（seandavi/awesome-single-cell、mdozmorov/sc{RNA,ATAC,HiC}-notes、crazyhottommy/{scRNAseq-analysis-notes,awesome_spatial_omics}）→ 合并成 ~100K token 的补充语料

**2. 章节切片**——原教程 16 章切到 12 个 per-chapter source 文件（Outlook / Acknowledgements / Glossary / Changelog 合并为 backmatter）；工具目录按模态切到 5 个 tool-ecosystem source 文件。

**3. 结构化蒸馏**——每章按固定模板生成：

```
Core Idea → Frameworks Introduced → Key Concepts → Mental Models →
Anti-patterns → Code Examples → Reference Tables → Worked Example →
Key Takeaways → Connects To
```

- Ch 01-12（理论层）：`DEPTH=study` + `BOOK_TYPE=technical` → 每章 2,000-3,000 token 预算，密度优先于长度
- Ch 13-17（工具生态层）：`DEPTH=study` + `BOOK_TYPE=text` → 每章 1,000-1,800 token 预算，按 pipeline stage 组织

**4. 交叉索引**——生成 `glossary.md`（术语→章节）、`patterns.md`（任务→技术）、`cheatsheet.md`（决策→阈值 / 方法）、`SKILL.md`（主索引）。索引覆盖理论层 + 工具生态层两套术语。

**5. 验证**——每个理论章节包含至少一个 Worked Example（NeurIPS 2021 BMMC、Kang PBMC IFN-β、Haniffa SARS-CoV-2、Lee 2019 mouse brain scHi-C、Visium mouse brain 等），与原书一致才通过；工具生态章节包含按 pipeline stage 的工具选择 walkthrough。

---

## 仓库结构

```
Single-cell-Omics-Skill/
├── CLAUDE.md                    # 仓库说明（双层级结构 + 查询路径）
├── SKILL.md                     # skill 主文件（13 个 Core Frameworks + 17 章索引）
├── glossary.md                  # 术语表（含工具生态术语）
├── patterns.md                  # 任务级技术模式（含工具选择模式）
├── cheatsheet.md                # 决策规则速查（含工具套件选择）
├── chapters/                    # 17 个蒸馏后的章节文件
│   ├── ch01-introduction.md
│   ├── ch02-preprocessing-visualization.md
│   ├── ch03-cellular-structure.md
│   ├── ch04-trajectories.md
│   ├── ch05-conditions.md
│   ├── ch06-modeling-mechanisms.md
│   ├── ch07-bulk-deconvolution.md
│   ├── ch08-chromatin-accessibility.md
│   ├── ch09-spatial-omics.md
│   ├── ch10-surface-protein.md
│   ├── ch11-airr.md
│   ├── ch12-multimodal-integration.md
│   ├── ch13-scrna-seq-tools.md          # 工具生态：scRNA-seq
│   ├── ch14-scatac-seq-tools.md         # 工具生态：scATAC-seq
│   ├── ch15-schic-tools.md              # 工具生态：scHi-C
│   ├── ch16-spatial-omics-tools.md      # 工具生态：空间组学
│   └── ch17-cross-modality-resources.md # 工具生态：跨模态资源
├── manuals/                     # 工具目录源材料（6 个 awesome-lists README）
│   ├── seandavi_awesome-single-cell_README.md
│   ├── scATAC-seq/mdozmorov_scATAC-seq_notes_README.md
│   ├── scHiC/mdozmorov_scHiC_notes_README.md
│   ├── scRNA-seq/mdozmorov_scRNA-seq_notes_README.md
│   ├── scRNA-seq/crazyhottommy_scRNAseq-analysis-notes_README.md
│   └── spatial_omics/crazyhottommy_awesome_spatial_omics_README.md
└── sc-best-practices/           # 原书源材料（16 章 .ipynb / .md）
    ├── 1-Introduction/
    ├── 2-Preprocessing_and_visualization/
    ├── ...
    └── 16-Changelog/
```

---

## 背后的故事

[Heumos et al. 2023 *Nat Rev Genet*](https://doi.org/10.1038/s41576-023-00586-w) 是单细胞领域的标杆综述，配套的 [single-cell-best-practices](https://www.sc-best-practices.org) 在线书籍是 scverse 生态的官方推荐读物。

但 16 章 50+ notebook，读一遍要几天，记住每章的 best practice 要更久。每次分析新数据集时翻文档、查阈值、决定用哪个方法，是单细胞分析员日常的时间黑洞。

把整本书蒸馏成一个 Skill，是为了让最佳实践在你需要的那一刻直接生效——不是查文档，是带着心智模型帮你做决策。

这个 Skill 不替代原教程。它提取单细胞分析的认知操作系统。原教程仍然是深度理解的最佳来源；Skill 是日常决策的快速通道。

工具生态层（Ch 13-17）不替代 awesome-lists。它把 6 个社区维护的工具目录按 pipeline stage 重组 + 加上决策规则，让你在选工具时不用在 6 个 README 之间跳转。原 awesome-lists 仍然是工具发现的最佳起点；Skill 是工具选择的快速通道。

**两层结构 = 理论（怎么做）+ 工具（用什么）**--面对新数据集时，先查理论层确定方法学，再查工具生态层选具体工具实现。
