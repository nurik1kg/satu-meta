# satu.kg — доступы и ссылки

> ⚠️ Файл содержит креды для dev/test окружения. Не коммитить в публичные репы.

## Тестовый сервер

- **IP:** 62.113.97.240 (Ubuntu 24.04)
- **SSH:** `ssh root@62.113.97.240`
- **Пароль:** `yEILYELC2*Ge`
- **Проект:** `/opt/satu/`

## URL'ы

### Локально (Docker)

| Сервис | URL |
|---|---|
| Admin frontend | http://localhost:5173 |
| Storefront | http://localhost:3000 |
| Storefront-API (FastAPI) | http://localhost:8001 |
| Backend API (Django) | http://localhost:8000 |
| Django Admin | http://localhost:8000/admin/ |

### Тест-сервер

| Сервис | URL |
|---|---|
| Admin frontend | http://62.113.97.240:5173 |
| Storefront landing | http://62.113.97.240:3000 |
| Demo витрина | http://62.113.97.240:3000/demo |
| Storefront-API | http://62.113.97.240:8001 |
| Backend API | http://62.113.97.240:8000 |
| Django Admin | http://62.113.97.240:8000/admin/ |

## Пользователи

### Локальная БД

- **Superuser:** `admin@satu.kg` / `admin12345` (телефон `+996700000000`)
- **Существующий шоп:** `IStore` (IStore Bishkek)

### Серверная БД

- **Super admin (Django):** `admin@satu.kg` / `admin12345` (телефон `+996700000000`) — для http://62.113.97.240:8000/admin/
- **Admin магазина:** `demo@satu.kg` / `demo12345` (телефон `+996700000001`) — для http://62.113.97.240:5173
- **Шоп:** slug `demo`, "Demo Магазин"
- **Клиент:** логина нет (анонимный покупатель витрины http://62.113.97.240:3000/demo)

## GitHub репозитории

| Что | Репо | Remote в локальной папке |
|---|---|---|
| Backend (Django) | `nurik1kg/satu-backend` | `origin` в `satu-backend/` |
| Admin frontend | `nurik1kg/satu-admin-front` | `nurik` в `satu-kg-admin/` |
| Storefront (Next.js) | `nurik1kg/satu-client` | `nurik` в `satu-storefront/` |
| Storefront-API (FastAPI) | `nurik1kg/satu-client-backend` | `origin` в `satu-storefront-api/` |

**PAT для clone на сервере (read access):** хранится в 1Password / личной заметке (не коммитить).

Пример клонирования с PAT:
```bash
git clone https://nurik1kg:<PAT>@github.com/nurik1kg/satu-backend.git
```

## База данных (dev)

| Параметр | Значение |
|---|---|
| Engine | PostgreSQL 16 (Docker) |
| Host (внутри compose) | `db` |
| Port | 5432 (наружу проброшен) |
| DB / User / Pass | `satu` / `satu` / `satu` |
| Redis | `redis://redis:6379` (db 0 — Django cache, db 1 — FastAPI cache) |

## Архитектура

```
satu.kg/
├── satu-backend/          # Django REST (источник истины, admin API)
├── satu-kg-admin/         # React + Vite (админка магазина)
├── satu-storefront-api/   # FastAPI BFF (читает Django + Redis-кэш)
├── satu-storefront/       # Next.js 16 (витрина для покупателей)
└── docker-compose.yml     # Все 6 сервисов
```

**Поток данных витрины:**
```
Next.js storefront → FastAPI BFF → Django public API → Postgres
                          ↓
                       Redis cache
```

## Полезные команды

### Локально

```bash
cd /Users/nuridin/projects/satu.kg

# Поднять весь стек
docker compose up -d

# Логи сервиса
docker compose logs -f <backend|storefront|storefront-api|frontend|db|redis>

# Django shell
docker compose exec backend python manage.py shell

# Создать суперюзера
docker compose exec backend python manage.py createsuperuser

# Миграции
docker compose exec backend python manage.py makemigrations
docker compose exec backend python manage.py migrate
```

### На сервере

```bash
cd /opt/satu

# Обновить код
(cd satu-backend && git pull)
(cd satu-storefront && git pull)
(cd satu-storefront-api && git pull)

# Пересобрать и перезапустить сервис
docker compose up -d --build <сервис>

# Логи
docker compose logs -f <сервис>
```

## Переменные окружения

Файлы `.env` в `.gitignore` — после `git clone` восстанови вручную.

### `satu-backend/.env`

```env
SECRET_KEY=django-insecure-change-me-in-production
DEBUG=True
ALLOWED_HOSTS=localhost,127.0.0.1,0.0.0.0,backend,62.113.97.240

POSTGRES_DB=satu
POSTGRES_USER=satu
POSTGRES_PASSWORD=satu
POSTGRES_HOST=db
POSTGRES_PORT=5432

REDIS_URL=redis://redis:6379/0

CORS_ALLOWED_ORIGINS=http://localhost:5173,http://62.113.97.240:5173,http://62.113.97.240:3000

GOOGLE_CLIENT_ID=
```

### `satu-storefront-api/.env`

```env
DJANGO_API_URL=http://backend:8000
REDIS_URL=redis://redis:6379/1
CACHE_TTL_SECONDS=60
CORS_ALLOWED_ORIGINS=http://localhost:3000,http://62.113.97.240:3000
```

### Storefront / Admin (через compose `environment:`)

- `NEXT_PUBLIC_API_URL=http://62.113.97.240:8001` (storefront)
- `VITE_API_URL=http://62.113.97.240:8000` (admin)

## API эндпоинты

### Публичные (для витрины)

```
GET  /api/public/shops/<slug>/
GET  /api/public/shops/<slug>/categories/
GET  /api/public/shops/<slug>/products/?category=&q=&page=&page_size=
GET  /api/public/shops/<slug>/products/<id>/
POST /api/public/shops/<slug>/orders/
```

### FastAPI BFF (storefront-api)

```
GET  /health
GET  /storefront/<slug>/shop
GET  /storefront/<slug>/categories
GET  /storefront/<slug>/products?category=&q=&page=&page_size=
GET  /storefront/<slug>/products/<id>
POST /storefront/<slug>/orders
```

### Авторизованные (для админки)

```
POST   /api/auth/register/
POST   /api/auth/token/        # login → JWT
POST   /api/auth/token/refresh/
GET    /api/auth/me/
POST   /api/auth/google/

GET    /api/shop/me/
POST   /api/shop/
PATCH  /api/shop/me/
GET    /api/shop/phones/        ... (CRUD: addresses, social-links, schedule)

GET    /api/catalog/categories/
GET    /api/catalog/products/
POST   /api/uploads/image/      # multipart, PNG/JPEG, ≤5MB

GET    /api/orders/
PATCH  /api/orders/<number>/status/
DELETE /api/orders/items/<id>/
```
