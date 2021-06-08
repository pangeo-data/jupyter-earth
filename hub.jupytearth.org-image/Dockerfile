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
# We only need to install packages not listed in this file already:
# https://github.com/pangeo-data/pangeo-docker-images/blob/master/pangeo-notebook/apt.txt
RUN echo "Installing apt-get packages..." \
 && apt-get update > /dev/null \
 && apt-get -qq install --yes \
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
        build-essential gfortran \
            # Regular build tools for compiling common stuff
        texlive-xetex texlive-fonts-recommended texlive-plain-generic \
            # Dependencies for nbconvert
        # common geospatial dependencies:
        libspatialindex-dev \
        libgeos-dev \
        libproj-dev \
        proj-data \
        proj-bin \
        graphviz \
        ffmpeg \
 && rm -rf /var/lib/apt/lists/*

# Switch user away from ROOT for the rest
USER ${NB_USER}


# We only need to install packages not listed in this file already:
# https://github.com/pangeo-data/pangeo-docker-images/blob/master/pangeo-notebook/packages.txt
RUN echo "Installing conda packages..." \
 && mamba install -n ${CONDA_ENV} -y \
        #
        # visualization:
        altair \
        bqplot \
        plotly \
        plotly_express \
        seaborn \
        ipycanvas \
        ipympl \
        jupyter_bokeh \
        jupyterlab-geojson \
        #
        # tests and formatting:
        black \
        flake8 \
        pep8 \
        pyflakes \
        pylint \
        pytest \
        pytest-cov \
        #
        # documentation:
        jupyter-book \
        jupytext \
        numpydoc \
        sphinx \
        #
        # data:
        ipydatagrid \
        ipyparallel \
        lxml \
        pytables \
        statsmodels \
        xlrd \
        #
        # jupyterlab and extensions:
        jupyterlab==3.1.0a11 \
            # ref: https://github.com/jupyterlab/jupyterlab
        jupyter-repo2docker \
        # jupyterlab-drawio \
            # DISABLED: One can't save work so it could be a disservice to have
            #           it enabled: https://github.com/QuantStack/jupyterlab-drawio/issues/6
        jupyterlab-link-share \
            # ref: https://github.com/jupyterlab-contrib/jupyterlab-link-share
        jupyterlab-git \
        jupyterlab-system-monitor \
        jupyterlab-variableinspector \
        nbdime \
        retrolab \
        #
        # other
        cython \
        fortran-magic \
        google-cloud-sdk \
        julia \
        nodejs \
        sympy \
 && echo "Installing conda packages complete!"


# We use a conda first approach in this Dockerfile, so only install pip packages
# if you have a clear reason to not use conda.
# https://github.com/pangeo-data/pangeo-docker-images/blob/master/pangeo-notebook/packages.txt
#
# RUN echo "Installing pip packages..." \
#  && export PATH=${NB_PYTHON_PREFIX}/bin:${PATH} \
#  && pip install --no-cache-dir \
#         <package-name-here>
#  && echo "Installing pip packages complete!"


# Configure conda/mamba to create new environments within the home folder by
# default. This allows the environments to remain in between restarts of the
# container if only the home folder is persisted.
RUN conda config --system --prepend envs_dirs '~/.conda/envs'


# User environment variables
# Configure PIP always installs to the user's home directory
ENV PIP_USER=True

# Set up micro as the default EDITOR (git, etc). Advanced users will reconfigure
# this to vim/emacs/etc, but this will ensure that less unix-experienced ones
# have a good first experience
ENV EDITOR=micro
ENV VISUAL=micro
