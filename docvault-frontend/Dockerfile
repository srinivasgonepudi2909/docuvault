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

# Copy SPA config & build output
COPY deploy/nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=build /app/dist /usr/share/nginx/html

# Prepare writable dirs for non-root Nginx
# The nginx user exists in this image; give it ownership
RUN mkdir -p /var/cache/nginx /var/run /var/log/nginx \
 && chown -R nginx:nginx /var/cache/nginx /var/run /var/log/nginx /usr/share/nginx /etc/nginx/conf.d

EXPOSE 8080

# Run as the 'nginx' user (uid ~101)
USER nginx

# Healthcheck (optional)
HEALTHCHECK --interval=30s --timeout=3s CMD wget -qO- http://127.0.0.1:8080/ >/dev/null 2>&1 || exit 1

