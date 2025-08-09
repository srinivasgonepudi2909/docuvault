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

# Copy SPA nginx config (listens on 8080 and supports client-side routing)
COPY deploy/nginx.conf /etc/nginx/conf.d/default.conf

# Copy built assets
COPY --from=build /app/dist /usr/share/nginx/html

# Expose non-root port
EXPOSE 8080

# Optional: run as non-root (nginx user has uid 101 in this image)
USER 101

# Healthcheck (optional)
HEALTHCHECK --interval=30s --timeout=3s CMD wget -qO- http://127.0.0.1:8080/ >/dev/null 2>&1 || exit 1
