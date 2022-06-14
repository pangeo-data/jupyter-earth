# References regarding our base image:
# - ubuntu:20.04
# - pangeo/base-image definition:       https://github.com/pangeo-data/pangeo-docker-images/blob/master/base-image
# - pangeo/pangeo-notebook definition:  https://github.com/pangeo-data/pangeo-docker-images/tree/master/pangeo-notebook
# - pangeo/pangeo-notebook tags:        https://hub.docker.com/r/pangeo/pangeo-notebook/tags
# - pangeo-notebook conda package:      https://github.com/conda-forge/pangeo-notebook-feedstock/blob/master/recipe/meta.yaml
#
FROM pangeo/pangeo-notebook:master
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
        # pymc3-theano relies on compiling some C++ code, this may be relevant
        # to help that compilation succeed. See this issue for details:
        # https://github.com/pangeo-data/jupyter-earth/issues/104#issuecomment-1027210956
        libc-dev \
    > /dev/null \
    # chown $HOME to workaround that the xorg installation creates a
    # /home/jovyan/.cache directory owned by root
 && chown -R $NB_UID:$NB_GID $HOME \
 && rm -rf /var/lib/apt/lists/*


# Install visual studio code-server
# ref: https://github.com/cdr/code-server
RUN curl -fsSL https://code-server.dev/install.sh | sh \
 && rm -rf "${HOME}/.cache"


# qgis, used by geographers, display gis data
# ref: https://qgis.org/en/site/forusers/alldownloads.html#debian-ubuntu
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


# Install Julia itself (Julia part 1/2)
#
# NOTE: Needs to be followed up by installing the Julia kernel in a location not
#       overridden by mounting the user storage as done below when we are no
#       longer acting as root.
#
# Latest version at https://julialang.org/downloads/
#
ENV JULIA_VERSION 1.7.1
ENV JULIA_PATH /srv/julia
ENV JULIA_DEPOT_PATH ${JULIA_PATH}/pkg
ENV PATH $PATH:${JULIA_PATH}/bin
RUN mkdir -p ${JULIA_PATH} \
 && curl -sSL "https://julialang-s3.julialang.org/bin/linux/x64/${JULIA_VERSION%[.-]*}/julia-${JULIA_VERSION}-linux-x86_64.tar.gz" \
  | tar -xz -C ${JULIA_PATH} --strip-components 1 \
 && mkdir -p ${JULIA_DEPOT_PATH} \
 && chown ${NB_UID}:${NB_UID} ${JULIA_DEPOT_PATH}


# Install the nix package manager, step 1/2
RUN mkdir -m 0755 /nix \
 && chown jovyan /nix


# Switch user away from ROOT for the rest
USER ${NB_USER}


# Install the nix package manager, step 2/2
RUN curl -L https://nixos.org/nix/install | sh


# Install Julia kernel (Julia part 2/2)
#
# NOTE: If we are not not setting JUPYTER_DATA_DIR, the Julia kernel gets
#       installed in ~/.local/share/jupyter/kernels, and that folder may be
#       overridden by a home directory mount. This was found out by using
#       "jupyter kernelspec list" during debugging.
#
#       Julia's installkernel function relates to JUPYTER_DATA_DIR via logic
#       defined at https://github.com/JuliaLang/IJulia.jl/blob/cc2a9bf61a2515596b177339f9a3514de8c38573/deps/kspec.jl#L32-L37
#
#       NB_PYTHON_PREFIX is presumed to be /srv/conda/envs/notebook in this
#       case, which is defined in this specific Dockerfile's base image.
#
RUN export JUPYTER_DATA_DIR="$NB_PYTHON_PREFIX/share/jupyter" \
 && julia --eval 'using Pkg; Pkg.add("IJulia"); using IJulia; installkernel("Julia");' \
 && julia --eval 'using Pkg; Pkg.instantiate(); Pkg.resolve(); pkg"precompile"'

# WARNING: Messy workaround to install pyMC3, relying on mkl, but failing
#          because of the base image has the nomkl package installed.
#          Uninstalling nomkl may have consequences, but I'm not sure about
#          them. I believe it can have big performance consequences to use or
#          not use mkl.
#
RUN echo "Removing nomkl to enable install of PyMC3" \
 && mamba uninstall -n ${CONDA_ENV} -y nomkl

# We only need to install packages not listed in this file already:
# https://github.com/pangeo-data/pangeo-docker-images/blob/master/pangeo-notebook/packages.txt
RUN echo "Installing conda packages..." \
 && mamba install -n ${CONDA_ENV} -y \
        # temporary upgrades, because sometimes we wish to have a more modern
        # version than installed in the base image.
        #
        "jupyterlab>=3.4.3" \
        "dask-gateway>=2022.6.1" \
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
        # pymc3 and dependencies start
        # - WARNING: we had to remove nomkl to be able to install this, see note
        #            about that above the mamba install step.
        # - installation instructions found at: https://github.com/pymc-devs/pymc/wiki/Installation-Guide-(Linux)
        # - installed for Abby and Facu following a slack discussion
        pymc3 \
        theano-pymc \
        mkl \
        mkl-service \
        # pymc3 and dependencies end
        ipydatagrid \
        ipyparallel \
        lxml \
        pyhdf \
        vaex \
        mhealpy \
        pytables \
        statsmodels \
        xlrd \
        jupyter-repo2docker \
        #
        # IDE:
        jupyter-vscode-proxy \
            # NOTE: Requires code-server to be installed.
            # https://pypi.org/project/jupyter-vscode-proxy/
        jupyterlab-link-share \
            # ref: https://github.com/jupyterlab-contrib/jupyterlab-link-share
        jupyterlab-git \
        jupyterlab-system-monitor \
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
        cxx-compiler \
        cython \
        fortran-magic \
        google-cloud-sdk \
        sympy \
        # Storage related
        #
        syncthing \
            # ref: https://anaconda.org/conda-forge/syncthing
            # We also install jupyter-syncthing-proxy from pip.
        #
        # GPU related
        #
        # We work against a specific NVIDIA Driver, so we should install the
        # latest cudatoolkit that supports that driver or update the driver to
        # support the cudatoolkit we want to install. Since cudatoolkit may
        # update and require more modern drivers than available, we pin this
        # explicitly to avoid such issues.
        #
        # I think the driver will be updatable by editing a DaemonSet installed
        # in the k8s cluster automatically by eksctl. Correction, I think it is
        # installed as part of the AMI (whats installed by default on the node)
        # for the machine.
        #
        # https://docs.cupy.dev/en/stable/install.html#installation
        # https://docs.nvidia.com/cuda/cuda-toolkit-release-notes/index.html
        #
        # Check latest cudatoolkit version on conda-forge, currently 11.2.2:
        # - https://anaconda.org/conda-forge/cudatoolkit
        # - https://github.com/conda-forge/cudatoolkit-feedstock
        #
        cupy \
        cudatoolkit=11.2.2 \
        #
        # temp workarounds
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
        github-app-user-auth \
            # Additional setup instructions: https://github.com/yuvipanda/github-app-user-auth#installation
            # Related issue: https://github.com/pangeo-data/jupyter-earth/issues/96
        julia \
            # To enable doing Julia stuff from Python
            # ref: https://pyjulia.readthedocs.io/en/latest/index.html
        jupyter-syncthing-proxy \
            # We install the conda-forge package syncthing along with this, as
            # this is just the glue to expose syncthing.
            #
            # ref, request:    https://github.com/pangeo-data/jupyter-earth/issues/103
            # ref, source:     https://github.com/yuvipanda/jupyter-syncthing-proxy
            # ref, dependency: https://anaconda.org/conda-forge/syncthing
        plotly-geo \
            # NOTE: This package is not available in conda (conda-forge or
            #       plotly), even though they describe it to be.
            # ref: https://github.com/plotly/plotly.py#extended-geo-support
 && echo "Installing pip packages complete!"


# TEMPORARY:
# Shane wants to trial this PR: https://github.com/dask/dask/pull/8223
#
RUN echo "Installing unreleased dask stuff..." \
 && export PATH=${NB_PYTHON_PREFIX}/bin:${PATH} \
 && pip install --no-cache-dir --upgrade \
    git+https://github.com/gjoseph92/dask@shuffle_service \
&& echo "Installing unreleased dask stuff complete!"


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
