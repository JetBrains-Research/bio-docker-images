FROM continuumio/miniconda3:latest

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
RUN apt-get install -y --no-install-recommends libfontconfig1

#  GNU/GCC compiler and related tools (such as make, debugger, man pages) collection
RUN apt-get install -y build-essential autoconf automake

# collection of debian utility tools, e.g. 'column', etc
RUN apt-get install -y bsdmainutils

# Install additional tools: ps, htop, vim, mc
RUN apt-get install -y procps htop vim mc nano tmux screen tree

RUN apt-get install -y locate

# --------------------------------------------------------
# Additional tools / libs for bioinformatics:
#
# The GNU Scientific Library
RUN apt-get install -y libgsl-dev

#-----------
# Install methpipe-3.4.3
RUN mkdir -p /opt && cd /opt && \
   wget http://smithlabresearch.org/downloads/methpipe-3.4.3.tar.bz2 && \
   tar -xvf methpipe-3.4.3.tar.bz2  && \
   cd methpipe-3.4.3 && \
   make all && make install && \
   rm -rf /opt/methpipe-3.4.3.tar.bz2

# Install HTSlib: https://github.com/samtools/htslib
RUN apt-get install -y perl zlib1g-dev libbz2-dev liblzma-dev libcurl4-gnutls-dev libssl-dev
#RUN apt-get install -ay libbz2-dev zlib1g-dev libncurses5-dev libncursesw5-dev liblzma-dev
RUN cd /usr/bin && \
    wget https://github.com/samtools/htslib/releases/download/1.11/htslib-1.11.tar.bz2 && \
    tar -vxjf htslib-1.11.tar.bz2 && cd /usr/bin/htslib-1.11 && \
    autoreconf && \
    ./configure && make && make install && \
    rm -rf /usr/bin/htslib-1.11

#-----------
# Installing methpipe-4.1.1
# smithlab_cpp lib:
RUN mkdir -p /opt && cd /opt && \
   wget https://github.com/smithlabcode/smithlab_cpp/releases/download/1.0.2/libsmithlab_cpp-1.0.2.tar.gz && \
   tar -xvf libsmithlab_cpp-1.0.2.tar.gz && \
   cd libsmithlab_cpp-1.0.2 && \
   ./configure --enable-hts && make && make install && \
   rm -rf /opt/libsmithlab_cpp-1.0.2*

# methpipe
RUN mkdir -p /opt && cd /opt && \
  wget https://github.com/smithlabcode/methpipe/releases/download/4.1.1/methpipe-4.1.1.tar.gz && \
   tar -zxvf methpipe-4.1.1.tar.gz && \
   cd methpipe-4.1.1 && mkdir -p build && cd build && \
   ../configure --prefix="$(dirname $(pwd))"  && make &&  make install && \
   rm -rf /opt/methpipe-4.1.1.tar.gz

# ---------
# Install xvfb & Co for plot.ly (orca)
# RUN apt-get install -y software-properties-common
RUN apt-get install -y xvfb libgtk2.0-0 libxtst-dev libxss1 gconf-gsettings-backend libnss3 libasound2

#-----------
# singularity
RUN apt-get install -y dh-autoreconf libarchive-dev squashfs-tools

WORKDIR /opt
ENV SING_VER=2.5.2
RUN wget https://github.com/singularityware/singularity/releases/download/$SING_VER/singularity-$SING_VER.tar.gz && \
   tar xvf singularity-$SING_VER.tar.gz  && \
   cd singularity-$SING_VER && \
   ./configure --prefix=/usr/local --sysconfdir=/etc && \
   make &&  make install

# --------------------------------------------------------
# --------------------------------------------------------
# --------------------------------------------------------

# Cleanup apt caches
RUN rm -rf /var/lib/apt/lists/*

# --------------------------------------------------------
# --------------------------------------------------------
# --------------------------------------------------------

# Install snakemake and other packages
COPY environement.yaml /root/environement.yaml
RUN conda update  --yes -n base -c defaults conda setuptools
RUN conda env update -n base --file /root/environement.yaml
RUN conda clean   --yes --all
RUN rm /root/environement.yaml

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
