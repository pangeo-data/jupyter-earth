# jupyter-earth

**Jupyter meets the Earth**: combining research use cases in geosciences with technical developments within the Jupyter and Pangeo ecosystems.

[![NSF-1928406](https://img.shields.io/badge/NSF-1928406-blue)](https://nsf.gov/awardsearch/showAward?AWD_ID=1928406)
[![NSF-1928374](https://img.shields.io/badge/NSF-1928374-blue)](https://nsf.gov/awardsearch/showAward?AWD_ID=1928374)

This repository consists of two parts:

1. The Dockerfile that builds into the experimental, cloud-based JupyterHub environment. 
2. The source files and notebooks for the Jupyter meets the Earth project website.

## Dockerfile

The `hub.jupytearth.org-image/` folder hosts the Dockerfile that builds into the base environment used in https://hub.jupytearth.org. This is an experimental environment our team uses to perform various geoscience workflows and co-develop itself based on user feedback / issue reports.

The cloud infrastructure declaration and Helm chart configuration for https://hub.jupytearth.org lives in https://github.com/2i2c-org/pilot-hubs/.

### How to work with this Dockerfile?

Whenever a pull request is made modifying the image, a GitHub Workflow will try
verify if it can successfully rebuild.

Whenever a pull request is merged modifying the image, a GitHub Workflow will
build and push the image and emit a message on how to update
https://hub.jupytearth.org to start making use of the new image. In short, it is
to visit https://hub.jupytearth.org/services/configurator/ and enter the image
name:tag which is described in the executed GitHub Workflow.

**The corresponding GitHub Workflow:** `.github/workflows/build-image.yaml`

## Project website

The `docs/` folder contains source files for the Jupyter meets the Earth project website: https://pangeo-data.github.io/jupyter-earth/. This is a Binder-ready site, and you can execute the example notebooks presented in the documentation interactively using the following Binder link:

[![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/pangeo-data/jupyter-earth/HEAD/?urlpath=lab)


### How to work with this documentation?

The documentation of this repository is built using the beta version of
[Jupyter Book](https://beta.jupyterbook.org). It will be automatically updated
any time changes are made to the `docs/` folder and pushed to master.

**The corresponding GitHub Workflow:** `.github/workflows/build-doc.yaml`

### To build the documentation locally...

Run these steps:

* **Install the Jupyter Book 2.0 CLI** using `conda`:

  ```
  conda env create -f environment.yml
  conda activate jupyter-earth
  ```
* **Build the `docs/` folder**:

  ```
  jupyter-book build docs/
  ```

This will create a folder in `docs/_build/html` where you can preview your
site. For example, with `chrome docs/_build/html/index.html`.
