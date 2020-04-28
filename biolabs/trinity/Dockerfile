FROM trinityrnaseq/trinityrnaseq

MAINTAINER Roman Chernyatchik, JetBrains Research <roman.cheryatchik@jetbrains.com>

# docker build -t biolabs/trinity .
#  docker run --rm -it biolabs/trinity

RUN wget --quiet https://repo.anaconda.com/miniconda/Miniconda3-4.5.11-Linux-x86_64.sh -O ~/miniconda.sh && \
    /bin/bash ~/miniconda.sh -b -p /opt/conda && \
    rm ~/miniconda.sh && \
    /opt/conda/bin/conda clean -tipsy && \
    ln -s /opt/conda/etc/profile.d/conda.sh /etc/profile.d/conda.sh && \
    echo ". /opt/conda/etc/profile.d/conda.sh" >> ~/.bashrc && \
    echo "conda activate base" >> ~/.bashrc

# Install snakemake and other packages
COPY environement.yaml /root/environement.yaml
RUN /opt/conda/bin/conda update  --yes -n base -c defaults conda setuptools
RUN /opt/conda/bin/conda env update -n base --file /root/environement.yaml
RUN /opt/conda/bin/conda clean   --yes --all
RUN rm /root/environement.yaml