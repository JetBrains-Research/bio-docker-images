# https://hub.docker.com/r/continuumio/miniconda3
# specify version, or use ':latest'
# (see docker file - https://hub.docker.com/r/continuumio/miniconda3/dockerfile)
# * continuumio/miniconda3:4.10.3 ~ python 3.9.5, conda 4.10.3
FROM continuumio/miniconda3:4.10.3

#  $ docker build . -t biolabs/snakemake
#  $ docker run --rm -it biolabs/snakemake /bin/bash
#  $ docker push biolabs/snakemake
MAINTAINER Roman Chernyatchik, JetBrains Research <roman.cheryatchik@jetbrains.com>

# Resolve issue with undefined user name for LDAP users, e.g remove warnings
#  group..
#  username..
RUN apt-get update && \
 apt-get upgrade -y && \
 apt-get install -y --no-install-recommends libnss-sss

# Solve locale issues when running bash.
#   /bin/bash: warning: setlocale: LC_ALL: cannot change locale (en_US.UTF-8)
#
# It breaks conda version check in snakemake:
RUN apt-get clean && apt-get update && apt-get install -y locales && \
    echo "LC_ALL=en_US.UTF-8" >> /etc/environment  && \
    echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen  && \
    echo "LANG=en_US.UTF-8" > /etc/locale.conf  && \
    locale-gen en_US.UTF-8

# JAVA 8 missing fonts workaround:
# see https://bugs.debian.org/793210
# and https://github.com/docker-library/java/issues/46#issuecomment-119026586
# P.S: our java comes from conda, seems the same issue
RUN apt-get update && apt-get install -y --no-install-recommends libfontconfig1

#  GNU/GCC compiler and related tools (such as make, debugger, man pages) collection
RUN apt-get update && apt-get install -y build-essential autoconf automake

# collection of debian utility tools, e.g. 'column', etc
RUN apt-get update && apt-get install -y bsdmainutils

# Install additional tools: ps, htop, vim, mc
RUN apt-get update && apt-get install -y procps htop vim mc nano tmux screen tree less

# Install additional dev tools: git
RUN apt-get update && apt-get install -y git

RUN apt-get update && apt-get install -y locate

# --------------------------------------------------------
# Additional tools / libs for bioinformatics:
#
# The GNU Scientific Library
RUN apt-get update && apt-get install -y libgsl-dev

#-----------
# Install methpipe-3.4.3
RUN mkdir -p /opt && cd /opt && \
   wget http://smithlabresearch.org/downloads/methpipe-3.4.3.tar.bz2 && \
   tar -xvf methpipe-3.4.3.tar.bz2  && \
   cd methpipe-3.4.3 && \
   make all && make install && \
   rm -rf /opt/methpipe-3.4.3.tar.bz2

# Install HTSlib: https://github.com/samtools/htslib
RUN apt-get update && apt-get install -y autoconf automake make gcc perl zlib1g-dev libbz2-dev liblzma-dev libcurl4-gnutls-dev libssl-dev
RUN cd /usr/bin && \
    wget https://github.com/samtools/htslib/releases/download/1.16/htslib-1.16.tar.bz2 && \
    tar -vxjf htslib-1.16.tar.bz2 && cd /usr/bin/htslib-1.16 && \
    autoreconf -i && \
    ./configure && make && make install && \
    rm -rf /usr/bin/htslib-1.16

#-----------
# Installing methpipe-5.0.1
RUN mkdir -p /opt && cd /opt && \
   wget https://github.com/smithlabcode/methpipe/releases/download/v5.0.1/methpipe-5.0.1.tar.gz && \
   tar -zxvf methpipe-5.0.1.tar.gz --no-same-owner && \
   cd methpipe-5.0.1 && mkdir -p build && cd build && \
   ../configure --prefix="$(dirname $(pwd))"  && make &&  make install && \
   rm -rf /opt/methpipe-5.0.1.tar.gz

