# References regarding our base image:
# - ubuntu:22.04
# - pangeo/base-image definition:       https://github.com/pangeo-data/pangeo-docker-images/blob/master/base-image
# - pangeo/pytorch-notebook definition: https://github.com/pangeo-data/pangeo-docker-images/tree/master/pytorch-notebook
# - pangeo/pytorch-notebook tags:       https://hub.docker.com/r/pangeo/pytorch-notebook/tags
# - pytorch-notebook conda package:     https://github.com/conda-forge/pytorch-notebook-feedstock/blob/master/recipe/meta.yaml
#
FROM pangeo/pytorch-notebook:master

# While NB_GID is often defined in these jupyter images, it isn't for
# pangeo/base-image and derivative images. Let's define it here so copy pasting
# various Dockerfile snippets will work.
ENV NB_GID=$NB_UID

USER root
# We only need to install packages not listed in this file already:
# https://github.com/pangeo-data/pangeo-docker-images/blob/master/pytorch-notebook/apt.txt
RUN echo "Installing apt-get packages..." \
 && apt-get -y update > /dev/null \
 && apt-get -y install \
        curl \
        emacs-nox emacs-goodies-el \
            # Basic Emacs configuration for general development.
            #
            # python-mode was part of this list, but removed as it was not
            # available for ubuntu 22.04 that is now used.
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
        unrar \
            # zipp is installed in the base image, but I think zipp provides
            # `zip` and `unzip` which doesn't help us decompress .rar files. See
            # https://github.com/pangeo-data/pangeo-docker-images/issues/366 for
            # an issue about installing this in the base image.
        # /desktop part 1: install desktop UI via apt
        #
            dbus-x11 \
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
 && apt-get -y remove \
        xfce4-screensaver \
            # /desktop part 2: xfce4-screensaver removal
            #
            # There is no great option to avoid installing it while installing
            # other xfce4 packages above it seems.
            #
    > /dev/null \
    # chown $HOME to workaround that the xorg installation creates a
    # /home/jovyan/.cache directory owned by root
 && chown -R $NB_UID:$NB_GID $HOME \
 && rm -rf /var/lib/apt/lists/*

# /desktop part 3: firefox as default browswer
#
RUN sed -i 's/WebBrowser=.*/WebBrowser=firefox/' /etc/xdg/xfce4/helpers.rc


# Install visual studio code-server
# ref: https://github.com/coder/code-server
#
RUN export VERSION=4.9.0 \
 && curl -fsSL https://code-server.dev/install.sh | sh \
 && rm -rf "${HOME}/.cache"


