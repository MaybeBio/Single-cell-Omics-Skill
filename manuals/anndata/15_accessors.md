# Accessors and paths

> anndata.acc provides accessor s that create reference s to axis-aligned 1D and 2D arrays in AnnData objects. You can use these to drive e.g. plotting or validation code. For these purposes, they ar...

# Accessors and paths[#](https://anndata.scverse.org/accessors.html#module-anndata.acc)

`anndata.acc` provides [accessor](https://anndata.scverse.org/accessors.html#term-accessor)s that create [reference](https://anndata.scverse.org/accessors.html#term-reference)s to axis-aligned 1D and 2D arrays in [`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData) objects. You can use these to drive e.g. plotting or validation code. For these purposes, they are

1. easy to create:

The central [`A`](https://anndata.scverse.org/accessors.html#anndata.acc.A) object is an [accessor](https://anndata.scverse.org/accessors.html#term-accessor) for the whole [`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData) object, and allows you to create [`AdRef`](https://anndata.scverse.org/generated/anndata.acc.AdRef.html#anndata.acc.AdRef) objects, which are [reference](https://anndata.scverse.org/accessors.html#term-reference)s to arrays spanning one or two dimensions of an [`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData) object (without being bound to a specific object):

```
>>> from anndata.acc import A
>>> A.X[:, "gene-3"]  # reference to `adata[:, "gene-3"].X` as 1D vector
A.X[:, 'gene-3']
>>> type(A.X[:, "gene-3"])
<class 'anndata.acc.AdRef'>
```

… and to use:

```
>>> import scanpy as sc
>>> adata = sc.datasets.pbmc3k_processed()
```

E.g. to check if `adata.varm["PCs"]` has at least 30 columns:

```
>>> A.varm["PCs"][:, 30] in adata
True
```

or to extract the referenced vector:

```
>>> adata[A.obs["louvain"]].categories[:2]
Index(['CD4 T cells', 'CD14+ Monocytes'], dtype=...)
>>> adata[A.varm["PCs"]]
array([[...
      ...]],... dtype=float32)
```
2. introspectible:

[`AdRef`](https://anndata.scverse.org/generated/anndata.acc.AdRef.html#anndata.acc.AdRef)s have the [`AdRef.dims`](https://anndata.scverse.org/generated/anndata.acc.AdRef.html#anndata.acc.AdRef.dims), [`AdRef.idx`](https://anndata.scverse.org/generated/anndata.acc.AdRef.html#anndata.acc.AdRef.idx), and [`AdRef.acc`](https://anndata.scverse.org/generated/anndata.acc.AdRef.html#anndata.acc.AdRef.acc) attributes, allowing you to inspect all relevant properties.

```
>>> pc0 = A.obsm["pca"][:, 0]
>>> pc0
A.obsm['pca'][:, 0]
>>> pc0.idx
0
>>> pc0.acc
A.obsm['pca']
>>> A.var["symbol"].dims
{'var'}
>>> pc0.acc.k
'pca'
```
3. convenient:

Want to reference multiple vectors from the same object? Pass a list of indices to the vector accessor:

```
>>> A.obsp["connectivities"][:, ["cell0", "cell1"]]
[A.obsp['connectivities'][:, 'cell0'], A.obsp['connectivities'][:, 'cell1']]
```
4. extensible: see [extending accessors](https://anndata.scverse.org/accessors.html#extending-accessors).

## API & Glossary[#](https://anndata.scverse.org/accessors.html#api-glossary)

The central [accessor](https://anndata.scverse.org/accessors.html#term-accessor) is `A`:

anndata.acc.A:[AdAcc](https://anndata.scverse.org/generated/anndata.acc.AdAcc.html#anndata.acc.AdAcc)[[AdRef](https://anndata.scverse.org/generated/anndata.acc.AdRef.html#anndata.acc.AdRef)]=A[#](https://anndata.scverse.org/accessors.html#anndata.acc.A)

A global accessor to create [`AdRef`](https://anndata.scverse.org/generated/anndata.acc.AdRef.html#anndata.acc.AdRef)s.

See [`AdAcc`](https://anndata.scverse.org/generated/anndata.acc.AdAcc.html#anndata.acc.AdAcc) for examples of how to use it to create [reference](https://anndata.scverse.org/accessors.html#term-reference)s (i.e., [`AdRef`](https://anndata.scverse.org/generated/anndata.acc.AdRef.html#anndata.acc.AdRef)s).

| AdRef(acc, idx) | A reference to a 1D or 2D array along one or two dimensions of an AnnData object. |
|-----------------|-----------------------------------------------------------------------------------|

reference[#](https://anndata.scverse.org/accessors.html#term-reference)

An instance of [`AdRef`](https://anndata.scverse.org/generated/anndata.acc.AdRef.html#anndata.acc.AdRef). References a 1D or 2D array in [`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData) objects. It is independent of individual objects and can be inspected, checked for equality, used as mapping keys, or applied to concrete objects, e.g. via `refinadata` or `adata[ref]`. An example of this would be `A.obsm["d"][:,2]` but not `A.obsm["d"]`, which is a [reference accessor](https://anndata.scverse.org/accessors.html#term-reference-accessor).

accessor[#](https://anndata.scverse.org/accessors.html#term-accessor)

An instance of any of the `*Acc` classes, i.e. [`AdAcc`](https://anndata.scverse.org/generated/anndata.acc.AdAcc.html#anndata.acc.AdAcc), or subclasses of [`MapAcc`](https://anndata.scverse.org/generated/anndata.acc.MapAcc.html#anndata.acc.MapAcc) or [`RefAcc`](https://anndata.scverse.org/generated/anndata.acc.RefAcc.html#anndata.acc.RefAcc). Can be descended into via attribute access to get deeper accessors (e.g. `A` → `A.obs`) or references (e.g. `A.obs.index`, `A.obs["c"]`). Their presence in an anndata object can also be checked via `accinadata`.

reference accessor[#](https://anndata.scverse.org/accessors.html#term-reference-accessor)

`RefAcc` subclasses directly create [reference](https://anndata.scverse.org/accessors.html#term-reference)s ([`AdRef`](https://anndata.scverse.org/generated/anndata.acc.AdRef.html#anndata.acc.AdRef) instances). They can be accessed from these references using the [`AdRef.acc`](https://anndata.scverse.org/generated/anndata.acc.AdRef.html#anndata.acc.AdRef.acc) attribute, and are therefore useful in [matches](https://docs.python.org/3/reference/compound_stmts.html#match) or [`isinstance()`](https://docs.python.org/3/builtins/functions.html#isinstance) checks:

|  |
|--|

| Class | has attributes | available asA.??? | Examplereferencecreation |
|-------|----------------|-------------------|--------------------------|
| LayerAcc | k | X,layers[key] | A.X[:,:],A.layers["c"][:,"g0"] |
| MetaAcc | dim | obs,var | A.obs["a"],A.var["b"] |
| MultiAcc | dim,k | obsm[key],varm[key] | A.obsm["d"][:,2] |
| GraphAcc | dim,k | obsp[key],varp[key] | A.obsp["e"][:,"c1"],A.vbsp["e"]["g0",:] |

mapping accessor[#](https://anndata.scverse.org/accessors.html#term-mapping-accessor)

`MapAcc` subclasses can be indexed with a string to create [reference accessor](https://anndata.scverse.org/accessors.html#term-reference-accessor)s, e.g. `A.layers` or `A.obsm` are both `MapAcc`s, while `A.layers["a"]` is a [`LayerAcc`](https://anndata.scverse.org/generated/anndata.acc.LayerAcc.html#anndata.acc.LayerAcc) and `A.obsm["b"]` is a [`MultiAcc`](https://anndata.scverse.org/generated/anndata.acc.MultiAcc.html#anndata.acc.MultiAcc). `MapAcc`s are mostly useful for extending, but might be useful for APIs that need to refer to a [`Mapping`](https://docs.python.org/3/library/collections.abc.html#collections.abc.Mapping) of arrays:

| LayerMapAcc(*, ref_class[, ref_acc_cls]) | Accessor for layers (A.layers). |
|------------------------------------------|---------------------------------|
| MultiMapAcc(dim, *, ref_class[, ref_acc_cls]) | Accessor for multi-dimensional array containers (A.obsm/A.varm). |
| GraphMapAcc(dim, *, ref_class[, ref_acc_cls]) | Accessor for graph containers (A.obsp/A.varp). |

## Serialization[#](https://anndata.scverse.org/accessors.html#serialization)

[reference](https://anndata.scverse.org/accessors.html#term-reference)s round-trip through JSON-compatible lists via [`AdAcc.to_json()`](https://anndata.scverse.org/generated/anndata.acc.AdAcc.html#anndata.acc.AdAcc.to_json) and [`AdAcc.from_json()`](https://anndata.scverse.org/generated/anndata.acc.AdAcc.html#anndata.acc.AdAcc.from_json):

```
>>> A.to_json(A.obsm["pca"][:, 0])
['obsm', 'pca', 0]
>>> A.from_json(["obsm", "pca", 0])
A.obsm['pca'][:, 0]
```

So do the [reference accessor](https://anndata.scverse.org/accessors.html#term-reference-accessor)s they came from, as the same lists without the trailing index:

```
>>> A.to_json(A.obsm["pca"])
['obsm', 'pca']
>>> A.from_json(["layers", None])
A.X
```

Pass `vec=True` or `vec=False` to [`from_json()`](https://anndata.scverse.org/generated/anndata.acc.AdAcc.html#anndata.acc.AdAcc.from_json) to accept only [reference](https://anndata.scverse.org/accessors.html#term-reference)s or only [reference accessor](https://anndata.scverse.org/accessors.html#term-reference-accessor)s, like [`AdAcc.resolve()`](https://anndata.scverse.org/generated/anndata.acc.AdAcc.html#anndata.acc.AdAcc.resolve) does for strings. [`MetaAcc`](https://anndata.scverse.org/generated/anndata.acc.MetaAcc.html#anndata.acc.MetaAcc) (i.e. [`obs`](https://anndata.scverse.org/generated/anndata.acc.AdAcc.html#anndata.acc.AdAcc.obs)/[`var`](https://anndata.scverse.org/generated/anndata.acc.AdAcc.html#anndata.acc.AdAcc.var)) is the exception: only its [reference](https://anndata.scverse.org/accessors.html#term-reference)s are serializable, as it has no array of its own.

Both are described by [acc-schema-v1.json](https://anndata.scverse.org/acc-schema-v1.json), as separate subschemas you can validate against independently using JSON pointers into it: `#/$defs/ref` matches [reference](https://anndata.scverse.org/accessors.html#term-reference)s, `#/$defs/acc`[reference accessor](https://anndata.scverse.org/accessors.html#term-reference-accessor)s, and the schema itself either.

## Extending accessors[#](https://anndata.scverse.org/accessors.html#extending-accessors)

There are three layers of extensibility:

1. subclassing [`AdRef`](https://anndata.scverse.org/generated/anndata.acc.AdRef.html#anndata.acc.AdRef) and creating a new [`AdAcc`](https://anndata.scverse.org/generated/anndata.acc.AdAcc.html#anndata.acc.AdAcc) instance for creating them:

```
from matplotlib import pyplot as plt
from anndata.acc import AdAcc, AdRef

class MplRef(AdRef, str):
    """Matplotlib will only treat strings as references, so we subclass `str`."""
    def __new__(cls, acc, idx) -> None:
        obj = str.__new__(cls, str(AdRef(acc, idx)))
        AdRef.__init__(obj, acc, idx)
        return obj

A = AdAcc(ref_class=MplRef)

adata = sc.datasets.pbmc3k_processed()
plt.scatter(*A.obsm["X_umap"][:, [0, 1]], c=A.obs["n_counts"], data=adata)
```
2. subclass one or more of the [reference accessor](https://anndata.scverse.org/accessors.html#term-reference-accessor)s, and create a new [`AdAcc`](https://anndata.scverse.org/generated/anndata.acc.AdAcc.html#anndata.acc.AdAcc) instance:

```
>>> from anndata.acc import AdAcc, AdRef, MetaAcc
>>>
>>> class TwoDRef(AdRef):
...     """A reference able to refer to multiple metadata columns."""
...     ...
>>>
>>> class MyMetaAcc(MetaAcc):
...     def __getitem__(self, k):
...         if isinstance(k, list):
...             # override default behavior of returning a list of refs
...             return self.ref_class(self, k)
...         return super().__getitem__(k)
>>>
>>> A = AdAcc(ref_class=TwoDRef, meta_cls=MyMetaAcc)
>>> A.obs[["a", "b"]]
A.obs[['a', 'b']]
```

Note that for implementing [`RefAcc.get()`](https://anndata.scverse.org/generated/anndata.acc.RefAcc.html#anndata.acc.RefAcc.get) when subclassing, you will need [`NO_IDX`](https://anndata.scverse.org/generated/anndata.acc.NO_IDX.html#anndata.acc.NO_IDX).
3. subclass [`AdAcc`](https://anndata.scverse.org/generated/anndata.acc.AdAcc.html#anndata.acc.AdAcc) to add new accessors:

```
>>> from dataclasses import dataclass, field
>>> from anndata.acc import AdAcc, MetaAcc
>>>
>>> @dataclass(frozen=True)
... class EHRAcc(AdAcc):
...     tem: MetaAcc = field(init=False)
...     def __post_init__(self) -> None:
...         super().__post_init__()
...         tem = MetaAcc("tem", ref_class=self.ref_class)
...         object.__setattr__(self, "tem", tem)  # necessary because it’s frozen
>>>
>>> A = EHRAcc()
>>> A.tem["visit_id"]
A.tem['visit_id']
```

To make your subclasses writable (see [accessor encoding](https://anndata.scverse.org/fileformat-prose.html#accessor)), register your [`AdAcc`](https://anndata.scverse.org/generated/anndata.acc.AdAcc.html#anndata.acc.AdAcc) instance as an `anndata.acc` entry point named after your project, and make sure its [`to_json()`](https://anndata.scverse.org/generated/anndata.acc.AdAcc.html#anndata.acc.AdAcc.to_json) and [`from_json()`](https://anndata.scverse.org/generated/anndata.acc.AdAcc.html#anndata.acc.AdAcc.from_json) round-trip them:

```
[project.entry-points."anndata.acc"]
my-project = "my_project.acc:A"
```

anndata picks the entry point whose module is in the same top-level package as your classes, and stores `my-project` with the written element so it can find your [`AdAcc`](https://anndata.scverse.org/generated/anndata.acc.AdAcc.html#anndata.acc.AdAcc) when reading it. If no entry point matches, it isn’t named after the project that registers it, or your [`AdAcc`](https://anndata.scverse.org/generated/anndata.acc.AdAcc.html#anndata.acc.AdAcc) cannot reproduce the original accessor from its JSON form, writing fails instead of losing information.
