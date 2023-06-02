# References regarding our base image:
# - ubuntu:22.04
# - pangeo/base-image definition:       https://github.com/pangeo-data/pangeo-docker-images/blob/master/base-image
# - pangeo/pytorch-notebook definition: https://github.com/pangeo-data/pangeo-docker-images/tree/master/pytorch-notebook
# - pangeo/pytorch-notebook tags:       https://hub.docker.com/r/pangeo/pytorch-notebook/tags
# - pytorch-notebook conda package:     https://github.com/conda-forge/pytorch-notebook-feedstock/blob/master/recipe/meta.yaml
#
FROM pangeo/pytorch-notebook:2023.05.18

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
RUN export VERSION=4.9.1 \
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
ARG JULIA_VERSION=1.9.0
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

# Install extra packages not present in base env
ADD environment.yml /tmp/environment.yml
RUN mamba env update -n ${CONDA_ENV} -y -f /tmp/environment.yml &&
        && find ${CONDA_DIR} -follow -type f -name '*.a' -delete \
        && find ${CONDA_DIR} -follow -type f -name '*.js.map' -delete
        
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
