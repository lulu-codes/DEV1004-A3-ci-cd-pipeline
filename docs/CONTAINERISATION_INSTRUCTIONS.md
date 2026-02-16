# Containerisation and Setup Instructions

This document provides instructions on how to containerise "The Century Screening Room" application (using Docker).

## Prerequisites

- Docker desktop installed and running
- Docker compose installed (included in Docker Desktop)
- Git (to clone the repository)

---

## Quick Start (prerequisites required)

1. **Build and start all containers:**
   docker-compose up --build

---

## Setup and Installation Instructions

1. To install Docker Desktop, visit [Docker's official website](https://www.docker.com/products/docker-desktop).

- Ensure Docker is running on your machine after installation.
  - Check by running:

  ```bash
  docker --version
  ```

It should show the installed Docker version (eg. Docker version 29.0.1, build eedd969)

1. Clone the repository:

From the terminal, run command:

```bash
git clone https://github.com/lulu-codes/DEV1004-A1-containerise-app.git
```

<!-- Note for marker: I have GPG issue that wouldn't allow me to make new commits to Github (which I need to fix) please refer to this complete file submission to run this app. -->

1. Navigate to the project directory:

```bash
cd DEV1004-A1-containerise-app
```

1. Build the Docker image:

```bash
docker-compose up --build
```

1. Access the application in your browser:

- Frontend: <http://localhost:5000>
- Backend API: <http://localhost:3000>
- Backend Health Check: <http://localhost:3000/health>

1. To stop the Application

```bash
docker-compose down        # Stop and remove containers
docker-compose down -v     # Also remove volumes (deletes database data)
```

---

### Common Docker Commands

```
# Start services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop services
docker-compose stop

# Rebuild after changes
docker-compose up --build

# Check container status
docker ps
```

---

## Setting up Environment Variables

- Environment variables are configured in `(/docker-compose.yml)`
- For local development, defaults are used.
- For production, set appropriate values for environment variables as per `(/.env.example)

Instructions for setting up environment variables can be found here in section 5. [Add Environment Variables](backend/docs/INSTALLATION_AND_DEPLOYMENT.md)
