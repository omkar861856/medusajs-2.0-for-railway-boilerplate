# Use Node 22 as specified in package.json
FROM node:22-alpine AS base

# Install pnpm and essentials
RUN npm install -g pnpm

# --- Dependencies ---
FROM base AS deps
WORKDIR /app
COPY package.json pnpm-lock.yaml .npmrc ./
# Use --frozen-lockfile for consistency
RUN pnpm install --frozen-lockfile

# --- Build ---
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
# Run build to generate .medusa/server
RUN pnpm run build

# --- Runner ---
FROM base AS runner
WORKDIR /app

ENV NODE_ENV=production

# Install medusa cli globally
RUN npm install -g @medusajs/cli

COPY --from=builder /app/package.json ./package.json
COPY --from=builder /app/pnpm-lock.yaml ./pnpm-lock.yaml
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/.medusa ./.medusa
COPY --from=builder /app/medusa-config.js ./medusa-config.js
COPY --from=builder /app/src ./src
# Public folder might not exist, but copy if it does
COPY --from=builder /app/public* ./public 2>/dev/null || true

EXPOSE 9000

CMD ["pnpm", "run", "start"]
