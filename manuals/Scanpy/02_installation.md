# Installation

> To use scanpy from another project, install it using your favourite environment manager: Hatch (recommended) Adding scanpy[leiden] to your dependencies is enough. See below for how to use Scanpy’s ...

# Installation[#](https://scanpy.scverse.org/en/stable/installation.html#installation)

To use `scanpy` from another project, install it using your favourite environment manager:

 Hatch (recommended)

Adding `scanpy[leiden]` to your dependencies is enough. See below for how to use Scanpy’s [Development Version](https://scanpy.scverse.org/en/stable/installation.html#dev-install-instructions).

 Pip

If you prefer to exclusively use pip run:

```
$ pip install 'scanpy[leiden]'
```

 Conda

After installing installing e.g. [Miniconda](https://docs.anaconda.com/miniconda/miniconda-install/), run:

```
$ conda install -c conda-forge scanpy python-igraph leidenalg
```

Pull Scanpy [from PyPI](https://pypi.org/project/scanpy):

```
$ pip install scanpy
```

If you use Hatch or pip, the extra `[leiden]` installs two packages that are needed for popular parts of scanpy but aren’t requirements: [igraph](https://python.igraph.org/)[[Csárdi and Nepusz, 2006](https://scanpy.scverse.org/en/stable/references.html#id17)] and [leiden](https://leidenalg.readthedocs.io/en/latest/index.html)[[Traag *et al.*, 2019](https://scanpy.scverse.org/en/stable/references.html#id70)]. If you use conda, you should to add these dependencies to your environment individually.

## Development Version[#](https://scanpy.scverse.org/en/stable/installation.html#dev-install-instructions)

To work with the latest version [on GitHub](https://github.com/scverse/scanpy): clone the repository and `cd` into its root directory.

```
$ gh repo clone scverse/scanpy
$ cd scanpy
```

 Hatch (recommended)

To use one of the predefined [Hatch environments](https://hatch.pypa.io/latest/tutorials/environment/basic-usage/) in `hatch.toml`, run either `hatchtest[args]` or `hatchrun[env:]command[...args]`, e.g.:

```
$ hatch test -p               # run tests in parallel
$ hatch run docs:build        # build docs
$ hatch run towncrier:create  # create changelog entry
```

 Pip

If you are using `pip&gt;=21.3`, an editable install can be made:

```
$ python -m venv .venv
$ source .venv/bin/activate
$ pip install --group=dev --group=test -e .
```

 Conda

If you want to let `conda` handle the installations of dependencies, do:

```
$ pipx install beni
$ beni pyproject.toml > environment.yml
$ conda env create -f environment.yml
$ conda activate scanpy
$ pip install --group=dev --group=test --group=doc -e .
```

For instructions on how to work with the code, see the [contribution guide](https://scanpy.scverse.org/en/stable/dev/index.html#contribution-guide).

## GPU acceleration[#](https://scanpy.scverse.org/en/stable/installation.html#gpu-acceleration)

If you have a GPU, consider [rapids-singlecell](https://rapids-singlecell.scverse.org/), which mirrors the scanpy API with order-of-magnitude speedups. See its [documentation](https://rapids-singlecell.scverse.org/) for installation and usage instructions.

## Docker[#](https://scanpy.scverse.org/en/stable/installation.html#docker)

If you’re using [Docker](https://en.wikipedia.org/wiki/Docker_(software)), you can use e.g. the image [gcfntnu/scanpy](https://hub.docker.com/r/gcfntnu/scanpy) from Docker Hub.

## Troubleshooting[#](https://scanpy.scverse.org/en/stable/installation.html#troubleshooting)

If you get a `Permissiondenied` error, never use `sudopip`. Instead, use virtual environments or:

```
$ pip install --user scanpy
```
