# Kittygram

Социальная сеть для обмена фотографиями котиков. Пользователи могут регистрироваться, добавлять своих котиков, указывать их достижения, цвет, год рождения и загружать фото.

## Функции

- Регистрация и аутентификация по токену.
- CRUD для котиков с загрузкой изображений.
- Список достижений (Achievements) с привязкой к котикам.
- Админка Django.
- REST API.

## Стек технологий

- **Backend:** Python 3.10, Django 3.2, Django REST framework, Djoser, Gunicorn
- **База данных:** PostgreSQL 13
- **Frontend:** React (Node.js 18)
- **Веб-сервер / прокси:** Nginx
- **Контейнеризация:** Docker, Docker Compose
- **CI/CD:** GitHub Actions, Docker Hub
- **Линтер:** ruff

## Архитектура

Четыре контейнера, связанные через docker-compose:

| Контейнер | Образ                       | Назначение                                     |
|-----------|-----------------------------|------------------------------------------------|
| `db`      | `postgres:13`               | База данных                                    |
| `backend` | `kittygram_backend`         | Django + Gunicorn на порту 9000                |
| `frontend`| `kittygram_frontend`        | Собирает React-бандл в volume `static`         |
| `gateway` | `kittygram_gateway` (Nginx) | Раздаёт статику/медиа, проксирует API и admin  |

Volumes:
- `pg_data` — данные PostgreSQL.
- `static` — собранная статика админки/DRF и фронтенда.
- `media` — пользовательские загрузки.

## Как развернуть проект

### 1. Клонировать репозиторий

```bash
git clone https://github.com/<your_login>/kittygram_final.git
cd kittygram_final
```

### 2. Заполнить `.env`

В корне проекта создайте файл `.env` (по образцу `.env.example`):

```env
POSTGRES_DB=kittygram
POSTGRES_USER=kittygram_user
POSTGRES_PASSWORD=kittygram_password
DB_HOST=db
DB_PORT=5432

SECRET_KEY=replace-with-your-secret
DEBUG=False
ALLOWED_HOSTS=127.0.0.1,localhost,your.domain
```

Переменные:
- `POSTGRES_DB / USER / PASSWORD` — креды для PostgreSQL.
- `DB_HOST` — имя сервиса БД (`db` для docker-compose).
- `DB_PORT` — порт БД (обычно 5432).
- `SECRET_KEY` — секретный ключ Django.
- `DEBUG` — `True` для разработки, `False` для продакшена.
- `ALLOWED_HOSTS` — список хостов через запятую.
- `USE_SQLITE=True` — необязательная, переключает БД на SQLite (для быстрых локальных тестов).

### 3. Запустить контейнеры

Локально (со сборкой образов):

```bash
docker compose up --build -d
```

В продакшене (использует готовые образы с Docker Hub):

```bash
docker compose -f docker-compose.production.yml up -d
```

Приложение будет доступно на `http://localhost:9000`.

### 4. Создать суперпользователя (по желанию)

```bash
docker compose exec backend python manage.py createsuperuser
```

## CI/CD

При пуше в любую ветку GitHub Actions:
1. Запускает `ruff check` и `ruff format --check` на бэкенде.
2. Запускает тесты бэкенда (matrix Python 3.10, 3.11, 3.12) и фронтенда.

При пуше в ветку `main` дополнительно:
3. Собирает и пушит на Docker Hub образы `kittygram_backend`, `kittygram_frontend`, `kittygram_gateway`.
4. Отправляет уведомление в Telegram.

### Необходимые GitHub Secrets

| Имя | Назначение |
|-----|-----------|
| `DOCKER_USERNAME` | Логин на Docker Hub |
| `DOCKER_PASSWORD` | Токен/пароль Docker Hub |
| `TELEGRAM_TO` | ID получателя в Telegram |
| `TELEGRAM_TOKEN` | Токен бота Telegram |

## Автор

Студент Яндекс.Практикума, курс «Python-разработчик».