# qgis, used by geographers, display gis data
# ref: https://qgis.org/en/site/forusers/alldownloads.html#debian-ubuntu
RUN wget -qO - https://qgis.org/downloads/qgis-2022.gpg.key \
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
ARG TURBOVNC_VERSION=3.0.2
RUN wget -q "https://sourceforge.net/projects/turbovnc/files/${TURBOVNC_VERSION}/turbovnc_${TURBOVNC_VERSION}_amd64.deb/download" -O turbovnc.deb \
 && apt-get install -y ./turbovnc.deb > /dev/null \
 && rm ./turbovnc.deb \
 && ln -s /opt/TurboVNC/bin/* /usr/local/bin/


# Install Julia itself (Julia part 1/2)
#
# NOTE: Needs to be followed up by installing the Julia kernel in a location not
#       overridden by mounting the user storage as done below when we are no
#       longer acting as root.
#
# NOTE: The following issue was observed, and we added the workaround of copying
#       libstdc++.so.6 from the system to the julia directory:
#       https://github.com/pangeo-data/jupyter-earth/issues/126.
#
#       I've concluded that julia declares a version of CSL
#       (CompilerSupportLibraries) as defined here
#       https://github.com/JuliaLang/julia/blob/35ac6e1823a2145fdbd8273d62cc19f10bde3543/stdlib/CompilerSupportLibraries_jll/Project.toml#L4-L7,
#       and that CSL version ships with stdlibc++.so.6 of a certain version. The
#       version can be found by inspecting the .tar files in the releases of CSL
#       as defined in
#       https://github.com/JuliaBinaryWrappers/CompilerSupportLibraries_jll.jl/releases.
#
#       At this point in time, Julia 1.7.3 and 1.8.0-beta.3 is referencing CSL
#       0.5.2, and only CSL 0.6.0+ included stdlibc++.so.6 version 6.0.30 as
#       required. Due to that, we remove the stdlibc++.so.6 symlink and the
#       stdlibc++.so.6.0.29 file provided by the julia installation currently
#       and copy the more modern ubuntu 22.04 symlink to 6.0.30.
#
#       There was one open PR to bump to a modern version of CSL to 0.6.1 that
#       that has stdlibc++.so.6.0.30 see https://github.com/JuliaLang/julia/pull/45582.
#
# Latest Julia version at https://julialang.org/downloads/
#
ARG JULIA_VERSION=1.8.1
ARG JULIA_PATH=/srv/julia
ENV JULIA_DEPOT_PATH=${JULIA_PATH}/pkg
ENV PATH=$PATH:${JULIA_PATH}/bin
RUN mkdir -p ${JULIA_PATH} \
 && curl -sSL "https://julialang-s3.julialang.org/bin/linux/x64/${JULIA_VERSION%[.-]*}/julia-${JULIA_VERSION}-linux-x86_64.tar.gz" \
  | tar -xz -C ${JULIA_PATH} --strip-components 1 \
 && mkdir -p ${JULIA_DEPOT_PATH} \
 && chown ${NB_UID}:${NB_UID} ${JULIA_DEPOT_PATH} \
 && rm $JULIA_PATH/lib/julia/libstdc++.so.6* \
 && cp /usr/lib/x86_64-linux-gnu/libstdc++.so.6 $JULIA_PATH/lib/julia/


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
# Make additional julia packages be installed in the home folder while retaining
# access to julia packages installed outside the home folder as well.
#
ENV JULIA_DEPOT_PATH=$HOME/.julia/pkg:$JULIA_DEPOT_PATH

# We only need to install packages not listed in this file already:
# https://github.com/pangeo-data/pangeo-docker-images/blob/master/pytorch-notebook/packages.txt
RUN echo "Installing conda packages..." \
 && mamba install -n ${CONDA_ENV} -y \
        # temporary upgrades, because sometimes we wish to have a more modern
        # version than installed in the base image.
        #
        # visualization:
        altair \
            # https://anaconda.org/conda-forge/altair
            # https://github.com/conda-forge/altair-feedstock/blob/main/recipe/meta.yaml
        bqplot \
            # https://anaconda.org/conda-forge/bqplot
            # https://github.com/conda-forge/bqplot-feedstock/blob/main/recipe/meta.yaml
        plotly \
            # https://anaconda.org/conda-forge/plotly
            # https://github.com/conda-forge/plotly-feedstock/blob/main/recipe/meta.yaml
        python-kaleido \
            # https://anaconda.org/conda-forge/python-kaleido
            # https://github.com/conda-forge/python-kaleido-feedstock/blob/main/recipe/meta.yaml
        seaborn \
            # https://anaconda.org/conda-forge/seaborn
            # https://github.com/conda-forge/seaborn-feedstock/blob/main/recipe/meta.yaml
        ipycanvas \
            # https://anaconda.org/conda-forge/ipycanvas
            # https://github.com/conda-forge/ipycanvas-feedstock/blob/main/recipe/meta.yaml
        ipympl \
            # https://anaconda.org/conda-forge/ipympl
            # https://github.com/conda-forge/ipympl-feedstock/blob/main/recipe/meta.yaml
        jupyter_bokeh \
            # https://anaconda.org/conda-forge/jupyter_bokeh
            # https://github.com/conda-forge/jupyter_bokeh-feedstock/blob/main/recipe/meta.yaml
        jupyterlab-geojson \
            # https://anaconda.org/conda-forge/jupyterlab-geojson
            # https://github.com/conda-forge/jupyterlab-geojson-feedstock/blob/main/recipe/meta.yaml
        #
        # common geospatial dependencies:
        qgis \
            # https://anaconda.org/conda-forge/qgis
            # https://github.com/conda-forge/qgis-feedstock/blob/main/recipe/meta.yaml
            #
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
            # FIXME: Installing qgis downgrades pytorch and lots of other files,
            #        causing a 1 GB download. The reason is unclear.
            #
            # NOTE: qgis depends on libspatialindex, geos, proj, but not
            #       proj-data. Fernando added these as apt packages initially.
            #
        proj-data \
            # https://anaconda.org/conda-forge/proj-data
            # https://github.com/conda-forge/proj-data-feedstock/blob/main/recipe/meta.yaml
        #
        # tests and formatting:
        black \
            # https://anaconda.org/conda-forge/black
            # https://github.com/conda-forge/black-feedstock/blob/main/recipe/meta.yaml
        flake8 \
            # https://anaconda.org/conda-forge/flake8
            # https://github.com/conda-forge/flake8-feedstock/blob/main/recipe/meta.yaml
        pep8 \
            # https://anaconda.org/conda-forge/pep8
            # https://github.com/conda-forge/pep8-feedstock/blob/main/recipe/meta.yaml
        pyflakes \
            # https://anaconda.org/conda-forge/pyflakes
            # https://github.com/conda-forge/pyflakes-feedstock/blob/main/recipe/meta.yaml
        pylint \
            # https://anaconda.org/conda-forge/pylint
            # https://github.com/conda-forge/pylint-feedstock/blob/main/recipe/meta.yaml
        pytest \
            # https://anaconda.org/conda-forge/pytest
            # https://github.com/conda-forge/pytest-feedstock/blob/main/recipe/meta.yaml
        pytest-cov \
            # https://anaconda.org/conda-forge/pytest-cov
            # https://github.com/conda-forge/pytest-cov-feedstock/blob/main/recipe/meta.yaml
        #
        # documentation:
        # jupyter-book \
              # NOTE: jupyter-book is disabled as its providing upper bounds on
              #       its dependencies, and that has led to ipywidgets 8 was
              #       held back.
              #
              #       https://github.com/executablebooks/jupyter-book/issues/1898.
              #
              # https://anaconda.org/conda-forge/jupyter-book
              # https://github.com/conda-forge/jupyter-book-feedstock/blob/main/recipe/meta.yaml
        jupytext \
            # https://anaconda.org/conda-forge/jupytext
            # https://github.com/conda-forge/jupytext-feedstock/blob/main/recipe/meta.yaml
        numpydoc \
            # https://anaconda.org/conda-forge/numpydoc
            # https://github.com/conda-forge/numpydoc-feedstock/blob/main/recipe/meta.yaml
        sphinx \
            # https://anaconda.org/conda-forge/sphinx
            # https://github.com/conda-forge/sphinx-feedstock/blob/main/recipe/meta.yaml
        #
        # data:
        ipydatagrid \
            # https://anaconda.org/conda-forge/ipydatagrid
            # https://github.com/conda-forge/ipydatagrid-feedstock/blob/main/recipe/meta.yaml
        ipyparallel \
            # https://anaconda.org/conda-forge/ipyparallel
            # https://github.com/conda-forge/ipyparallel-feedstock/blob/main/recipe/meta.yaml
        lxml \
            # https://anaconda.org/conda-forge/lxml
            # https://github.com/conda-forge/lxml-feedstock/blob/main/recipe/meta.yaml
        pyhdf \
            # https://anaconda.org/conda-forge/pyhdf
            # https://github.com/conda-forge/pyhdf-feedstock/blob/main/recipe/meta.yaml
        vaex \
            # https://anaconda.org/conda-forge/vaex
            # https://github.com/conda-forge/vaex-feedstock/blob/main/recipe/meta.yaml
        mhealpy \
            # https://anaconda.org/conda-forge/mhealpy
            # https://github.com/conda-forge/mhealpy-feedstock/blob/main/recipe/meta.yaml
        pytables \
            # https://anaconda.org/conda-forge/pytables
            # https://github.com/conda-forge/pytables-feedstock/blob/main/recipe/meta.yaml
        statsmodels \
            # https://anaconda.org/conda-forge/statsmodels
            # https://github.com/conda-forge/statsmodels-feedstock/blob/main/recipe/meta.yaml
        xlrd \
            # https://anaconda.org/conda-forge/xlrd
            # https://github.com/conda-forge/xlrd-feedstock/blob/main/recipe/meta.yaml
        jupyter-repo2docker \
            # https://anaconda.org/conda-forge/jupyter-repo2docker
            # https://github.com/conda-forge/jupyter-repo2docker-feedstock/blob/main/recipe/meta.yaml
        #
        # IDE:
        jupyter-vscode-proxy \
            # https://anaconda.org/conda-forge/jupyter-vscode-proxy
            # https://github.com/conda-forge/jupyter-vscode-proxy-feedstock/blob/main/recipe/meta.yaml
            # NOTE: Requires code-server to be installed.
            # https://pypi.org/project/jupyter-vscode-proxy/
        jupyterlab-link-share \
            # https://anaconda.org/conda-forge/jupyterlab-link-share
            # https://github.com/conda-forge/jupyterlab-link-share-feedstock/blob/main/recipe/meta.yaml
            # ref: https://github.com/jupyterlab-contrib/jupyterlab-link-share
        jupyterlab-git \
            # https://anaconda.org/conda-forge/jupyterlab-git
            # https://github.com/conda-forge/jupyterlab-git-feedstock/blob/main/recipe/meta.yaml
        jupyterlab-system-monitor \
            # https://anaconda.org/conda-forge/jupyterlab-system-monitor
            # https://github.com/conda-forge/jupyterlab-system-monitor-feedstock/blob/main/recipe/meta.yaml
        jupyterlab-favorites \
            # https://anaconda.org/conda-forge/jupyterlab-favorites
            # https://github.com/conda-forge/jupyterlab-favorites-feedstock/blob/main/recipe/meta.yaml
        nbdime \
            # https://anaconda.org/conda-forge/nbdime
            # https://github.com/conda-forge/nbdime-feedstock/blob/main/recipe/meta.yaml
        gh-scoped-creds \
            # https://anaconda.org/conda-forge/gh-scoped-creds
            # https://github.com/conda-forge/gh-scoped-creds-feedstock/blob/main/recipe/meta.yaml
            #
            # Additional setup instructions: https://github.com/yuvipanda/gh-scoped-creds#installation
            # Additional setup done: https://github.com/2i2c-org/infrastructure/commit/5a9f69b11727965fd4f07571c03eb65de5279fa4
            # Related issue: https://github.com/pangeo-data/jupyter-earth/issues/96
        retrolab \
            # https://anaconda.org/conda-forge/retrolab
            # https://github.com/conda-forge/retrolab-feedstock/blob/main/recipe/meta.yaml
        ipydrawio \
            # a drawio IDE launchable from jupyterlab's launcher
            # https://anaconda.org/conda-forge/ipydrawio
            # https://github.com/conda-forge/ipydrawio-feedstock/blob/main/recipe/meta.yaml
        cython \
        fortran-magic \
            # https://anaconda.org/conda-forge/fortran-magic
            # https://github.com/conda-forge/fortran-magic-feedstock/blob/main/recipe/meta.yaml
        google-cloud-sdk \
            # https://anaconda.org/conda-forge/google-cloud-sdk
            # https://github.com/conda-forge/google-cloud-sdk-feedstock/blob/main/recipe/meta.yaml
        sympy \
            # https://anaconda.org/conda-forge/sympy
            # https://github.com/conda-forge/sympy-feedstock/blob/main/recipe/meta.yaml
        # Storage related
        #
        syncthing \
            # https://anaconda.org/conda-forge/syncthing
            # We also install jupyter-syncthing-proxy from pip.
        #
        # /desktop part 4: websockify and firefox
        websockify \
            # https://anaconda.org/conda-forge/websockify
            # optional performance benefit for jupyter-remote-desktop-proxy
        firefox \
            # https://anaconda.org/conda-forge/firefox
            # optional good to have for use with jupyter-remote-desktop-proxy,
            # note that we also set it as a default in a command below.
            #
 && echo "Installing conda packages complete!"


# We use a conda first approach in this Dockerfile, so only install pip packages
# if you have a clear reason to not use conda.
# https://github.com/pangeo-data/pangeo-docker-images/blob/master/pytorch-notebook/packages.txt
#
RUN echo "Installing pip packages..." \
 && export PATH=${NB_PYTHON_PREFIX}/bin:${PATH} \
 && pip install --no-cache-dir \
        https://github.com/jupyterhub/jupyter-remote-desktop-proxy/archive/main.zip \
            # /desktop part 5: jupyter-remote-desktop-proxy
            #
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
        https://github.com/minrk/jupyter-keepalive/archive/main.zip \
            # This is a jupyter_server extension that is controllable via a
            # JupyterLab plugin to keep a server running.
            #
            # ref: https://github.com/minrk/jupyter-keepalive
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
        # jupyter-datasette-proxy \
            # FIXME: this is disabled because it malfunctions, and when it does,
            #        all other jupyter-server-proxy processes fail and we
            #        observe for example the vscode launcher entry in the
            #        jupyterlab UI disappears.
            # This package is not available in conda
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
