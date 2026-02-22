# DEV1004 Assessment 3: Develop a CI/CD Pipeline

## The Century Screening Room - Continuous Integration and Deployment

### Overview
This project sets up a CI/CD pipeline for the Century Screening Room MERN app.
The pipeline automates testing and deployment using GitHub Actions, Docker and Render with frontend served on Netlify.
This document outlines the tools and systems used in this CI/CD Pipeline and includes a pipeline overview of each phase, it's purpose and what it does.

---

## CI/CD Tools & Systems

These are the main tools and systems I chose including a description of what they do, what it's used for and why I chose them over alternative options.

### 1. GitHub Actions

**What it does:**
- GitHub Actions is the CI/CD platform.
- It runs workflow (YAML files) automatically on triggers like code pushes or PRs.

**What it's used for in this project:**
In this project, it is used to:
- Run tests (backend and frontend in Docker) at multiple phases to check code and catch bugs early, acts as a validation check to ensure that tests passes successfully.
- Build the Docker image and push to Docker Hub on merge to main.
- Trigger CD deployment of the backend to Render after a successful build.

**Why chosen over alternatives:**
I chose GitHub Actions because:
    - Integrates into GitHub (with no extra setup).
    - It is free for public repos, which is beneficial for a student project.
    - Has reusable workflows to optimise DRY code.

In comparison to the alternative options, such as:
- Jenkins: It needs your own server which can be more complex to setup and can incur costs
- CircleCI: Has paid limits for bigger projects, in this case GitHub is more simple and free making it a more suitable student option for projects of this scale.

### 2. Docker and Docker Compose

**What it does:**
- Docker packages the app into containers for consistent environments.
- Docker Compose manages multi-container setups (like test DB if needed).

**What it's used for in this project:**
- It's used in `test_backend.yml` to run isolated tests (in a Docker container to control environment tests)
- It's also used in `build_push.yml` to build and push the production image to Docker Hub (ready for next step for backend to be deployed to Render)

**Why chosen over alternatives**:
I chose to use Docker because:
- It is a highly industry standard tool used for containerisation, to makes tests and deployments reliable across different environments.
- It is free and more simpler to use than compared to the alternative options below.

In comparison to the alternative options, such as:
- Kubernetes: In this case, I considered Kubernetes too complex for a small application

### 3. Render (Cloud Platform)

**What it does:**
- Render is the cloud host used for backend deployment.
- It pulls the Docker images and runs them as a web service.

**What it's used for in this project:**
- In `deploy_render.yml`, it triggers deployment via a webhook to deploy the latest image, then does a health check to verify it's live.
- It handles scaling and connects to services like a database via environment variables setup.

**Why chosen over alternatives**:
- Render has a free tier suitable for the current scope of this project.
- It provides seamless Docker support and auto-deploys.

In comparison to other Cloud Hosting Platforms such as:
- Google Cloud Run: 
- AWS ECS: Requires more complex configurations and billing setup and can incur costs especially if not managed properly.
- Heroku:
- Railway:

### 4. Docker Hub (Image Registry)
**What it does**:
- Docker Hub stores built Docker images.

**What it's used for in this project:**
- It is used in `build_push.yml` to push built Docker images to Docker Hub, which Render pulls for auto-deployment.

**Why chosen over alternatives**:
- Free public repos
- Integrates seamlessly with GitHub Actions.

Other alternatives such as AWS ECR can incur costs.

### 4. Netlify

### 5. MongoDB Atlas

---

## CI/CD Pipeline Overview

### Diagram: How the tools integrate together

---

## Pipeline Phases

1. `ci_pr.yml`
    - Trigger: on pull_request
    - Purpose: Runs test before merge as quality validation check
2. `ci_main.yml`
    - Trigger: on push to main
    - Purpose: Builds production image of backend and pushes to Docker Hub
3. `cd_production.yml`
    - Trigger: workflow_run
    - Purpose: Deplos to Render after CI checks and tests passes successfully

---

## Workflow Purpose and Functionality

### Phase 1: PR Validation (`ci_pr.yml`)

**Purpose:** Ensures code is tested and passes successfully before merge
**What it does:**
    - Runs backend tests in Docker container (using Jest)
    - Runs frontend tests in Docker container (using Vitest)
    - The PR quality validation checks block merge if either of the tests fail
    - Uploads test logs as artifacts
**Reusable workflows called:**
    - `test_backend.yml`
    - `test_frontend.yml`
    - (These workflow tests can also be run independently on workflow_displatch)

### Phase 2: Build Pipeline (`ci_main.yml`)

**Purpose:** Build and publish production Docker image after merge to main
**What it does:**
    - Re-runs backend tests
    - Builds production image from backend/Dockerfile (target: production)
    - Pushes to Docker Hub with tags: :latest and :main-<sha>
    - Writes a pipeline summary with commit and image tag
**Reusable workflows called:**
    - `build_push.yml`

## Phase 3: Deployment - CD Production Pipeline (`cd_production.yml`)

**Purpose:** Deploy backend to Render after successful build
**What it does:**
    - Triggers only when CI Main Pipeline succeeds (using workflow_run + if successful condition)
    - Sends POST request to Render deploy hook
    - Waits 30 seconds for Render to pull new image
    - Health check

**Reusable workflows called:**
    - `deploy_render.yml`

---

