# Dockerfile для simple-app (Go)
# Робив multi-stage, щоб фінальний образ був маленький.

# --- Stage 1: збираємо бінарник ---
FROM golang:1.23-alpine AS builder

WORKDIR /src

# Копіюємо весь код і збираємо Go-застосунок
COPY . .
RUN go build -o /app .


# --- Stage 2: фінальний образ ---
# Беремо маленький alpine, бо Go toolchain тут вже не потрібен
FROM alpine:3.20

# Створюємо юзера, щоб не запускати від root
RUN adduser -D -u 1000 app

# Переносимо тільки готовий бінарник з першого стейджу
COPY --from=builder /app /app

# Перемикаємось на нерут-юзера
USER app

# Порт, на якому застосунок слухає
EXPOSE 8080

# Запуск
CMD ["/app"]
