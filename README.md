# github-runner-dockerfile

[![Open in DevPod!](https://devpod.sh/assets/open-in-devpod.svg)](https://devpod.sh/open#https://github.com/joshyorko/github-runner-dockerfile)

Dockerfile for the creation of a GitHub Actions runner image to be deployed dynamically. [Find the full explanation and tutorial here](https://baccini-al.medium.com/creating-a-dockerfile-for-dynamically-creating-github-actions-self-hosted-runners-5994cc08b9fb).

When running the docker image, or when executing docker compose, environment variables for repo-owner/repo-name and github-token must be included. 

Credit to [testdriven.io](https://testdriven.io/blog/github-actions-docker/) for the original start.sh script, which I slightly modified to make it work with a regular repository rather than with an enterprise. 

Whene generating your GitHub PAT you will need to include `repo`, `workflow`, and `admin:org` permissions.

## Overview

This repository provides tools and scripts for managing GitHub Actions runners, including support for multi-architecture Docker images and a secure script for managing local runners.

## Managing Local Runners

This repository provides a `manage-local.sh` script to help manage local GitHub Actions runner containers. The script can launch or tear down runners interactively or start a specific number of runners in one command.

Prerequisites:

1. Create a `.env` file in the root directory with the following variables:
   ```env
   REPO=owner/repo
   ACCESS_TOKEN=your-github-personal-access-token
   SERVICE_NAME=your-service-name
   ENVIRONMENT=dev
   ```
2. Make the script executable:
   ```bash
   chmod +x manage-local.sh
   ```

Usage:

- To run the interactive menu:
  ```bash
  ./manage-local.sh
  ```
- To start N runners directly:
  ```bash
  ./manage-local.sh 4
  ```

The script will load variables from `.env` and prompt you for any that are missing.

## Multi-Architecture Support

This repository now supports multi-architecture Docker image builds and deployments for GitHub Actions runners across both arm64 and amd64 environments.

### Building Multi-Arch Images

To build the multi-arch images, you can use the following command:

```sh
docker buildx build --platform linux/amd64,linux/arm64 -t <your-dockerhub-username>/github-runner:latest --push .
```

### Deploying Multi-Arch Images

To deploy the multi-arch images, you can use the following command:

```sh
docker run -e REPO=<owner>/<repo> -e ACCESS_TOKEN=<your-github-personal-access-token> -e SERVICE_NAME=<your-service-name> -e ENVIRONMENT=<environment> <your-dockerhub-username>/github-runner:latest
```

Alternatively, you can use `docker-compose` to deploy the multi-arch images:

```yaml
version: '3.8'

services:
  runner:
    image: <your-dockerhub-username>/github-runner:latest
    restart: unless-stopped
    env_file: .env
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    deploy:
      mode: replicated
      replicas: 4
      resources:
        reservations:
          cpus: 0.5
          memory: 1024M
  cache:
    image: ghcr.io/falcondev-oss/github-actions-cache-server:latest
    restart: unless-stopped
    ports:
      - '3000:3000'
    environment:
      API_BASE_URL: http://cache:3000
    volumes:
      - cache:/app/.data

volumes:
  cache:
```

Make sure to replace `<your-dockerhub-username>` with your actual Docker Hub username.

## Usage

To use the GitHub Actions runner, you need to set the following environment variables:

- `REPO`: The owner and repository name in the format `<owner>/<repo>`.
- `ACCESS_TOKEN`: Your GitHub personal access token with `repo`, `workflow`, and `admin:org` permissions.
- `SERVICE_NAME`: A name to label your runner service.
- `ENVIRONMENT`: The environment name (e.g. dev, prod).

You can set these in your `.env` file or pass them directly to `docker run`.

Example `.env` file:

```env
REPO=owner/repo
ACCESS_TOKEN=your-github-personal-access-token
SERVICE_NAME=github-runner
ENVIRONMENT=dev
```

Example `docker run` command:

```sh
docker run \
  -e REPO=owner/repo \
  -e ACCESS_TOKEN=your-github-personal-access-token \
  -e SERVICE_NAME=github-runner \
  -e ENVIRONMENT=dev \
  <your-dockerhub-username>/github-runner:latest
```
