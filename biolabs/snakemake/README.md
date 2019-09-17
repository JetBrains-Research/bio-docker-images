Docker Image with snakemake support
=====================================

This image provides conda+snakemake with some tweaks using to use it with LSF submission system. 
* Image based on [continuumio/miniconda3](https://hub.docker.com/r/continuumio/miniconda3/dockerfile), Debian linux 
* latest snakemake installed 
* cmd_wrapper.sh script, that allows customizing image environment before pipeline execution on a cluster with LSF and submission using docker

Build
-----
Change directory to `./docker/biolabs/washu` 

```bash
docker build -t biolabs/snakemake .
```

Tag
---
Check latest `biolabs/snakemake` image ID  
```bash
docker images
```
and use it to tag your image with `tag_name` tag, e.g. `snakemake_5.5.4`
```bash                                                         
docker tag <image id, e.g. bb38976d03cf> `biolabs/snakemake:<tag_name>
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