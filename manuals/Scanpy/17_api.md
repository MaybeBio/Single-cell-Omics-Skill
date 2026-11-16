# API

> Import Scanpy as: Array type support: Different APIs have different levels of support for array types, and this page lists the supported array types for each function(⚡ indicates support of the typ...

# API[#](https://scanpy.scverse.org/en/stable/api/index.html#api)

Import Scanpy as:

```
import scanpy as sc
```

Note

Additional functionality is available across the broader [scverse ecosystem](https://scverse.org/packages/#ecosystem), with some tools wrapped in the [`scanpy.external`](https://scanpy.scverse.org/en/stable/external/index.html#module-scanpy.external) module.

## Array type support[#](https://scanpy.scverse.org/en/stable/api/index.html#array-support)

Different APIs have different levels of support for array types, and this page lists the supported array types for each function (⚡ indicates support of the type as chunk in a dask [`Array`](https://docs.dask.org/en/stable/generated/dask.array.Array.html#dask.array.Array)):

| Function | ndarray | csr_array/csr_matrix | csc_array/csc_matrix | Array API |
|----------|---------|----------------------|----------------------|-----------|
| ✅ | ✅ | ✅ | ❌ |  |
| ✅⚡ | ✅⚡ | ✅⚡ | ❌ |  |
| ✅ | ✅ | ✅ | ✅ |  |
| ✅⚡ | ✅⚡ | ✅⚡ | ✅ |  |
| ✅ | ❌ | ❌ | ❌ |  |
| ✅ | ✅ | ❌ | ❌ |  |
| ✅⚡ | ✅⚡ | ✅⚡ | ✅ |  |
| ✅⚡ | ✅⚡ | ✅⚡ | ✅ |  |
| ✅ | ❌ | ❌ | ❌ |  |
| ✅ | ✅ | ✅ | ❌ |  |
| ✅⚡ | ✅⚡ | ✅ | ✅[*] |  |
| ✅⚡ | ✅⚡ | ✅⚡ | ✅ |  |
| ✅ | ✅ | ✅ | ✅ |  |
| ✅⚡ | ✅⚡ | ✅⚡ | ✅ |  |
| ✅⚡ | ✅⚡ | ❌ | ✅ |  |
| ✅⚡ | ✅⚡ | ✅ | ✅ |  |
| ✅ | ❌ | ❌ | ❌ |  |
| ✅⚡ | ✅⚡ | ✅⚡ | ✅ |  |
| ✅⚡ | ✅⚡ | ✅⚡ | ✅ |  |
| ✅ | ✅ | ✅ | ❌ |  |
| ✅ | ✅ | ✅ | ❌ |  |
| ✅ | ✅ | ✅ | ❌ |  |
| ✅ | ✅ | ✅ | ❌ |  |
| ✅ | ✅ | ✅ | ❌ |  |
| ✅ | ✅ | ✅ | ❌ |  |
| ✅ | ❌ | ❌ | ❌ |  |
| ✅ | ✅ | ✅ | ❌ |  |
| ✅ | ✅ | ✅ | ❌ |  |
| ✅ | ✅ | ✅ | ❌ |  |
| ✅ | ✅ | ✅ | ❌ |  |
| ✅ | ✅ | ✅ | ✅ |  |
| ✅ | ✅ | ✅ | ✅ |  |
| ✅ | ✅ | ✅ | ❌ |  |
| ✅ | ✅ | ✅ | ❌ |  |

[[*](https://scanpy.scverse.org/en/stable/api/index.html#array-support-pp-highly-variable-genes-xp-ref-0)]

Except for `flavor='seurat_v3'` with a `batch_key`.
