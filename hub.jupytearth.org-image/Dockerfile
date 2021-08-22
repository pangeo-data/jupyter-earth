# References regarding our base image:
# - ubuntu:20.04
# - pangeo/base-image definition:       https://github.com/pangeo-data/pangeo-docker-images/blob/master/base-image
# - pangeo/pangeo-notebook definition:  https://github.com/pangeo-data/pangeo-docker-images/tree/master/pangeo-notebook
# - pangeo/pangeo-notebook tags:        https://hub.docker.com/r/pangeo/pangeo-notebook/tags
# - pangeo-notebook conda package:      https://github.com/conda-forge/pangeo-notebook-feedstock/blob/master/recipe/meta.yaml
#
# NOTE: Newer versions than 2021.07.06 bumped to ubuntu 21.10 that qgis doesn't
#       support.
FROM pangeo/pangeo-notebook:2021.07.06
ARG DEBIAN_FRONTEND=noninteractive

# While NB_GID is often defined in these jupyter images, it isn't for
# pangeo/base-image and derivative images. Let's define it here so copy pasting
# various Dockerfile snippets will work.
ENV NB_GID=$NB_UID

USER root
# We only need to install packages not listed in this file already:
# https://github.com/pangeo-data/pangeo-docker-images/blob/master/pangeo-notebook/apt.txt
RUN echo "Installing apt-get packages..." \
 && apt-get -y update > /dev/null \
 && apt-get -y install \
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
        # jupyter-remote-desktop-proxy dependencies
        dbus-x11 \
        firefox \
        xfce4 \
        xfce4-panel \
        xfce4-session \
        xfce4-settings \
        xorg \
        xubuntu-icon-theme \
        # qgis
            gnupg \
            software-properties-common \
    > /dev/null \
    # chown $HOME to workaround that the xorg installation creates a
    # /home/jovyan/.cache directory owned by root
 && chown -R $NB_UID:$NB_GID $HOME \
 && rm -rf /var/lib/apt/lists/*


# qgis, used by geographers, display gis data
RUN wget -qO - https://qgis.org/downloads/qgis-2021.gpg.key \
  | gpg --no-default-keyring --keyring gnupg-ring:/etc/apt/trusted.gpg.d/qgis-archive.gpg --import \
 && chmod a+r /etc/apt/trusted.gpg.d/qgis-archive.gpg \
 && add-apt-repository "deb https://qgis.org/ubuntu $(lsb_release -c -s) main" \
 && apt-get -y update > /dev/null \
 && apt-get -y install \
        qgis \
        qgis-plugin-grass \
    > /dev/null \
 && rm -rf /var/lib/apt/lists/*



# Install TurboVNC (https://github.com/TurboVNC/turbovnc)
ARG TURBOVNC_VERSION=2.2.6
RUN wget -q "https://sourceforge.net/projects/turbovnc/files/${TURBOVNC_VERSION}/turbovnc_${TURBOVNC_VERSION}_amd64.deb/download" -O turbovnc.deb \
 && apt-get install -y ./turbovnc.deb > /dev/null \
    # remove light-locker to prevent screen lock
 && apt-get remove -y light-locker > /dev/null \
 && rm ./turbovnc.deb \
 && ln -s /opt/TurboVNC/bin/* /usr/local/bin/


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
        #
        # IDE:
        jupyterlab \
        jupyterlab-link-share \
            # ref: https://github.com/jupyterlab-contrib/jupyterlab-link-share
        jupyterlab-git \
        jupyterlab-system-monitor \
        jupyterlab-variableinspector \
        nbdime \
        qgis \
            # We install this as an apt-get package but on startup we got errors
            # about Python integration not being available. But installing this
            # by itself didn't seem to give us the application. Installing both
            # though makes things work, but seem to cause an initial install
            # followed by a downgrade as the conda-forge version isn't as well
            # updated. It also makes the installation take ~5-10 minutes longer.
            #
            # FIXME: Install qgis in a way that provides us with a recent
            #        version, a shortcut from the desktop UI, and with Python
            #        support - without also taking 5-10 minutes more than needed
            #        to install.
            #
        retrolab \
        ipydrawio \
            # a drawio IDE launchable from jupyterlab's launcher
        #
        # other
        websockify \
            # dependency for jupyter-remote-desktop-proxy
        compilers \
        cxx-compiler \
        cython \
        fortran-magic \
        google-cloud-sdk \
        sympy \
        #
        # temp workarounds
        # - jupyter_server pin can be removed when we bump the pangeo-notebook
        #   base image so it includes this or a more recent version of
        #   jupyter_server
        jupyter_server=1.10.2 \
        notebook=6.4.3 \
 && echo "Installing conda packages complete!"


# We use a conda first approach in this Dockerfile, so only install pip packages
# if you have a clear reason to not use conda.
# https://github.com/pangeo-data/pangeo-docker-images/blob/master/pangeo-notebook/packages.txt
#
RUN echo "Installing pip packages..." \
 && export PATH=${NB_PYTHON_PREFIX}/bin:${PATH} \
 && pip install --no-cache-dir \
        https://github.com/jupyterhub/jupyter-remote-desktop-proxy/archive/main.zip \
            # jupyter-remote-desktop-proxy enables us to visit the /desktop path
            # just like we visit the /lab path. Visiting /desktop provides us
            # with an actual remote desktop experience.
            #
            # NOTE: This package is not available on conda-forge, but available
            #       on PyPI as jupyter-desktop-server I think but maybe not.
            #
            # NOTE: This install requires websockify to be installed via
            #       conda-forge. We have also installed TurboVNC for performance
            #       I think, and also various apt packages to get a desktop UI.
            #
        julia \
            # To enable doing Julia stuff from Python
            # ref: https://pyjulia.readthedocs.io/en/latest/index.html
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
