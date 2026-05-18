# AWS ECS Deployment

This folder documents the AWS ECS deployment path derived from the TeamMart CI/CD pipeline.
It focuses on building and pushing a Docker image to ECR, then deploying a new task definition to ECS on the `master` branch.

## Overview

- Branches: `master` deploys to ECS production
- Build step: Docker image built and pushed to Amazon ECR
- ECS cluster: `teammart-cluster`
- ECS service: `tm-core-service`
- Task definition name: `tm-core-prod-new`
- Image repository: `${{ secrets.ECR_REPO_TM_CORE }}`

## Required GitHub secrets

- `AWS_REGION`
- `AWS_ACCOUNT_ID`
- `ECR_REPO_TM_CORE`
- `AWS_OIDC_ROLE`
- `SLACK_WEBHOOK_URL` (optional for notifications)

## How it works

1. `build-and-push-image` builds the Docker image locally in GitHub Actions and pushes it to ECR.
2. The `deploy-to-prod-ecs` job runs only on `master`.
3. It downloads the existing ECS task definition, updates the container image, and registers a new revision.
4. It then updates the ECS service and waits for stability.

## Task definition sample

A sample task definition is available in `task-definition.json`.
It is based on your production ECS task definition for the `tm-core` service:

- `containerPort`: 7050
- `hostPort`: 7050
- Mounted config volume: `/var/web/html_new/tm_core_prod:/app/config`
- ECS compatibilities: `EC2`
- Logs: AWS Logs to `/tm/core/java/prod`
- Health check: `http://localhost:7050/actuator/health`
- Task role / execution role: `ecs-cicd-deployer-role`

Adjust only the account, region, repository, and environment-specific values.

## ECS workflow sample

The `cicd-ecs.yml` file contains a production ECS deployment workflow sample.
Use it as a starting point for your `.github/workflows/` configuration.

## Notes

- The sample uses AWS OIDC via `aws-actions/configure-aws-credentials@v4`.
- The image tag is the short commit SHA and the branch-specific `LATEST_TAG`.
- Keep the build and deployment jobs aligned with the repository and ECR naming.
