Docker Compose Build & Push
---
A very simple GitHub Action that builds, tag and push images from docker compose file.

## Features
- support multiple docker-compose files. e.g. using a specific override docker compose file for CI   
- support passing multiple `--profile` flags
- support passing desired service names 

## Requirements
All desired services on docker compose file should include both `build` and `image` properties.    
the `build` property determines how to build an image while the `image` property will be used to determine the image tag. it must include registry, name and version of the image. the registery part should match with the url of intended registery.    

Also, you are responsible to login into docker registry before calling this action, you can use official docker login action
```yaml
# Login against a Docker registry
# https://github.com/docker/login-action
- name: Log into registry ${{ env.REGISTRY }}
  uses: docker/login-action@343f7c4344506bcbf9b4de18042ae17996df046d # v3.0.0
  with:
    registry: ghcr.io
    username: ${{ github.actor }}
    password: ${{ secrets.GITHUB_TOKEN }}
```
Examples
---
## Simple usage
given  we have Hard-Coded image name and version in our docker-compose.yml.   
Note that `ghcr.io` should match the registry you have logged into
```yaml
services:
  foo:
    image: "ghcr.io/alice/awesome-github/foo:v1"
    build:
      context: .
      dockerfile: Dockerfile
  bar:
    image: "ghcr.io/alice/awesome-github/bar:v1"
    build:
      context: .
      dockerfile: docker/bar/Dockerfile
```
in your workflow

```yaml
- name: publish
  uses: 0x4r45h/docker-compose-ci@v0.1.0
```

## Multiple compose files, profiles and services
the default compose file is `docker-compose.yml` and default profile is `--profile '*'`, mean it will try to build every available profiles.
default value for services is empty which means all, you select only a few services to build.   
you can override them like below
```yaml
- name: publish
  uses: 0x4r45h/docker-compose-ci@v0.1.0
  with:
    docker_compose_files: '-f docker-compose.yml -f docker-compose.ci.yml'
    profiles: "--profile alpha --profile beta"
    services: "foo baz"
```
### Using ENV Variables for images
```yaml
  foo:
    image: ${IMAGE_REGISTRY}/${FOO_IMAGE_NAME}:${FOO_IMAGE_VERSION}
    build:
      context: .
      dockerfile: Dockerfile
```
your workflow should set these ENV variables
```yaml
name: Docker
on:
  push: [main]
env:
  IMAGE_REGISTRY: "ghcr.io"
  FOO_IMAGE_NAME: ${{ github.repository }}/foo
  FOO_IMAGE_VERSION: "v2"
jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - name: Log into registry ${{ env.IMAGE_REGISTRY }}
        uses: docker/login-action@343f7c4344506bcbf9b4de18042ae17996df046d # v3.0.0
        with:
          registry: ${{ env.IMAGE_REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: publish
        uses: 0x4r45h/docker-compose-ci@v0.1.0
```
