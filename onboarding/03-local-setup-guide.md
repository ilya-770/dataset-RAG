# Локальный запуск проекта

Инструкция позволяет поднять полный стек проекта на своём компьютере:
бот, административную панель и базу данных. Время — около 10–15 минут
при первом запуске.

---

## Предварительные требования

- Docker Desktop (Windows/Mac) или Docker Engine + Docker Compose (Linux).
  Версии: Docker 24+, Compose 2.x.
- Git.
- Доступ к репозиторию (см. `/onboarding/02-access-checklist.md`).

Проверь, что Docker запущен:
```bash
docker --version
docker compose version
```

---

## Шаг 1. Клонирование репозитория

```bash
git clone <url-репозитория>
cd hse-bot
```

Структура репозитория:
hse-bot/
├── bot/          # Telegram-бот (Python)
├── admin/        # Административная панель (FastAPI)
├── migrations/   # Миграции Alembic
├── deploy/       # Скрипты деплоя
├── parser/       # Парсер XML (в разработке)
├── docker-compose.yml
├── .env.example
└── README.md

---

## Шаг 2. Переменные окружения

Скопируй шаблон и заполни значения:

```bash
cp .env.example .env
```

Обязательные переменные:

| Переменная | Описание | Где взять |
|---|---|---|
| `BOT_TOKEN` | Токен Telegram-бота | У Орлова О. О. |
| `DATABASE_URL` | DSN PostgreSQL | Заполняется автоматически для локального запуска |
| `REDIS_URL` | URL Redis | Заполняется автоматически для локального запуска |
| `ADMIN_SECRET_KEY` | Секрет для JWT в панели | Придумать самостоятельно |

Для локального запуска `DATABASE_URL` и `REDIS_URL` уже прописаны
в `.env.example` с дефолтными значениями — менять не нужно.

---

## Шаг 3. Запуск

```bash
docker compose up -d
```

Docker скачает образы, соберёт контейнеры и запустит все сервисы
в фоне. При первом запуске — 3–5 минут на загрузку образов.

---

## Шаг 4. Инициализация базы данных

После первого запуска применить миграции:

```bash
docker compose exec admin alembic upgrade head
```

Создать первого суперменеджера:

```bash
docker compose exec admin python scripts/create_superuser.py
```

Скрипт спросит логин и пароль — введи любые, запомни.

---

## Шаг 5. Проверка

**Бот:** напиши `/start` в Telegram-чате `@hes_bot_dev_bot`.
Бот должен ответить главным меню.

**Административная панель:** открой `http://localhost:8000`
в браузере. Войди с данными суперменеджера из шага 4.

**Логи** (если что-то не работает):

```bash
docker compose logs bot --tail=50
docker compose logs admin --tail=50
```

---

## Обновление после изменений в коде

```bash
git pull
docker compose up -d --build
```

Если изменилась схема БД — дополнительно:

```bash
docker compose exec admin alembic upgrade head
```

---

## Частые проблемы

**Порт 8000 занят.** Поменяй порт в `docker-compose.yml`:
`"8001:8000"` вместо `"8000:8000"`, затем открывай `localhost:8001`.

**Бот не отвечает.** Проверь, что `BOT_TOKEN` в `.env` верный
и бот не запущен где-то ещё с тем же токеном.

**Ошибка миграции.** Убедись, что контейнер `db` запущен и здоров:
`docker compose ps`. Если статус `unhealthy` — подожди 20–30 секунд
и повтори команду.

---

## Остановка

```bash
docker compose down
```

Данные PostgreSQL сохраняются в Docker volume — при следующем
`docker compose up` база будет в том же состоянии.