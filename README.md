# Docker (base) images for Azure DevOps Container Jobs

Yaml pipelines enable uses to easily containerize their [jobs](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/container-phases?view=azure-devops&tabs=linux).

This comes with a set of [requirements](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/container-phases?view=azure-devops&tabs=linux#additional-container-requirements), which is mostly about
being able to run Node.js to execute the tasks for the job. The container does not need to have Node.js installed. The hosted agent will supply a suitable version of Node.js using [NodeSource](https://github.com/nodesource/distributions).

This however comes at a price for older operating systems and/or non-glibc-based systems. In such cases it is more stable or required to [pre-install Node.js](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/container-phases?view=azure-devops&tabs=linux#nonglibc-based-containers) and
tell the hosted agent where to find the Node.js installation.

## Disclaimer

Images are published to ghcr.io/smkanadl/ado-base-images. Use at your own risk!

The process of creating the images is heavily inspired by the [official nodejs docker repository](https://github.com/nodejs/docker-node).
Those images are not used directly as they 1) have an entry point and also do not support a wide range of older OS versions.
