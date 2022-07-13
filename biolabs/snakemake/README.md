Docker Image with snakemake support
=====================================

This image provides miniconda3+snakemake with some tweaks using to use it with LSF submission system. 
* Image based on [continuumio/miniconda3](https://hub.docker.com/r/continuumio/miniconda3/dockerfile), Debian linux 
* latest snakemake installed
* latest pandas, pathlib, cookiecutter packages, usefull for snakemake pipelines 
* cmd_wrapper.sh script, that allows customizing image environment before pipeline execution on a cluster with LSF and submission using docker

Configure
---
Update `environment.yaml`
```bash
cd biolabs/snakemake
snakedeploy update-conda-envs environment.yaml 
```

Build
-----
Change directory to `biolabs/snakemake` 

```bash
docker build -t biolabs/snakemake .
```

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

Prepare next release
---
[DockerHub](https://hub.docker.com/repository/docker/biolabs/snakemake/builds) automatically triggers builds of two types:
* On each master commit, `biolabs/snakemake:latest`
* On tag push. Tag should match pattern `/^snakemake([0-9.]+)_conda([0-9.]+)_py([0-9.]+_v[0-9]+$/`, e.g. for `5.8.2_conda4.8.0_py37_v1` `biolabs/snakemake:5.8.2_conda4.8.0_py37`

DockerHub is able to build image only from new tags, you cannot re-trigger build for an already existing tag. Although regexp doesn't work as expected, so we use `_vNNN` suffix which will be trimmed from docker build tag.

So you need to:
* Build image locally, ensure that image works ok
* GitHub:
  * Commit and Push changes to GitHub 
  * Tag build in GitHub, e.g. `git tag -a snakemake7.8.5_conda4.12.0_py38_v1 -m "Snakemake 7.8.5_conda4.12.0"`
      * List tags: `git tag -l`
      * Remove local and remote tag `git tag -d tagname` or `git push --delete origin tagname`
  * Push GitHub tags, e.g. `git push --tags`
* DockerHub:
  * Ensure that build was triggered and finished at [DockerHub](https://hub.docker.com/repository/docker/biolabs/snakemake/builds) 
  * If you need to update image for the same tag, then make & push commits, tag them with increased latest version component, e.g. `git tag -a snakemake5.13.0_conda4.8.3_py37_v2 -m "Snakemake 5.13.0_conda4.8.3 fixed"` and push tags.
  * If auto-trigger is off, push Docker Image manually:
```shell
docker build -t biolabs/snakemake .
# docker images
docker tag biolabs/snakemake biolabs/snakemake:7.8.5_conda4.12.0_py38
docker login -u biolabs
docker push biolabs/snakemake
docker push biolabs/snakemake:7.8.5_conda4.12.0_py38
```

Tag Docker Image
----
Tag release in Docker hub:
```shell
# docker images
docker tag biolabs/snakemake biolabs/snakemake:7.8.1_conda4.11_py38
```

Push Docker Image
----
Before push you have to login to docker hub first.
```bash
docker login -u biolabs
```

Then you just push current image 
```bash
docker push biolabs/snakemake
docker push  biolabs/snakemake:7.8.1_conda4.11_py38
```