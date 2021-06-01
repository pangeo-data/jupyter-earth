# References regarding our base image:
# - ubuntu:20.04
# - pangeo/base-image definition:       https://github.com/pangeo-data/pangeo-docker-images/blob/master/base-image
# - pangeo/pangeo-notebook definition:  https://github.com/pangeo-data/pangeo-docker-images/tree/master/pangeo-notebook
# - pangeo/pangeo-notebook tags:        https://hub.docker.com/r/pangeo/pangeo-notebook/tags
# - pangeo-notebook conda package:      https://github.com/conda-forge/pangeo-notebook-feedstock/blob/master/recipe/meta.yaml
#
FROM pangeo/pangeo-notebook:2021.05.15
ARG DEBIAN_FRONTEND=noninteractive

# DISABLED: apt packages.
#
#           Enable if we have a apt package we want to install!
#
# USER root
# RUN echo "Installing apt-get packages..." \
#  && apt-get update \
#  && apt-get install -y \
#         gcc \
#  && rm -rf /var/lib/apt/lists/*
# USER ${NB_USER}


# We only need to install packages not listed in this file already:
# https://github.com/pangeo-data/pangeo-docker-images/blob/master/pangeo-notebook/packages.txt
RUN echo "Installing conda packages..." \
 && mamba install -n ${CONDA_ENV} -y \
        google-cloud-sdk \
 && echo "Installing conda packages complete!"


# We only need to install packages not listed in this file already:
# https://github.com/pangeo-data/pangeo-docker-images/blob/master/pangeo-notebook/packages.txt
RUN echo "Installing pip packages..." \
 && export PATH=${NB_PYTHON_PREFIX}/bin:${PATH} \
 && pip install --no-cache-dir \
        jupyter-resource-usage \
            # ref: https://github.com/jupyter-server/jupyter-resource-usage
        jupyterlab==3.1.0a10 \
            # ref: https://github.com/jupyterlab/jupyterlab
        jupyterlab-link-share \
            # ref: https://github.com/jupyterlab-contrib/jupyterlab-link-share
 && echo "Installing pip packages complete!"



# DISABLED: JupyterLab 2 extensions.
#
#           If we want to install JupyterLab 2 based extensions we should first
#           install node and add extensions here, but JupyterLab 3 extensions
#           should preferably be installed as Python packages.
#
# RUN echo "Installing jupyterlab extensions..." \
#  && export PATH=${NB_PYTHON_PREFIX}/bin:${PATH} \
#  && jupyter labextension install -y --clean \
#         @lckr/jupyterlab_variableinspector \
#  && echo "Installing jupyterlab extensions complete!"



# Configure conda/mamba to create new environments within the home folder by
# default. This allows the environments to remain in between restarts of the
# container if only the home folder is persisted.
RUN conda config --system --prepend envs_dirs '~/.conda/envs'
