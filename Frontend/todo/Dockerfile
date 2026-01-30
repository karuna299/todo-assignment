# ---------- Build stage ----------
FROM node:18-alpine AS builder
WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .
RUN npm run build

# ---------- Runtime stage ----------
FROM node:18-alpine
WORKDIR /app

# create non-root user
RUN adduser -D appuser
USER appuser

COPY --from=builder /app/build ./build

EXPOSE 3000
CMD ["npx", "serve", "-s", "build"]
