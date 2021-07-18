# References regarding our base image:
# - ubuntu:20.04
# - pangeo/base-image definition:       https://github.com/pangeo-data/pangeo-docker-images/blob/master/base-image
# - pangeo/pangeo-notebook definition:  https://github.com/pangeo-data/pangeo-docker-images/tree/master/pangeo-notebook
# - pangeo/pangeo-notebook tags:        https://hub.docker.com/r/pangeo/pangeo-notebook/tags
# - pangeo-notebook conda package:      https://github.com/conda-forge/pangeo-notebook-feedstock/blob/master/recipe/meta.yaml
#
FROM pangeo/pangeo-notebook:2021.07.06
ARG DEBIAN_FRONTEND=noninteractive


USER root
# We only need to install packages not listed in this file already:
# https://github.com/pangeo-data/pangeo-docker-images/blob/master/pangeo-notebook/apt.txt
RUN echo "Installing apt-get packages..." \
 && apt-get update > /dev/null \
 && apt-get -qq install --yes \
        curl \
        groff \
            # The aws CLI apparently relies on "groff"
            # Issue about including it in future versions of pangeo-notebook:
            # https://github.com/pangeo-data/pangeo-docker-images/issues/216
        emacs-nox python-mode emacs-goodies-el \
            # Basic Emacs configuration for general development.
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
        rsync \
            # for use with jupyterhub-ssh, generate a token at
            # hub.jupytearth.org/hub/token and connect to
            # <hub-username>@hub.jupytearth.org using the token as password
        # common geospatial dependencies:
        libspatialindex-dev \
        libgeos-dev \
        libproj-dev \
        proj-data \
        proj-bin \
        graphviz \
        ffmpeg \
 && rm -rf /var/lib/apt/lists/*


# Install Julia
ENV JULIA_PATH /srv/julia
ENV JULIA_DEPOT_PATH ${JULIA_PATH}/pkg
ENV JULIA_VERSION 1.6.1
ENV PATH $PATH:${JULIA_PATH}/bin
RUN mkdir -p ${JULIA_PATH} \
 && curl -sSL "https://julialang-s3.julialang.org/bin/linux/x64/${JULIA_VERSION%[.-]*}/julia-${JULIA_VERSION}-linux-x86_64.tar.gz" \
  | tar -xz -C ${JULIA_PATH} --strip-components 1 \
 && mkdir -p ${JULIA_DEPOT_PATH} \
 && chown ${NB_UID}:${NB_UID} ${JULIA_DEPOT_PATH}


# Switch user away from ROOT for the rest
USER ${NB_USER}

# Prepare Julia
RUN julia --eval 'using Pkg; Pkg.add("IJulia"); using IJulia; installkernel("Julia");' \
 && julia --eval 'using Pkg; Pkg.instantiate(); Pkg.resolve(); pkg"precompile"'


# We only need to install packages not listed in this file already:
# https://github.com/pangeo-data/pangeo-docker-images/blob/master/pangeo-notebook/packages.txt
RUN echo "Installing conda packages..." \
 && mamba install -n ${CONDA_ENV} -y \
        #
        # visualization:
        altair \
        bqplot \
        plotly \
        python-kaleido \
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
        jupyter-repo2docker \
        jupyterlab-link-share \
            # ref: https://github.com/jupyterlab-contrib/jupyterlab-link-share
        jupyterlab-git \
        jupyterlab-system-monitor \
        jupyterlab-variableinspector \
        nbdime \
        retrolab \
        #
        # other
        ipydrawio \
            # a drawio IDE launchable from jupyterlab's launcher
        compilers \
        cxx-compiler \
        cython \
        fortran-magic \
        google-cloud-sdk \
        sympy \
 && echo "Installing conda packages complete!"


# We use a conda first approach in this Dockerfile, so only install pip packages
# if you have a clear reason to not use conda.
# https://github.com/pangeo-data/pangeo-docker-images/blob/master/pangeo-notebook/packages.txt
#
RUN echo "Installing pip packages..." \
 && export PATH=${NB_PYTHON_PREFIX}/bin:${PATH} \
 && pip install --no-cache-dir \
        julia \
            # To enable doing Julia stuff from Python
            # ref: https://pyjulia.readthedocs.io/en/latest/index.html
        jupyterlab==3.1.0a12 \
            # ref: https://github.com/jupyterlab/jupyterlab
        plotly-geo \
            # NOTE: This package is not available in conda (conda-forge or
            #       plotly), even though they describe it to be.
            # ref: https://github.com/plotly/plotly.py#extended-geo-support
 && echo "Installing pip packages complete!"


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
