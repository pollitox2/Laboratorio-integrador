FROM node:20-alpine AS builder
WORKDIR /app

COPY package*.json ./
RUN npm ci --omit=dev

FROM node:20-alpine
WORKDIR /app

COPY --from=builder --chown=node:node /app/node_modules ./node_modules
COPY --chown=node:node . .

USER node

HEALTHCHECK --interval=15s --timeout=3s --start-period=10s --retries=3 \
  CMD wget -qO /dev/null http://127.0.0.1:3000/healthz || exit 1

EXPOSE 3000
CMD ["node", "server.js"]