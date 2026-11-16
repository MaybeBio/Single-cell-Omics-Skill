# Interfacing PyTorch models with collections of AnnData objects via annbatch

> Author: Sergei Rybakov Author: Ilan Gold This tutorial demonstrates how to set up a annbatch.Loader for an in-memory AnnData object to interface with a PyTorch model and lay out complex covariates....

# Interfacing PyTorch models with collections of AnnData objects via `annbatch`[#](https://anndata.scverse.org/tutorials/notebooks/annbatch.html#interfacing-pytorch-models-with-collections-of-anndata-objects-via-annbatch)

**Author:**[Sergei Rybakov](https://github.com/koncopd)**Author:**[Ilan Gold](https://github.com/ilan-gold)

This tutorial demonstrates how to set up a [`annbatch.Loader`](https://annbatch.readthedocs.io/en/stable/generated/annbatch.Loader.html#annbatch.Loader) for an in-memory [`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData) object to interface with a PyTorch model and lay out complex covariates.

Note, however, that when training models on many large `AnnData` objects, loading them all into memory at once is impractical as this notebook does.

`annbatch.Loader` solves this by allowing users to streams batches formed of contiguous on-memory runs of the training data across multiple AnnData objects without requiring a full in-memory concatenation — compatible with PyTorch, JAX, and other frameworks while being extremely fast. Preshuffling is a crucial part of making this “contiguous fetching” strategy work – see [`annbatch.DatasetCollection`](https://annbatch.readthedocs.io/en/stable/generated/annbatch.DatasetCollection.html#annbatch.DatasetCollection) for more information on performing the on-disk, lazy preshuffle.

To install `annbatch` for in-memory sparse data (which is most common in single-cell RNA-seq and will be used in this tutorial), use `pipinstall"annbatch[numba]"` – note that the `numba` extra is required for in-memory sparse data. To use the `preload_to_gpu` option in [`annbatch.Loader`](https://annbatch.readthedocs.io/en/stable/generated/annbatch.Loader.html#annbatch.Loader), install the `cupy-cudaxx` extra as well where `xx` is your `cuda` version. See the [docs](https://annbatch.readthedocs.io/en/stable/installation.html) for more info on installation.

```
from __future__ import annotations

from typing import TYPE_CHECKING

import numpy as np
import pandas as pd
import pooch
import pyro
import pyro.distributions as dist
import scanpy as sc
import torch
from annbatch import Loader
from sklearn.preprocessing import OneHotEncoder
from torch import nn

import anndata as ad

if TYPE_CHECKING:
    from collections.abc import Sequence


pyro.clear_param_store()
```

## VAE model definition[#](https://anndata.scverse.org/tutorials/notebooks/annbatch.html#vae-model-definition)

The task is semi-supervised label transfer: `pbmc` cells have annotated cell types, while `covid` cells do not. The model learns a shared latent space across both datasets and uses the labeled pbmc cells to predict cell types for the unlabeled covid cells. Batch labels (study of origin) are included as covariates to correct for dataset-specific effects.

Show code cell content

Hide code cell content

```
class MLP(nn.Module):
    def __init__(self, input_dim, hidden_dims, out_dim):
        super().__init__()

        modules = []
        for in_size, out_size in zip(
            [input_dim, *hidden_dims], hidden_dims, strict=True
        ):
            modules.append(nn.Linear(in_size, out_size))
            modules.append(nn.LayerNorm(out_size))
            modules.append(nn.ReLU())
            modules.append(nn.Dropout(p=0.05))
        modules.append(nn.Linear(hidden_dims[-1], out_dim))
        self.fc = nn.Sequential(*modules)

    def forward(self, *inputs):
        shape = (*dist.util.broadcast_shape(*[s.shape[:-1] for s in inputs]), -1)
        inputs = [s.expand(shape) for s in inputs]

        input_cat = torch.cat(inputs, dim=-1)
        return self.fc(input_cat)
```

Show code cell content

Hide code cell content

```
# The code is based on the scArches trVAE model: https://github.com/theislab/scarches/blob/v0.3.5/scarches/models/trvae/trvae.py
# and on the pyro.ai Variational Autoencoders tutorial: http://pyro.ai/examples/vae.html
# and the Semi-Supervised VAE tutorial: http://pyro.ai/examples/ss-vae.html
class CVAE(nn.Module):
    def __init__(
        self,
        input_dim,
        *,
        n_conds: int,
        n_classes: int,
        hidden_dims: Sequence[int],
        latent_dim: int,
        classifier_dims: Sequence[int] = (128,),
    ) -> None:
        super().__init__()
        # output - mean and logvar of z
        self.encoder = MLP(input_dim + n_conds, hidden_dims, 2 * latent_dim)
        self.decoder = MLP(
            latent_dim + n_conds + n_classes, hidden_dims[::-1], input_dim
        )
        self.theta = nn.Linear(n_conds, input_dim, bias=False)
        self.classifier = MLP(latent_dim, classifier_dims, n_classes)
        self.latent_dim = latent_dim

    def model(self, x, batches, classes, size_factors, supervised):
        pyro.module("cvae", self)

        batch_size = x.shape[0]

        with pyro.plate("data", batch_size):
            z_loc = x.new_zeros((batch_size, self.latent_dim))
            z_scale = x.new_ones((batch_size, self.latent_dim))
            z = pyro.sample("Z", dist.Normal(z_loc, z_scale).to_event(1))

            classes_probs = self.classifier(z).softmax(dim=-1)
            obs = classes if supervised else None
            classes = pyro.sample(
                "Class", dist.OneHotCategorical(probs=classes_probs), obs=obs
            )

            dec_mu = (
                self.decoder(z, batches, classes).softmax(dim=-1)
                * size_factors[:, None]
            )
            dec_theta = torch.exp(self.theta(batches))

            logits = (dec_mu + 1e-6).log() - (dec_theta + 1e-6).log()

            pyro.sample(
                "X",
                dist.NegativeBinomial(total_count=dec_theta, logits=logits).to_event(1),
                obs=x.int(),
            )

    def guide(self, x, batches, classes, size_factors, supervised):
        batch_size = x.shape[0]

        with pyro.plate("data", batch_size):
            z_loc_scale = self.encoder(x, batches)

            z_mu = z_loc_scale[:, : self.latent_dim]
            z_var = torch.sqrt(torch.exp(z_loc_scale[:, self.latent_dim :]) + 1e-4)

            z = pyro.sample("Z", dist.Normal(z_mu, z_var).to_event(1))

            if not supervised:
                classes_probs = self.classifier(z).softmax(dim=-1)
                pyro.sample("Class", dist.OneHotCategorical(probs=classes_probs))
```

## Create a concatenated `AnnData` from two `AnnData` objects[#](https://anndata.scverse.org/tutorials/notebooks/annbatch.html#create-a-concatenated-anndata-from-two-anndata-objects)

The data is from this [scvi reproducibility notebook](https://yoseflab.github.io/scvi-tools-reproducibility/scarches_totalvi_seurat_data/).

Show code cell content

Hide code cell content

```
def download(url, fname, sha_hash):
    pooch.retrieve(
        url=url,
        known_hash=sha_hash,
        fname=fname,
        path=".",
        downloader=pooch.HTTPDownloader(
            progressbar=True,
            chunk_size=1024,
            timeout=120,
            headers={
                "User-Agent": "an/ndata/1.0.0 (https://github.com/scverse/anndata)"
            },
        ),
    )
```

```
download(
    "https://scverse-exampledata.s3.eu-west-1.amazonaws.com/anndata/cite_covid_full.h5ad",
    "covid_cite.h5ad",
    "cb9745b2d642459926194961f34110a17c16ef6b11777304a3478e12bd682657",
)
download(
    "https://scverse-exampledata.s3.eu-west-1.amazonaws.com/anndata/pbmc_seurat_v4.h5ad",
    "pbmc_seurat_v4.h5ad",
    "c3b0100a6ce27beb64eff53692e09f98da2a58cfdfea08d15ff204f834b41396",
)
```

Show code cell output

Hide code cell output

```
covid = ad.read_h5ad("covid_cite.h5ad")
pbmc = ad.read_h5ad("pbmc_seurat_v4.h5ad")
```

```
covid.obs["size_factors"] = np.asarray(covid.X.sum(1)).ravel()
pbmc.obs["size_factors"] = np.asarray(pbmc.X.sum(1)).ravel()
```

We select highly variable genes using pbmc as the reference, since it has rich cell type annotations and multiple batches to compute variability across. Concatenating with `join='inner'` then restricts covid to the same gene set.

```
sc.pp.highly_variable_genes(
    pbmc,
    n_top_genes=4000,
    flavor="seurat_v3",
    batch_key="orig.ident",
    subset=True,
)
```

```
covid
```

```
AnnData object with n_obs × n_vars = 57669 × 33538
    obs: 'orig.ident', 'nCount_RNA', 'nFeature_RNA', 'RNA_snn_res.0.4', 'seurat_clusters', 'set', 'Resp', 'disease', 'subj_code', 'covidpt_orhealth', 'mito', 'ncount', 'nfeat', 'bust_21', 'og_clust', 'severmod_other', 'og_clusts', 'nCount_ADT', 'nFeature_ADT', 'UMAP1', 'UMAP2', 'final_clust', 'final_clust_v2', 'new_pt_id', 'Resp_og', 'final_clust_withnum', 'final_clust_review', 'Age', 'Gender', 'Gender_num', 'size_factors'
    obsm: 'pro_exp'
```

```
pbmc
```

```
AnnData object with n_obs × n_vars = 161764 × 4000
    obs: 'nCount_ADT', 'nFeature_ADT', 'nCount_RNA', 'nFeature_RNA', 'orig.ident', 'lane', 'donor', 'time', 'celltype.l1', 'celltype.l2', 'celltype.l3', 'Phase', 'nCount_SCT', 'nFeature_SCT', 'X_index', 'size_factors'
    var: 'highly_variable', 'highly_variable_rank', 'means', 'variances', 'variances_norm', 'highly_variable_nbatches'
    uns: 'hvg'
    obsm: 'protein_counts'
```

```
dataset = ad.concat(
    {"covid": covid, "pbmc": pbmc},
    join="inner",
    label="dataset",
)
```

```
dataset
```

```
AnnData object with n_obs × n_vars = 219433 × 4000
    obs: 'orig.ident', 'nCount_RNA', 'nFeature_RNA', 'nCount_ADT', 'nFeature_ADT', 'size_factors', 'dataset'
```

Setup important covariates in the concatenated object as well as one-hot encoders.

```
dataset.obs["orig.ident"] = dataset.obs["orig.ident"].astype(str)
dataset.obs.loc[covid.obs_names, "orig.ident"] = covid.obs["set"]
dataset.obs["orig.ident"] = dataset.obs["orig.ident"].astype("category")
```

```
dataset.obs["size_factors"] = dataset.X.sum(1)
```

```
labels = "celltype.l1"
```

```
dataset.obs[labels] = pd.concat([
    pd.Series(index=covid.obs_names, data=["not-annotated"] * covid.shape[0]),
    pbmc.obs["celltype.l1"].astype("str"),
]).astype("category")
```

```
dataset.obs[labels]
```

```
AAACCCACACCAGCGT-1       not-annotated
AAACCCACATCTCAAG-1       not-annotated
AAACGAAAGACCTGGA-1       not-annotated
AAACGCTCAGTGGGTA-1       not-annotated
AAACGCTGTAGCTTGT-1       not-annotated
                             ...      
E2L8_TTTGTTGGTCGTGATT            CD8 T
E2L8_TTTGTTGGTGTGCCTG             Mono
E2L8_TTTGTTGGTTAGTTCG                B
E2L8_TTTGTTGGTTGGCTAT             Mono
E2L8_TTTGTTGTCTCATGGA             Mono
Name: celltype.l1, Length: 219433, dtype: category
Categories (9, object): ['B', 'CD4 T', 'CD8 T', 'DC', ..., 'NK', 'not-annotated', 'other', 'other T']
```

```
use_cuda = torch.cuda.is_available()
use_cuda
```

```
True
```

```
encoder_study = OneHotEncoder(sparse_output=False, dtype=np.float32)
encoder_study.fit(dataset.obs["orig.ident"].to_numpy()[:, None])

encoder_celltype = OneHotEncoder(sparse_output=False, dtype=np.float32)
encoder_celltype.fit(pbmc.obs[labels].to_numpy()[:, None])
```

Show code cell output

Hide code cell output

```
OneHotEncoder(dtype=<class 'numpy.float32'>, sparse_output=False)
```

**In a Jupyter environment, please rerun this cell to show the HTML representation or trust the notebook. On GitHub, the HTML representation is unable to render, please try loading this page with nbviewer.org.**

## Loader and model initialization[#](https://anndata.scverse.org/tutorials/notebooks/annbatch.html#loader-and-model-initialization)

We initialize a [`annbatch.Loader`](https://annbatch.readthedocs.io/en/stable/generated/annbatch.Loader.html#annbatch.Loader) from the concatenated dataset.

- **`chunk_size`**: number of AnnData chunks streamed at a time. For in-memory data you can safely choose `chunk_size=1` for perfect random sampling; for on-disk data, larger values with pre-shuffled data are important for maximizing both training quality and speed.
- **`preload_nchunks`**: how many chunks to buffer before yielding batches – here it can be the same as `batch_size` because we have `chunk_size=1`.
- **`batch_size`**: number of cells per training batch. Typical values are 256–8192; larger batches train faster but require more GPU memory.
- **`preload_to_gpu`**: reads directly into pinned GPU memory for a speed boost; requires the `cupy-cudaxx` extra.

```
loader = Loader(
    chunk_size=1,
    preload_nchunks=8192,
    batch_size=8192,
    shuffle=True,
    preload_to_gpu=True,
).add_adata(dataset)
```

```
pyro.clear_param_store()
torch.set_float32_matmul_precision("high")

n_conds = len(dataset.obs["orig.ident"].cat.categories)
n_classes = len(dataset.obs[labels].cat.categories) - 1

latent_dim = 20

cvae = CVAE(
    dataset.shape[1],
    n_conds=n_conds,
    n_classes=n_classes,
    hidden_dims=[1280, 256],
    latent_dim=latent_dim,
)

if use_cuda:
    cvae.cuda()
```

## Train the model[#](https://anndata.scverse.org/tutorials/notebooks/annbatch.html#train-the-model)

```
optimizer = pyro.optim.Adam({"lr": 1e-3})

svi_sup = pyro.infer.SVI(
    cvae.model, cvae.guide, optimizer, loss=pyro.infer.TraceMeanField_ELBO()
)

# very slow for sequential
svi_unsup = pyro.infer.SVI(
    cvae.model,
    pyro.infer.config_enumerate(cvae.guide, "parallel", expand=True),
    optimizer,
    loss=pyro.infer.TraceEnum_ELBO(),
)
```

```
def train(svi_sup, svi_unsup, train_loader):
    epoch_loss = 0.0

    for batch in train_loader:
        # select indices of labelled cells
        sup_idx = (batch["obs"]["dataset"] == "pbmc").to_numpy()
        X = batch["X"].to_dense().int()
        cols = ["orig.ident", labels, "size_factors"]
        obs_dict = batch["obs"][cols].to_dict("series")
        obs_dict["size_factors"] = torch.Tensor(
            obs_dict["size_factors"].to_numpy()
        ).cuda()
        obs_dict["orig.ident"] = torch.Tensor(
            encoder_study.transform(obs_dict["orig.ident"].to_numpy()[:, None])
        ).cuda()
        # do supervised step for the labelled cells
        sup_data = {
            "orig.ident": obs_dict["orig.ident"][sup_idx],
            labels: torch.Tensor(
                encoder_celltype.transform(
                    obs_dict[labels].loc[sup_idx].to_numpy()[:, None]
                )
            ).cuda(),
            "size_factors": obs_dict["size_factors"][sup_idx],
        }

        sup_data["supervised"] = True
        epoch_loss += svi_sup.step(X[sup_idx], *sup_data.values())

        # do unsupervised step for unlabelled cells
        unsup_data = {
            "orig.ident": obs_dict["orig.ident"][~sup_idx],
            labels: torch.empty(((~sup_idx).sum(), n_classes)).cuda(),
            "size_factors": obs_dict["size_factors"][~sup_idx],
        }

        unsup_data["supervised"] = False
        epoch_loss += svi_unsup.step(X[~sup_idx], *unsup_data.values())
    normalizer_train = train_loader.n_obs
    total_epoch_loss_train = epoch_loss / normalizer_train

    return total_epoch_loss_train
```

```
NUM_EPOCHS = 250

for epoch in range(NUM_EPOCHS):
    total_epoch_loss_train = train(svi_sup, svi_unsup, loader)
    if epoch % 40 == 0 or epoch == NUM_EPOCHS - 1:
        print(f"[epoch {epoch:03}]  average training loss: {total_epoch_loss_train:.4}")
```

Show code cell output

Hide code cell output

```
[epoch 000]  average training loss: 1255.0783
[epoch 040]  average training loss: 919.0868
[epoch 080]  average training loss: 903.4886
[epoch 120]  average training loss: 898.2071
[epoch 160]  average training loss: 895.3621
[epoch 200]  average training loss: 894.0363
[epoch 240]  average training loss: 893.1848
[epoch 249]  average training loss: 893.0040
```

Training is complete. We now run inference over the full dataset to extract latent representations and transfer cell type labels from pbmc to the unlabeled covid cells.

## Check the results[#](https://anndata.scverse.org/tutorials/notebooks/annbatch.html#check-the-results)

Get means of latent variables and cell labels predictions for the unlabelled data.

```
dataset.obsm["X_cvae"] = np.empty((dataset.shape[0], latent_dim), dtype="float32")
dataset.obs["cell_type_pred"] = pd.Series(dtype=pbmc.obs[labels].dtype)
loader = Loader(
    chunk_size=512,
    preload_nchunks=8192,
    batch_size=8192,
    shuffle=False,
    preload_to_gpu=True,
    return_index=True,
).add_adata(dataset)

for batch in loader:
    latents = cvae.encoder(
        batch["X"].to_dense(),
        torch.Tensor(
            encoder_study.transform(batch["obs"]["orig.ident"].to_numpy()[:, None])
        ).cuda(),
    )[:, :latent_dim].detach()

    predict = cvae.classifier(latents).detach()
    one_hot = torch.zeros(predict.shape[0], predict.shape[1], device=predict.device)
    one_hot = (
        one_hot.scatter_(1, predict.argmax(dim=-1, keepdim=True), 1.0).cpu().numpy()
    )

    dataset.obsm["X_cvae"][batch["index"]] = latents.cpu().numpy()
    dataset.obs.iloc[batch["index"], 8] = np.ravel(
        encoder_celltype.inverse_transform(one_hot)
    )
```

```
dataset.obs["cell_type_joint"] = dataset.obs["cell_type_pred"]
dataset.obs.loc[pbmc.obs_names, "cell_type_joint"] = pbmc.obs[labels]
```

```
sc.pp.neighbors(dataset, use_rep="X_cvae")
sc.tl.umap(dataset)
```

```
sc.pl.umap(dataset, color=["dataset"])
```

![../../_images/17bb3c5d6ef8dab848e89fbb2c0b79c9db8cf6e9c89e69f43ccb609960f34477.png](https://anndata.scverse.org/_images/17bb3c5d6ef8dab848e89fbb2c0b79c9db8cf6e9c89e69f43ccb609960f34477.png)

```
accuracy = (
    dataset.obs["cell_type_pred"].loc[pbmc.obs_names] == pbmc.obs[labels]
).sum().item() / pbmc.n_obs
accuracy
```

```
0.9397764644791178
```

The model achieves ~94% accuracy on pbmc cell types, confirming that the shared latent space captures biologically meaningful structure. Below we visualize the joint UMAP colored by predicted cell type, which also covers the unlabeled covid cells.

```
sc.pl.umap(dataset, color="cell_type_joint")
```

![../../_images/5d6c89415ca83fba8b331fea85fbfef4b8309b5280d6586ca0acd3601e3fc58c.png](https://anndata.scverse.org/_images/5d6c89415ca83fba8b331fea85fbfef4b8309b5280d6586ca0acd3601e3fc58c.png)
