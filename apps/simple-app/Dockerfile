# syntax=docker/dockerfile:1.7
# Dockerfile для apps/simple-app (Go)
# -----------------------------------------------------------
# Multi-stage build:
#   1) builder — повний Go toolchain, завантажує модулі, компілює статичний бінарник;
#   2) runtime — мінімальний alpine, лише сам бінарник + ca-certificates,
#                запуск під нерут-користувачем.

############################################
# Stage 1 — builder
############################################
FROM golang:1.23-alpine AS builder

# CGO_ENABLED=0 — статичний бінарник без залежності від libc,
# який без проблем запуститься навіть у scratch/distroless.
ENV CGO_ENABLED=0 \
    GOOS=linux

WORKDIR /data

# Спочатку копіюємо лише маніфести модулів — це дає кешування шару
# із залежностями, поки go.mod/go.sum не змінюються.
COPY go.mod go.sum* ./
RUN go mod download

# Копіюємо решту коду і збираємо.
COPY . .

# -trimpath      — прибирає локальні шляхи з бінарника (репродюсибельність);
# -ldflags "-s -w" — викидає debug-символи та DWARF-таблиці, зменшує розмір.
RUN go build -trimpath -ldflags="-s -w" -o /out/app .


############################################
# Stage 2 — runtime (фінальний образ)
############################################
FROM alpine:3.20 AS runtime

# ca-certificates — якщо застосунок робить HTTPS-запити назовні;
# одразу створюємо системного нерут-користувача app.
RUN apk add --no-cache ca-certificates \
 && addgroup -S app \
 && adduser  -S -G app -H -s /sbin/nologin app

WORKDIR /app

# Переносимо ЛИШЕ зібраний бінарник зі стейджу builder.
# Ніякого Go toolchain, вихідного коду чи модулів у фінальному образі немає.
COPY --from=builder /out/app /app/app

# Перемикаємось на нерут-користувача.
USER app

# Відкриваємо порт назовні.
EXPOSE 8080

# Простий healthcheck: wget вбудований у busybox alpine.
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget -qO- http://127.0.0.1:8080/ >/dev/null 2>&1 || exit 1

ENTRYPOINT ["/app/app"]
