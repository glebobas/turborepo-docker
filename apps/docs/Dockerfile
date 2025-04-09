FROM node:18-alpine AS base

# --------------------------------------------
# Builder Stage — установка Turbo и prune
# --------------------------------------------
FROM base AS builder

RUN apk update && apk add --no-cache libc6-compat
WORKDIR /app

# Устанавливаем Turbo CLI глобально через npm
RUN npm install -g turbo

COPY . .

# Обрезаем workspace для docker
RUN turbo prune docs --docker

# --------------------------------------------
# Installer Stage — установка зависимостей
# --------------------------------------------
FROM base AS installer

RUN apk update
RUN apk add --no-cache libc6-compat
WORKDIR /app

# Копируем package.json и lock-файлы из builder
COPY --from=builder /app/out/json/ .

RUN corepack disable

# Установка зависимостей
RUN npm install

# Копируем все нужные файлы для сборки
COPY --from=builder /app/out/full/ .

ENV NEXT_TELEMETRY_DISABLED=1

RUN npm run build

# --------------------------------------------
# Runner Stage — финальный образ
# --------------------------------------------
FROM base AS runner

WORKDIR /app

# Создаём безопасного пользователя
RUN addgroup --system --gid 1001 nodejs \
 && adduser --system --uid 1001 nextjs

USER nextjs

# COPY --from=installer /app/public ./public

# RUN mkdir /app/apps/docs/.next/.next
# RUN chown nextjs:nodejs .next

# Копируем production-сборку
COPY --from=installer --chown=nextjs:nodejs /app/apps/docs/.next/standalone ./
COPY --from=installer --chown=nextjs:nodejs /app/apps/docs/.next/static ./apps/docs/.next/static

EXPOSE 3000

CMD node apps/docs/server.js