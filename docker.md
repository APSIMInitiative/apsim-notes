# APSIM Docker Images

The apsim initiative provides several official docker images, which are all hosted on dockerhub in the [apsiminititiave organisation](https://hub.docker.com/orgs/apsiminitiative). The dockerfiles for all of these images are stored in the [APSIM.Docker](https://github.com/APSIMInitiative/APSIM.Docker) repository.

The apsimng, apsimng-server, and apsimng-complete images are all built from a single dockerfile (they use many of the same build steps). There are build instructions in the dockerfile comments, but in short, you must use the `--target` argument to specify which image you want to build.

## Images

### apsiminitiative/apsimng

An image containing apsim. Updated automatically with every apsim release. [dockerfile](https://github.com/APSIMInitiative/APSIM.Docker/blob/master/NextGen/apsimng/Dockerfile) [dockerhub](https://hub.docker.com/repository/docker/apsiminitiative/apsimng)

### apsiminitiative/apsimng-server

An image containing the apsim server; used for cluster runs. Updated automatically with every apsim release. [dockerfile](https://github.com/APSIMInitiative/APSIM.Docker/blob/master/NextGen/apsimng/Dockerfile) [dockerhub](https://hub.docker.com/repository/docker/apsiminitiative/apsimng-server)

### apsiminitiative/apsimng-complete

This image contains all dependencies for building, running, and testing apsim. This includes R packages used for the builtin senstivity analysis and parameter optimisation methods. Updated automatically with every apsim release. [dockerfile](https://github.com/APSIMInitiative/APSIM.Docker/blob/master/NextGen/apsimng/Dockerfile) [dockerhub](https://hub.docker.com/repository/docker/apsiminitiative/apsimng-complete)

### apsiminitiative/apsimng-r

This image contains the R dependencies for apsim, but not apsim itself. It's updated irregularly and manually. This needs to be tested manually before being pushed to dockerhub, as the R dependencies **frequently** break. [dockerfile](https://github.com/APSIMInitiative/APSIM.Docker/tree/master/NextGen/apsimng-r) [dockerhub](https://hub.docker.com/repository/docker/apsiminitiative/apsimng-r)

### apsiminitiative/apsimng-build

Contains dependencies (e.g. .net sdk) required to build apsim and the apsim installers. Updated irregularly and manually. [dockerfile](https://github.com/APSIMInitiative/APSIM.Docker/blob/master/NextGen/apsimng-build/Dockerfile) [dockerhub](https://hub.docker.com/repository/docker/apsiminitiative/apsimng-build)

### apsiminitiative/apsimng-build-win

Dependencies for building new apsim on windows. This is used by jenkins to build the apsim windows installers. It's updated irregularly and manually. [dockerfile](https://github.com/APSIMInitiative/APSIM.Docker/blob/master/NextGen/apsimng-windows/Dockerfile) [dockerhub](https://hub.docker.com/repository/docker/apsiminitiative/apsimng-build-win)

### apsiminitiative/buildapsim

A windows image containing dependencies to build old apsim on windows. This is updated irregularly and manually. Used by jenkins for old apsim builds. There are reports that this doesn't actually work (e.g. compilation errors), but it *does* work on jenkins. Somehow. [dockerfile](https://github.com/APSIMInitiative/APSIM.Docker/blob/master/OldApsim/dockerfile) [dockerhub](https://hub.docker.com/repository/docker/apsiminitiative/buildapsim)

## Manual Updates

Some of these images must be manually updated from time to time. To do this, you will first need a dockerhub account with push access to the apsiminitiative organisation (ie you must be a member of the organisation). To update the image, you must then run (after rebuilding the image):

```bash
docker login
docker push apsiminitiative/apsimng-build:latest
```
