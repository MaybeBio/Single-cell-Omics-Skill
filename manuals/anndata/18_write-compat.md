# Write compatibility profiles

> Every so often, the python package anndata starts using an on-disk encoding that older readers won’t be able to read, whether that’s an older python anndata or an implementation in another language...

# Write compatibility profiles[#](https://anndata.scverse.org/write-compat.html#write-compatibility-profiles)

Every so often, the python package `anndata` starts using an on-disk encoding that older readers won’t be able to read, whether that’s an older python `anndata` or an implementation in another language.

Every write function takes a `compat` argument selecting which anndata version’s write behavior to target, as a [`WriteCompat`](https://anndata.scverse.org/generated/anndata.WriteCompat.html#anndata.WriteCompat) member or its string value ([`WriteCompatStr`](https://anndata.scverse.org/generated/anndata.types.WriteCompatStr.html#anndata.types.WriteCompatStr)):

```
adata.write_h5ad("old.h5ad", compat="0.13")
```

A profile is the oldest anndata python-package version whose I/O behavior we promise to be compatible with, i.e. encodings that version can’t read are disallowed. New encodings only land in minor releases until major releases start, so e.g. `"0.13"` files are readable by every 0.13.x release.

This is different from the `encoding-version` attribute (see [On-disk format](https://anndata.scverse.org/fileformat-prose.html)): each element (including the root `anndata` group), uses it to version only its own layout, not its children’s, while a profile is a promise about every element in the file: which `anndata` version can read them all.

Profiles are named after versions of the python package, but they describe sets of encodings, so they apply to other implementations too: a reader that supports every encoding that python `anndata` X.Y can read can read all files written with profile `"X.Y"`.

## Profiles[#](https://anndata.scverse.org/write-compat.html#profiles)

| Profile | Released | Effect |
|---------|----------|--------|
| "0.13" | 2026-07-07 | This is the default. |
| "0.14" | unreleased | Enables theaccessorencoding foranndata.accaccessors.
Older versions can’t read it. |

## Picking individual features[#](https://anndata.scverse.org/write-compat.html#picking-individual-features)

Each profile is a [`WriteConfig`](https://anndata.scverse.org/generated/anndata.WriteConfig.html#anndata.WriteConfig) with one boolean flag per feature. To pick features individually, pass a [`WriteConfig`](https://anndata.scverse.org/generated/anndata.WriteConfig.html#anndata.WriteConfig) as `compat`:

```
adata.write_h5ad("new.h5ad", compat=ad.WriteConfig(accessor=True))
```

## Checking before writing[#](https://anndata.scverse.org/write-compat.html#checking-before-writing)

[`anndata.AnnData.unwriteable()`](https://anndata.scverse.org/generated/anndata.AnnData.unwriteable.html#anndata.AnnData.unwriteable) takes the same `compat` argument, so you can check an object against a profile without writing anything:

```
if adata.unwriteable(compat="0.13"):
    ...
```

## The default profile[#](https://anndata.scverse.org/write-compat.html#the-default-profile)

The default is [`anndata.WriteCompat.DEFAULT`](https://anndata.scverse.org/generated/anndata.WriteCompat.html#anndata.WriteCompat.DEFAULT), later features are disabled.

The default keeps lagging behind the newest profile by roughly a year, so files stay readable by the anndata versions people are likely to have. Pass the newest profile as `compat` to opt into everything the installed anndata can write.
