Docker Image with snakemake support
=====================================

This image provides miniconda3+snakemake with some tweaks using to use it with LSF submission system. 
* Image based on [continuumio/miniconda3](https://hub.docker.com/r/continuumio/miniconda3/dockerfile), Debian linux 
* latest snakemake installed
* latest pandas, pathlib, cookiecutter packages, usefull for snakemake pipelines 
* cmd_wrapper.sh script, that allows customizing image environment before pipeline execution on a cluster with LSF and submission using docker

Run
-----
Use with LSF docker submission application. To run container locally, use one of examples::
```bash
docker run -it biolabs/snakemake
```
```bash
docker run -it biolabs/snakemake /bin/bash 
```
```bash
docker run -it biolabs/snakemake snakemake --help  
```

To source custom bash script with additional environment settings use:
```bash
docker run -it biolabs/snakemake use-source /some/script/to/source/by/bash snakemake --help  
```

You could override default `/usr/bin/cmd_wrapper.sh` entry point, e.g.:

```bash
docker run -it --entrypoint /bin/bash biolabs/snakemake
```

If your run docker with changed user id and home directory you will not be able to install packages into this container directly, but you could always create conda environment using custom environment directory in location where your user has write access, e.g home directory.

Build
-----
Change directory to ` biolabs/snakemake` 

```bash
docker build -t biolabs/snakemake .
```

Tag
---
Check latest `biolabs/snakemake` image ID  
```bash
docker images
```
and use it to tag your image with `tag_name` tag, e.g. `5.6.0_conda4.7.12`
```bash                                                         
docker tag <image id, e.g. bb38976d03cf> biolabs/snakemake:<tag_name>
```

Push
----
Before push you have to login to docker hub first.
```bash
docker login -u biolabs
```

Then you just push current image 
```bash
docker push biolabs/snakemake
```

Prepare next release
---
[DockerHub](https://hub.docker.com/repository/docker/biolabs/snakemake/builds) automatically triggers builds of two types:
* On each master commit, `biolabs/snakemake:latest`
* On tag push. Tag should match pattern `/^snakemake([0-9.]+)_conda([0-9.]+)_v[0-9]+$/`, e.g. for `5.8.2_conda4.8.0_v1` `biolabs/snakemake:5.8.2_conda4.8.0`

DockerHub is able to build image only from new tags, you cannot re-trigger build for an already existing tag. Although regexp doesn't work as expected, so we use `_vNNN` suffix which will be trimmed from docker build tag.

So you need to:
* Build image locally, ensure that image works ok
* Commit and Push changes to GitHub 
* Tag build, e.g. `git tag -a snakemake5.8.2_conda4.8.0_v1 -m "Snakemake 5.8.2_conda4.8.0"`
* Push tags, e.g. `git push --tags`
* Ensure that build was triggered and finished at [DockerHub](https://hub.docker.com/repository/docker/biolabs/snakemake/builds) 
* If you need to update image for the same tag, then make & push commits, tag them with increased latest version component, e.g. `git tag -a snakemake5.8.2_conda4.8.0_v2 -m "Snakemake 5.8.2_conda4.8.0 fixed"` and push tags.
