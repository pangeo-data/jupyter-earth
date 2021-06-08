# References regarding our base image:
# - ubuntu:20.04
# - pangeo/base-image definition:       https://github.com/pangeo-data/pangeo-docker-images/blob/master/base-image
# - pangeo/pangeo-notebook definition:  https://github.com/pangeo-data/pangeo-docker-images/tree/master/pangeo-notebook
# - pangeo/pangeo-notebook tags:        https://hub.docker.com/r/pangeo/pangeo-notebook/tags
# - pangeo-notebook conda package:      https://github.com/conda-forge/pangeo-notebook-feedstock/blob/master/recipe/meta.yaml
#
FROM pangeo/pangeo-notebook:2021.05.15
ARG DEBIAN_FRONTEND=noninteractive

USER root
RUN echo "Installing apt-get packages..." \
 && apt-get update \
 && apt-get install -y \
        groff \
            # The aws CLI apparently relies on "groff"
            # Issue about including it in future versions of pangeo-notebook:
            # https://github.com/pangeo-data/pangeo-docker-images/issues/216
        nano \
            # A terminal file editor, vim is already made available
        micro \
            # A friendly terminal editor, even easier than nano for new users
        mc \
            # powerful terminal-based file manager, better than the one in JLab
        build-essential gfortran 
            # Regular build tools for compiling common stuff
            
# for nbconvert
RUN apt-get update > /dev/null && \
    apt-get -qq install --yes \
            texlive-xetex \
            texlive-fonts-recommended \
            texlive-plain-generic > /dev/null

# Some common geospatial dependencies
RUN apt-get update > /dev/null && \
    apt-get -qq install --yes \
            libspatialindex-dev \
            libgeos-dev \
            libproj-dev \
            proj-data \
            proj-bin \
            graphviz \
            ffmpeg > /dev/null
            
RUN rm -rf /var/lib/apt/lists/*

# Switch user away from ROOT for the rest
USER ${NB_USER}

# We only need to install packages not listed in this file already:
# https://github.com/pangeo-data/pangeo-docker-images/blob/master/pangeo-notebook/packages.txt
RUN echo "Installing conda packages..." \
 && mamba install -n ${CONDA_ENV} -y \
        google-cloud-sdk \
        ipython jupyter jupyterlab jupyter-book jupyter-repo2docker \
        retrolab jupyterlab-git jupyterlab-system-monitor \
        jupyterlab-variableinspector jupyterlab-drawio nbdime \
        ipywidgets ipympl ipycanvas ipydatagrid \
        numpy scipy matplotlib sympy scikit-image scikit-learn networkx \
        pandas seaborn statsmodels xarray xlrd \
        geopandas h5py netcdf4 pytables jupyterlab-geojson \
        zarr intake intake-xarray intake-esm pydap rasterio \
        altair bokeh jupyter_bokeh plotly plotly_express bqplot \
        cython numba fortran-magic \
        dask dask-labextension distributed ipyparallel \
        black jedi pep8 flake8 pyflakes pylint pytest pytest-cov \
        sphinx numpydoc pandoc jupytext \
        yaml lxml requests \
        nodejs \
        pip \
 && echo "Installing conda packages complete!"


# We only need to install packages not listed in this file already:
# https://github.com/pangeo-data/pangeo-docker-images/blob/master/pangeo-notebook/packages.txt
RUN echo "Installing pip packages..." \
 && export PATH=${NB_PYTHON_PREFIX}/bin:${PATH} \
 && pip install --no-cache-dir \
        jupyterlab==3.1.0a11 \
            # ref: https://github.com/jupyterlab/jupyterlab
        jupyterlab-link-share \
            # ref: https://github.com/jupyterlab-contrib/jupyterlab-link-share
        julia \
            # If we install julia w/conda, it pulls the entirety of Julia in...
 && echo "Installing pip packages complete!"

# Configure conda/mamba to create new environments within the home folder by
# default. This allows the environments to remain in between restarts of the
# container if only the home folder is persisted.
RUN conda config --system --prepend envs_dirs '~/.conda/envs'

# User environment variables
# Configure PIP always installs to the user's home directory
ENV PIP_USER=True

# Set up micro as the default EDITOR (git, etc). Advanced users will reconfigure this 
# to vim/emacs/etc, but this will ensure that less unix-experienced ones have a good 
# first experience
ENV EDITOR=micro
ENV VISUAL=micro
