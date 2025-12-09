
---
weight: 1
title: "Ch 4: Docker"
bookToc: true
type: "docs"
bookFlatSection: true
---

# Docker

Containerization is awesome.

I want to build and host my own Docker images.

## Docker Engine

Also known as "Docker CE", this is the free, open-source version of Docker.

It installs in less than 1 minute following the [debian installation guide](https://docs.docker.com/engine/install/debian/).

Systemd begins running it automatically.

![docker running](/docker_running.png)

I verify the 'hello world' image runs successfully.

![docker hello world](/docker_hello_world.png)

## Container Registry

Images can be built and ran on the same machine for local dev.

However, I'll soon want to run my own images on a self-hosted kubernetes cluster.

Therefore, its better to use a container registry.

A registry acts like centralized catalog where I can push/pull images to/from a remote machine.

There are several major 3rd party registries out there like:
- [Docker Hub](https://hub.docker.com/)
- [Amazon ECR](https://aws.amazon.com/ecr/)
- [Google Cloud Artifact](https://cloud.google.com/artifact-registry/docs)

However, I can DIY this!

## Self-hosted Registry

Docker has an officially supported ["registry"](https://hub.docker.com/_/registry) image for self-hosting.

The docs include [setup instructions for using an nginx reverse proxy](https://distribution.github.io/distribution/recipes/nginx/) to handle TLS.

TLS is a security requirement for the docker client if it wants to use a registry on a remote machine.

### Docker client TLS requirement

As a security requirement, the docker client will refuse to connect to any registry that isn't using TLS.

Localhost access on the same machine is an exception to this rule, 

but it won't help me when connecting from a different machine across my home network.

I follow the recipe for an nginx proxy 