#-----------
# DNMTOOLS
# New MethPipe repo - https://github.com/smithlabcode/dnmtools
# Release 1.2.1
# * If your HTSlib (or some other library) is not installed system-wide, then you might need to udpate your library path:
#     $ export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:/path/to/htslib/lib
# * If you installed HTSlib yourself in some non-standard directory, you must specify the location like this:
#    $ ../configure CPPFLAGS='-I /path/to/htslib/headers' LDFLAGS='-L/path/to/htslib/lib'

RUN mkdir -p /opt && cd /opt && \
   wget https://github.com/smithlabcode/dnmtools/releases/download/v1.2.1/dnmtools-1.2.1.tar.gz && \
   tar -zxvf dnmtools-1.2.1.tar.gz --no-same-owner && \
   cd dnmtools-1.2.1 && mkdir -p build && cd build && \
   ../configure --prefix="$(dirname $(pwd))"  && make &&  make install && \
   rm -rf /opt/dnmtools-1.2.1.tar.gz

# ---------
# Install xvfb & Co for plot.ly (orca)
# RUN apt-get install -y software-properties-common
RUN apt-get update && apt-get install -y xvfb libgtk2.0-0 libxtst-dev libxss1 gconf-gsettings-backend libnss3 libasound2

# --------------------------------------------------------
# Install graphviz to get 'dot' command used for snakemake DAG plots:
RUN apt-get update && apt-get install -y graphviz


# --------------------------------------------------------
# Install curl
RUN apt-get update && apt-get install -y curl

# --------------------------------------------------------
# --------------------------------------------------------

# Cleanup apt caches
RUN rm -rf /var/lib/apt/lists/*

# --------------------------------------------------------
# --------------------------------------------------------
# --------------------------------------------------------
# Install mamba
# NB: /root/environment.yaml depends on python 3.8
RUN conda install mamba "python=3.8" -n base -c conda-forge
#RUN conda install "mamba>=0.24" "python=3.8" -n base -c conda-forge

# resolves module 'lib' has no attribute 'X509_V_FLAG_CB_ISSUER_CHECK' for pip 22.2.2 - 22.3
RUN pip3 install pyOpenSSL --upgrade

# Install snakemake and other packages
COPY environment.yaml /root/environment.yaml
RUN mamba update  --yes -n base -c defaults conda setuptools
RUN mamba env update -n base --file /root/environment.yaml
RUN mamba clean --yes --all
RUN rm /root/environment.yaml


# Fix PATH and profile when running in LSF cluster
RUN echo "PATH=/opt/conda/condabin:/opt/conda/bin:\$PATH" > /opt/conda/.bashrc && \
    cat /root/.bashrc >> /opt/conda/.bashrc

# Simulate `conda init` and write config to global bash config. It is needed for docker containers
# which are launched with custom user folder and thus lack of original ~/.bashrc settings
RUN echo "export  PATH=/opt/conda/condabin:\$PATH" > /etc/bash.bashrc && \
    echo "# >>> conda initialize >>>" >> /etc/bash.bashrc && \
    echo "__conda_setup=\"\$('/opt/conda/bin/conda' 'shell.bash' 'hook' 2> /dev/null)\"" >> /etc/bash.bashrc && \
    echo "eval \"\$__conda_setup\"" >> /etc/bash.bashrc && \
    echo "unset __conda_setup" >> /etc/bash.bashrc && \
    echo "# <<< conda initialize <<<" >> /etc/bash.bashrc

# Wrapper to pass custom source file before docker command execution, e.g. could be used
# to pass some env variables or source other config files
COPY cmd_wrapper.sh /usr/bin
RUN chmod a+x /usr/bin/cmd_wrapper.sh

# Could be overriden, e.g. run: docker run -it --entrypoint /bin/bash biolabs/snakemake
ENTRYPOINT [ "/usr/bin/cmd_wrapper.sh" ]

# E.g.: use w/o additional scripts to source:
#CMD ["/bin/bash", "-l"]

# seems enough is 'use-source  /etc/bash.bashrc'

# E.g.: Specify 1 or multiple times 'use-source' with script to source, e.g.
CMD ["use-source", "/opt/conda/.bashrc", "/bin/bash", "-l"]

# XXX: Current git version 2.20.1 is outdated, but is latest available via apt-get