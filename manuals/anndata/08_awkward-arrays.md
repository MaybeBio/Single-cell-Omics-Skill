# Awkward Arrays in AnnData

> Author: Gregor Sturm Awkward Array is a library for working with nested, variable-sized data using NumPy-like idioms. It is considerably faster than working with lists-of-lists or lists-of-dicts in...

# Awkward Arrays in AnnData[#](https://anndata.scverse.org/tutorials/notebooks/awkward-arrays.html#awkward-arrays-in-anndata)

**Author**: [Gregor Sturm](https://github.com/grst)

Warning

Support for awkward arrays in AnnData is **experimental**.

Behavior, in particular of [`concat()`](https://anndata.scverse.org/generated/anndata.concat.html#anndata.concat), may change in the future. Please report any issues using the [issue tracker](https://github.com/scverse/anndata)

```
from __future__ import annotations

import awkward as ak
import scanpy as sc
from biothings_client import get_client as get_biothings_client
```

[Awkward Array](https://awkward-array.org/doc/main/) is a library for working with **nested, variable-sized data** using **NumPy-like idioms**. It is considerably faster than working with lists-of-lists or lists-of-dicts in Python.

Here are two simple examples what an awkward array could look like:

**ragged array:**

```
ragged = ak.Array([
    None,
    [1, 2, 3],
    [3, 4],
])
ragged[:, 1]
```

```
[None,
 2,
 4]
----------------
type: 3 * ?int64
```

**list of records:**

```
records = ak.Array([
    {"a": 1, "b": 2},
    {"a": 3, "c": 4},
    {"d": 5},
])
records["a"]
```

```
[1,
 3,
 None]
----------------
type: 3 * ?int64
```

Please refer to the [awkward array documentation](https://awkward-array.org) for more information.

Since v0.9, awkward arrays are supported in AnnData in the `.layers`, `.obsm`, `.varm` and `.uns` slots.

In the following, we will explore how awkward arrays can be useful when working with single-cell data.

## Storing transcripts in `.varm`[#](https://anndata.scverse.org/tutorials/notebooks/awkward-arrays.html#storing-transcripts-in-varm)

Every gene can have one or many transcripts. Using awkward arrays, we can store a ragged list of transcripts for each gene in `adata.varm`.

```
adata = sc.datasets.pbmc3k()
```

```
adata.var.head()
```

|  | gene_ids |
|--|----------|
|  |  |
| ENSG00000243485 |  |
| ENSG00000237613 |  |
| ENSG00000186092 |  |
| ENSG00000238009 |  |
| ENSG00000239945 |  |

Let’s retrieve a list of transcripts for each gene using the [MyGene.info API](https://docs.mygene.info/en/latest/).

```
mygene = get_biothings_client("gene")
```

```
%%capture
mygene_res = mygene.querymany(
    adata.var["gene_ids"],
    scopes=["ensembl.gene"],
    fields=["ensembl.transcript"],
    species="human",
    as_dataframe=True,
)
# remove duplicated results
mygene_res = mygene_res.loc[~mygene_res.index.duplicated()]
assert adata.var["gene_ids"].tolist() == mygene_res.index.tolist(), (
    "Order of genes does not match"
)
```

The API call returns a data frame with transcripts in the `ensembl.transcript` key:

```
mygene_res.head()
```

|  | _id | _score | ensembl.transcript | notfound | ensembl |
|--|-----|--------|--------------------|----------|---------|
|  |  |  |  |  |  |
| ENSG00000243485 | 25.719067 | [ENST00000469289, ENST00000473358] | NaN | NaN |  |
| 645520 | 25.719007 | [ENST00000417324, ENST00000461467] | NaN | NaN |  |
| 79501 | 24.912605 | ENST00000641515 | NaN | NaN |  |
| ENSG00000238009 | 25.719582 | [ENST00000453576, ENST00000466430, ENST0000047... | NaN | NaN |  |
| ENSG00000239945 | 25.719145 | ENST00000495576 | NaN | NaN |  |

Let’s construct an awkward Array from the list of dictionaries and assign the ragged list of transcripts to `adata.varm`

```
adata.varm["transcripts"] = ak.Array(mygene_res["ensembl.transcript"])
```

We can now access transcripts of individual genes by slicing the AnnData object:

```
adata[:, ["CD8A", "CXCL13"]].varm["transcripts"]
```

```
[['ENST00000283635', 'ENST00000352580', ..., 'ENST00000699439'],
 ['ENST00000286758', 'ENST00000506590', 'ENST00000682537']]
----------------------------------------------------------------------------------------------------------------
type: 2 * union[
    var * string,
    string,
    float64, 
parameters={"_view_args": ["target-140242149270096", "varm", ["transcripts"]], "__array__": "AwkwardArrayView"}]
```
