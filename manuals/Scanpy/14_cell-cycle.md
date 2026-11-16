# Cell-Cycle Scoring and Regression

> This is a Scanpy demo that shows how to regress cell cycle effect, following the approach showed in Seurat’s vignette. As for the R example, toy dataset consists of murine hematopoietic progenitors...

# Cell-Cycle Scoring and Regression[#](https://scanpy.scverse.org/en/stable/how-to/cell-cycle.html#cell-cycle-scoring-and-regression)

This is a *Scanpy* demo that shows how to regress cell cycle effect, following the approach showed in [Seurat’s vignette](http://satijalab.org/seurat/cell_cycle_vignette.html#assign-cell-cycle-scores). As for the R example, toy dataset consists of murine hematopoietic progenitors from [Nestorowa et al., Blood 2016](https://doi.org/10.1182/blood-2016-05-716480). The files of the Seurat tutorial – used here for reasons of benchmarking – can be downloaded [here](https://www.dropbox.com/s/3dby3bjsaf5arrw/cell_cycle_vignette_files.zip?dl=1). A more recent version of the dataset can be downloaded [here](http://blood.stemcells.cam.ac.uk/data/nestorowa_corrected_log2_transformed_counts.txt.gz).

```
from __future__ import annotations

import zipfile

import pooch

import scanpy as sc

sc.logging.print_header()
```

Load data

```
p_zip = pooch.retrieve(
    "https://www.dropbox.com/s/3dby3bjsaf5arrw/cell_cycle_vignette_files.zip?dl=1",
    known_hash="sha256:6557fe2a2761d1550d41edf61b26c6d5ec9b42b4933d56c3161164a595f145ab",
    path="../data",
)
```

Load cell cycle genes defined in [Tirosh et al, 2015](https://doi.org/10.1126/science.aad0501). It is a list of 97 genes, represented by their gene symbol. The list here is for humans, in case of alternate organism, a list of ortologues should be compiled. There are major differences in the way *Scanpy* and *Seurat* manage data, in particular we need to filter out cell cycle genes that are not present in our dataset to avoid errors.

```
with zipfile.ZipFile(p_zip, "r") as f_zip:
    f_csv = zipfile.Path(f_zip, "nestorawa_forcellcycle_expressionMatrix.txt").open()
    adata = sc.read_csv(f_csv, delimiter="\t").T
    cell_cycle_genes = (
        zipfile.Path(f_zip, "regev_lab_cell_cycle_genes.txt").read_text().splitlines()
    )
```

Here we define two lists, genes associated to the S phase and genes associated to the G2M phase

```
s_genes = [x for x in cell_cycle_genes[:43] if x in adata.var_names]
g2m_genes = [x for x in cell_cycle_genes[43:] if x in adata.var_names]
cell_cycle_genes = [*s_genes, *g2m_genes]
```

Standard filters applied. Note that we do not extract variable genes and work on the whole dataset, instead. This is because, for this demo, almost 70 cell cycle genes would not be scored as variable. Cell cycle scoring on ~20 genes is ineffective.

```
sc.pp.filter_cells(adata, min_genes=200)
sc.pp.filter_genes(adata, min_cells=3)
sc.pp.normalize_total(adata, target_sum=1e4)
```

Log-transformation of data and scaling should always be performed before scoring

```
sc.pp.log1p(adata)
```

We here perform cell cycle scoring. The function is actually a wrapper to [`scanpy.tl.score_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.score_genes.html#scanpy.tl.score_genes), which is launched twice, to score separately S and G2M phases. Both [`scanpy.tl.score_genes()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.score_genes.html#scanpy.tl.score_genes) and [`scanpy.tl.score_genes_cell_cycle()`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.tl.score_genes_cell_cycle.html#scanpy.tl.score_genes_cell_cycle) are a port from *Seurat* and are supposed to work in a very similar way. To score a gene list, the algorithm calculates the difference of mean expression of the given list and the mean expression of reference genes. To build the reference, the function randomly chooses a bunch of genes matching the distribution of the expression of the given list. Cell cycle scoring adds three slots in data, a score for S phase, a score for G2M phase and the predicted cell cycle phase.

```
sc.tl.score_genes_cell_cycle(adata, s_genes=s_genes, g2m_genes=g2m_genes)
```

Here comes another difference from *Seurat*. The R package stores raw data, scaled data and variable genes information in separate slots, *Scanpy* instead keeps only one snapshot of the data. This implies that PCA is always calculated on the entire dataset. In order to calculate PCA reduction using only a subset of genes (like `cell_cycle_genes`), a trick should be used. Basically we create a dummy object to store information of PCA projection, which is then reincorporated into original dataset.

```
adata_cc_genes = adata[:, cell_cycle_genes].copy()
sc.tl.pca(adata_cc_genes)
sc.pl.pca(adata_cc_genes, color="phase")
```

[![../_images/1d57a6a6f897c88b6acfc435bbfeb70798036d973008ed199b9b93305844c407.png](https://scanpy.scverse.org/en/stable/_images/1d57a6a6f897c88b6acfc435bbfeb70798036d973008ed199b9b93305844c407.png)](https://scanpy.scverse.org/en/stable/_images/1d57a6a6f897c88b6acfc435bbfeb70798036d973008ed199b9b93305844c407.png)

As in the original vignette, cells can be easily separated by their cell cycle status when cell cycle genes are used. Now we can regress out both S score and G2M score.

```
sc.pp.regress_out(adata, ["S_score", "G2M_score"])
```

Finally, we reproject dataset using cell cycle genes again. Since we regressed the scores, no effect of cell cycle is now evident.

```
adata_cc_genes = adata[:, cell_cycle_genes].copy()
sc.tl.pca(adata_cc_genes)
sc.pl.pca(adata_cc_genes, color="phase")
```

[![../_images/9201238033dc3d71525ffa4c3fd342931493cbc4154f10dad379a148746ccc9b.png](https://scanpy.scverse.org/en/stable/_images/9201238033dc3d71525ffa4c3fd342931493cbc4154f10dad379a148746ccc9b.png)](https://scanpy.scverse.org/en/stable/_images/9201238033dc3d71525ffa4c3fd342931493cbc4154f10dad379a148746ccc9b.png)

Author: *Davide Cittaro*
