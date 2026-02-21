# MULTI-STAGE DOCKERFILE FOR BACKEND API -
# This file contains instructions to build Docker images for all environments: development, test and production
#
# Usage:
#   Development: docker-compose up
#   Test:        docker-compose -f docker-compose.test.yml up 
#   Production:  docker build --target=production

# STAGE 1: Base foundation layer
# Contains base image, system packages and shared setup for all environments

# Use Debian-based node as base image instead of Alpine for mongodb-memory-server compatibility
FROM node:22-slim AS base

# Set the working directory of the Docker image (where everything else will happen in)
WORKDIR /app

# Install system dependencies including:
# - wget: for health checks
# - libcurl4: for mongodb-memory-server (needed for test env)
# - ca-certificates: for HTTPS requests connections
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      wget \
      libcurl4 \
      ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Copy the package files first for dependencies installation
COPY package.json package-lock.json ./


# STAGE 2: Dependencies installation stage (for dev and test envs)
FROM base AS dependencies

# Installs all dependencies including dev dependencies (need for jest, mongodb-memory-server, etc.)
RUN npm install

# STAGE 3: Runtime base - common setup (non-root user, files and configs - to make more DRY)
FROM dependencies AS runtime-base

# Copy the whole project (.dockerignore is used to exclude specific files and folders from the build)
COPY . .

# Create non-root user (for security reasons)
RUN groupadd -g 1001 nodejs && \
    useradd -u 1001 -g nodejs nodejs && \
    chown -R nodejs:nodejs /app

# Switch to non-root user
USER nodejs

# Expose backend API port
EXPOSE 3000


# STAGE 4: Development environment (with hot-reload)
FROM runtime-base AS development

# Set the development environment variables
ENV NODE_ENV=development

# Configure a health check to ensure the container is running properly
# This will check by calling the /health endpoint every 30 secs,
# If after 3 failed attempts, then it will mark the container as unhealthy
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

# Start the application (when container starts)
CMD ["npm", "start"]


# STAGE 5: Test environment
FROM runtime-base AS test

# Set test environment
ENV NODE_ENV=test

# Run tests
CMD ["npm", "test"]


# STAGE 6: Builder stage - to build production image
FROM base AS builder

# Install all dependencies
RUN npm ci

# Copy only source code
COPY src ./src/


# STAGE 7: Production environment - minimal image to run the app (used by CI/CD pipelines for deployment)
FROM base AS production

# Install only production dependencies (and clean up npm cache to reduce image size)
RUN npm ci --only=production && \
    npm cache clean --force

# Copy production source from builder stage
COPY --from=builder /app/src ./src/

# Create non-root user for security
RUN groupadd -g 1001 nodejs && \
    useradd -u 1001 -g nodejs nodejs && \
    chown -R nodejs:nodejs /app

# Switch to non-root user
USER nodejs

# Set production environment
ENV NODE_ENV=production

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

# Start the application
CMD ["npm", "start"]