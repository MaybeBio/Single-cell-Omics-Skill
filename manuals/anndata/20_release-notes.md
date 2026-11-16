# Release notes

> Version 0.14: 0.14.0.dev98+g7c3b0f8ac 2026-11-12: Breaking changes: Drop support for Python 3.12 and raise the minimum versions of h5py to 3.12, numpy to 2.2, and scipy to 1.15, following SPEC 0@fl...

# Release notes[#](https://anndata.scverse.org/release-notes/index.html#release-notes)

## Version 0.14[#](https://anndata.scverse.org/release-notes/index.html#v0-14)

### 0.14.0.dev98+g7c3b0f8ac 2026-11-12[#](https://anndata.scverse.org/release-notes/index.html#v0-14-0-dev98-g7c3b0f8ac)

#### Breaking changes[#](https://anndata.scverse.org/release-notes/index.html#breaking-changes)

- Drop support for Python 3.12 and raise the minimum versions of `h5py` to 3.12, `numpy` to 2.2, and `scipy` to 1.15, following [SPEC 0](https://scientific-python.org/specs/spec-0000/)[@flying-sheep](https://github.com/sponsors/flying-sheep) ([#2703](https://github.com/scverse/anndata/pull/2703))
- Reject [`cupy`](https://docs.cupy.dev/en/stable/index.html#module-cupy) sparse matrices other than CSR and CSC (e.g. COO) when assigning them to [`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData), like it was already done for [`scipy.sparse`](https://docs.scipy.org/doc/scipy/reference/sparse.html#module-scipy.sparse)[@flying-sheep](https://github.com/sponsors/flying-sheep) ([#2716](https://github.com/scverse/anndata/pull/2716))

#### Miscellaneous changes[#](https://anndata.scverse.org/release-notes/index.html#miscellaneous-changes)

- Type check `src` and `tests` with [mypy](https://mypy.readthedocs.io/) and sync the repository configuration with the [cookiecutter-scverse](https://github.com/scverse/cookiecutter-scverse) template ([#2577](https://github.com/scverse/anndata/pull/2577))
- Disable [`zarr`](https://zarr.readthedocs.io/en/stable/api/zarr/#zarr) format v2 writing via the now-removed setting `anndata.settings.zarr_write_format`[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2628](https://github.com/scverse/anndata/pull/2628))

#### Documentation[#](https://anndata.scverse.org/release-notes/index.html#documentation)

- Remove the outdated benchmarks documentation page Zethson ([#2538](https://github.com/scverse/anndata/pull/2538))
- Group the docs sidebar into Getting started, Guides, Reference and Project like the other scverse core packages, with new installation and citation pages [@Zethson](https://github.com/sponsors/Zethson) ([#2731](https://github.com/scverse/anndata/pull/2731))

#### Features[#](https://anndata.scverse.org/release-notes/index.html#features)

- Added type [`AlignedArray`](https://anndata.scverse.org/typing.html#anndata.typing.AlignedArray) and renamed `AxisStorable` to [`Storable`](https://anndata.scverse.org/typing.html#anndata.typing.Storable)[@flying-sheep](https://github.com/sponsors/flying-sheep) ([#2577](https://github.com/scverse/anndata/pull/2577))
- [`write_h5ad()`](https://anndata.scverse.org/generated/anndata.io.write_h5ad.html#anndata.io.write_h5ad) and [`write_zarr()`](https://anndata.scverse.org/generated/anndata.io.write_zarr.html#anndata.io.write_zarr) now store dask arrays with on-disk chunks matching the dask array chunks. ([#2629](https://github.com/scverse/anndata/pull/2629)) ([#2629](https://github.com/scverse/anndata/pull/2629))
- Add write compatibility profiles: a `compat` argument on the write functions and [`unwriteable()`](https://anndata.scverse.org/generated/anndata.AnnData.unwriteable.html#anndata.AnnData.unwriteable) selects which anndata version’s write behavior to target ([`WriteCompat`](https://anndata.scverse.org/generated/anndata.WriteCompat.html#anndata.WriteCompat), or individual [`WriteConfig`](https://anndata.scverse.org/generated/anndata.WriteConfig.html#anndata.WriteConfig) flags), see [Write compatibility profiles](https://anndata.scverse.org/write-compat.html)P Angerer ([#2642](https://github.com/scverse/anndata/pull/2642))
- Added the `accessor` encoding, enabled via `compat="0.14"`: [`anndata.acc`](https://anndata.scverse.org/accessors.html#module-anndata.acc) accessors such as `A.obsm["pca"][0]` are stored as their [`to_json()`](https://anndata.scverse.org/generated/anndata.acc.AdAcc.html#anndata.acc.AdAcc.to_json) form, serialized to a JSON string. Packages extending [`anndata.acc`](https://anndata.scverse.org/accessors.html#module-anndata.acc) can make their accessors writable via an `anndata.acc` entry point P Angerer ([#2658](https://github.com/scverse/anndata/pull/2658))
- Add a `use` argument to [`write_h5ad()`](https://anndata.scverse.org/generated/anndata.AnnData.write_h5ad.html#anndata.AnnData.write_h5ad) and [`anndata.AnnData.write_zarr()`](https://anndata.scverse.org/generated/anndata.AnnData.write_zarr.html#anndata.AnnData.write_zarr) that the enables writing of subelements directly from the [`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData) object via [Accessors and paths](https://anndata.scverse.org/accessors.html)[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2707](https://github.com/scverse/anndata/pull/2707))
- Add [`anndata.AnnData.__setitem__()`](https://anndata.scverse.org/generated/anndata.AnnData.__setitem__.html#anndata.AnnData.__setitem__) and [`anndata.acc.RefAcc.set()`](https://anndata.scverse.org/generated/anndata.acc.RefAcc.html#anndata.acc.RefAcc.set) to set arrays via [Accessors and paths](https://anndata.scverse.org/accessors.html), e.g. `adata[A.obs["x"]]=...`[@flying-sheep](https://github.com/sponsors/flying-sheep) ([#2724](https://github.com/scverse/anndata/pull/2724))

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#bug-fixes)

- Fix a number of latent crashes surfaced by type checking, among them reading a backed `zarr` element (an internal `filename` lookup always raised `AttributeError` since zarr v3), `read_elem` on an attribute with a default, `obs_keys()`/`var_keys()` on xarray-backed objects, `write_loom` emitting a `None`-named layer, and `__sizeof__` on a COO matrix in `uns` ([#2577](https://github.com/scverse/anndata/pull/2577))
- Remove requirement that `{obs,var}m`[`pandas.DataFrame`](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.html#pandas.DataFrame) indexes match `obs_names` because it is not in the spec. [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2641](https://github.com/scverse/anndata/pull/2641))
- Raise an [`IndexError`](https://docs.python.org/3/builtins/exceptions.html#IndexError) for float index arrays where only some values are non-integers, instead of truncating them [@haeganm](https://github.com/sponsors/haeganm) ([#2652](https://github.com/scverse/anndata/pull/2652))
- Validate the structure of sparse arrays on read and write, so corrupt files raise a [`ValueError`](https://docs.python.org/3/builtins/exceptions.html#ValueError) instead of allocating memory according to a bogus `shape`[@flying-sheep](https://github.com/sponsors/flying-sheep) ([#2665](https://github.com/scverse/anndata/pull/2665))
- Support [`pandas`](https://pandas.pydata.org/docs/index.html#module-pandas) 3.1: writing nullable string arrays no longer triggers a deprecation warning by accessing the `future.infer_string` option [@flying-sheep](https://github.com/sponsors/flying-sheep) ([#2671](https://github.com/scverse/anndata/pull/2671))
- Honour `max_loaded_elems` in [`anndata.experimental.concat_on_disk()`](https://anndata.scverse.org/generated/anndata.experimental.concat_on_disk.html#anndata.experimental.concat_on_disk) when the inputs need no reindexing, instead of loading whole sparse matrices into memory [@JuanTejedor](https://github.com/sponsors/JuanTejedor) ([#2678](https://github.com/scverse/anndata/pull/2678))
- If auto-sharding is on for zarr, the default, on-disk shards will match dask chunks and sensible subchunks will be formed. [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2680](https://github.com/scverse/anndata/pull/2680))
- Ensure columns of an on-disk `dataframe` are added to `column-order` in the on-disk metadata when done via [`write_elem()`](https://anndata.scverse.org/generated/anndata.io.write_elem.html#anndata.io.write_elem)[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2708](https://github.com/scverse/anndata/pull/2708))
- Raise a clear [`TypeError`](https://docs.python.org/3/builtins/exceptions.html#TypeError) when using [`anndata.acc`](https://anndata.scverse.org/accessors.html#module-anndata.acc) to get a vector from a backed (not in-memory) array [@flying-sheep](https://github.com/sponsors/flying-sheep) ([#2716](https://github.com/scverse/anndata/pull/2716))

#### Performance[#](https://anndata.scverse.org/release-notes/index.html#performance)

- Subset [`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData) objects with many observations or variables faster by removing unused categories of categorical annotations only once and only if there are any, and by not re-checking the uniqueness of unchanged names [@Intron7](https://github.com/sponsors/Intron7) ([#2675](https://github.com/scverse/anndata/pull/2675))

## Version 0.13[#](https://anndata.scverse.org/release-notes/index.html#v0-13)

### 0.13.4 2026-09-18[#](https://anndata.scverse.org/release-notes/index.html#v0-13-4)

#### Miscellaneous changes[#](https://anndata.scverse.org/release-notes/index.html#id2)

- The [`zarr`](https://zarr.readthedocs.io/en/stable/api/zarr/#zarr) format v2 writing is now deprecated via `anndata.settings.zarr_write_format` and this setting will be removed in 0.14 [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2628](https://github.com/scverse/anndata/pull/2628))

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id3)

- Fall back to copying via `asarray` when moving arrays between Array API namespaces fails with a [`BufferError`](https://docs.python.org/3/builtins/exceptions.html#BufferError), e.g. for implementations that don’t support DLPack 1.0 [@amalia-k510](https://github.com/sponsors/amalia-k510) ([#2561](https://github.com/scverse/anndata/pull/2561))
- Remove hard-coded scheduler when writing dask arrays to zarr stores [@joshua-gould](https://github.com/sponsors/joshua-gould) ([#2584](https://github.com/scverse/anndata/pull/2584))

#### Performance[#](https://anndata.scverse.org/release-notes/index.html#id4)

- Use the new [`FusedCodecPipeline`](https://zarr.readthedocs.io/en/stable/user-guide/experimental/#fusedcodecpipeline) where we control reading/writing to a zarr store if [zarrs-python](https://zarrs-python.readthedocs.io/en/stable/index.html) is not installed i.e., [`zarr`](https://zarr.readthedocs.io/en/stable/api/zarr/#zarr) i/o [@ilan-gold](https://github.com/sponsors/ilan-gold)

### 0.13.3 2026-08-27[#](https://anndata.scverse.org/release-notes/index.html#v0-13-3)

#### Features[#](https://anndata.scverse.org/release-notes/index.html#id6)

- [`to_json()`](https://anndata.scverse.org/generated/anndata.acc.AdAcc.html#anndata.acc.AdAcc.to_json) and [`from_json()`](https://anndata.scverse.org/generated/anndata.acc.AdAcc.html#anndata.acc.AdAcc.from_json) now also (de)serialize whole container accessors ([`LayerAcc`](https://anndata.scverse.org/generated/anndata.acc.LayerAcc.html#anndata.acc.LayerAcc)/[`MultiAcc`](https://anndata.scverse.org/generated/anndata.acc.MultiAcc.html#anndata.acc.MultiAcc)/[`GraphAcc`](https://anndata.scverse.org/generated/anndata.acc.GraphAcc.html#anndata.acc.GraphAcc)), with `from_json` gaining a `vec` parameter like [`resolve()`](https://anndata.scverse.org/generated/anndata.acc.AdAcc.html#anndata.acc.AdAcc.resolve)[@flying-sheep](https://github.com/sponsors/flying-sheep) ([#2614](https://github.com/scverse/anndata/pull/2614))

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id7)

- Suggest the store class to use when [`read_zarr()`](https://anndata.scverse.org/generated/anndata.io.read_zarr.html#anndata.io.read_zarr) is passed the path of a single-file store such as `adata.zarr.zip`, instead of only raising `GroupNotFoundError`[@JOhnsonKC201](https://github.com/sponsors/JOhnsonKC201) ([#2571](https://github.com/scverse/anndata/pull/2571))
- Fix writing the result of `concat`/`concat_on_disk` with `join="outer"` and a union `merge` strategy when alternate-axis annotations contain `bool` columns, by promoting them to nullable `boolean` on reindexing instead of `object`[@srikarjy](https://github.com/sponsors/srikarjy) ([#2609](https://github.com/scverse/anndata/pull/2609))
- Raise `IndexError` instead of returning an empty array when a `MultiAcc` column index is out of range, and accept negative indices [@dylanpulver](https://github.com/sponsors/dylanpulver) ([#2613](https://github.com/scverse/anndata/pull/2613))

### 0.13.2 2026-07-13[#](https://anndata.scverse.org/release-notes/index.html#v0-13-2)

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id9)

- Keep [`X`](https://anndata.scverse.org/generated/anndata.AnnData.X.html#anndata.AnnData.X) (stored as `layers[None]`) when clearing or reassigning [`layers`](https://anndata.scverse.org/generated/anndata.AnnData.layers.html#anndata.AnnData.layers), matching the existing behavior of `deladata.layers`. `adata.layers={}`, `adata.layers.clear()` and `deladata.layers` now warn that a future release may drop `.X` in these cases without more expliit handling i.e., pass `adata.layers.clear(keep_x=True/False)` or include the `None` key explicitly (`adata.layers={None:adata.X,...}` to keep `X` and `{None:None,...}` to drop) to opt in and silence the warning. [@slambert-git](https://github.com/sponsors/slambert-git) ([#2549](https://github.com/scverse/anndata/pull/2549))
- Allow [`resolve()`](https://anndata.scverse.org/generated/anndata.acc.AdAcc.html#anndata.acc.AdAcc.resolve) to resolve `AdAcc`s (e.g. `A.X`, `A.layers['y']`, `A.obsm['c']`, `A.obsp['g']`) in addition to `AdRef`s, when selected via `vec=False|None` (Resolve to `AdRef`s with `vec=True|None`) [@flying-sheep](https://github.com/sponsors/flying-sheep) ([#2550](https://github.com/scverse/anndata/pull/2550))
- Change `layers:None` in the repr to be `layers:None(.X)` to highlight that `None` is `X`[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2553](https://github.com/scverse/anndata/pull/2553))

### 0.13.1 2026-07-08[#](https://anndata.scverse.org/release-notes/index.html#v0-13-1)

#### Documentation[#](https://anndata.scverse.org/release-notes/index.html#id11)

- Move the concatenation guide into the tutorials section Zethson ([#2539](https://github.com/scverse/anndata/pull/2539))

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id12)

- Lower bound `typing_extensions` module to contain the `sentinel` object [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2540](https://github.com/scverse/anndata/pull/2540))
- Accept array-API arrays that cannot export via DLPack (e.g. [pydata/sparse](https://github.com/pydata/sparse)) in [`X`](https://anndata.scverse.org/generated/anndata.AnnData.X.html#anndata.AnnData.X)/[`layers`](https://anndata.scverse.org/generated/anndata.AnnData.layers.html#anndata.AnnData.layers). [`SupportsArrayApi`](https://anndata.scverse.org/generated/anndata.types.SupportsArrayApi.html#anndata.types.SupportsArrayApi) is split into a DLPack-free [`SupportsArrayApiBase`](https://anndata.scverse.org/generated/anndata.types.SupportsArrayApiBase.html#anndata.types.SupportsArrayApiBase) (used to gate storage) and the DLPack-requiring [`SupportsArrayApi`](https://anndata.scverse.org/generated/anndata.types.SupportsArrayApi.html#anndata.types.SupportsArrayApi) (used only where DLPack is actually needed). [@eroell](https://github.com/sponsors/eroell) ([#2541](https://github.com/scverse/anndata/pull/2541))

### 0.13.0 2026-07-07[#](https://anndata.scverse.org/release-notes/index.html#v0-13-0)

Attention

In anndata 0.14, [`anndata.settings.allow_write_nullable_strings`](https://anndata.scverse.org/generated/anndata.settings.html#anndata.settings.allow_write_nullable_strings) will default to `True`, so take care to test your code with the upcoming default value. (See also “Miscellaneous changes” below.)

#### Breaking changes[#](https://anndata.scverse.org/release-notes/index.html#id14)

- Remove `dtype` argument from [`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData) constructor. [@flying-sheep](https://github.com/sponsors/flying-sheep) ([#1707](https://github.com/scverse/anndata/pull/1707))
- Remove internal use of inheritance inside [`CSRDataset`](https://anndata.scverse.org/generated/anndata.abc.CSRDataset.html#anndata.abc.CSRDataset) and [`CSCDataset`](https://anndata.scverse.org/generated/anndata.abc.CSCDataset.html#anndata.abc.CSCDataset) from [`scipy.sparse.spmatrix`](https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.spmatrix.html#scipy.sparse.spmatrix). Users can thus no longer write to backed sparse stores but now the backed sparse classes can return [`cupyx.scipy.sparse.csr_matrix`](https://docs.cupy.dev/en/stable/reference/generated/cupyx.scipy.sparse.csr_matrix.html#cupyx.scipy.sparse.csr_matrix) and [`cupyx.scipy.sparse.csc_matrix`](https://docs.cupy.dev/en/stable/reference/generated/cupyx.scipy.sparse.csc_matrix.html#cupyx.scipy.sparse.csc_matrix) when configured properly with [`zarr`](https://zarr.readthedocs.io/en/stable/api/zarr/#zarr). [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1927](https://github.com/scverse/anndata/pull/1927))
- Remove support for [`zarr`](https://zarr.readthedocs.io/en/stable/api/zarr/#zarr)`&lt;3` dependency [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2309](https://github.com/scverse/anndata/pull/2309))
- [`anndata.AnnData.X`](https://anndata.scverse.org/generated/anndata.AnnData.X.html#anndata.AnnData.X) is now copy-on-write like the other elements like [`layers`](https://anndata.scverse.org/generated/anndata.AnnData.layers.html#anndata.AnnData.layers). Operations like `my_subset.X=0` will *not* propagate back to original object. [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2338](https://github.com/scverse/anndata/pull/2338))
- Remove `Anndata.__{set,del}item__`[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2367](https://github.com/scverse/anndata/pull/2367))
- Remove `AnnData.concatenate`[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2370](https://github.com/scverse/anndata/pull/2370))

#### Miscellaneous changes[#](https://anndata.scverse.org/release-notes/index.html#id15)

- Switch [`anndata.settings.allow_write_nullable_strings`](https://anndata.scverse.org/generated/anndata.settings.html#anndata.settings.allow_write_nullable_strings) to new default `None`, which means the writing of pandas string arrays will be controlled by `pandas.options.future.infer_string` (see the [Migration Guides](https://pandas.pydata.org/docs/user_guide/migration.html)). Setting `allow_write_nullable_strings=False` explicitly now means to try writing the array in the non-nullable string array format, which was the only supported format up to anndata 0.11. P Angerer ([#2133](https://github.com/scverse/anndata/pull/2133))
- Flip default for [`anndata.settings.disallow_forward_slash_in_h5ad`](https://anndata.scverse.org/generated/anndata.settings.html#anndata.settings.disallow_forward_slash_in_h5ad) to `True`. Document writing to `k="/"` in [`write_elem()`](https://anndata.scverse.org/generated/anndata.io.write_elem.html#anndata.io.write_elem) and check `k` more strictly [@flying-sheep](https://github.com/sponsors/flying-sheep) ([#2436](https://github.com/scverse/anndata/pull/2436))
- Add `consolidate_metadata` argument to [`write_zarr()`](https://anndata.scverse.org/generated/anndata.AnnData.write_zarr.html#anndata.AnnData.write_zarr) to allow opting out of consolidated metadata easily. No longer warn on its writing as it appears to be heading for stability in zarr. [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2497](https://github.com/scverse/anndata/pull/2497))

#### Features[#](https://anndata.scverse.org/release-notes/index.html#id16)

- Move [`X`](https://anndata.scverse.org/generated/anndata.AnnData.X.html#anndata.AnnData.X) into [`anndata.AnnData.layers`](https://anndata.scverse.org/generated/anndata.AnnData.layers.html#anndata.AnnData.layers) under the `None` key. As result, `layers:X` shows up in `AnnData`’s text representation, `None` appears in `.layers.items()`/`.keys()`, [`concat()`](https://anndata.scverse.org/generated/anndata.concat.html#anndata.concat) now works when some `AnnData`s have no `X`. [@flying-sheep](https://github.com/sponsors/flying-sheep) & [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1707](https://github.com/scverse/anndata/pull/1707))
- Support setting with [array-api compatible](https://data-apis.org/array-api/latest/) objects i.e., `adata.X=jax.numpy.array([[0,1],[2,3]])`[@ilan-gold](https://github.com/sponsors/ilan-gold)[@amalia-k510](https://github.com/sponsors/amalia-k510) ([#2071](https://github.com/scverse/anndata/pull/2071))
- Add support for Dask [`dask.array.Array`](https://docs.dask.org/en/stable/generated/dask.array.Array.html#dask.array.Array)s containing [`sparray`](https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.sparray.html#scipy.sparse.sparray)s [@flying-sheep](https://github.com/sponsors/flying-sheep) ([#2087](https://github.com/scverse/anndata/pull/2087))
- Add [Accessors and paths](https://anndata.scverse.org/accessors.html#accessors) for referencing vectors in [`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData) and [`MuData`](https://mudata.scverse.org/stable/generated/mudata.MuData.html#mudata.MuData)[@flying-sheep](https://github.com/sponsors/flying-sheep) ([#2290](https://github.com/scverse/anndata/pull/2290))
- New [`anndata.AnnData.unwriteable()`](https://anndata.scverse.org/generated/anndata.AnnData.unwriteable.html#anndata.AnnData.unwriteable) for checking if an `AnnData` can be written [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2372](https://github.com/scverse/anndata/pull/2372))
- Create a new setting [`anndata.settings.restrict_index_types`](https://anndata.scverse.org/generated/anndata.settings.html#anndata.settings.restrict_index_types) to allow for custom in-memory indexes (i.e., for [`obs`](https://anndata.scverse.org/generated/anndata.AnnData.obs.html#anndata.AnnData.obs) and [`var`](https://anndata.scverse.org/generated/anndata.AnnData.var.html#anndata.AnnData.var)) to be passed to an [`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData) object without being converted to a string type. Note that indexes recognized by [`pandas.api.types.is_integer_dtype()`](https://pandas.pydata.org/docs/reference/api/pandas.api.types.is_integer_dtype.html#pandas.api.types.is_integer_dtype) will still be stringified regardless of the setting. [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2379](https://github.com/scverse/anndata/pull/2379))
- `anndata.settings.zarr_write_format` is now 3 and [`anndata.settings.auto_shard_zarr_v3()`](https://anndata.scverse.org/generated/anndata.settings.html#anndata.settings.auto_shard_zarr_v3) is now `True` (with `None` removed as an option). This means that sharded zarr v3 stores will be written with a target uncompressed shard size of 1GB unless the user either

- changes one of these flags, or
- explicitly creates a [`zarr.Group`](https://zarr.readthedocs.io/en/stable/api/zarr/group/#zarr.Group) with `zarr_format=2`, or
- overrides the `shards` in `dataset_kwargs` passed in to [`write_elem()`](https://anndata.scverse.org/generated/anndata.io.write_elem.html#anndata.io.write_elem)

[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2439](https://github.com/scverse/anndata/pull/2439))
- Add `extra` for CUDA 13 (`anndata[cu13]`) [@grst](https://github.com/sponsors/grst)

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id17)

- Fix [`anndata.AnnData.copy()`](https://anndata.scverse.org/generated/anndata.AnnData.copy.html#anndata.AnnData.copy) so that it provides an informative error when trying to `copy` and object that contains [`h5py.Dataset`](https://docs.h5py.org/en/latest/high/dataset.html#h5py.Dataset), [`zarr.Array`](https://zarr.readthedocs.io/en/stable/api/zarr/array/#zarr.Array), [`anndata.abc.CSRDataset`](https://anndata.scverse.org/generated/anndata.abc.CSRDataset.html#anndata.abc.CSRDataset), or [`anndata.abc.CSCDataset`](https://anndata.scverse.org/generated/anndata.abc.CSCDataset.html#anndata.abc.CSCDataset)[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2406](https://github.com/scverse/anndata/pull/2406))
- Enforce the 2-dimensional [`X`](https://anndata.scverse.org/generated/anndata.AnnData.X.html#anndata.AnnData.X)/[`layers`](https://anndata.scverse.org/generated/anndata.AnnData.layers.html#anndata.AnnData.layers) spec at the IO boundary: higher-dimensional arrays now warn (rather than error for [`X`](https://anndata.scverse.org/generated/anndata.AnnData.X.html#anndata.AnnData.X), and silent accept for [`layers`](https://anndata.scverse.org/generated/anndata.AnnData.layers.html#anndata.AnnData.layers)) in memory, writing one raises a clear error, and reading a non-conforming file warns instead of silently loading it. [@eroell](https://github.com/sponsors/eroell)

#### Performance[#](https://anndata.scverse.org/release-notes/index.html#id18)

- Use [zarrs-python](https://zarrs-python.readthedocs.io/en/stable/index.html) by default for [`anndata.io.read_zarr()`](https://anndata.scverse.org/generated/anndata.io.read_zarr.html#anndata.io.read_zarr) and [`anndata.io.write_zarr()`](https://anndata.scverse.org/generated/anndata.io.write_zarr.html#anndata.io.write_zarr) if it is installed [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2326](https://github.com/scverse/anndata/pull/2326))

## Version 0.12[#](https://anndata.scverse.org/release-notes/index.html#v0-12)

### 0.12.19 2026-07-02[#](https://anndata.scverse.org/release-notes/index.html#v0-12-19)

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id20)

- Fix a version-check path for [`scverse_misc`](https://scverse-misc.readthedocs.io/stable/api.html#module-scverse_misc)[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2525](https://github.com/scverse/anndata/pull/2525))

### 0.12.18 2026-06-22[#](https://anndata.scverse.org/release-notes/index.html#v0-12-18)

No significant changes.

### 0.12.17 2026-06-16[#](https://anndata.scverse.org/release-notes/index.html#v0-12-17)

#### Miscellaneous changes[#](https://anndata.scverse.org/release-notes/index.html#id23)

- Deprecate [`AnnLoader`](https://anndata.scverse.org/generated/anndata.experimental.AnnLoader.html#anndata.experimental.AnnLoader) in favor of [`annbatch.Loader`](https://annbatch.readthedocs.io/en/stable/generated/annbatch.Loader.html#annbatch.Loader)[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2493](https://github.com/scverse/anndata/pull/2493))

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id24)

- Fix [`read_lazy()`](https://anndata.scverse.org/generated/anndata.experimental.read_lazy.html#anndata.experimental.read_lazy) crashing on empty (zero-length) arrays, e.g. an empty array stored in [`uns`](https://anndata.scverse.org/generated/anndata.AnnData.uns.html#anndata.AnnData.uns)gaoflow ([#2473](https://github.com/scverse/anndata/pull/2473))

### 0.12.16 2026-05-18[#](https://anndata.scverse.org/release-notes/index.html#v0-12-16)

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id26)

- `0.12.15` was released against main accidentally and yanked. This version contains the correct branch.

### 0.12.15 2026-05-18[#](https://anndata.scverse.org/release-notes/index.html#v0-12-15)

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id28)

- Make sure `indices`, `data`, and `indptr` have `zarr.config.set({'array.target_shard_size_bytes'})` applied instead of having one override the other’s setting when writing sparse matrices. [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2445](https://github.com/scverse/anndata/pull/2445))
- Don’t warn when `shards` are explicitly set but [`anndata.settings.auto_shard_zarr_v3`](https://anndata.scverse.org/generated/anndata.settings.html#anndata.settings.auto_shard_zarr_v3) is not [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2451](https://github.com/scverse/anndata/pull/2451))

### 0.12.14 2026-05-12[#](https://anndata.scverse.org/release-notes/index.html#v0-12-14)

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id30)

- [`anndata.settings.auto_shard_zarr_v3`](https://anndata.scverse.org/generated/anndata.settings.html#anndata.settings.auto_shard_zarr_v3) and `anndata.settings.zarr_write_format` are no longer dependent on each other, as stated in the docs [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2427](https://github.com/scverse/anndata/pull/2427))

#### Performance[#](https://anndata.scverse.org/release-notes/index.html#id31)

- [`anndata.settings.auto_shard_zarr_v3`](https://anndata.scverse.org/generated/anndata.settings.html#anndata.settings.auto_shard_zarr_v3) now utilizes zarr’s support for a target shard via `zarr.config.set({"array.target_shard_size_bytes"...})` (only `zarr` version `&gt;=3.1.4`) to make the target shard size 1GB (uncompressed) if not otherwise set. [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2427](https://github.com/scverse/anndata/pull/2427))

### 0.12.13 2026-05-07[#](https://anndata.scverse.org/release-notes/index.html#v0-12-13)

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id33)

- Ensure non-string indexes are preserved in [`uns`](https://anndata.scverse.org/generated/anndata.AnnData.uns.html#anndata.AnnData.uns) using [`read_lazy()`](https://anndata.scverse.org/generated/anndata.experimental.read_lazy.html#anndata.experimental.read_lazy) (introduced by [#2419](https://github.com/scverse/anndata/pull/2419)) [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2423](https://github.com/scverse/anndata/pull/2423))

### 0.12.12 2026-05-06[#](https://anndata.scverse.org/release-notes/index.html#v0-12-12)

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id35)

- Disallow [`anndata.AnnData.transpose()`](https://anndata.scverse.org/generated/anndata.AnnData.transpose.html#anndata.AnnData.transpose) when `X` or `layers` contains [`h5py.Dataset`](https://docs.h5py.org/en/latest/high/dataset.html#h5py.Dataset), [`zarr.Array`](https://zarr.readthedocs.io/en/stable/api/zarr/array/#zarr.Array) ,[`anndata.abc.CSRDataset`](https://anndata.scverse.org/generated/anndata.abc.CSRDataset.html#anndata.abc.CSRDataset), or [`anndata.abc.CSCDataset`](https://anndata.scverse.org/generated/anndata.abc.CSCDataset.html#anndata.abc.CSCDataset)[@ilan-gold](https://github.com/sponsors/ilan-gold). ([#2399](https://github.com/scverse/anndata/pull/2399))
- Don’t force [`anndata.settings.check_uniqueness`](https://anndata.scverse.org/generated/anndata.settings.html#anndata.settings.check_uniqueness) if user loads an annotation index with [`anndata.experimental.read_lazy()`](https://anndata.scverse.org/generated/anndata.experimental.read_lazy.html#anndata.experimental.read_lazy)[@ilan-gold](https://github.com/sponsors/ilan-gold). ([#2419](https://github.com/scverse/anndata/pull/2419))

### 0.12.11 2026-04-21[#](https://anndata.scverse.org/release-notes/index.html#v0-12-11)

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id37)

- `anndata.settings.zarr_write_format` and [`anndata.settings.auto_shard_zarr_v3`](https://anndata.scverse.org/generated/anndata.settings.html#anndata.settings.auto_shard_zarr_v3) can now be set together [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2342](https://github.com/scverse/anndata/pull/2342))
- Check `Raw` length on creation and fix associated `.adata` in `Raw` slicing [@P Angerer](https://github.com/sponsors/P Angerer) ([#2351](https://github.com/scverse/anndata/pull/2351))
- Ensure the index name along `obs` and `var` are not lost when reading/writing to disk after reading with [`read_lazy()`](https://anndata.scverse.org/generated/anndata.experimental.read_lazy.html#anndata.experimental.read_lazy)[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2358](https://github.com/scverse/anndata/pull/2358))

#### Performance[#](https://anndata.scverse.org/release-notes/index.html#id38)

- Accelerate outer joins on dask-sparse matrices with unchunked minor axes in [`anndata.concat()`](https://anndata.scverse.org/generated/anndata.concat.html#anndata.concat)[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2395](https://github.com/scverse/anndata/pull/2395))

### 0.12.10 2026-02-06[#](https://anndata.scverse.org/release-notes/index.html#v0-12-10)

#### Miscellaneous changes[#](https://anndata.scverse.org/release-notes/index.html#id40)

- Add `anndata.settings.copy_on_write_X` to allow for forward-compatibility with future copy-on-write behavior (release `0.13`). Currently, setting [`X`](https://anndata.scverse.org/generated/anndata.AnnData.X.html#anndata.AnnData.X) on a view implicitly updates the object from which the view was created. [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2327](https://github.com/scverse/anndata/pull/2327))

### 0.12.9 2026-01-29[#](https://anndata.scverse.org/release-notes/index.html#v0-12-9)

#### Performance[#](https://anndata.scverse.org/release-notes/index.html#id42)

- Add a `write_csr_csc_indices_with_min_possible_dtype` option to [`anndata.settings`](https://anndata.scverse.org/generated/anndata.settings.html#anndata.settings) to enable downcasting of the `indices` of csr and csc matrices to a smaller dtype when writing. For example, if your csr matrix only has 30000 columns, then you can write out the `indices` of that matrix as `uint16` instead of `int64`. [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2159](https://github.com/scverse/anndata/pull/2159))

### 0.12.8 2026-01-27[#](https://anndata.scverse.org/release-notes/index.html#v0-12-8)

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id44)

- Actually copy single zarr store input in [`anndata.experimental.concat_on_disk()`](https://anndata.scverse.org/generated/anndata.experimental.concat_on_disk.html#anndata.experimental.concat_on_disk)[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2267](https://github.com/scverse/anndata/pull/2267))
- Fix `compressor` kwarg handling when writing to zarr v3 [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2270](https://github.com/scverse/anndata/pull/2270))
- Only open HDF5 file once in backed mode [@flying-sheep](https://github.com/sponsors/flying-sheep) ([#2274](https://github.com/scverse/anndata/pull/2274))
- Fix [`numpy.uint`](https://numpy.org/doc/stable/reference/arrays.scalars.html#numpy.uint) support in [`anndata.experimental.read_lazy()`](https://anndata.scverse.org/generated/anndata.experimental.read_lazy.html#anndata.experimental.read_lazy) and [`anndata.experimental.read_elem_lazy()`](https://anndata.scverse.org/generated/anndata.experimental.read_elem_lazy.html#anndata.experimental.read_elem_lazy)[@flying-sheep](https://github.com/sponsors/flying-sheep) ([#2287](https://github.com/scverse/anndata/pull/2287))
- `var` index is loaded into memory if it wasn’t previously for [`anndata.concat()`](https://anndata.scverse.org/generated/anndata.concat.html#anndata.concat) when [`anndata.AnnData.var`](https://anndata.scverse.org/generated/anndata.AnnData.var.html#anndata.AnnData.var) is a [`anndata.experimental.backed.Dataset2D`](https://anndata.scverse.org/generated/anndata.experimental.backed.Dataset2D.html#anndata.experimental.backed.Dataset2D)[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2299](https://github.com/scverse/anndata/pull/2299))

### 0.12.7 2025-12-16[#](https://anndata.scverse.org/release-notes/index.html#v0-12-7)

#### Documentation[#](https://anndata.scverse.org/release-notes/index.html#id46)

- Add cruft config & update contributors zethson ([#2223](https://github.com/scverse/anndata/pull/2223))

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id47)

- Handle `pandas.options.future.infer_string`P AngererI Gold ([#2133](https://github.com/scverse/anndata/pull/2133))
- Ensure there are no dangling file handles in [`anndata.experimental.concat_on_disk()`](https://anndata.scverse.org/generated/anndata.experimental.concat_on_disk.html#anndata.experimental.concat_on_disk) when passing `.h5ad`-terminated strings as path inputs. [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2218](https://github.com/scverse/anndata/pull/2218))
- Ensure [`anndata.experimental.backed.Dataset2D.true_index`](https://anndata.scverse.org/generated/anndata.experimental.backed.Dataset2D.html#anndata.experimental.backed.Dataset2D.true_index) is carried over when subsetting by columns i.,e `ds[[col1,col2]]`[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2220](https://github.com/scverse/anndata/pull/2220))

### 0.12.6 2025-11-06[#](https://anndata.scverse.org/release-notes/index.html#v0-12-6)

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id49)

- Attach [`h5py.File`](https://docs.h5py.org/en/latest/high/file.html#h5py.File) object to [`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData) at the `file` attribute returned from [`read_lazy()`](https://anndata.scverse.org/generated/anndata.experimental.read_lazy.html#anndata.experimental.read_lazy)[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2204](https://github.com/scverse/anndata/pull/2204))

### 0.12.5 2025-11-03[#](https://anndata.scverse.org/release-notes/index.html#v0-12-5)

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id51)

- Remove use of private `read_dataset` internally inside [`anndata.experimental.read_elem_lazy()`](https://anndata.scverse.org/generated/anndata.experimental.read_elem_lazy.html#anndata.experimental.read_elem_lazy)[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2158](https://github.com/scverse/anndata/pull/2158))
- Unblock version restriction on `dask` distributed writing by using threading scheduler always (see [#2172](https://github.com/scverse/anndata/pull/2172)) [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2183](https://github.com/scverse/anndata/pull/2183))

#### Performance[#](https://anndata.scverse.org/release-notes/index.html#id52)

- Use `name` on [`dask.array.map_blocks()`](https://docs.dask.org/en/stable/generated/dask.array.map_blocks.html#dask.array.map_blocks) internally when concatenating [`anndata.experimental.backed.Dataset2D`](https://anndata.scverse.org/generated/anndata.experimental.backed.Dataset2D.html#anndata.experimental.backed.Dataset2D) objects whose categoricals/nullable types must be converted to dask arrays [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2121](https://github.com/scverse/anndata/pull/2121))
- Enable automatic sharding in zarr v3 via [`anndata.settings.auto_shard_zarr_v3`](https://anndata.scverse.org/generated/anndata.settings.html#anndata.settings.auto_shard_zarr_v3) (via [`zarr`](https://zarr.readthedocs.io/en/stable/api/zarr/#zarr)’s own auto sharding mechanism i.e., `shards="auto"`) for all types except [`numpy.recarray`](https://numpy.org/doc/stable/reference/generated/numpy.recarray.html#numpy.recarray)[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2167](https://github.com/scverse/anndata/pull/2167))

### 0.12.4 2025-10-27[#](https://anndata.scverse.org/release-notes/index.html#v0-12-4)

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id54)

- [`dask.array.store()`](https://docs.dask.org/en/stable/generated/dask.array.store.html#dask.array.store) was producing corrupted data with zarr v3 + distributed scheduler + a lock (which we used internally): see [dask/dask#12109](https://github.com/dask/dask/issues/12109). Thus dense arrays were potentially being stored with corrupted data. The solution is to remove the lock for newer versions of dask but without the lock in older versions, it is impossible to store the data. Thus versions of dask older than `2025.4.0` will not be supported for writing dense data. [@ilan-gold](https://github.com/sponsors/ilan-gold)

### 0.12.3 2025-10-16[#](https://anndata.scverse.org/release-notes/index.html#v0-12-3)

#### Miscellaneous changes[#](https://anndata.scverse.org/release-notes/index.html#id56)

- Deprecate `AnnData.*_keys()` methods. [@flying-sheep](https://github.com/sponsors/flying-sheep) ([#2102](https://github.com/scverse/anndata/pull/2102))

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id57)

- Deprecate `__version__` and use standard [`version()`](https://docs.python.org/3/library/importlib.metadata.html#importlib.metadata.version) API [@flying-sheep](https://github.com/sponsors/flying-sheep) ([#1318](https://github.com/scverse/anndata/pull/1318))
- Allow writing of views of [`dask.array.Array`](https://docs.dask.org/en/stable/generated/dask.array.Array.html#dask.array.Array)[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2084](https://github.com/scverse/anndata/pull/2084))
- Enable writing of views of [`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData) in backed mode [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2092](https://github.com/scverse/anndata/pull/2092))
- Reallow writing of keys in `h5ad` files with forward slashes instead of erroring. Now a warning will be raised that the behavior will be disallowed in the future. To enable the new behavior, use [`anndata.settings.disallow_forward_slash_in_h5ad`](https://anndata.scverse.org/generated/anndata.settings.html#anndata.settings.disallow_forward_slash_in_h5ad). [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2097](https://github.com/scverse/anndata/pull/2097))
- Respect off-axis merge options in [`anndata.experimental.concat_on_disk()`](https://anndata.scverse.org/generated/anndata.experimental.concat_on_disk.html#anndata.experimental.concat_on_disk)[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2122](https://github.com/scverse/anndata/pull/2122))

### 0.12.2 2025-08-11[#](https://anndata.scverse.org/release-notes/index.html#v0-12-2)

### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id59)

- Revert accidental change where [`X`](https://anndata.scverse.org/generated/anndata.AnnData.X.html#anndata.AnnData.X) got written to disk when it was `None`[@flying-sheep](https://github.com/sponsors/flying-sheep) ([#2054](https://github.com/scverse/anndata/pull/2054))

### 0.12.1 2025-07-23[#](https://anndata.scverse.org/release-notes/index.html#v0-12-1)

### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id61)

- Fix `chunks` argument for [`anndata.experimental.read_elem_lazy()`](https://anndata.scverse.org/generated/anndata.experimental.read_elem_lazy.html#anndata.experimental.read_elem_lazy) so that it uses the on-disk chunking when possible, and allow users to pass this argument through to the reading of [`anndata.experimental.backed.Dataset2D`](https://anndata.scverse.org/generated/anndata.experimental.backed.Dataset2D.html#anndata.experimental.backed.Dataset2D)[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2033](https://github.com/scverse/anndata/pull/2033))

### Performance[#](https://anndata.scverse.org/release-notes/index.html#id62)

- Improve integer indexing performance of `h5` 1d arrays that are opened via [`anndata.experimental.read_elem_lazy()`](https://anndata.scverse.org/generated/anndata.experimental.read_elem_lazy.html#anndata.experimental.read_elem_lazy)[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2035](https://github.com/scverse/anndata/pull/2035))

### 0.12.0 2025-07-16[#](https://anndata.scverse.org/release-notes/index.html#v0-12-0)

- rc4 2025-06-18
- rc3 2025-05-20
- rc2 2025-05-15
- rc1 2025-04-09

#### Breaking changes[#](https://anndata.scverse.org/release-notes/index.html#id64)

- rc1 Remove `anndata.read`[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1766](https://github.com/scverse/anndata/pull/1766))
- rc1 Tighten usage of [`scipy.sparse.spmatrix`](https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.spmatrix.html#scipy.sparse.spmatrix) for describing sparse matrices in types and instance checks to only [`scipy.sparse.csr_matrix`](https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.csr_matrix.html#scipy.sparse.csr_matrix) and [`scipy.sparse.csc_matrix`](https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.csc_matrix.html#scipy.sparse.csc_matrix)[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1768](https://github.com/scverse/anndata/pull/1768))
- rc1 Disallow declaration of [`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData) with non-`cs{r,c}` sparse data-structures [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1829](https://github.com/scverse/anndata/pull/1829))
- rc1 Upgrade all `DeprecationWarning`s to `FutureWarning`s [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1874](https://github.com/scverse/anndata/pull/1874))
- rc4 Lower bound `xarray` by `2025.06.01`. [`pandas.arrays.StringArray`](https://pandas.pydata.org/docs/reference/api/pandas.arrays.StringArray.html#pandas.arrays.StringArray) was previously used as the in-memory `nullable-string-array` container in `xarray`, but due to [pydata/xarray#10419](https://github.com/pydata/xarray/issues/10419) now uses [`numpy.ndarray`](https://numpy.org/doc/stable/reference/generated/numpy.ndarray.html#numpy.ndarray) with an object data type. [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2008](https://github.com/scverse/anndata/pull/2008))

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id65)

- Fix [`anndata.experimental.backed.Dataset2D.reindex()`](https://anndata.scverse.org/generated/anndata.experimental.backed.Dataset2D.html#anndata.experimental.backed.Dataset2D.reindex) internal setting [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2018](https://github.com/scverse/anndata/pull/2018))
- rc1 Disallow writing of [`Dataset2D`](https://anndata.scverse.org/generated/anndata.experimental.backed.Dataset2D.html#anndata.experimental.backed.Dataset2D) objects [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1887](https://github.com/scverse/anndata/pull/1887))
- rc1 Upgrade old deprecation warning to a `FutureWarning` on `BaseCompressedSparseDataset.__setitem__`, showing our intent to remove the feature in the next release. [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1928](https://github.com/scverse/anndata/pull/1928))
- rc1 Don’t use [`asyncio.run()`](https://docs.python.org/3/library/asyncio-runner.html#asyncio.run) internally for any operations [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1933](https://github.com/scverse/anndata/pull/1933))
- rc1 Disallow forward slashes in keys for writing [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1940](https://github.com/scverse/anndata/pull/1940))
- rc2 Convert 1d [`numpy.ndarray`](https://numpy.org/doc/stable/reference/generated/numpy.ndarray.html#numpy.ndarray) and [`cupy.ndarray`](https://docs.cupy.dev/en/stable/reference/generated/cupy.ndarray.html#cupy.ndarray)s in [`anndata.AnnData.obsm`](https://anndata.scverse.org/generated/anndata.AnnData.obsm.html#anndata.AnnData.obsm) and [`anndata.AnnData.varm`](https://anndata.scverse.org/generated/anndata.AnnData.varm.html#anndata.AnnData.varm) to 2d [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1962](https://github.com/scverse/anndata/pull/1962))
- rc3 Update zarr v3 bound to &gt;3.0.8 to prevent corrupted data [zarr-developers/zarr-python#3061](https://github.com/zarr-developers/zarr-python/issues/3061)[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1993](https://github.com/scverse/anndata/pull/1993))

#### Features[#](https://anndata.scverse.org/release-notes/index.html#id66)

- rc1[`None`](https://docs.python.org/3/builtins/constants.html#None) values can now be serialized to `.h5ad` and `.zarr`, preserving e.g. [`uns`](https://anndata.scverse.org/generated/anndata.AnnData.uns.html#anndata.AnnData.uns) structure through saving and loading [@flying-sheep](https://github.com/sponsors/flying-sheep) ([#999](https://github.com/scverse/anndata/pull/999))
- rc1 Add [`read_elem_lazy()`](https://anndata.scverse.org/generated/anndata.experimental.read_elem_lazy.html#anndata.experimental.read_elem_lazy) (in place of `read_elem_as_dask`) to handle backed dataframes, sparse arrays, and dense arrays, as well as a [`read_lazy()`](https://anndata.scverse.org/generated/anndata.experimental.read_lazy.html#anndata.experimental.read_lazy) to handle reading in as much of the on-disk data as possible to produce a [`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData) object [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1247](https://github.com/scverse/anndata/pull/1247))
- rc1 Support [`zarr`](https://zarr.readthedocs.io/en/stable/api/zarr/#zarr) version 3 python package [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1726](https://github.com/scverse/anndata/pull/1726))
- rc1 Adopt the Scientific Python [deprecation schedule](https://scientific-python.org/specs/spec-0000/)[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1768](https://github.com/scverse/anndata/pull/1768))
- rc1 Allow [`zarr`](https://zarr.readthedocs.io/en/stable/api/zarr/#zarr) v3 writing of data [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1892](https://github.com/scverse/anndata/pull/1892))
- rc1[`anndata.register_anndata_namespace()`](https://anndata.scverse.org/generated/anndata.register_anndata_namespace.html#anndata.register_anndata_namespace) functionality for adding custom functionality to an [`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData) object [@srivarra](https://github.com/sponsors/srivarra) ([#1870](https://github.com/scverse/anndata/pull/1870))
- rc2 Allow xarray Datasets to be used for obs/var/obsm/varm. [@ilia-kats](https://github.com/sponsors/ilia-kats) ([#1966](https://github.com/scverse/anndata/pull/1966))
- rc4[`anndata.experimental.backed.Dataset2D`](https://anndata.scverse.org/generated/anndata.experimental.backed.Dataset2D.html#anndata.experimental.backed.Dataset2D) now takes a compositional approach to wrapping [`xarray.Dataset`](https://docs.xarray.dev/en/stable/generated/xarray.Dataset.html#xarray.Dataset) which may have breaking changes over the past release versions. [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1997](https://github.com/scverse/anndata/pull/1997))
- rc4 Use [`numpy.dtypes.StringDType`](https://numpy.org/doc/stable/reference/routines.dtypes.html#numpy.dtypes.StringDType) with `na_object` set to [`pandas.NA`](https://pandas.pydata.org/docs/reference/api/pandas.NA.html#pandas.NA) for nullable string data with [`anndata.experimental.backed.Dataset2D`](https://anndata.scverse.org/generated/anndata.experimental.backed.Dataset2D.html#anndata.experimental.backed.Dataset2D)[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2011](https://github.com/scverse/anndata/pull/2011))

#### Performance[#](https://anndata.scverse.org/release-notes/index.html#id67)

- rc2 Load AnnLoader lazily to prevent expensive unnecessary `torch` imports when its available on the system. [@Zethson](https://github.com/sponsors/Zethson) & [@flying-sheep](https://github.com/sponsors/flying-sheep) ([#1950](https://github.com/scverse/anndata/pull/1950))
- rc4 Improve [`read_elem_lazy()`](https://anndata.scverse.org/generated/anndata.experimental.read_elem_lazy.html#anndata.experimental.read_elem_lazy) performance for `h5ad` files by not caching `indptr`. [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2005](https://github.com/scverse/anndata/pull/2005))

#### Development[#](https://anndata.scverse.org/release-notes/index.html#development)

- rc4 Temporarily bound [`zarr`](https://zarr.readthedocs.io/en/stable/api/zarr/#zarr) to `&lt;3.1` until [#1995](https://github.com/scverse/anndata/pull/1995) is merged to handle the new data type structure. [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#2013](https://github.com/scverse/anndata/pull/2013))

## Version 0.11[#](https://anndata.scverse.org/release-notes/index.html#v0-11)

### 0.11.4 2025-03-26[#](https://anndata.scverse.org/release-notes/index.html#v0-11-4)

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id69)

- Raise [`ImplicitModificationWarning`](https://anndata.scverse.org/generated/anndata.ImplicitModificationWarning.html#anndata.ImplicitModificationWarning) when setting `X` on a view. [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1853](https://github.com/scverse/anndata/pull/1853))
- Bound `dask` due to [dask/dask#11752](https://github.com/dask/dask/issues/11752)[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1859](https://github.com/scverse/anndata/pull/1859))
- Fix concatenation of [`anndata.AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData) objects along `var` using `join="outer"` when `varm` is not empty. [@ilia-kats](https://github.com/sponsors/ilia-kats) ([#1911](https://github.com/scverse/anndata/pull/1911))
- Add `convert_strings_to_categoricals` parameter also to [`write_h5ad()`](https://anndata.scverse.org/generated/anndata.AnnData.write_h5ad.html#anndata.AnnData.write_h5ad) and [`write_zarr()`](https://anndata.scverse.org/generated/anndata.AnnData.write_zarr.html#anndata.AnnData.write_zarr) as intended [@flying-sheep](https://github.com/sponsors/flying-sheep) ([#1914](https://github.com/scverse/anndata/pull/1914))
- Allow initialization of [`anndata.AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData) objects without `X` (since they could be constructed previously by deleting `X`) [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1941](https://github.com/scverse/anndata/pull/1941))

#### Development Process[#](https://anndata.scverse.org/release-notes/index.html#development-process)

- Fix version number inference in development environments (CI and local) [@flying-sheep](https://github.com/sponsors/flying-sheep) ([#1831](https://github.com/scverse/anndata/pull/1831))

### 0.11.3 2025-01-10[#](https://anndata.scverse.org/release-notes/index.html#v0-11-3)

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id71)

- Upper bound `zarr` at runtime [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1819](https://github.com/scverse/anndata/pull/1819))

### 0.11.2 2025-01-07[#](https://anndata.scverse.org/release-notes/index.html#v0-11-2)

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id73)

- Cache accesses to the `data` and `indices` arrays in [`CSRDataset`](https://anndata.scverse.org/generated/anndata.abc.CSRDataset.html#anndata.abc.CSRDataset) and [`CSCDataset`](https://anndata.scverse.org/generated/anndata.abc.CSCDataset.html#anndata.abc.CSCDataset)[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1744](https://github.com/scverse/anndata/pull/1744))
- Error out on floating point indices that are not actually integers [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1746](https://github.com/scverse/anndata/pull/1746))
- `write_elem` now filters out incompatible `dataset_kwargs` when saving zero-dimensional arrays [@ilia-kats](https://github.com/sponsors/ilia-kats) ([#1783](https://github.com/scverse/anndata/pull/1783))
- Add [`scipy`](https://docs.scipy.org/doc/scipy/index.html#module-scipy) 1.5 compatibility [@flying-sheep](https://github.com/sponsors/flying-sheep) ([#1806](https://github.com/scverse/anndata/pull/1806))

### Performance[#](https://anndata.scverse.org/release-notes/index.html#id74)

- Batch slice-based indexing in [`anndata.abc.CSRDataset`](https://anndata.scverse.org/generated/anndata.abc.CSRDataset.html#anndata.abc.CSRDataset) and [`anndata.abc.CSCDataset`](https://anndata.scverse.org/generated/anndata.abc.CSCDataset.html#anndata.abc.CSCDataset) for performance boost in `zarr`[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1790](https://github.com/scverse/anndata/pull/1790))

### 0.11.1 2024-11-12[#](https://anndata.scverse.org/release-notes/index.html#v0-11-1)

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id76)

- Remove upper pin on `dask` and exclude versions broken with sparse indexing [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1725](https://github.com/scverse/anndata/pull/1725))
- Fix chunking with -1 in `chunks` argument of `~anndata.experimental.read_elem_as_dask`[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1743](https://github.com/scverse/anndata/pull/1743))
- Fix `cupy&lt;0.13` imports in non-gpu environments [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1754](https://github.com/scverse/anndata/pull/1754))

### 0.11.0 2024-11-07[#](https://anndata.scverse.org/release-notes/index.html#v0-11-0)

Release candidates:

- rc3 2024-10-14
- rc2 2024-09-24
- rc1 2024-09-04

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id78)

- Ensure [`anndata.concat()`](https://anndata.scverse.org/generated/anndata.concat.html#anndata.concat) of [`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData) object with [`scipy.sparse.spmatrix`](https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.spmatrix.html#scipy.sparse.spmatrix) and [`scipy.sparse.sparray`](https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.sparray.html#scipy.sparse.sparray) dask arrays uses the correct fill value of 0. [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1719](https://github.com/scverse/anndata/pull/1719))
- Ensure that views of AwkwardArrays have their “view” attributes removed on saving an [`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData) object to disk. [@grst](https://github.com/sponsors/grst) ([#1736](https://github.com/scverse/anndata/pull/1736))

#### Breaking changes[#](https://anndata.scverse.org/release-notes/index.html#id79)

- rc3 Drop support for `python` 3.9 [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1712](https://github.com/scverse/anndata/pull/1712))
- rc2 A new `anndata.io` module contains all `read_*` and `write_*` functions, and all imports of such functions should go through this module. Old ways of importing these functions i.e., `fromanndataimportread_csv` or `fromanndata._io.specsimportread_elem` will still work, but are now considered deprecated and give a warning on import with the exception of [`anndata.io.read_zarr()`](https://anndata.scverse.org/generated/anndata.io.read_zarr.html#anndata.io.read_zarr) and [`anndata.io.read_h5ad()`](https://anndata.scverse.org/generated/anndata.io.read_h5ad.html#anndata.io.read_h5ad), which will remain at the top-level `anndata` without warning. [@ilan-gold ({pr}](https://github.com/sponsors/ilan-gold ({pr})1682`)
- rc1 Removed deprecated modules `anndata.core` and `anndata.readwrite`[@ivirshup](https://github.com/sponsors/ivirshup) ([#1197](https://github.com/scverse/anndata/pull/1197))
- rc1 No longer export `sparse_dataset` from `anndata.experimental`, instead exporting [`anndata.io.sparse_dataset()`](https://anndata.scverse.org/generated/anndata.io.sparse_dataset.html#anndata.io.sparse_dataset)[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1642](https://github.com/scverse/anndata/pull/1642))
- rc1 Move `RWAble` and `InMemoryElem` out of `experimental`, renaming `RWAble` to `AxisStorable`[[1]](https://anndata.scverse.org/release-notes/index.html#storable) and `InMemoryElem` to [`RWAble`](https://anndata.scverse.org/typing.html#anndata.typing.RWAble)[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1643](https://github.com/scverse/anndata/pull/1643))

#### Development Process[#](https://anndata.scverse.org/release-notes/index.html#id81)

- rc2 Add extra `dask` dependency for installation i.e., `pipinstallanndata[dask]`[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1677](https://github.com/scverse/anndata/pull/1677))
- rc2 Remove `shall_` from variable names in `settings`[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1685](https://github.com/scverse/anndata/pull/1685))
- rc1 Create new `cupy` installation options for cuda 11 & 12 called `cu11` and `cu12`[@Intron7](https://github.com/sponsors/Intron7) ([#1596](https://github.com/scverse/anndata/pull/1596))

#### Documentation[#](https://anndata.scverse.org/release-notes/index.html#id82)

- rc1 Correct [`anndata.AnnData.X`](https://anndata.scverse.org/generated/anndata.AnnData.X.html#anndata.AnnData.X) type to include [`CSRDataset`](https://anndata.scverse.org/generated/anndata.abc.CSRDataset.html#anndata.abc.CSRDataset) and [`CSCDataset`](https://anndata.scverse.org/generated/anndata.abc.CSCDataset.html#anndata.abc.CSCDataset) as possible types and being deprecation process for non-csr/csc [`scipy.sparse.spmatrix`](https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.spmatrix.html#scipy.sparse.spmatrix) types in [`anndata.AnnData.X`](https://anndata.scverse.org/generated/anndata.AnnData.X.html#anndata.AnnData.X)[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1616](https://github.com/scverse/anndata/pull/1616))

#### Features[#](https://anndata.scverse.org/release-notes/index.html#id83)

- Add support for ellipsis indexing of the [`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData) object [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1729](https://github.com/scverse/anndata/pull/1729))
- rc1`scipy.sparse.csr_array` and `scipy.sparse.csc_array` are now supported when constructing `AnnData` objects [@ilan-gold](https://github.com/sponsors/ilan-gold)[@isaac-virshup](https://github.com/sponsors/isaac-virshup) ([#1028](https://github.com/scverse/anndata/pull/1028))
- rc1 Allow `axis` parameter of e.g. [`anndata.concat()`](https://anndata.scverse.org/generated/anndata.concat.html#anndata.concat) to accept `'obs'` and `'var'`[@flying-sheep](https://github.com/sponsors/flying-sheep) ([#1244](https://github.com/scverse/anndata/pull/1244))
- rc1 Add `settings` object with methods for altering internally-used options, like checking for uniqueness on `obs`’ index [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1270](https://github.com/scverse/anndata/pull/1270))
- rc1 Add [`remove_unused_categories`](https://anndata.scverse.org/generated/anndata.settings.html#anndata.settings.remove_unused_categories) option to [`anndata.settings`](https://anndata.scverse.org/generated/anndata.settings.html#anndata.settings) to override current behavior [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1340](https://github.com/scverse/anndata/pull/1340))
- rc1 Add `~anndata.experimental.read_elem_as_dask` function to handle i/o with sparse and dense arrays [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1469](https://github.com/scverse/anndata/pull/1469))
- rc1 Add ability to convert strings to categoricals on write in [`write_h5ad()`](https://anndata.scverse.org/generated/anndata.AnnData.write_h5ad.html#anndata.AnnData.write_h5ad) and [`write_zarr()`](https://anndata.scverse.org/generated/anndata.AnnData.write_zarr.html#anndata.AnnData.write_zarr) via `convert_strings_to_categoricals` parameter [@falexwolf](https://github.com/sponsors/falexwolf) ([#1474](https://github.com/scverse/anndata/pull/1474))
- rc1 Add [`check_uniqueness`](https://anndata.scverse.org/generated/anndata.settings.html#anndata.settings.check_uniqueness) option to [`anndata.settings`](https://anndata.scverse.org/generated/anndata.settings.html#anndata.settings) to override current behavior [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1507](https://github.com/scverse/anndata/pull/1507))
- rc1 Add functionality to write from GPU [`dask.array.Array`](https://docs.dask.org/en/stable/generated/dask.array.Array.html#dask.array.Array) to disk [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1550](https://github.com/scverse/anndata/pull/1550))
- rc1 Read and write support for nullable string arrays ([`pandas.arrays.StringArray`](https://pandas.pydata.org/docs/reference/api/pandas.arrays.StringArray.html#pandas.arrays.StringArray)). Use pandas’ [Options and settings](https://pandas.pydata.org/docs/user_guide/options.html)`mode.string_storage` to control which storage mode is used when reading `dtype="string"` columns. [@flying-sheep](https://github.com/sponsors/flying-sheep) ([#1558](https://github.com/scverse/anndata/pull/1558))
- rc1 Export [`write_elem()`](https://anndata.scverse.org/generated/anndata.io.write_elem.html#anndata.io.write_elem) and [`read_elem()`](https://anndata.scverse.org/generated/anndata.io.read_elem.html#anndata.io.read_elem) directly from the main package instead of `experimental`[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1598](https://github.com/scverse/anndata/pull/1598))
- rc1 Allow reading sparse data (via [`read_elem()`](https://anndata.scverse.org/generated/anndata.io.read_elem.html#anndata.io.read_elem) or [`sparse_dataset()`](https://anndata.scverse.org/generated/anndata.io.sparse_dataset.html#anndata.io.sparse_dataset)) into either [`scipy.sparse.csr_array`](https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.csr_array.html#scipy.sparse.csr_array) or [`scipy.sparse.csc_array`](https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.csc_array.html#scipy.sparse.csc_array) via [`anndata.settings.use_sparse_array_on_read`](https://anndata.scverse.org/generated/anndata.settings.html#anndata.settings.use_sparse_array_on_read)[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1633](https://github.com/scverse/anndata/pull/1633))

## Version 0.10[#](https://anndata.scverse.org/release-notes/index.html#v0-10)

### 0.10.9 2024-08-28[#](https://anndata.scverse.org/release-notes/index.html#v0-10-9)

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id85)

- Fix writing large number of columns for `h5` files [@ilan-gold](https://github.com/sponsors/ilan-gold)[@selmanozleyen](https://github.com/sponsors/selmanozleyen) ([#1147](https://github.com/scverse/anndata/pull/1147))
- Add warning for setting `X` on a view with repeated indices [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1501](https://github.com/scverse/anndata/pull/1501))
- Coerce [`numpy.matrix`](https://numpy.org/doc/stable/reference/generated/numpy.matrix.html#numpy.matrix) classes to arrays when trying to store them in `AnnData`[@flying-sheep](https://github.com/sponsors/flying-sheep) ([#1516](https://github.com/scverse/anndata/pull/1516))
- Fix for setting a dense `X` view with a sparse matrix [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1532](https://github.com/scverse/anndata/pull/1532))
- Upper bound [`numpy`](https://numpy.org/doc/stable/reference/index.html#module-numpy) for `gpu` installation on account of [cupy/cupy#8391](https://github.com/cupy/cupy/issues/8391)[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1540](https://github.com/scverse/anndata/pull/1540))
- Upper bound dask on account of [#1579](https://github.com/scverse/anndata/issues/1579)[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1580](https://github.com/scverse/anndata/pull/1580))
- Ensure setting [`pandas.DataFrame.index`](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.index.html#pandas.DataFrame.index) on a view of a [`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData) instantiates the [`DataFrame`](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.html#pandas.DataFrame) from the view [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1586](https://github.com/scverse/anndata/pull/1586))
- Disallow using [`DataFrame`](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.html#pandas.DataFrame)s with multi-index columns [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1589](https://github.com/scverse/anndata/pull/1589))

#### Development Process[#](https://anndata.scverse.org/release-notes/index.html#id86)

- create new `cupy` installation options for cuda 11 & 12 called `cu11` and `cu12`[@Intron7](https://github.com/sponsors/Intron7) ([#1596](https://github.com/scverse/anndata/pull/1596))

#### Documentation[#](https://anndata.scverse.org/release-notes/index.html#id87)

- add `callback` typing for [`read_dispatched()`](https://anndata.scverse.org/generated/anndata.experimental.read_dispatched.html#anndata.experimental.read_dispatched) and [`write_dispatched()`](https://anndata.scverse.org/generated/anndata.experimental.write_dispatched.html#anndata.experimental.write_dispatched)[@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1557](https://github.com/scverse/anndata/pull/1557))

#### Performance[#](https://anndata.scverse.org/release-notes/index.html#id88)

- Support for `concat_on_disk` outer join [@ilan-gold](https://github.com/sponsors/ilan-gold) ([#1504](https://github.com/scverse/anndata/pull/1504))

### 0.10.8 2024-06-20[#](https://anndata.scverse.org/release-notes/index.html#v0-10-8)

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id90)

- Write out `64bit` indptr when appropriate for [`concat_on_disk()`](https://anndata.scverse.org/generated/anndata.experimental.concat_on_disk.html#anndata.experimental.concat_on_disk)[#1493](https://github.com/scverse/anndata/pull/1493)[@ilan-gold](https://github.com/sponsors/ilan-gold)
- Support for Numpy 2 [#1499](https://github.com/scverse/anndata/pull/1499)[@flying-sheep](https://github.com/sponsors/flying-sheep)
- Fix [`sparse_dataset()`](https://anndata.scverse.org/generated/anndata.io.sparse_dataset.html#anndata.io.sparse_dataset) docstring test on account of new [`scipy`](https://docs.scipy.org/doc/scipy/index.html#module-scipy) version [#1514](https://github.com/scverse/anndata/pull/1514)[@ilan-gold](https://github.com/sponsors/ilan-gold)

#### Documentation[#](https://anndata.scverse.org/release-notes/index.html#id91)

- Improved example for [`sparse_dataset()`](https://anndata.scverse.org/generated/anndata.io.sparse_dataset.html#anndata.io.sparse_dataset)[#1468](https://github.com/scverse/anndata/pull/1468)[@ivirshup](https://github.com/sponsors/ivirshup)

### 0.10.7 2024-04-09[#](https://anndata.scverse.org/release-notes/index.html#v0-10-7)

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id93)

- Handle upstream `numcodecs` bug where read-only string arrays cannot be encoded [@ivirshup](https://github.com/sponsors/ivirshup)[#1421](https://github.com/scverse/anndata/pull/1421)
- Use in-memory sparse matrix directly to fix compatibility with `scipy``1.13`[@ilan-gold](https://github.com/sponsors/ilan-gold)[#1435](https://github.com/scverse/anndata/pull/1435)

#### Performance[#](https://anndata.scverse.org/release-notes/index.html#id94)

- Remove `vindex` for subsetting `dask.array.Array` because of its slowness and memory consumption [@ilan-gold](https://github.com/sponsors/ilan-gold)[#1432](https://github.com/scverse/anndata/pull/1432)

### 0.10.6 2024-03-11[#](https://anndata.scverse.org/release-notes/index.html#v0-10-6)

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id96)

- Defer import of zarr in test helpers, as scanpy CI job relies on them [#1343](https://github.com/scverse/anndata/pull/1343)[@ilan-gold](https://github.com/sponsors/ilan-gold)
- Writing a dataframe with non-unique column names now throws an error, instead of silently overwriting [#1335](https://github.com/scverse/anndata/pull/1335)[@ivirshup](https://github.com/sponsors/ivirshup)
- Bring optimization from [#1233](https://github.com/scverse/anndata/pull/1233) to indexing on the whole `AnnData` object, not just the sparse dataset itself [#1365](https://github.com/scverse/anndata/pull/1365)[@ilan-gold](https://github.com/sponsors/ilan-gold)
- Fix mean slice length checking to use improved performance when indexing backed sparse matrices with boolean masks along their major axis [#1366](https://github.com/scverse/anndata/pull/1366)[@ilan-gold](https://github.com/sponsors/ilan-gold)
- Fixed overflow occurring when writing dask arrays with sparse chunks by always writing dask arrays with 64 bit indptr and indices, and adding an overflow check to `.append` method of sparse on disk structures [#1348](https://github.com/scverse/anndata/pull/1348)[@ivirshup](https://github.com/sponsors/ivirshup)
- Modified `ValueError` message for invalid `.X` during construction to show more helpful list instead of ambiguous `__name__`[#1395](https://github.com/scverse/anndata/pull/1395)[@eroell](https://github.com/sponsors/eroell)
- Pin `array-api-compat!=1.5` to avoid incorrect implementation of `asarray`[#1411](https://github.com/scverse/anndata/pull/1411)[@ivirshup](https://github.com/sponsors/ivirshup)

#### Documentation[#](https://anndata.scverse.org/release-notes/index.html#id97)

- Type hints and docstrings for `.to_df` method are updated and fixed [#1402](https://github.com/scverse/anndata/pull/1402)[@WeilerP](https://github.com/sponsors/WeilerP)

#### Development[#](https://anndata.scverse.org/release-notes/index.html#id98)

- `anndata`’s CI now tests against minimum versions of it’s dependencies. As a result, several dependencies had their minimum required version bumped. See diff for details [#1314](https://github.com/scverse/anndata/pull/1314)[@ivirshup](https://github.com/sponsors/ivirshup)
- `anndata` now tests against Python 3.12 [#1373](https://github.com/scverse/anndata/pull/1373)[@ivirshup](https://github.com/sponsors/ivirshup)

### 0.10.5 2024-01-25[#](https://anndata.scverse.org/release-notes/index.html#v0-10-5)

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id100)

- Fix outer concatenation along variables when only a subset of objects had an entry in layers [#1291](https://github.com/scverse/anndata/pull/1291)[@ivirshup](https://github.com/sponsors/ivirshup)
- Fix comparison of &gt;2d arrays in `uns` during concatenation [#1300](https://github.com/scverse/anndata/pull/1300)[@ivirshup](https://github.com/sponsors/ivirshup)
- Fix IO with awkward array version 2.5.2 [#1328](https://github.com/scverse/anndata/pull/1328)[@ivirshup](https://github.com/sponsors/ivirshup)
- Fix bug (introduced in 0.10.4) where indexing an AnnData with `list[bool]` would return the wrong result [#1332](https://github.com/scverse/anndata/pull/1332)[@ivirshup](https://github.com/sponsors/ivirshup)

#### Documentation[#](https://anndata.scverse.org/release-notes/index.html#id101)

- Re-add search-as-you-type, this time via `readthedocs-sphinx-search`[#1311](https://github.com/scverse/anndata/pull/1311)[@flying-sheep](https://github.com/sponsors/flying-sheep)

#### Performance[#](https://anndata.scverse.org/release-notes/index.html#id102)

- `BaseCompressedSparseDataset`’s `indptr` is cached [#1266](https://github.com/scverse/anndata/pull/1266)[@ilan-gold](https://github.com/sponsors/ilan-gold)
- Improved performance when indexing backed sparse matrices with boolean masks along their major axis [#1233](https://github.com/scverse/anndata/pull/1233)[@ilan-gold](https://github.com/sponsors/ilan-gold)

### 0.10.4 2024-01-04[#](https://anndata.scverse.org/release-notes/index.html#v0-10-4)

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id104)

- Only try to use `Categorical.map(na_action=…)` in actually supported Pandas ≥2.1 [#1226](https://github.com/scverse/anndata/pull/1226)[@flying-sheep](https://github.com/sponsors/flying-sheep)
- `AnnData.__sizeof__()` support for backed datasets [#1230](https://github.com/scverse/anndata/pull/1230)[@Neah-Ko](https://github.com/sponsors/Neah-Ko)
- `adata[:,[]]` now returns an `AnnData` object empty on the appropriate dimensions instead of erroring [#1243](https://github.com/scverse/anndata/pull/1243)[@ilan-gold](https://github.com/sponsors/ilan-gold)
- `adata.X[mask]` works in newer `numpy` versions when `X` is `backed`[#1255](https://github.com/scverse/anndata/pull/1255)[@ilan-gold](https://github.com/sponsors/ilan-gold)
- `adata.X[...]` fixed for `X` as a `BaseCompressedSparseDataset` with `zarr` backend [#1265](https://github.com/scverse/anndata/pull/1265)[@ilan-gold](https://github.com/sponsors/ilan-gold)
- Improve read/write error reporting [#1273](https://github.com/scverse/anndata/pull/1273)[@flying-sheep](https://github.com/sponsors/flying-sheep)

#### Documentation[#](https://anndata.scverse.org/release-notes/index.html#id105)

- Improve aligned mapping error messages [#1252](https://github.com/scverse/anndata/pull/1252)[@flying-sheep](https://github.com/sponsors/flying-sheep)

### 0.10.3 2023-10-31[#](https://anndata.scverse.org/release-notes/index.html#v0-10-3)

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id107)

- Prevent pandas from causing infinite recursion when setting a slice of a categorical column [#1211](https://github.com/scverse/anndata/pull/1211)[@flying-sheep](https://github.com/sponsors/flying-sheep)

#### Documentation[#](https://anndata.scverse.org/release-notes/index.html#id108)

- Stop showing “Support for Awkward Arrays is currently experimental” warnings when reading, concatenating, slicing, or transposing AnnData objects [#1182](https://github.com/scverse/anndata/pull/1182)[@flying-sheep](https://github.com/sponsors/flying-sheep)

#### Other updates[#](https://anndata.scverse.org/release-notes/index.html#other-updates)

- Fail canary CI job when tests raise unexpected warnings. [#1182](https://github.com/scverse/anndata/pull/1182)[@flying-sheep](https://github.com/sponsors/flying-sheep)

### 0.10.2 2023-10-11[#](https://anndata.scverse.org/release-notes/index.html#v0-10-2)

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id110)

- Added compatibility layer for packages relying on `anndata._core.sparse_dataset.SparseDataset`. Note that this API is *deprecated* and new code should use `anndata.CSRDataset`, `~anndata.CSCDataset`, and `anndata.sparse_dataset` instead. [#1185](https://github.com/scverse/anndata/pull/1185)[@ivirshup](https://github.com/sponsors/ivirshup)
- Handle deprecation warning from `pd.Categorical.map` thrown during `anndata.concat`[#1189](https://github.com/scverse/anndata/pull/1189)[@flying-sheep](https://github.com/sponsors/flying-sheep)[@ivirshup](https://github.com/sponsors/ivirshup)
- Fixed extra steps being included in IO tracebacks [#1193](https://github.com/scverse/anndata/pull/1193)[@flying-sheep](https://github.com/sponsors/flying-sheep)
- `as_dense` argument of `write_h5ad` no longer writes an array without encoding metadata [#1193](https://github.com/scverse/anndata/pull/1193)[@flying-sheep](https://github.com/sponsors/flying-sheep)

#### Performance[#](https://anndata.scverse.org/release-notes/index.html#id111)

- Improved performance of `concat_on_disk` with dense arrays in some cases [#1169](https://github.com/scverse/anndata/pull/1169)[@selmanozleyen](https://github.com/sponsors/selmanozleyen)

### 0.10.1 2023-10-08[#](https://anndata.scverse.org/release-notes/index.html#v0-10-1)

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id113)

- Fix `ad.concat` erroring when concatenating a categorical and object column [#1171](https://github.com/scverse/anndata/pull/1171)[@ivirshup](https://github.com/sponsors/ivirshup)

### 0.10.0 2023-10-06[#](https://anndata.scverse.org/release-notes/index.html#v0-10-0)

#### Features[#](https://anndata.scverse.org/release-notes/index.html#id115)

**GPU Support**

- Dense and sparse [`CuPy`](https://docs.cupy.dev/) arrays are now supported [#1066](https://github.com/scverse/anndata/pull/1066)[@ivirshup](https://github.com/sponsors/ivirshup)

- Once you have `CuPy` arrays in your anndata, use it with: [`rapids-singlecell`](https://rapids-singlecell.readthedocs.io/en/latest/index.html) from v0.9+
- anndata now has GPU enabled CI. Made possibly by a grant from [CZI’s EOSS program](https://chanzuckerberg.com/eoss/) and managed via [Cirun](https://Cirun.io)[#1066](https://github.com/scverse/anndata/pull/1066)[#1084](https://github.com/scverse/anndata/pull/1084)[@Zethson](https://github.com/sponsors/Zethson)[@ivirshup](https://github.com/sponsors/ivirshup)

**Out of core**

- Concatenate on-disk anndata objects with [`anndata.experimental.concat_on_disk()`](https://anndata.scverse.org/generated/anndata.experimental.concat_on_disk.html#anndata.experimental.concat_on_disk)[#955](https://github.com/scverse/anndata/pull/955)[@selmanozleyen](https://github.com/sponsors/selmanozleyen)
- AnnData can now hold dask arrays with `scipy.sparse.spmatrix` chunks [#1114](https://github.com/scverse/anndata/pull/1114)[@ivirshup](https://github.com/sponsors/ivirshup)
- Public API for interacting with on disk sparse arrays: [`sparse_dataset()`](https://anndata.scverse.org/generated/anndata.io.sparse_dataset.html#anndata.io.sparse_dataset), [`CSRDataset`](https://anndata.scverse.org/generated/anndata.abc.CSRDataset.html#anndata.abc.CSRDataset), and [`CSCDataset`](https://anndata.scverse.org/generated/anndata.abc.CSCDataset.html#anndata.abc.CSCDataset)[#765](https://github.com/scverse/anndata/pull/765)[@ilan-gold](https://github.com/sponsors/ilan-gold)[@ivirshup](https://github.com/sponsors/ivirshup)
- Improved performance for simple slices of OOC sparse arrays [#1131](https://github.com/scverse/anndata/pull/1131)[@ivirshup](https://github.com/sponsors/ivirshup)

**Improved errors and warnings**

- Improved error messages when combining dataframes with duplicated column names [#1029](https://github.com/scverse/anndata/pull/1029)[@ivirshup](https://github.com/sponsors/ivirshup)
- Improved warnings when modifying views of `AlingedMappings`[#1016](https://github.com/scverse/anndata/pull/1016)[@flying-sheep](https://github.com/sponsors/flying-sheep)[@ivirshup](https://github.com/sponsors/ivirshup)
- `AnnDataReadError`s have been removed. The original error is now thrown with additional information in a note [#1055](https://github.com/scverse/anndata/pull/1055)[@ivirshup](https://github.com/sponsors/ivirshup)

#### Documentation[#](https://anndata.scverse.org/release-notes/index.html#id116)

- Added zarr examples to [file format docs](https://anndata.scverse.org/fileformat-prose.html)[#1162](https://github.com/scverse/anndata/pull/1162)[@ivirshup](https://github.com/sponsors/ivirshup)

#### Breaking changes[#](https://anndata.scverse.org/release-notes/index.html#id117)

- [`anndata.AnnData.transpose()`](https://anndata.scverse.org/generated/anndata.AnnData.transpose.html#anndata.AnnData.transpose) no longer copies unnecessarily. If you rely on the copying behavior, call `.copy` on the resulting object. [#1114](https://github.com/scverse/anndata/pull/1114)[@ivirshup](https://github.com/sponsors/ivirshup)

#### Other updates[#](https://anndata.scverse.org/release-notes/index.html#id118)

- Bump minimum python version to 3.9 [#1117](https://github.com/scverse/anndata/pull/1117)[@flying-sheep](https://github.com/sponsors/flying-sheep)

#### Deprecations[#](https://anndata.scverse.org/release-notes/index.html#deprecations)

- Deprecate `anndata.read`, which was just an alias for [`anndata.io.read_h5ad()`](https://anndata.scverse.org/generated/anndata.io.read_h5ad.html#anndata.io.read_h5ad)[#1108](https://github.com/scverse/anndata/pull/1108)[@ivirshup](https://github.com/sponsors/ivirshup).
- `dtype` argument to `AnnData` constructor is now deprecated [#1153](https://github.com/scverse/anndata/pull/1153)[@ivirshup](https://github.com/sponsors/ivirshup)

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id119)

- Fix shape inference on initialization when `X=None` is specified [#1121](https://github.com/scverse/anndata/pull/1121)[@flying-sheep](https://github.com/sponsors/flying-sheep)

## Version 0.9[#](https://anndata.scverse.org/release-notes/index.html#v0-9)

### 0.9.2 2023-07-25[#](https://anndata.scverse.org/release-notes/index.html#v0-9-2)

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id121)

- Views of `awkward.Array`s now work with `awkward&gt;=2.3`[#1040](https://github.com/scverse/anndata/pull/1040)[@ivirshup](https://github.com/sponsors/ivirshup)
- Fix ufuncs of views like `adata.X[:10].cov(axis=0)` returning views [#1043](https://github.com/scverse/anndata/pull/1043)[@flying-sheep](https://github.com/sponsors/flying-sheep)
- Fix instantiating AnnData where `.X` is a `DataFrame` with an integer valued index [#1002](https://github.com/scverse/anndata/pull/1002)[@flying-sheep](https://github.com/sponsors/flying-sheep)
- Fix [`read_zarr()`](https://anndata.scverse.org/generated/anndata.io.read_zarr.html#anndata.io.read_zarr) when used on `zarr.Group`[#1057](https://github.com/scverse/anndata/pull/1057)[@ivirshup](https://github.com/sponsors/ivirshup)

### 0.9.1 2023-04-11[#](https://anndata.scverse.org/release-notes/index.html#v0-9-1)

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id123)

- Fixing windows support [#958](https://github.com/scverse/anndata/pull/958)[@Koncopd](https://github.com/sponsors/Koncopd)

### 0.9.0 2023-04-11[#](https://anndata.scverse.org/release-notes/index.html#v0-9-0)

#### Features[#](https://anndata.scverse.org/release-notes/index.html#id125)

- Added experimental support for dask arrays [#813](https://github.com/scverse/anndata/pull/813)[@syelman](https://github.com/sponsors/syelman)[@rahulbshrestha](https://github.com/sponsors/rahulbshrestha)
- `obsm`, `varm` and `uns` can now hold [AwkwardArrays](https://awkward-array.org/quickstart.html)[#647](https://github.com/scverse/anndata/pull/647)[@giovp](https://github.com/sponsors/giovp), [@grst](https://github.com/sponsors/grst), [@ivirshup](https://github.com/sponsors/ivirshup)
- Added experimental functions [`anndata.experimental.read_dispatched()`](https://anndata.scverse.org/generated/anndata.experimental.read_dispatched.html#anndata.experimental.read_dispatched) and [`anndata.experimental.write_dispatched()`](https://anndata.scverse.org/generated/anndata.experimental.write_dispatched.html#anndata.experimental.write_dispatched) which allow customizing IO with a callback [#873](https://github.com/scverse/anndata/pull/873)[@ilan-gold](https://github.com/sponsors/ilan-gold)[@ivirshup](https://github.com/sponsors/ivirshup)
- Better error messages during IO [#734](https://github.com/scverse/anndata/pull/734)[@flying-sheep](https://github.com/sponsors/flying-sheep), [@ivirshup](https://github.com/sponsors/ivirshup)
- Unordered categorical columns are no longer cast to object during [`anndata.concat()`](https://anndata.scverse.org/generated/anndata.concat.html#anndata.concat)[#763](https://github.com/scverse/anndata/pull/763)[@ivirshup](https://github.com/sponsors/ivirshup)

#### Documentation[#](https://anndata.scverse.org/release-notes/index.html#id126)

- New tutorials for experimental features

> - [Dask Array Support to AnnData](https://anndata.scverse.org/tutorials/notebooks/anndata_dask_array.html) – [#886](https://github.com/scverse/anndata/pull/886)[@syelman](https://github.com/sponsors/syelman)
- [Dask + Zarr, but Remote!](https://anndata.scverse.org/tutorials/notebooks/%7Bread%2Cwrite%7D_dispatched.html) – [scverse/anndata-tutorials#17](https://github.com/scverse/anndata-tutorials/pull/17)[@ilan-gold](https://github.com/sponsors/ilan-gold)
- [Awkward Arrays in AnnData](https://anndata.scverse.org/tutorials/notebooks/awkward-arrays.html) – [scverse/anndata-tutorials#15](https://github.com/scverse/anndata-tutorials/pull/15)[@grst](https://github.com/sponsors/grst)
- [File format description](https://anndata.scverse.org/fileformat-prose.html) now includes a more formal specification [#882](https://github.com/scverse/anndata/pull/882)[@ivirshup](https://github.com/sponsors/ivirshup)
- [Interoperability](https://anndata.scverse.org/interoperability.html): new page on interoperability with other packages [#831](https://github.com/scverse/anndata/pull/831)[@ivirshup](https://github.com/sponsors/ivirshup)
- Expanded docstring more documentation for `backed` argument of [`anndata.io.read_h5ad()`](https://anndata.scverse.org/generated/anndata.io.read_h5ad.html#anndata.io.read_h5ad)[#812](https://github.com/scverse/anndata/pull/812)[@jeskowagner](https://github.com/sponsors/jeskowagner)
- Documented how to use alternative compression methods for the `h5ad` file format, see [`AnnData.write_h5ad()`](https://anndata.scverse.org/generated/anndata.AnnData.write_h5ad.html#anndata.AnnData.write_h5ad)[#857](https://github.com/scverse/anndata/pull/857)[@nigeil](https://github.com/sponsors/nigeil)
- General typo corrections 😅 [#870](https://github.com/scverse/anndata/pull/870)[@folded](https://github.com/sponsors/folded)

#### Breaking changes[#](https://anndata.scverse.org/release-notes/index.html#id127)

- The `AnnData``dtype` argument no longer defaults to `float32`[#854](https://github.com/scverse/anndata/pull/854)[@ivirshup](https://github.com/sponsors/ivirshup)
- Previously deprecated `force_dense` argument [`AnnData.write_h5ad()`](https://anndata.scverse.org/generated/anndata.AnnData.write_h5ad.html#anndata.AnnData.write_h5ad) has been removed. [#855](https://github.com/scverse/anndata/pull/855)[@ivirshup](https://github.com/sponsors/ivirshup)
- Previously deprecated behaviour around storing adjacency matrices in `uns` has been removed [#866](https://github.com/scverse/anndata/pull/866)[@ivirshup](https://github.com/sponsors/ivirshup)

#### Other updates[#](https://anndata.scverse.org/release-notes/index.html#id128)

- Bump minimum python version to 3.8 [#820](https://github.com/scverse/anndata/pull/820)[@ivirshup](https://github.com/sponsors/ivirshup)

#### Deprecations[#](https://anndata.scverse.org/release-notes/index.html#id129)

- `AnnData.concatenate()` is now deprecated in favour of [`anndata.concat()`](https://anndata.scverse.org/generated/anndata.concat.html#anndata.concat)[#845](https://github.com/scverse/anndata/pull/845)[@ivirshup](https://github.com/sponsors/ivirshup)

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id130)

- Fix warning from `rename_categories`[#790](https://github.com/scverse/anndata/pull/790)I Virshup
- Remove backwards compat checks for categories in `uns` when we can tell the file is new enough [#790](https://github.com/scverse/anndata/pull/790)I Virshup
- Categorical arrays are now created with a python `bool` instead of a `numpy.bool_`[#856](https://github.com/scverse/anndata/pull/856)
- Fixed order dependent outer concatenation bug [#904](https://github.com/scverse/anndata/pull/904)[@ivirshup](https://github.com/sponsors/ivirshup), reported by [@szalata](https://github.com/sponsors/szalata)
- Fixed bug in renaming categories [#790](https://github.com/scverse/anndata/pull/790)[@ivirshup](https://github.com/sponsors/ivirshup), reported by [@perrin-isir](https://github.com/sponsors/perrin-isir)
- Fixed IO bug when keys in `uns` ended in `_categories`[#806](https://github.com/scverse/anndata/pull/806)[@ivirshup](https://github.com/sponsors/ivirshup), reported by [@Hrovatin](https://github.com/sponsors/Hrovatin)
- Fixed `raw.to_adata` not populating `obs` aligned values when `raw` was assigned through the setter [#939](https://github.com/scverse/anndata/pull/939)[@ivirshup](https://github.com/sponsors/ivirshup)

## Version 0.8[#](https://anndata.scverse.org/release-notes/index.html#v0-8)

### 0.8.0 14th March, 2022[#](https://anndata.scverse.org/release-notes/index.html#v0-8-0)

#### IO Specification[#](https://anndata.scverse.org/release-notes/index.html#io-specification)

Warning

The on disk format of AnnData objects has been updated with this release. Previous releases of `anndata` will not be able to read all files written by this version.

For discussion of possible future solutions to this issue, see [#698](https://github.com/scverse/anndata/issues/698)

Internal handling of IO has been overhauled. This should make it much easier to support new datatypes, use partial access, and use `AnnData` internally in other formats.

- Each element should be tagged with an `encoding_type` and `encoding_version`. See updated docs on the [file format](https://anndata.scverse.org/fileformat-prose.html)
- Support for nullable integer and boolean data arrays. More data types to come!
- Experimental support for low level access to the IO API via [`read_elem()`](https://anndata.scverse.org/generated/anndata.io.read_elem.html#anndata.io.read_elem) and [`write_elem()`](https://anndata.scverse.org/generated/anndata.io.write_elem.html#anndata.io.write_elem)

#### Features[#](https://anndata.scverse.org/release-notes/index.html#id131)

- Added PyTorch dataloader [`AnnLoader`](https://anndata.scverse.org/generated/anndata.experimental.AnnLoader.html#anndata.experimental.AnnLoader) and lazy concatenation object [`AnnCollection`](https://anndata.scverse.org/generated/anndata.experimental.AnnCollection.html#anndata.experimental.AnnCollection). See the [tutorials](https://anndata-tutorials.readthedocs.io/en/latest/index.html)[#416](https://github.com/scverse/anndata/pull/416)S Rybakov
- Compatibility with `h5ad` files written from Julia [#569](https://github.com/scverse/anndata/pull/569)I Kats
- Many logging messages that should have been warnings are now warnings [#650](https://github.com/scverse/anndata/pull/650)I Virshup
- Significantly more efficient [`anndata.io.read_umi_tools()`](https://anndata.scverse.org/generated/anndata.io.read_umi_tools.html#anndata.io.read_umi_tools)[#661](https://github.com/scverse/anndata/pull/661)I Virshup
- Fixed deepcopy of a copy of a view retaining sparse matrix view mixin type [#670](https://github.com/scverse/anndata/pull/670)M Klein
- In many cases [`X`](https://anndata.scverse.org/generated/anndata.AnnData.X.html#anndata.AnnData.X) can now be `None`[#463](https://github.com/scverse/anndata/pull/463)R Cannoodt[#677](https://github.com/scverse/anndata/pull/677)I Virshup. Remaining work is documented in [#467](https://github.com/scverse/anndata/issues/467).
- Removed hard `xlrd` dependency I Virshup
- `obs` and `var` dataframes are no longer copied by default on `AnnData` instantiation [#371](https://github.com/scverse/anndata/issues/371)I Virshup

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id132)

- Fixed issue where `.copy` was creating sparse matrices views when copying [#670](https://github.com/scverse/anndata/pull/670)michalk8
- Fixed issue where `.X` matrix read in from `zarr` would always have `float32` values [#701](https://github.com/scverse/anndata/pull/701)I Virshup
- `Raw.to_adata`` now includes `obsp` in the output [#404](https://github.com/scverse/anndata/pull/404)G Eraslan

#### Dependencies[#](https://anndata.scverse.org/release-notes/index.html#dependencies)

- `xlrd` dropped as a hard dependency
- Now requires `h5py``v3.0.0` or newer

## Version 0.7[#](https://anndata.scverse.org/release-notes/index.html#v0-7)

### 0.7.8 9 November, 2021[#](https://anndata.scverse.org/release-notes/index.html#v0-7-8)

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id133)

- Re-include test helpers [#641](https://github.com/scverse/anndata/pull/641)I Virshup

### 0.7.7 9 November, 2021[#](https://anndata.scverse.org/release-notes/index.html#v0-7-7)

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id135)

- Fixed propagation of import error when importing `write_zarr` but not all dependencies are installed [#579](https://github.com/scverse/anndata/pull/579)R Hillje
- Fixed issue with `.uns` sub-dictionaries being referenced by copies [#576](https://github.com/scverse/anndata/pull/576)I Virshup
- Fixed out-of-bounds integer indices not raising [`IndexError`](https://docs.python.org/3/builtins/exceptions.html#IndexError)[#630](https://github.com/scverse/anndata/pull/630)M Klein
- Fixed backed `SparseDataset` indexing with scipy 1.7.2 [#638](https://github.com/scverse/anndata/pull/638)I Virshup

#### Development processes[#](https://anndata.scverse.org/release-notes/index.html#development-processes)

- Use PEPs 621 (standardized project metadata), 631 (standardized dependencies), and 660 (standardized editable installs) [#639](https://github.com/scverse/anndata/pull/639)I Virshup

### 0.7.6 11 April, 2021[#](https://anndata.scverse.org/release-notes/index.html#v0-7-6)

#### Features[#](https://anndata.scverse.org/release-notes/index.html#id136)

- Added [`anndata.AnnData.to_memory()`](https://anndata.scverse.org/generated/anndata.AnnData.to_memory.html#anndata.AnnData.to_memory) for returning an in memory object from a backed one [#470](https://github.com/scverse/anndata/pull/470)[#542](https://github.com/scverse/anndata/pull/542)V BergenI Virshup
- [`anndata.AnnData.write_loom()`](https://anndata.scverse.org/generated/anndata.AnnData.write_loom.html#anndata.AnnData.write_loom) now writes `obs_names` and `var_names` using the `Index`’s `.name` attribute, if set [#538](https://github.com/scverse/anndata/pull/538)I Virshup

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id137)

- Fixed bug where `np.str_` column names errored at write time [#457](https://github.com/scverse/anndata/pull/457)I Virshup
- Fixed “value.index does not match parent’s axis 0/1 names” error triggered when a data frame is stored in obsm/varm after obs_names/var_names is updated [#461](https://github.com/scverse/anndata/pull/461)G Eraslan
- Fixed `adata.write_csvs` when `adata` is a view [#462](https://github.com/scverse/anndata/pull/462)I Virshup
- Fixed null values being converted to strings when strings are converted to categorical [#529](https://github.com/scverse/anndata/pull/529)I Virshup
- Fixed handling of compression key word arguments [#536](https://github.com/scverse/anndata/pull/536)I Virshup
- Fixed copying a backed `AnnData` from changing which file the original object points at [#533](https://github.com/scverse/anndata/pull/533)ilia-kats
- Fixed a bug where calling `AnnData.concatenate` an `AnnData` with no variables would error [#537](https://github.com/scverse/anndata/pull/537)I Virshup

#### Deprecations[#](https://anndata.scverse.org/release-notes/index.html#id138)

- Passing positional arguments to [`anndata.io.read_loom()`](https://anndata.scverse.org/generated/anndata.io.read_loom.html#anndata.io.read_loom) besides the path is now deprecated [#538](https://github.com/scverse/anndata/pull/538)I Virshup
- [`anndata.io.read_loom()`](https://anndata.scverse.org/generated/anndata.io.read_loom.html#anndata.io.read_loom) arguments `obsm_names` and `varm_names` are now deprecated in favour of `obsm_mapping` and `varm_mapping`[#538](https://github.com/scverse/anndata/pull/538)I Virshup

### 0.7.5 12 November, 2020[#](https://anndata.scverse.org/release-notes/index.html#v0-7-5)

#### Functionality[#](https://anndata.scverse.org/release-notes/index.html#functionality)

- Added ipython tab completion and a useful return from `.keys` to `adata.uns`[#415](https://github.com/scverse/anndata/pull/415)I Virshup

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id139)

- Compatibility with `h5py&gt;=3` strings [#444](https://github.com/scverse/anndata/pull/444)I Virshup
- Allow `adata.raw=None`, as is documented [#447](https://github.com/scverse/anndata/pull/447)I Virshup
- Fix warnings from pandas 1.1 [#425](https://github.com/scverse/anndata/pull/425)I Virshup

### 0.7.4 10 July, 2020[#](https://anndata.scverse.org/release-notes/index.html#v0-7-4)

#### Concatenation overhaul [#378](https://github.com/scverse/anndata/pull/378)I Virshup[#](https://anndata.scverse.org/release-notes/index.html#concatenation-overhaul-378-i-virshup)

- New function [`anndata.concat()`](https://anndata.scverse.org/generated/anndata.concat.html#anndata.concat) for concatenating `AnnData` objects along either observations or variables
- New documentation section: [Concatenation](https://anndata.scverse.org/tutorials/concatenation.html)

#### Functionality[#](https://anndata.scverse.org/release-notes/index.html#id140)

- AnnData object created from dataframes with sparse values will have sparse `.X`[#395](https://github.com/scverse/anndata/pull/395)I Virshup

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id141)

- Fixed error from `AnnData.concatenate` by bumping minimum versions of numpy and pandas [#385](https://github.com/scverse/anndata/issues/385)
- Fixed colors being incorrectly changed when `AnnData` object was subset [#388](https://github.com/scverse/anndata/pull/388)

### 0.7.3 20 May, 2020[#](https://anndata.scverse.org/release-notes/index.html#v0-7-3)

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id142)

- Fixed bug where graphs used too much memory when copying [#381](https://github.com/scverse/anndata/pull/381)I Virshup

### 0.7.2 15 May, 2020[#](https://anndata.scverse.org/release-notes/index.html#v0-7-2)

#### Concatenation overhaul I Virshup[#](https://anndata.scverse.org/release-notes/index.html#concatenation-overhaul-i-virshup)

- Elements of `uns` can now be merged, see [#350](https://github.com/scverse/anndata/pull/350)
- Outer joins now work for `layers` and `obsm`, see [#352](https://github.com/scverse/anndata/pull/352)
- Fill value for outer joins can now be specified
- Expect improvements in performance, see [#303](https://github.com/scverse/anndata/issues/303)

#### Functionality[#](https://anndata.scverse.org/release-notes/index.html#id144)

- [`obsp`](https://anndata.scverse.org/generated/anndata.AnnData.obsp.html#anndata.AnnData.obsp) and [`varp`](https://anndata.scverse.org/generated/anndata.AnnData.varp.html#anndata.AnnData.varp) can now be transposed [#370](https://github.com/scverse/anndata/pull/370)A Wolf
- [`obs_names_make_unique()`](https://anndata.scverse.org/generated/anndata.AnnData.obs_names_make_unique.html#anndata.AnnData.obs_names_make_unique) is now better at making values unique, and will warn if ambiguities arise [#345](https://github.com/scverse/anndata/pull/345)M Weiden
- [`obsp`](https://anndata.scverse.org/generated/anndata.AnnData.obsp.html#anndata.AnnData.obsp) is now preferred for storing pairwise relationships between observations. In practice, this means there will be deprecation warnings and reformatting applied to objects which stored connectivities under `uns["neighbors"]`. Square matrices in [`uns`](https://anndata.scverse.org/generated/anndata.AnnData.uns.html#anndata.AnnData.uns) will no longer be sliced (use `.{obs,var}p` instead). [#337](https://github.com/scverse/anndata/pull/337)I Virshup
- [`ImplicitModificationWarning`](https://anndata.scverse.org/generated/anndata.ImplicitModificationWarning.html#anndata.ImplicitModificationWarning) is now exported [#315](https://github.com/scverse/anndata/pull/315)P Angerer
- Better support for [`ndarray`](https://numpy.org/doc/stable/reference/generated/numpy.ndarray.html#numpy.ndarray) subclasses stored in `AnnData` objects [#335](https://github.com/scverse/anndata/pull/335)michalk8

#### Bug fixes[#](https://anndata.scverse.org/release-notes/index.html#id145)

- Fixed inplace modification of [`Index`](https://pandas.pydata.org/docs/reference/api/pandas.Index.html#pandas.Index) objects by the make unique function [#348](https://github.com/scverse/anndata/pull/348)I Virshup
- Passing ambiguous keys to `AnnData.obs_vector` and `AnnData.var_vector` now throws errors [#340](https://github.com/scverse/anndata/pull/340)I Virshup
- Fix instantiating [`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData) objects from [`DataFrame`](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.html#pandas.DataFrame)[#316](https://github.com/scverse/anndata/pull/316)P Angerer
- Fixed indexing into `AnnData` objects with arrays like `adata[adata[:,gene].X&gt;0]`[#332](https://github.com/scverse/anndata/pull/332)I Virshup
- Fixed type of version [#315](https://github.com/scverse/anndata/pull/315)P Angerer
- Fixed deprecated import from [`pandas`](https://pandas.pydata.org/docs/index.html#module-pandas)[#319](https://github.com/scverse/anndata/pull/319)P Angerer

### 0.7.0 22 January, 2020[#](https://anndata.scverse.org/release-notes/index.html#v0-7-0)

Warning

Breaking changes introduced between `0.6.22.post1` and `0.7`:

- Elements of [`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData)s don’t have their dimensionality reduced when the main object is subset. This is to maintain consistency when subsetting. See discussion in [#145](https://github.com/scverse/anndata/issues/145).
- Internal modules like `anndata.core` are private and their contents are not stable: See [#174](https://github.com/scverse/anndata/issues/174).
- The old deprecated attributes `.smp*`. `.add` and `.data` have been removed.

#### View overhaul [#164](https://github.com/scverse/anndata/pull/164)[#](https://anndata.scverse.org/release-notes/index.html#view-overhaul-164)

- Indexing into a view no longer keeps a reference to intermediate view, see [#62](https://github.com/scverse/anndata/issues/62).
- Views are now lazy. Elements of view of AnnData are not indexed until they’re accessed.
- Indexing with scalars no longer reduces dimensionality of contained arrays, see [#145](https://github.com/scverse/anndata/issues/145).
- All elements of AnnData should now follow the same rules about how they’re subset, see [#145](https://github.com/scverse/anndata/issues/145).
- Can now index by observations and variables at the same time.

#### IO overhaul [#167](https://github.com/scverse/anndata/pull/167)[#](https://anndata.scverse.org/release-notes/index.html#io-overhaul-167)

- Reading and writing has been overhauled for simplification and speed.
- Time and memory usage can be half of previous in typical use cases
- Zarr backend now supports sparse arrays, and generally is closer to having the same features as HDF5.
- Backed mode should see significant speed and memory improvements for access along compressed dimensions and IO. PR [#241](https://github.com/scverse/anndata/pull/241).
- [`Categorical`](https://pandas.pydata.org/docs/reference/api/pandas.Categorical.html#pandas.Categorical)s can now be ordered (PR [#230](https://github.com/scverse/anndata/pull/230)) and written to disk with a large number of categories (PR [#217](https://github.com/scverse/anndata/pull/217)).

#### Mapping attributes overhaul (obsm, varm, layers, …)[#](https://anndata.scverse.org/release-notes/index.html#mapping-attributes-overhaul-obsm-varm-layers)

- New attributes [`obsp`](https://anndata.scverse.org/generated/anndata.AnnData.obsp.html#anndata.AnnData.obsp) and [`varp`](https://anndata.scverse.org/generated/anndata.AnnData.varp.html#anndata.AnnData.varp) have been added for two dimensional arrays where each axis corresponds to a single axis of the AnnData object. PR [#207](https://github.com/scverse/anndata/pull/207).
- These are intended to store values like cell-by-cell graphs, which are currently stored in [`uns`](https://anndata.scverse.org/generated/anndata.AnnData.uns.html#anndata.AnnData.uns).
- Sparse arrays are now allowed as values in all mapping attributes.
- DataFrames are now allowed as values in [`obsm`](https://anndata.scverse.org/generated/anndata.AnnData.obsm.html#anndata.AnnData.obsm) and [`varm`](https://anndata.scverse.org/generated/anndata.AnnData.varm.html#anndata.AnnData.varm).
- All mapping attributes now share an implementation and will have the same behaviour. PR [#164](https://github.com/scverse/anndata/pull/164).

#### Miscellaneous improvements[#](https://anndata.scverse.org/release-notes/index.html#miscellaneous-improvements)

- Mapping attributes now have ipython tab completion (e.g. `adata.obsm["\\t` can provide suggestions) PR [#183](https://github.com/scverse/anndata/pull/183).
- [`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData) attributes are now delete-able (e.g. `deladata.raw`) PR [#242](https://github.com/scverse/anndata/pull/242).
- Many many bug fixes

## Version 0.6[#](https://anndata.scverse.org/release-notes/index.html#v0-6)

### 0.6.0 1 May, 2018[#](https://anndata.scverse.org/release-notes/index.html#v0-6-0)

- compatibility with Seurat converter
- tremendous speedup for `~anndata.AnnData.concatenate`
- bug fix for deep copy of unstructured annotation after slicing
- bug fix for reading HDF5 stored single-category annotations
- `'outerjoin'` concatenation: adds zeros for concatenation of sparse data and nans for dense data
- better memory efficiency in loom exports

## Version 0.5[#](https://anndata.scverse.org/release-notes/index.html#v0-5)

### 0.5.0 9 February, 2018[#](https://anndata.scverse.org/release-notes/index.html#v0-5-0)

- inform about duplicates in [`var_names`](https://anndata.scverse.org/generated/anndata.AnnData.var_names.html#anndata.AnnData.var_names) and resolve them using [`var_names_make_unique()`](https://anndata.scverse.org/generated/anndata.AnnData.var_names_make_unique.html#anndata.AnnData.var_names_make_unique)
- automatically remove unused categories after slicing
- read/write [.loom](https://loompy.org) files using loompy 2
- fixed read/write for a few text file formats
- read [UMI tools](https://github.com/CGATOxford/UMI-tools) files: [`read_umi_tools()`](https://anndata.scverse.org/generated/anndata.io.read_umi_tools.html#anndata.io.read_umi_tools)

## Version 0.4[#](https://anndata.scverse.org/release-notes/index.html#v0-4)

### 0.4.0 23 December, 2017[#](https://anndata.scverse.org/release-notes/index.html#v0-4-0)

- read/write [.loom](https://loompy.org) files
- scalability beyond dataset sizes that fit into memory: see this [blog post](http://falexwolf.de/blog/171223_AnnData_indexing_views_HDF5-backing/)
- [`AnnData`](https://anndata.scverse.org/generated/anndata.AnnData.html#anndata.AnnData) has a [`raw`](https://anndata.scverse.org/generated/anndata.AnnData.raw.html#anndata.AnnData.raw) attribute, which simplifies storing the data matrix when you consider it *raw*: see the [clustering tutorial](https://github.com/scverse/scanpy_usage/tree/master/170505_seurat)

---[[1](https://anndata.scverse.org/release-notes/index.html#id80)]

`anndata.typing.AxisStorable` was later again renamed to [`Storable`](https://anndata.scverse.org/typing.html#anndata.typing.Storable)
