# Scanpy – Single-Cell Analysis in Python

> Stars PyPI PyPI Downloads Conda Forge Conda Forge Downloads Docs CI Discourse topics Chat Powered by NumFOCUS Scanpy is a scalable toolkit for analyzing single-cell gene expression data built joint...

[![Stars](https://img.shields.io/github/stars/scverse/scanpy?style=flat&logo=GitHub&color=yellow)](https://github.com/scverse/scanpy/stargazers)[![PyPI](https://img.shields.io/pypi/v/scanpy)](https://pypi.org/project/scanpy)[![PyPI Downloads](https://img.shields.io/pepy/dt/scanpy?logo=pypi)](https://pepy.tech/project/scanpy)[![Conda Forge](https://img.shields.io/conda/vn/conda-forge/scanpy)![Conda Forge Downloads](https://img.shields.io/conda/dn/conda-forge/scanpy?logo=condaforge)](https://anaconda.org/conda-forge/scanpy)[![Docs](https://readthedocs.com/projects/icb-scanpy/badge/?version=latest)](https://scanpy.scverse.org/)[![CI](https://github.com/scverse/scanpy/actions/workflows/ci.yml/badge.svg)](https://github.com/scverse/scanpy/actions/workflows/ci.yml)[![Discourse topics](https://img.shields.io/discourse/posts?color=yellow&logo=discourse&server=https%3A%2F%2Fdiscourse.scverse.org)](https://discourse.scverse.org/)[![Chat](https://img.shields.io/badge/zulip-join_chat-%2367b08f.svg)](https://scverse.zulipchat.com)[![Powered by NumFOCUS](https://img.shields.io/badge/powered%20by-NumFOCUS-orange.svg?style=flat&colorA=E1523D&colorB=007D8A)](https://numfocus.org/)

# Scanpy – Single-Cell Analysis in Python[#](https://scanpy.scverse.org/en/stable/index.html#scanpy-single-cell-analysis-in-python)

Scanpy is a scalable toolkit for analyzing single-cell gene expression data built jointly with [anndata](https://anndata.scverse.org/). It includes preprocessing, visualization, clustering, trajectory inference and differential expression testing. The Python-based implementation efficiently deals with datasets of more than one million cells. For datasets too large to fit into memory, [many scanpy functions](https://github.com/scverse/scanpy/issues/2578) are now compatible with [dask](https://docs.dask.org/) (**warning: experimental**).

Discuss usage on the scverse [Discourse](https://discourse.scverse.org/). Read the [documentation](https://scanpy.scverse.org/). If you’d like to contribute by opening an issue or creating a pull request, please take a look at our [contribution guide](https://scanpy.scverse.org/en/stable/dev/index.html).

scanpy is part of the scverse® project ([website](https://scverse.org), [governance](https://scverse.org/about/roles)) and is fiscally sponsored by [NumFOCUS](https://numfocus.org/). If you like scverse® and want to support our mission, please consider making a tax-deductible [donation](https://numfocus.org/donate-to-scverse) to help the project pay for developer time, professional services, travel, workshops, and a variety of other needs.

[![](https://raw.githubusercontent.com/numfocus/templates/master/images/numfocus-logo.png)](https://numfocus.org/project/scverse)

## Public API[#](https://scanpy.scverse.org/en/stable/index.html#public-api)

Our public API is documented in the [API section](https://scanpy.scverse.org/page/api/) of these docs. We cannot guarantee the stability of our internal APIs, whether it’s the location of a function, its arguments, or something else. In other words, we do not officially support (or encourage users to do) something like `fromscanpy.loggingimportdebug` as `logging` is not documented, even though it does not contain a [leading underscore](https://peps.python.org/pep-0008/#public-and-internal-interfaces). However, we are aware that many users do use these internal APIs and thus encourage them to [open an issue](https://github.com/scverse/scanpy/issues/new/choose) or migrate to the public API. That is, if something is missing from our public API as documented, for example a feature you wish to be exported publicly, please open an issue.

 Installation 

New to *scanpy*? Check out the installation guide.

[Installation](https://scanpy.scverse.org/en/stable/installation.html) Tutorials 

The tutorials walk you through real-world applications of scanpy.

[Tutorials](https://scanpy.scverse.org/en/stable/tutorials/index.html) API reference 

The API reference contains a detailed description of the scanpy API.

[API](https://scanpy.scverse.org/en/stable/api/index.html) Discussion 

Need help? Reach out on our forum to get your questions answered!

[https://discourse.scverse.org](https://discourse.scverse.org) GitHub 

Find a bug? Interested in improving scanpy? Checkout our GitHub for the latest developments.

[https://github.com/scverse/scanpy](https://github.com/scverse/scanpy) GPU acceleration 

Working with millions of cells and have a GPU available? [rapids-singlecell](https://rapids-singlecell.scverse.org/) mirrors the scanpy API with order-of-magnitude speedups on core preprocessing, neighbors, embedding, and clustering steps.

[https://rapids-singlecell.scverse.org/](https://rapids-singlecell.scverse.org/)

**Other resources**

- Follow changes in the [release notes](https://scanpy.scverse.org/en/stable/release-notes/index.html#release-notes).
- Discover tools that build on or complement scanpy in the [scverse ecosystem](https://scverse.org/packages/#ecosystem), and follow scverse-wide news at [scverse.org/blog](https://scverse.org/blog/).
- Check out our [contribution guide](https://scanpy.scverse.org/en/stable/dev/index.html#contribution-guide) for development practices.
- Consider citing [Genome Biology (2018)](https://doi.org/10.1186/s13059-017-1382-0) along with original [references](https://scanpy.scverse.org/en/stable/references.html).
