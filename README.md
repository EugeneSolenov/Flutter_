# Flutter

Социальная веб-лента для голосовых публикаций с автоматической транскрибацией, русскоязычной постобработкой текста и базовыми социальными механиками.

<p align="center">
  <img alt="Python" src="https://img.shields.io/badge/Python-3.11-3776AB?style=for-the-badge&logo=python&logoColor=white">
  <img alt="FastAPI" src="https://img.shields.io/badge/FastAPI-Backend-009688?style=for-the-badge&logo=fastapi&logoColor=white">
  <img alt="React" src="https://img.shields.io/badge/React-18-61DAFB?style=for-the-badge&logo=react&logoColor=111111">
  <img alt="PostgreSQL" src="https://img.shields.io/badge/PostgreSQL-16-4169E1?style=for-the-badge&logo=postgresql&logoColor=white">
  <img alt="Docker" src="https://img.shields.io/badge/Docker-Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white">
</p>

> В коде и настройках местами осталось историческое рабочее название `Voice Twitter`: оно встречается в имени базы данных, frontend-пакета, Celery-приложения и части переменных. Пользовательское название проекта в этой документации: `Flutter`.

## Содержание

- [Коротко о проекте](#коротко-о-проекте)
- [Для кого предназначен проект](#для-кого-предназначен-проект)
- [Возможности](#возможности)
- [Технологический стек](#технологический-стек)
- [Пользовательские маршруты](#пользовательские-маршруты)
- [Архитектура](#архитектура)
- [Структура репозитория](#структура-репозитория)
- [Основные потоки данных](#основные-потоки-данных)
- [Быстрый старт через Docker](#быстрый-старт-через-docker)
- [Локальный запуск для разработки](#локальный-запуск-для-разработки)
- [Настройки окружения](#настройки-окружения)
- [Транскрибация и обработка аудио](#транскрибация-и-обработка-аудио)
- [Постобработка текста](#постобработка-текста)
- [Хранение файлов](#хранение-файлов)
- [Аутентификация и безопасность](#аутентификация-и-безопасность)
- [API](#api)
- [База данных](#база-данных)
- [Realtime-события](#realtime-события)
- [Администрирование](#администрирование)
- [Тестирование и качество кода](#тестирование-и-качество-кода)
- [Production-чеклист](#production-чеклист)
- [Частые проблемы](#частые-проблемы)


## Коротко о проекте

Flutter это full-stack приложение, в котором пользователь может записать или загрузить короткое аудио, опубликовать его в ленту, получить автоматическую транскрибацию и взаимодействовать с другими авторами через ответы, реакции, подписки и уведомления.

Главная ценность проекта в том, что голосовая публикация становится одновременно аудиозаписью и читаемым текстом. Аудио остается доступным для прослушивания, а транскрипция помогает искать записи, быстро понимать содержание и обсуждать публикации без обязательного прослушивания.

Проект состоит из трех основных частей:

- `backend`: FastAPI API, бизнес-логика, БД, аутентификация, транскрибация, realtime-события.
- `frontend`: React/Vite интерфейс социальной ленты.
- `infrastructure`: Docker Compose, PostgreSQL, Redis, Celery worker, Nginx и опциональный Flower.

## Для кого предназначен проект

| Аудитория | Что получает |
| --- | --- |
| Гость | Просмотр публичной ленты, поиск, публичные профили, регистрация и вход. |
| Пользователь | Публикация аудио, ответы, лайки, дизлайки, подписки, профиль, уведомления и настройки приватности. |
| Администратор | Панель состояния системы, просмотр жалоб, управление спорными публикациями и блокировка пользователей. |
| Разработчик | Документированную структуру backend/frontend, локальный запуск, тесты, миграции и API-контракты. |
| Эксплуатация | Docker-сборку, health/readiness checks, Redis/Celery, хранилище, Sentry и production-ограничения. |

## Возможности

| Направление | Реализовано |
| --- | --- |
| Аккаунты | Регистрация, вход, refresh-сессии, выход, выход со всех устройств, смена пароля, email/password flow. |
| Сессии | Access и refresh JWT в HttpOnly cookie, список активных сессий, отзыв отдельной сессии. |
| CSRF | Отдельная cookie `csrf_token` и заголовок `X-CSRF-Token` для небезопасных HTTP-методов. |
| Публикации | Создание голосовых постов, ответы, удаление, редактирование подписи и транскрипции, повторная транскрибация. |
| Аудио | Загрузка MP3, M4A, OGG, WebM и WAV, проверка MIME/заголовка, ограничение размера и длительности, обрезка. |
| Транскрибация | Celery worker, Faster-Whisper, русская модель по умолчанию, VAD, retry-логика, английский fallback при низкой уверенности. |
| Постобработка | Очистка мусорных токенов, автозамены, нормализация пробелов и пунктуации, опциональная LLM-коррекция. |
| Социальные функции | Лайки, дизлайки, подписки, скрытие пользователей, жалобы. |
| Поиск | Поиск по ленте, авторам, подписям и транскрипциям; в PostgreSQL используется полнотекстовый поиск. |
| Realtime | Server-Sent Events поверх Redis Pub/Sub для обновления ленты, статусов транскрибации и уведомлений. |
| Админка | Dashboard со статистикой, нагрузкой процесса, глубиной очереди, пользователями, публикациями и жалобами. |
| Хранилище | Локальное файловое хранилище для разработки и S3-совместимое хранилище для production. |
| Наблюдаемость | JSON-логи, readiness checks, Sentry для FastAPI и Celery при наличии DSN. |

## Технологический стек

| Слой | Технологии |
| --- | --- |
| Backend | Python 3.11, FastAPI, SQLAlchemy 2, Alembic, Pydantic Settings. |
| База данных | PostgreSQL 16. |
| Очереди и realtime | Redis, Celery, Redis Pub/Sub, Server-Sent Events. |
| Аудио | FFmpeg, ffprobe, Faster-Whisper. |
| Frontend | React 18, Vite, React Router, Tailwind CSS, Framer Motion. |
| Медиа-интерфейс | MediaRecorder API, Web Audio API, Canvas waveform, Lucide React icons. |
| Контейнеризация | Docker, Docker Compose, Nginx. |
| Проверки | Pytest, Vitest, Ruff, Mypy. |
| Наблюдаемость | Sentry SDK, структурированные JSON-логи. |

## Пользовательские маршруты

| Маршрут | Назначение | Доступ |
| --- | --- | --- |
| `/` | Главная лента с вкладками `Для вас` и `Подписки`. | Все, вкладка подписок полезна после входа. |
| `/search` | Поиск записей, авторов и тем. | Все. |
| `/post/:postId` | Страница публикации, родительский пост и ответы. | Все, с учетом блокировок. |
| `/profile` | Личный профиль пользователя. | Пользователь, админ. |
| `/profile/:profileId` | Публичный профиль другого пользователя. | Все, если профиль discoverable и пользователь не заблокирован. |
| `/settings` | Настройки профиля, уведомлений, приватности и пароля. | Пользователь, админ. |
| `/admin` | Панель администратора. | Только админ. |
| `/login` | Вход. | Гость. |
| `/register` | Регистрация. | Гость. |

## Архитектура

```text
Browser
  |
  | React SPA, EventSource, fetch with credentials
  v
Nginx frontend container
  |
  | /api/* and /uploads/* proxy
  v
FastAPI backend
  |       |        |
  |       |        +--> Local storage or S3-compatible storage
  |       +----------> PostgreSQL
  +------------------> Redis
                         |
                         +--> Celery worker
                         |      |
                         |      +--> FFmpeg normalization
                         |      +--> Faster-Whisper transcription
                         |      +--> rules/LLM postprocessing
                         |
                         +--> Pub/Sub channels for SSE
```

### Backend

Backend отвечает за API, доменную логику, транзакции БД, cookie-аутентификацию, CSRF, rate limits, публикацию realtime-событий и постановку задач транскрибации в очередь.

Главный модуль: `backend/app/main.py`.

Основные пакеты:

- `auth.py`: JWT, cookie, refresh-сессии, роли и зависимости FastAPI.
- `csrf.py`: double-submit CSRF protection.
- `database.py`: SQLAlchemy engine, session factory, bootstrap администратора.
- `models.py`: SQLAlchemy-модели и таблицы связей.
- `schemas.py`: Pydantic-схемы API.
- `routers/`: HTTP-маршруты.
- `transcription.py`: Celery-приложение и распознавание аудио.
- `media.py`: FFmpeg/ffprobe, нормализация и обрезка аудио.
- `storage.py`: local/S3 хранилище.
- `postprocess.py` и `replacements.py`: правила очистки транскрипции.
- `events.py`: Redis Pub/Sub публикация событий.

### Frontend

Frontend это SPA на React. Он хранит состояние аутентификации в контексте, обращается к API через `frontend/src/api/client.js`, автоматически добавляет CSRF-заголовок для небезопасных запросов и слушает SSE-поток для обновлений без ручного refresh.

Главный модуль: `frontend/src/App.jsx`.

Ключевые зоны:

- `components/PostComposer.jsx`: запись, загрузка, обрезка и отправка аудио.
- `components/PostCard.jsx`: карточка публикации, реакции, действия автора.
- `components/PostThreadPage.jsx`: отдельная публикация и ответы.
- `components/PublicProfilePage.jsx`: публичный профиль.
- `components/SettingsPage.jsx`: профиль, уведомления, приватность, пароль.
- `components/AdminDashboard.jsx`: админская панель.
- `context/AuthContext.jsx`: сессия и auth flow.
- `context/ToastContext.jsx`: уведомления интерфейса.

### Инфраструктура

`docker-compose.yml` поднимает:

- `db`: PostgreSQL 16.
- `redis`: Redis 7.
- `migrate`: одноразовый контейнер Alembic-миграций.
- `api`: FastAPI backend.
- `worker`: Celery worker для транскрибации.
- `frontend`: production-сборка React под Nginx.
- `flower`: опциональный мониторинг Celery через профиль `ops`.

## Структура репозитория

```text
Voice/
|-- backend/
|   |-- alembic/                 # миграции БД
|   |-- app/
|   |   |-- routers/             # FastAPI routers
|   |   |-- auth.py              # cookie/JWT/roles/sessions
|   |   |-- celery_app.py        # экспорт Celery app
|   |   |-- config.py            # настройки из backend/.env
|   |   |-- database.py          # SQLAlchemy и bootstrap админа
|   |   |-- events.py            # Redis Pub/Sub события
|   |   |-- main.py              # FastAPI app
|   |   |-- media.py             # FFmpeg/ffprobe
|   |   |-- models.py            # SQLAlchemy models
|   |   |-- observability.py     # Sentry
|   |   |-- postprocess.py       # правила и LLM-коррекция
|   |   |-- replacements.py      # автозамены
|   |   |-- schemas.py           # Pydantic schemas
|   |   |-- serializers.py       # преобразование ORM в API
|   |   |-- social.py            # счетчики, отношения, уведомления
|   |   |-- storage.py           # local/S3 storage
|   |   `-- transcription.py     # Celery task + Whisper
|   |-- tests/                   # pytest tests
|   |-- .env.example             # backend-настройки
|   |-- requirements.txt
|   `-- requirements-dev.txt
|-- frontend/
|   |-- public/                  # manifest, sw, иконки
|   |-- src/
|   |   |-- api/                 # API-клиент
|   |   |-- components/          # UI-компоненты
|   |   |-- context/             # React contexts
|   |   |-- utils/               # утилиты
|   |   |-- App.jsx
|   |   |-- index.css
|   |   `-- main.jsx
|   |-- nginx.conf
|   |-- package.json
|   `-- Dockerfile
|-- report/                     # отчет, диаграммы, скриншоты
|-- uploads/                    # локальные загруженные файлы
|-- .env.example                # переменные Docker Compose
|-- docker-compose.yml
|-- Dockerfile                  # backend image
|-- pyproject.toml              # pytest, ruff, mypy
`-- README.md
```

## Основные потоки данных

### Регистрация и вход

1. Frontend запрашивает `GET /api/auth/csrf` или получает CSRF cookie автоматически.
2. Пользователь отправляет email, пароль и, при регистрации, username.
3. Backend проверяет уникальность пользователя и хеширует пароль через `pwdlib[argon2]`.
4. Создается запись в `auth_sessions`.
5. Backend выставляет `access_token`, `refresh_token` и `csrf_token` в cookie.
6. Frontend получает `AuthResponse` и открывает приложение.

### Публикация голосовой записи

1. Пользователь записывает аудио в браузере или загружает файл.
2. `PostComposer` отправляет `multipart/form-data` на `POST /api/tweets/create`.
3. Backend проверяет тип файла, размер и длительность.
4. Если указана обрезка, файл проходит через FFmpeg.
5. Файл сохраняется в local/S3 хранилище.
6. В БД создается `voice_tweets` со статусом `processing`.
7. Задача транскрибации ставится в Redis/Celery.
8. Frontend сразу видит публикацию в ленте.
9. Worker нормализует аудио, запускает Whisper, очищает текст и обновляет статус.
10. Backend публикует SSE-событие `tweet.transcription_updated`.

### Ответ на публикацию

Ответы одноуровневые: можно ответить на исходный пост, но нельзя отвечать на комментарий. Ответ может содержать аудио или только текст. Если аудио нет, статус сразу становится `completed`.

### Лента и поиск

`GET /api/tweets/feed` возвращает только верхнеуровневые записи. Поддерживаются:

- `limit`: размер страницы от 1 до 100.
- `cursor_created_at` и `cursor_id`: курсор для следующей страницы.
- `q`: поиск по username, caption и transcription.
- `scope=all|following`: вся лента или авторы, на которых подписан пользователь.

Для PostgreSQL поиск использует `websearch_to_tsquery` и `to_tsvector`; в других диалектах остается `ILIKE`.

### Уведомления

Уведомления создаются для подписки, лайка, ответа и готовой транскрипции. Дизлайки не создают уведомления. Если пользователь отключил уведомления, новые уведомления для него не создаются. Realtime-доставка идет через личный Redis Pub/Sub канал пользователя и SSE.

### Жалобы и админка

Пользователь может пожаловаться на публикацию или аккаунт. Администратор видит жалобы в dashboard, может менять статус жалобы, банить пользователей и удалять публикации.

## Быстрый старт через Docker

Требования:

- Docker.
- Docker Compose.
- Свободный порт `5173` для frontend.
- Доступ в интернет при первом запуске, потому что worker скачивает модель Faster-Whisper.

Подготовьте переменные окружения:

```powershell
Copy-Item .env.example .env
Copy-Item backend\.env.example backend\.env
```

Отредактируйте секреты перед запуском:

```dotenv
POSTGRES_PASSWORD=replace-with-a-strong-postgres-password
SECRET_KEY=replace-with-a-long-random-secret-key-at-least-32-characters
ADMIN_PASSWORD=ChangeMeAdmin123!
```

Запустите проект:

```powershell
docker compose up --build
```

После запуска:

| Что | Адрес |
| --- | --- |
| Приложение | `http://localhost:5173` |
| API через frontend proxy | `http://localhost:5173/api` |
| Health check | `http://localhost:5173/api/health` |
| Ready check | `http://localhost:5173/api/ready` |
| Загруженные файлы | `http://localhost:5173/uploads/...` |

В Docker Compose сервис `api` не проброшен наружу отдельным портом. Внешние запросы идут через Nginx в контейнере `frontend`.

### Flower

Flower запускается отдельным ops-профилем:

```powershell
docker compose --profile ops up --build
```

Интерфейс Celery будет доступен на `http://localhost:5555`. Логин и пароль задаются переменной `FLOWER_BASIC_AUTH`, а если она не указана, используется небезопасное значение по умолчанию `flower:change-me-before-sharing`.

## Локальный запуск для разработки

Для запуска без Docker нужны:

- Python 3.11.
- PostgreSQL.
- Redis.
- FFmpeg и ffprobe в `PATH`.
- Node.js 20 или совместимая актуальная версия.

### Backend

Создайте виртуальное окружение и установите зависимости:

```powershell
python -m venv .venv
.venv\Scripts\activate
pip install -r backend\requirements-dev.txt
Copy-Item backend\.env.example backend\.env
```

Проверьте `backend\.env`:

```dotenv
DATABASE_URL=postgresql+psycopg://voice:voice@localhost:5432/voice_twitter
REDIS_URL=redis://localhost:6379/0
BACKEND_ORIGIN=http://localhost:8000
FRONTEND_ORIGIN=http://localhost:5173
COOKIE_SECURE=false
```

Примените миграции из папки `backend`:

```powershell
cd backend
alembic -c alembic.ini upgrade head
cd ..
```

Запустите API:

```powershell
uvicorn app.main:app --reload --app-dir backend
```

Запустите worker во втором терминале:

```powershell
celery -A app.celery_app:celery_app worker --loglevel=info --pool=solo --workdir backend
```

Локальный OpenAPI доступен при прямом запуске backend:

- Swagger UI: `http://localhost:8000/docs`
- ReDoc: `http://localhost:8000/redoc`
- OpenAPI JSON: `http://localhost:8000/openapi.json`

### Frontend

Установите зависимости и запустите Vite:

```powershell
cd frontend
npm install
npm run dev
```

Для локального Vite-режима задайте:

```dotenv
VITE_API_BASE_URL=http://localhost:8000/api
VITE_BACKEND_ORIGIN=http://localhost:8000
```

Для Docker/production-сборки используется:

```dotenv
VITE_API_BASE_URL=/api
VITE_BACKEND_ORIGIN=
```

## Настройки окружения

### Корневой `.env`

Корневой `.env` читает `docker-compose.yml`.

| Переменная | Назначение | Пример |
| --- | --- | --- |
| `POSTGRES_DB` | Имя базы данных. | `voice_twitter` |
| `POSTGRES_USER` | Пользователь PostgreSQL. | `voice` |
| `POSTGRES_PASSWORD` | Пароль PostgreSQL. | `replace-with-a-strong-postgres-password` |
| `FRONTEND_PORT` | Порт Nginx/frontend на хосте. | `5173` |
| `FLOWER_PORT` | Порт Flower на хосте. | `5555` |
| `VITE_SENTRY_DSN` | DSN Sentry для frontend-сборки. | пусто |
| `VITE_SENTRY_TRACES_SAMPLE_RATE` | Sampling frontend traces. | `0` |

### Backend `.env`

Backend-настройки определены в `backend/app/config.py` и читаются из `backend/.env`.

| Группа | Переменные |
| --- | --- |
| Приложение | `APP_NAME`, `ENVIRONMENT`, `DEBUG`, `LOG_LEVEL`, `BACKEND_ORIGIN`, `FRONTEND_ORIGIN`. |
| Наблюдаемость | `SENTRY_DSN`, `SENTRY_TRACES_SAMPLE_RATE`. |
| База и Redis | `DATABASE_URL`, `REDIS_URL`. |
| JWT | `SECRET_KEY`, `JWT_ALGORITHM`, `ACCESS_TOKEN_EXPIRE_MINUTES`, `REFRESH_TOKEN_EXPIRE_DAYS`. |
| Cookie | `ACCESS_COOKIE_NAME`, `REFRESH_COOKIE_NAME`, `CSRF_COOKIE_NAME`, `CSRF_HEADER_NAME`, `COOKIE_SECURE`, `COOKIE_SAMESITE`, `COOKIE_DOMAIN`. |
| Загрузки | `UPLOADS_DIR`, `MAX_UPLOAD_BYTES`, `MAX_AUDIO_SECONDS`. |
| Аудио | `AUDIO_ENHANCEMENT_ENABLED`, `AUDIO_HIGHPASS_HZ`, `AUDIO_LOWPASS_HZ`, `AUDIO_NOISE_REDUCTION_ENABLED`, `AUDIO_LOUDNORM_*`, `AUDIO_FFMPEG_TIMEOUT_SECONDS`. |
| Хранилище | `STORAGE_BACKEND`, `STORAGE_BUCKET`, `STORAGE_REGION`, `STORAGE_ENDPOINT_URL`, `STORAGE_ACCESS_KEY_ID`, `STORAGE_SECRET_ACCESS_KEY`, `STORAGE_PRESIGN_EXPIRE_SECONDS`. |
| Whisper | `WHISPER_MODEL_SIZE`, `WHISPER_DEVICE`, `WHISPER_COMPUTE_TYPE`, `WHISPER_LANGUAGE`, `WHISPER_BEAM_SIZE`, `WHISPER_VAD_FILTER` и остальные `WHISPER_*`. |
| Celery | `CELERY_QUEUE_NAME`, `TRANSCRIPTION_MAX_RETRIES`, `TRANSCRIPTION_RETRY_DELAY_SECONDS`, `TRANSCRIPTION_SOFT_TIME_LIMIT_SECONDS`, `TRANSCRIPTION_HARD_TIME_LIMIT_SECONDS`. |
| Постобработка | `TRANSCRIPTION_REPLACEMENTS`, `TRANSCRIPTION_POSTPROCESS_*`. |
| Rate limits | `AUTH_LOGIN_RATE_LIMIT`, `AUTH_REGISTER_RATE_LIMIT`, `TWEET_CREATE_RATE_LIMIT`, `TWEET_DELETE_RATE_LIMIT`. |
| Администратор | `ADMIN_EMAIL`, `ADMIN_USERNAME`, `ADMIN_PASSWORD`. |

Production-режим дополнительно валидирует небезопасные настройки. Приложение не стартует, если:

- `ENVIRONMENT=production`, но `SECRET_KEY` оставлен дефолтным.
- `ENVIRONMENT=production`, но `COOKIE_SECURE=false`.
- `ENVIRONMENT=production`, но `STORAGE_BACKEND=local`.
- выбран `STORAGE_BACKEND=s3`, но не заданы bucket или ключи доступа.

## Транскрибация и обработка аудио

### Ограничения загрузки

| Параметр | Значение по умолчанию |
| --- | --- |
| Поддерживаемые типы | MP3, M4A, OGG, WebM, WAV. |
| Максимальный размер | `10 MB`, через `MAX_UPLOAD_BYTES`. |
| Максимальная длительность | `300` секунд, через `MAX_AUDIO_SECONDS`. |
| Avatar upload | JPEG, PNG, WebP, GIF до `5 MB`. |

Тип аудио определяется в несколько этапов:

1. MIME-тип из загрузки.
2. Расширение файла.
3. Сигнатура первых байтов файла.

Длительность определяется через ffprobe. Если ffprobe не вернул длительность, backend пробует нормализовать аудио и проверить длительность повторно.

### Нормализация

Перед Whisper worker приводит аудио к формату:

- WAV PCM `s16le`;
- частота `16000 Hz`;
- моно;
- без видеодорожки.

Если включено улучшение аудио, FFmpeg применяет:

- high-pass фильтр;
- low-pass фильтр;
- опциональное шумоподавление `afftdn`;
- loudness normalization `loudnorm`.

Если фильтры не сработали, worker повторяет нормализацию без enhancement-фильтра.

### Whisper

Рекомендуемые настройки проекта:

```dotenv
WHISPER_MODEL_SIZE=dvislobokov/faster-whisper-large-v3-turbo-russian
WHISPER_DEVICE=cpu
WHISPER_COMPUTE_TYPE=int8
WHISPER_LANGUAGE=ru
WHISPER_BEAM_SIZE=7
WHISPER_BEST_OF=3
WHISPER_VAD_FILTER=true
WHISPER_CONDITION_ON_PREVIOUS_TEXT=false
WHISPER_INITIAL_PROMPT=
WHISPER_HOTWORDS=
WHISPER_MODEL_DIR=../.cache/whisper
```

Важные детали:

- модель загружается лениво при первой задаче;
- веса модели сохраняются в `.cache/whisper` или Docker volume `whisper_cache`;
- загрузка модели ограничена `WHISPER_LOAD_TIMEOUT_SECONDS`;
- Celery использует `worker_prefetch_multiplier=1`, `task_acks_late=true` и лимит задач на worker process;
- при пустой или низкоуверенной русской транскрипции worker может попробовать английский fallback;
- после исчерпания retry публикация получает статус `error`.

## Постобработка текста

Пайплайн транскрибации:

1. Faster-Whisper возвращает сегменты.
2. Сегменты склеиваются в один текст.
3. Удаляются не-речевые аннотации вроде `[музыка]`, `[смех]`, `[noise]`.
4. Убираются подозрительные ASR-артефакты и чрезмерные повторы букв.
5. Применяются `TRANSCRIPTION_REPLACEMENTS`.
6. Выполняется `postprocess_transcript_text`.
7. Результат сохраняется в `voice_tweets.transcription_text`.

Режимы:

| Режим | Поведение |
| --- | --- |
| `rules` | Только локальные правила. |
| `llm` | Только LLM-коррекция. |
| `rules+llm` | Сначала правила, затем LLM. |
| `llm+rules` | Сначала LLM, затем финальные правила. |

Пример:

```dotenv
TRANSCRIPTION_POSTPROCESS_ENABLED=true
TRANSCRIPTION_POSTPROCESS_MODE=rules+llm
TRANSCRIPTION_POSTPROCESS_LLM_REQUIRED=false
TRANSCRIPTION_POSTPROCESS_REPLACEMENTS=превед=привет;щас=сейчас
TRANSCRIPTION_POSTPROCESS_CAPITALIZE_SENTENCES=true
TRANSCRIPTION_POSTPROCESS_LLM_API_KEY=
TRANSCRIPTION_POSTPROCESS_LLM_BASE_URL=https://api.openai.com/v1
TRANSCRIPTION_POSTPROCESS_LLM_MODEL=
```

Если `TRANSCRIPTION_POSTPROCESS_LLM_REQUIRED=false`, отсутствие API-ключа или модели не ломает транскрибацию: LLM-этап пропускается, а локальные правила продолжают работать.

LLM-результат принимается только если он проходит базовые safety-проверки:

- текст не пустой;
- длина не отличается слишком сильно от исходной;
- URL, `@mentions` и `#hashtags` не потеряны.

Формат автозамен:

```dotenv
TRANSCRIPTION_REPLACEMENTS=ошибочный текст=правильный текст;ещё ошибка=исправление
TRANSCRIPTION_POSTPROCESS_REPLACEMENTS=превед=привет;щас=сейчас
```

## Хранение файлов

### Local

Для разработки используется:

```dotenv
STORAGE_BACKEND=local
UPLOADS_DIR=../uploads
```

Файлы сохраняются в `uploads/<user_id>/<filename>` и отдаются через `/uploads/...`.

### S3-compatible

Для production нужно S3-совместимое хранилище:

```dotenv
STORAGE_BACKEND=s3
STORAGE_BUCKET=flutter-media
STORAGE_REGION=ru-central1
STORAGE_ENDPOINT_URL=https://storage.example.com
STORAGE_ACCESS_KEY_ID=...
STORAGE_SECRET_ACCESS_KEY=...
STORAGE_PRESIGN_EXPIRE_SECONDS=3600
```

В БД сохраняется ссылка вида `s3://bucket/voice-tweets/user_id/file`. При выдаче API backend генерирует presigned URL.

## Аутентификация и безопасность

### Cookie-модель

| Cookie | HttpOnly | Назначение |
| --- | --- | --- |
| `access_token` | Да | Короткоживущий JWT для API-запросов. |
| `refresh_token` | Да | JWT для обновления access token и восстановления сессии. |
| `csrf_token` | Нет | Значение, которое frontend читает и отправляет в `X-CSRF-Token`. |

Access token живет `ACCESS_TOKEN_EXPIRE_MINUTES`, refresh token живет `REFRESH_TOKEN_EXPIRE_DAYS`.

### CSRF

Все небезопасные методы (`POST`, `PATCH`, `DELETE`) требуют совпадения:

- cookie `csrf_token`;
- заголовка `X-CSRF-Token`.

Frontend делает это автоматически в `apiFetch`.

### CORS

Backend разрешает credentials и origin из `FRONTEND_ORIGIN`. Для локальной разработки это обычно `http://localhost:5173`.

### Rate limits

Ограничения настраиваются в backend `.env`:

| Переменная | Пример |
| --- | --- |
| `AUTH_LOGIN_RATE_LIMIT` | `5/minute` |
| `AUTH_REGISTER_RATE_LIMIT` | `3/minute` |
| `TWEET_CREATE_RATE_LIMIT` | `10/minute` |
| `TWEET_DELETE_RATE_LIMIT` | `30/minute` |

Nginx дополнительно ограничивает `/api/auth/login`, `/api/auth/register`, `/api/tweets/create` и `/api/tweets/upload`.

### Роли

| Роль | Значение enum | Доступ |
| --- | --- | --- |
| Гость | нет текущего пользователя | Публичная лента, поиск, профили, login/register. |
| Пользователь | `user` | Все пользовательские действия. |
| Админ | `admin` | Пользовательские действия плюс `/admin`, ban, admin-only moderation. |

Администратор создается автоматически при старте backend, если в БД еще нет пользователя с `ADMIN_EMAIL` и `ADMIN_USERNAME`.

## API

Базовый путь API: `/api`.

В Docker запросы идут через frontend proxy:

```text
http://localhost:5173/api
```

При локальном прямом запуске backend:

```text
http://localhost:8000/api
```

Большинство пользовательских методов возвращают JSON. Создание публикаций и загрузка аватаров используют `multipart/form-data`. Auth работает через cookie, поэтому клиент должен отправлять запросы с credentials.

### Служебные маршруты

| Метод | Путь | Доступ | Назначение |
| --- | --- | --- | --- |
| `GET` | `/api/health` | Все | Проверка, что API отвечает. |
| `GET` | `/api/ready` | Все | Проверка БД, Redis и хранилища. |
| `GET` | `/api/events/stream` | Все | SSE-поток публичных и, при входе, личных событий. |

### Auth

| Метод | Путь | Body | Ответ |
| --- | --- | --- | --- |
| `GET` | `/api/auth/csrf` | нет | `CsrfTokenResponse` |
| `POST` | `/api/auth/register` | `username`, `email`, `password` | `AuthResponse`, cookie |
| `POST` | `/api/auth/login` | `email`, `password` | `AuthResponse`, cookie |
| `POST` | `/api/auth/refresh` | нет | новая пара cookie |
| `GET` | `/api/auth/session` | нет | текущий пользователь или 401 |
| `GET` | `/api/auth/sessions` | нет | список активных refresh-сессий |
| `DELETE` | `/api/auth/sessions/{session_id}` | нет | отзыв сессии |
| `POST` | `/api/auth/logout` | нет | 204 и очистка cookie |
| `POST` | `/api/auth/logout-all` | нет | отзыв всех сессий |
| `POST` | `/api/auth/change-password` | `current_password`, `new_password` | отзыв всех сессий |
| `POST` | `/api/auth/request-email-verification` | нет | token в `debug_token` только не в production |
| `POST` | `/api/auth/verify-email` | `token` | подтверждение email |
| `POST` | `/api/auth/request-password-reset` | `email` | token в `debug_token` только не в production |
| `POST` | `/api/auth/reset-password` | `token`, `password` | смена пароля |

Пример регистрации:

```json
{
  "username": "demo_user",
  "email": "demo@example.com",
  "password": "strong-password"
}
```

Username должен содержать 3-30 символов: латинские буквы, цифры и `_`.

### Публикации

| Метод | Путь | Доступ | Назначение |
| --- | --- | --- | --- |
| `GET` | `/api/tweets/feed` | Все | Лента с курсорной пагинацией, поиском и scope. |
| `GET` | `/api/tweets/{tweet_id}` | Все | Пост, родитель и ответы. |
| `POST` | `/api/tweets/create` | User/Admin | Создать голосовой пост. |
| `POST` | `/api/tweets/{tweet_id}/reply` | User/Admin | Создать ответ с аудио или текстом. |
| `PATCH` | `/api/tweets/{tweet_id}` | Автор/Admin | Изменить caption или transcription. |
| `POST` | `/api/tweets/{tweet_id}/rerun-transcription` | Автор/Admin | Повторить транскрибацию. |
| `DELETE` | `/api/tweets/{tweet_id}` | Автор/Admin | Удалить пост и файл. |
| `POST` | `/api/tweets/{tweet_id}/like` | User/Admin | Поставить лайк. |
| `DELETE` | `/api/tweets/{tweet_id}/like` | User/Admin | Убрать лайк. |
| `POST` | `/api/tweets/{tweet_id}/dislike` | User/Admin | Поставить дизлайк. Лайк и дизлайк взаимоисключающие. |
| `DELETE` | `/api/tweets/{tweet_id}/dislike` | User/Admin | Убрать дизлайк. |

`POST /api/tweets/upload` оставлен как скрытый совместимый маршрут для старого upload-клиента. Скрытые `POST /api/tweets/{tweet_id}/repost` и `DELETE /api/tweets/{tweet_id}/repost` тоже остались для legacy-клиентов, но сейчас они вызывают ту же логику, что и дизлайк. В текущем интерфейсе пользователь ставит именно дизлайк, не репост.

Поля `multipart/form-data` для создания поста:

| Поле | Тип | Обязательно | Назначение |
| --- | --- | --- | --- |
| `audio` | file | Да для нового поста | Аудиофайл. |
| `caption` | string | Нет | Подпись до 500 символов. |
| `parent_tweet_id` | int | Нет | Родительский пост, если создается ответ. |
| `trim_start_seconds` | float | Нет | Начало обрезки. |
| `trim_end_seconds` | float | Нет | Конец обрезки. |

Пример ответа `VoiceTweetRead`:

```json
{
  "id": 42,
  "audio_url": "/uploads/1/example.webm",
  "duration_seconds": 12.4,
  "caption": "Короткая заметка",
  "transcription_text": "Короткая заметка голосом.",
  "status": "completed",
  "mime_type": "audio/webm",
  "error_message": null,
  "likes_count": 3,
  "dislikes_count": 0,
  "reposts_count": 0,
  "reply_count": 1,
  "liked_by_viewer": false,
  "disliked_by_viewer": false,
  "reposted_by_viewer": false,
  "created_at": "2026-05-20T10:00:00Z",
  "parent_tweet_id": null,
  "user": {
    "id": 1,
    "username": "demo_user",
    "bio": null,
    "avatar_url": null,
    "role": "user",
    "is_following": false
  }
}
```

В ответе пока остаются legacy-поля `reposts_count` и `reposted_by_viewer`. Они дублируют счетчик и состояние дизлайков из старой схемы данных и не означают, что в текущем продукте есть пользовательская функция репоста.

Статусы публикации:

| Статус | Значение |
| --- | --- |
| `processing` | Аудио принято, транскрибация идет или будет повторена. |
| `completed` | Текст готов или публикация является текстовым ответом. |
| `error` | Worker не смог обработать аудио после retry. |

### Профиль, пользователи и настройки

| Метод | Путь | Доступ | Назначение |
| --- | --- | --- | --- |
| `GET` | `/api/profile` | User/Admin | Свой профиль и свои публикации. |
| `PATCH` | `/api/profile` | User/Admin | Изменить `bio` или `avatar_url`. |
| `POST` | `/api/profile/avatar` | User/Admin | Загрузить аватар. |
| `GET` | `/api/settings/preferences` | User/Admin | Настройки уведомлений и видимости. |
| `PATCH` | `/api/settings/preferences` | User/Admin | Обновить настройки. |
| `GET` | `/api/users/search` | Все | Поиск discoverable пользователей. |
| `GET` | `/api/users/suggestions` | Все | Рекомендации авторов. |
| `GET` | `/api/users/{user_id}` | Все | Публичный профиль. |
| `POST` | `/api/users/{user_id}/follow` | User/Admin | Подписаться. |
| `DELETE` | `/api/users/{user_id}/follow` | User/Admin | Отписаться. |
| `POST` | `/api/users/{user_id}/block` | Admin | Заблокировать пользователя от имени админа. |
| `DELETE` | `/api/users/{user_id}/block` | User/Admin | Снять свою блокировку. |
| `POST` | `/api/users/{user_id}/mute` | User/Admin | Скрыть пользователя из своей ленты. |
| `DELETE` | `/api/users/{user_id}/mute` | User/Admin | Вернуть пользователя в ленту. |
| `POST` | `/api/reports` | User/Admin | Создать жалобу на пользователя или пост. |
| `PATCH` | `/api/users/{user_id}/ban` | Admin | Забанить или разбанить пользователя. |

### Уведомления

| Метод | Путь | Доступ | Назначение |
| --- | --- | --- | --- |
| `GET` | `/api/notifications` | User/Admin | Последние уведомления и `unread_count`. |
| `POST` | `/api/notifications/{notification_id}/read` | User/Admin | Пометить одно уведомление прочитанным. |
| `POST` | `/api/notifications/read-all` | User/Admin | Пометить все уведомления прочитанными. |

Текущие пользовательские сценарии создают такие типы уведомлений:

- `follow`
- `like`
- `reply`
- `transcription_ready`

В enum модели остается значение `repost` для совместимости со старым названием, но новые дизлайки не создают `repost`-уведомления.

### Админка

| Метод | Путь | Доступ | Назначение |
| --- | --- | --- | --- |
| `GET` | `/api/admin/dashboard` | Admin | Статистика, нагрузка, пользователи, публикации и жалобы. |
| `PATCH` | `/api/admin/reports/{report_id}` | Admin | Обновить статус жалобы. Скрыт из OpenAPI. |

Параметры `GET /api/admin/dashboard`:

- `users_limit`, `users_offset`
- `tweets_limit`, `tweets_offset`
- `reports_limit`, `reports_offset`
- `user_q`

## База данных

Основные таблицы:

| Таблица | Назначение |
| --- | --- |
| `users` | Аккаунты, роли, профиль, настройки уведомлений и приватности. |
| `voice_tweets` | Публикации, ответы, аудио, длительность, подпись, транскрипция, статус. |
| `auth_sessions` | Refresh-сессии, user agent, IP, дата отзыва. |
| `notifications` | Уведомления пользователя. |
| `reports` | Жалобы на пользователей и публикации. |
| `follows` | Подписки. |
| `tweet_likes` | Лайки. |
| `tweet_reposts` | Историческое имя таблицы; в текущей логике хранит дизлайки. |
| `user_blocks` | Блокировки. |
| `user_mutes` | Скрытие пользователей из ленты. |

Миграции лежат в `backend/alembic/versions`. При старте Docker Compose сначала выполняется сервис `migrate`, затем запускается API.

После миграций `init_db()` проверяет наличие таблицы `users` и создает администратора из `ADMIN_EMAIL`, `ADMIN_USERNAME`, `ADMIN_PASSWORD`.

## Realtime-события

SSE endpoint: `GET /api/events/stream`.

Канал состоит из:

- публичного Redis-канала `voice_x:events:public`;
- личного Redis-канала `voice_x:events:user:{user_id}`, если пользователь авторизован.

Служебные события:

| Событие | Назначение |
| --- | --- |
| `ready` | Поток открыт. |
| `heartbeat` | Keep-alive каждые примерно 15 секунд без событий. |

Доменные события:

| Событие | Когда публикуется |
| --- | --- |
| `tweet.created` | Создан пост или ответ. |
| `tweet.reply_created` | Создан ответ на пост. |
| `tweet.deleted` | Удалена публикация. |
| `tweet.engagement_updated` | Изменились лайки или дизлайки. |
| `tweet.transcription_updated` | Изменился статус или текст транскрибации. |
| `notification.created` | Создано личное уведомление. |

Пример SSE-сообщения:

```text
event: tweet.transcription_updated
data: {"type":"tweet.transcription_updated","timestamp":"2026-05-20T10:00:00Z","tweet_id":42,"status":"completed","user_id":1}
```

## Администрирование

Админ-панель находится на `/admin` и требует роль `admin`.

Dashboard показывает:

- общее количество пользователей;
- количество публикаций;
- количество публикаций в `processing`;
- количество забаненных пользователей;
- количество открытых жалоб;
- CPU и память процесса API;
- глубину Celery-очереди;
- активную Whisper-модель и device;
- последние пользователи, публикации и жалобы.

Админ bootstrap:

```dotenv
ADMIN_EMAIL=admin@voice-tweet.com
ADMIN_USERNAME=admin
ADMIN_PASSWORD=ChangeMeAdmin123!
```

Перед демонстрацией или production-запуском обязательно замените `ADMIN_PASSWORD`.

## Тестирование и качество кода

### Backend

```powershell
pip install -r backend\requirements-dev.txt
pytest
python -m compileall backend\app
ruff check .
mypy backend
```

`pyproject.toml` задает:

- `pythonpath = ["backend"]`;
- тесты в `backend/tests`;
- Ruff правила `E`, `F`, `I`, `B`;
- Mypy для Python 3.11 с исключением `frontend` и `backend/alembic`.

### Frontend

```powershell
npm install --prefix frontend
npm test --prefix frontend
npm run build --prefix frontend
```

Vitest работает в `jsdom`, setup-файл: `frontend/src/test/setup.js`.

### Docker

```powershell
docker compose config
docker compose up --build
```

Для проверки готовности:

```powershell
Invoke-RestMethod http://localhost:5173/api/ready
```

## Production-чеклист

Перед production:

- Установить `ENVIRONMENT=production`.
- Заменить `SECRET_KEY` на случайный секрет длиной минимум 32 символа.
- Включить `COOKIE_SECURE=true`.
- Использовать HTTPS.
- Настроить `FRONTEND_ORIGIN` и `BACKEND_ORIGIN` на реальные origin.
- Использовать `STORAGE_BACKEND=s3`, а не local.
- Заменить `ADMIN_PASSWORD`.
- Задать сильный `POSTGRES_PASSWORD`.
- Настроить резервное копирование PostgreSQL и object storage.
- Проверить размер Docker volumes `postgres_data`, `uploads_data`, `whisper_cache`.
- Настроить `SENTRY_DSN` при необходимости.
- Ограничить внешний доступ к Flower или не запускать профиль `ops`.
- Проверить, что `.env` и `backend/.env` не попадают в git.

## Частые проблемы

### Первая транскрибация идет долго

Первый запуск скачивает модель Faster-Whisper в `.cache/whisper` или volume `whisper_cache`. Следующие запуски обычно быстрее.

### Worker падает с ошибкой FFmpeg

Проверьте, что `ffmpeg` и `ffprobe` установлены и доступны в `PATH`. В Docker они устанавливаются в backend image автоматически.

### Frontend не видит backend

Для Docker:

```dotenv
VITE_API_BASE_URL=/api
VITE_BACKEND_ORIGIN=
```

Для локального Vite:

```dotenv
VITE_API_BASE_URL=http://localhost:8000/api
VITE_BACKEND_ORIGIN=http://localhost:8000
```

Также проверьте `FRONTEND_ORIGIN` в `backend/.env`.

### Cookie не сохраняются локально

Для локального HTTP:

```dotenv
COOKIE_SECURE=false
COOKIE_SAMESITE=lax
```

Для HTTPS production:

```dotenv
COOKIE_SECURE=true
COOKIE_SAMESITE=lax
```

### Запросы возвращают `Invalid CSRF token`

Сначала вызовите:

```http
GET /api/auth/csrf
```

Затем для `POST`, `PATCH` и `DELETE` отправляйте значение cookie `csrf_token` в заголовке:

```http
X-CSRF-Token: <csrf_token>
```

Frontend делает это автоматически.

### LLM-постобработка не запускается

Нужно указать:

```dotenv
TRANSCRIPTION_POSTPROCESS_LLM_API_KEY=...
TRANSCRIPTION_POSTPROCESS_LLM_MODEL=...
TRANSCRIPTION_POSTPROCESS_LLM_BASE_URL=https://api.openai.com/v1
```

Если `TRANSCRIPTION_POSTPROCESS_LLM_REQUIRED=false`, отсутствие этих переменных не считается ошибкой.

### Русская транскрибация стала хуже

Проверьте, что не заданы неподходящие `WHISPER_INITIAL_PROMPT` и `WHISPER_HOTWORDS`. Для текущей русской модели они намеренно пустые.

### Production не стартует

Проверьте валидаторы в `backend/app/config.py`. Чаще всего причина одна из следующих:

- дефолтный `SECRET_KEY`;
- `COOKIE_SECURE=false`;
- `STORAGE_BACKEND=local`;
- неполная S3-конфигурация.

