# Reproducible Builds

The docker image inherits from the pangeo-notebook base image, with this
directory being a fork and modification of the pangeo pytorch-notebook.
Builds are designed to be modified by the other files in this directory,
mainly conda-lock.yml, and conda-linux-64.lock which are built using the
environment.yml files and conda-lock. 

## Updating builds

Technically docker only needs environment.yml, but conda-lock.yml ensures
reproducible builds that are known to create an image and working python
environment. When the Dockerfile inherits from other images, it will build
using each image's repo environment files, cascading to this repo. However,
it's useful to be able to use the same conda lock files to build locally as
well, since that let's us ensure the same local environment as the docker
image which is being pulled as an worker--dask expects the same environment
across workers, clients, and the scheduler. To do that, we use conda-lock to
combine multiple environment yml files together when producing output lock
files, which can be used both by the docker build process and locally to
mirror the environment.

When adding or updating dependencies, first edit the environment.yml file,
then run:

```conda-lock -f environment.yml -f NBenvironment.yml -p linux-64```

This will create `conda-lock.yml` ; note that the [NBevironment.yml](https://github.com/pangeo-data/pangeo-docker-images/blob/master/pangeo-notebook/environment.yml) file may need
to be manually updated from time to time, as it is not automatically tracked.
If the above command errors, it most likely the result of a package being
added that is not in conda-forge, and needs to be moved to the pip install
section (check this using `mamba search PackageName`). 

## Installing locally

The created `conda-lock.yml` file can be used to re-create an environment using
conda-lock, but not all systems have conda-lock. The following command will
create a `conda-linux-64.lock` that can be used with a plain conda install:

```conda-lock -f NBenvironment.yml -f environment.yml -p linux-64 -k explicit --filename-template "conda-linux-64.lock"```

To build locally, upload the `conda-linux-64.lock` file and run:

```conda create -n MyEnv --file conda-linux-64.lock```

...setting MyEnv to whatever name you prefer. Once the build is complete, add
to the local notebook kernel with:

```python -m ipykernel install --user --name=MyEnv```

### To do

 [ ] Kernel install isn't persistant using above instructions
 [ ] Auto-updaing the NBenvironment.yml file could probably be automated as a
github action.
 [ ] Same with running conda-lock commands (this is what pangeo does!)
 [ ] Add links and references

