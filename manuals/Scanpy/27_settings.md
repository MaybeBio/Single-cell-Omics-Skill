# Settings

> A convenience function for setting some default matplotlib.rcParams and a high-resolution jupyter display backend useful for use in notebooks. An object that allows configuring Scanpy. Some selecte...

# Settings[#](https://scanpy.scverse.org/en/stable/api/settings.html#settings)

A convenience function for setting some default [`matplotlib.rcParams`](https://matplotlib.org/stable/api/matplotlib_configuration_api.html#matplotlib.rcParams) and a high-resolution jupyter display backend useful for use in notebooks.

|  |
|--|

An object that allows configuring Scanpy.

| settings.override | Provides local override via keyword arguments as a context manager. |
|-------------------|---------------------------------------------------------------------|

Some selected settings are discussed in the following.

Presets allow to set the behavior of many scanpy functions at once:

|  |
|--|

Verbosity controls the amount of logging output:

|  |
|--|

Influence the global behavior of plotting functions. In non-interactive scripts, you’d usually want to set [`settings.autoshow`](https://scanpy.scverse.org/en/stable/api/generated/scanpy.settings.html#scanpy.settings.autoshow) to `False`.

| settings.autosave |  |
|-------------------|--|

IO related settings for saving figures, caching files and storing datasets.

| settings.cachedir |  |
|-------------------|--|
| settings.datasetdir |  |
| settings.file_format_figs |  |
| settings.file_format_data |  |

Print versions of packages that might influence numerical results.

|  |
|--|
