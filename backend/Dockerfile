# Backend Dockerfile -
# This file contains instructions for Docker to build a Development environment container for Express.js API server

# # Use node as starting base image
# FROM node:22-alpine

# Use Debian-based node image instead of Alpine for mongodb-memory-server compatibility
FROM node:22-slim

# Set the working directory of the Docker image (where everything else will happen in)
WORKDIR /app

# Install wget as not included in node:22-slim (needed for health checks)
RUN apt-get update && \
    apt-get install -y wget && \
    rm -rf /var/lib/apt/lists/*

# Copy the package files (for better layer caching as Docker will reuse if pkg files don't change)
COPY package.json package-lock.json ./

# Install all dependencies of the repo before copying the project source code
# (so Docker can run during build for image, caches this layer if package files don't change)
RUN npm install

# Copy the application source code
COPY src ./src/

# Create non-root user for security (Docker runs as root by default, this is for best practice)
# Then change ownership of all files to the nodejs user and group
RUN groupadd -g 1001 nodejs && \
    useradd -u 1001 -g nodejs nodejs && \
    chown -R nodejs:nodejs /app

# Switch to non-root user
USER nodejs

# Set the development environment variables
ARG NODE_ENV=development
ENV NODE_ENV=${NODE_ENV}

# Expose the backend port (tells Docker that this app listens on this port)
EXPOSE 3000

# Start the application (when container starts)
CMD ["npm", "start"]

# Configure a health check to ensure the container is running properly
# This will check by calling the /health endpoint every 30 secs,
# If after 3 failed attempts, then it will mark the container as unhealthy
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1
    # added this health check endpoint for Docker in server.js file
