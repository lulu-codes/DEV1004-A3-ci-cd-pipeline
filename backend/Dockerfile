# Backend Dockerfile -
# This file contains instructions for Docker to build a container image for the backend express API server

# Use node as starting base image
FROM node:22-alpine

# Set the working directory of the Docker image (where everything else will happen in)
WORKDIR /app

# Copy the repo files into the DOcker image
COPY package.json package-lock.json ./

# Install all dependencies of the repo before copying the project source code
# (so Docker can run during build for image, caches this layer if package files don't change)
RUN npm install

# Copy the application source code
COPY src ./src/

# Create non-root user for security (Docker runs as root by default, this is for best practice)
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Change ownership of all files and switch to the nodejs user
RUN chown -R nodejs:nodejs /app
USER nodejs

# Set the environment variables (can be overridden at runtime)
ARG NODE_ENV=development
ENV NODE_ENV=${NODE_ENV}

# Run the app (when container starts)
CMD ["npm", "start"]

# Expose the app port (tells Docker that this app listens on this port)
EXPOSE 3000

# Configure a health check to ensure the container is running properly
# This will check by calling the /health endpoint every 30 secs,
# If after 3 failed attempts, then it will mark the container as unhealthy
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:5000 || exit 1
    # added this health check endpoint for Docker in server.js file
