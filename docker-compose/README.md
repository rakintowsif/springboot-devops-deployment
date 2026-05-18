# Docker Compose Deployment

This folder documents the Docker Compose deployment path for the `tm-core` service.
It is based on the CI/CD pattern from the TeamMart pipeline and provides a dev deployment flow.

## Overview

- Branch: `dev`
- Deployment target: remote host via SSH
- Deployment directory: `/opt/tm-docker`
- Docker Compose service: `tm-core`
- Image tag variable: `IMAGE_TAG_CORE`

## Files

- `docker-compose.yml`: dev service definition for the application container.
- `docker-compose.prod.yml`: prod service definition for the application container.
- `.env.example`: example environment variables file.
- `cicd-docker-compose.yml`: sample GitHub Actions workflow for deploying the dev branch.

## Deployment flow

1. Build and push the Docker image to ECR in GitHub Actions.
2. SSH into the dev host and log into ECR.
3. Update `.env` with the new image tag.
4. Pull and restart the service with `docker compose up -d tm-core`.
5. Wait for the health check to succeed.

## Environment variables

Create a local `.env` file in the deployment host directory, based on `.env.example`:

```bash
cp .env.example .env
```

## Notes

- The service name in Docker Compose is `tm-core` and the container will be restarted in place.
- Rolling back is handled automatically if the health check fails.
- Customize ports and health-check routes according to your Spring Boot application.
