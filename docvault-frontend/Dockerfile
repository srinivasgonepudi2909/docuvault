# -----------------------------
# Stage 1: Build the React app
# -----------------------------
FROM node:20-alpine AS build
WORKDIR /app

# Install dependencies (prefer lockfile if present)
COPY package.json package-lock.json* pnpm-lock.yaml* yarn.lock* ./
RUN npm ci || npm install

# Copy sources and build
COPY . .
RUN npm run build

# ------------------------------------
# Stage 2: Serve static files via Nginx
# ------------------------------------
FROM nginx:1.27-alpine

# Remove default config
RUN rm -rf /etc/nginx/conf.d/*

# Copy custom config
COPY deploy/nginx.conf /etc/nginx/conf.d/default.conf

# Copy build output
COPY --from=build /app/dist /usr/share/nginx/html

# Prepare writable dirs for non-root Nginx
RUN mkdir -p /var/run/nginx && \
    chown -R nginx:nginx /var/run/nginx /var/cache/nginx /var/log/nginx /usr/share/nginx/html && \
    chmod -R 755 /var/run/nginx

# Important: Ensure PID path matches nginx.conf
ENV NGINX_PID_PATH=/var/run/nginx/nginx.pid

# Run as the 'nginx' user (uid ~101)
USER nginx

# Explicitly set the PID path in the CMD
CMD ["nginx", "-g", "daemon off; pid /var/run/nginx/nginx.pid;"]

EXPOSE 8080

# Healthcheck (optional)
HEALTHCHECK --interval=30s --timeout=3s CMD wget -qO- http://127.0.0.1:8080/ >/dev/null 2>&1 || exit 1