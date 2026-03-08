# Docker (base) images for Azure DevOps Container Jobs

Yaml pipelines enable uses to easily containerize their [jobs](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/container-phases?view=azure-devops&tabs=linux).

This comes with a set of [requirements](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/container-phases?view=azure-devops&tabs=linux#additional-container-requirements), which is mostly about
being able to run Node.js to execute the tasks for the job. The container does not need to have Node.js installed. The hosted agent will supply a suitable version of Node.js using [NodeSource](https://github.com/nodesource/distributions).

This however comes at a price for older operating systems and/or non-glibc-based systems. In such cases it is more stable or required to
[pre-install Node.js](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/container-phases?view=azure-devops&tabs=linux#nonglibc-based-containers) and
tell the hosted agent where to find the Node.js installation.

## Supported Node.js versions

The beginning support of Node.js 24 puts some burden on older OSes like Ubuntu18.04.

https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/nodejs-runners?view=azure-devops#nodejs-version-support

### Ubuntu 18.04

The glibc version is too old and building from code requires Python 3.9 and GCC 12. 
This image uses ppa:savoury1/gcc-defaults-12 to get a compatible GCC version which
is able to compile both Python and Node from source.

This additional ppa is kept in the configuration so derived images wil get easy access to GCC 12 as well.

### Alpine 3.18

The provided installers are not compatible but a regular source build works.

## Disclaimer

Images are published to ghcr.io/smkanadl/ado-base-images. Use at your own risk!

The process of creating the images is heavily inspired by the [official nodejs docker repository](https://github.com/nodejs/docker-node).
Those images are not used directly as they 1) have an entry point and also do not support a wide range of older OS versions.
