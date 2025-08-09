# -----------------------------
# Stage 1: Build the React app
# -----------------------------
FROM node:20-alpine AS build
WORKDIR /app

# Install dependencies (prefer lockfile if present)
COPY package.json package-lock.json* ./
RUN npm ci

# Copy sources and build
COPY . .
RUN npm run build

# ------------------------------------
# Stage 2: Serve static files via Nginx
# ------------------------------------
FROM nginx:1.27-alpine

# Remove default config to prevent conflicts
RUN rm -rf /etc/nginx/conf.d/default.conf

# Create custom PID directory with correct permissions
RUN mkdir -p /var/run/nginx && \
    chown -R nginx:nginx /var/run/nginx && \
    chmod -R 755 /var/run/nginx

# Copy custom Nginx config
COPY deploy/nginx.conf /etc/nginx/nginx.conf

# Copy built app from build stage
COPY --from=build /app/dist /usr/share/nginx/html

# Set permissions for nginx user
RUN chown -R nginx:nginx /usr/share/nginx/html && \
    chmod -R 755 /usr/share/nginx/html

# Switch to nginx user (UID 101 in alpine)
USER nginx

# Explicitly set PID path and disable daemon mode
CMD ["nginx", "-g", "daemon off; pid /var/run/nginx/nginx.pid;"]

EXPOSE 8080

# Healthcheck
HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget -qO- http://localhost:8080/ >/dev/null 2>&1 || exit 1