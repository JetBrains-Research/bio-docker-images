# Docker Image with Trinity + Snakemake

This image is based on official Trinity docker image with addition of miniconda and snakemake.

We use it as temporary solution to get Trinity 2.10.0 in our LSF jobs submitted via snakemake pipeline

# Build

Doesn't triggered automatically
```shell script
cd biolabs/trinity

docker build . -t biolabs/snakemake
```

Please use:
* Git tags naming conventions: 
```shell script
 git tag -a trinity2.10.0_snakemake5.13.0_py37_v1
````

* Docker image naming conventions:
```shell script
 docker tag <image id> biolabs/trinity:2.10.0_snakemake_5.13.0_py37

 docker push biolabs/trinity:2.10.0_snakemake_5.13.0_py37
```