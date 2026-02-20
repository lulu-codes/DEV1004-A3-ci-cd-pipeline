# Frontend Dockerfile - Development environment container for React/Vite application

# Use Debian-based node image (consistent with backend)
FROM node:22-slim

# Set the working directory
WORKDIR /app

# Build arguments (set at build time, used by Vite for API URL and NODE_ENV)
ARG NODE_ENV=development
ARG VITE_API_URL=http://localhost:3000

# Set environment variables (available at runtime for Vite dev server)
ENV NODE_ENV=${NODE_ENV}
ENV VITE_API_URL=${VITE_API_URL}

# Install wget as not included in node:22-slim (needed for health checks)
RUN apt-get update && \
    apt-get install -y wget && \
    rm -rf /var/lib/apt/lists/*

# Copy package files first for better Docker layer caching
COPY package.json package-lock.json ./

# Install all dependencies (as dev dependencies needed for Vite dev server)
RUN npm ci

# Copy application source code
COPY . .

# Create non-root user for security (Docker runs as root by default, this is for best practice)
# Then change ownership of all files to the nodejs user and group
RUN groupadd -g 1001 nodejs && \
    useradd -u 1001 -g nodejs nodejs && \
    chown -R nodejs:nodejs /app

# Switch to non-root user
USER nodejs

# Expose the frontend port
EXPOSE 5000

# Health check to verify container is running
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:5000 || exit 1

# Start Vite development server with hot-reload
CMD ["npm", "run", "dev", "--", "--host", "0.0.0.0", "--port", "5000"]
