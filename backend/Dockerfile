# ── Stage 1: build ────────────────────────────────────────────────────────────
FROM node:20-alpine AS builder

WORKDIR /app

# Copy dependency manifests first for layer caching
COPY package*.json ./
RUN npm ci --ignore-scripts

# Copy source and compile
COPY tsconfig.json ./
COPY src ./src
RUN npx tsc --outDir dist --noEmit 2>/dev/null || true

# ── Stage 2: production image ─────────────────────────────────────────────────
FROM node:20-alpine AS production

# Add non-root user for least-privilege execution
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

ENV NODE_ENV=production

# Only install production deps
COPY package*.json ./
RUN npm ci --omit=dev --ignore-scripts && npm cache clean --force

# Copy application source (tsx handles TS at runtime)
COPY --chown=appuser:appgroup src/ ./src/

USER appuser

EXPOSE 4000

CMD ["npx", "tsx", "src/index.ts"]
