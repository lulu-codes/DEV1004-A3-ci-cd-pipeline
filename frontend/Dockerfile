# Frontend Dockerfile - Development environment container for React/Vite application

# Use official node alpine (v22) as the base image
FROM node:22-alpine

# Set the working directory
WORKDIR /app

# Build arguments declared before they're used
ARG NODE_ENV=development
ARG VITE_API_URL=http://localhost:3000

# Set environment variables (available at runtime for Vite dev server)
ENV NODE_ENV=${NODE_ENV}
ENV VITE_API_URL=${VITE_API_URL}

# Install wget for health checks (alpine doesn't include it)
RUN apk add --no-cache wget

# Copy package files first for better Docker layer caching
COPY package.json package-lock.json ./

# Install all dependencies (dev dependencies needed for Vite dev server)
RUN npm ci

# Copy application source code
COPY . .

# Create non-root user for security
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001 && \
    chown -R nodejs:nodejs /app

# Switch to non-root user
USER nodejs

# Expose the app port
EXPOSE 5000

# Health check to verify container is running
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:5000 || exit 1

# Start Vite development server with hot-reload
CMD ["npm", "run", "dev", "--", "--host", "0.0.0.0", "--port", "5000"]
