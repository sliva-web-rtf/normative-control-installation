# normative-control-installation

Пример конфигурации для запуска «В Норме!» через Docker Compose.

## Сервисы

| Сервис     | Образ                                                               | Назначение                                     |
| ---------- | ------------------------------------------------------------------- | ---------------------------------------------- |
| `gateway`  | `nginx:1.30.4-alpine`                                               | Точка входа: `/api/*` в `api`, остальное во `frontend` |
| `frontend` | `normocontrol.registry.twcstorage.ru/normative-control-onprem-web`  | Веб-интерфейс                                  |
| `api`      | `normocontrol.registry.twcstorage.ru/normative-control-onprem-api`  | Backend                                        |
| `checker`  | `normocontrol.registry.twcstorage.ru/normative-control-onprem-core` | Проверка документов                            |
| `postgres` | `postgres:18.6`                                                     | База данных                                    |
| `rabbitmq` | `rabbitmq:4.4.0-alpine`                                         | Очередь проверок                               |
| `minio`    | `bitnamilegacy/minio:2025.4.22`                                     | Хранилище файлов                               |

## Запуск

```bash
docker login normocontrol.registry.twcstorage.ru
cp .env.example .env
docker compose up -d --wait
```

Сервис доступен на `http://127.0.0.1:8080`.

## OIDC-клиент

Вход выполняется через Identity Provider по OpenID Connect. Требования к
клиенту:

- Authorization Code flow с PKCE `S256`, confidential-клиент с секретом;
- redirect URI: `<HOST_URL>/api/auth/oidc/callback`;
- post logout redirect URI: `<HOST_URL>/`;
- discovery IdP содержит `end_session_endpoint`, иначе `api` не стартует;
- в ID token или userinfo передаётся claim `person` (допускается внутри claim
  `user`) с полями `guid`, `id`, `email`, `lastName`, `firstName` и
  необязательным `middleName`;
- необязательный claim `roles`: институт и группа берутся из роли с
  `userType: Student` (`instituteTitle`, `instituteId`, `groupTitle`,
  `groupId`).

## Переменные окружения

Значения задаются в `.env`. Адреса PostgreSQL, RabbitMQ и S3 по умолчанию
указывают на сервисы `postgres`, `rabbitmq` и `minio` из `docker-compose.yml`.

### Приложение

| Переменная               | По умолчанию           | Описание                                                       |
| ------------------------ | ---------------------- | -------------------------------------------------------------- |
| `IMAGE_TAG`              | обязательна            | Тег образов `frontend`, `api`, `checker`, например `v1.0.0`    |
| `HOST_URL`               | обязательна            | Публичный адрес сервиса без `/` в конце                        |
| `JWT_SECRET`             | обязательна            | Ключ подписи сессий, Base64 не короче 32 байт                  |
| `APP_BIND`               | `127.0.0.1`            | Адрес, на котором слушает `gateway`                            |
| `APP_PORT`               | `8080`                 | Порт `gateway`                                                 |
| `DOCUMENT_CHECK_WORKERS` | `1`                    | Число параллельных проверок в `checker`                        |

### OIDC

| Переменная           | По умолчанию           | Описание                                       |
| -------------------- | ---------------------- | ---------------------------------------------- |
| `OIDC_ISSUER_URI`    | обязательна            | Issuer Identity Provider                       |
| `OIDC_CLIENT_ID`     | обязательна            | Идентификатор OIDC-клиента                     |
| `OIDC_CLIENT_SECRET` | обязательна            | Секрет OIDC-клиента                            |
| `OIDC_SCOPES`        | `openid,profile,email` | Запрашиваемые scope через запятую              |
| `OIDC_ADMIN_EMAILS`  | пусто                  | Email администраторов через запятую            |

### PostgreSQL

| Переменная          | По умолчанию                                         | Описание                  |
| ------------------- | ---------------------------------------------------- | ------------------------- |
| `POSTGRES_URL`      | `jdbc:postgresql://postgres:5432/normative-control`  | JDBC-адрес базы для `api` |
| `POSTGRES_USER`     | `postgres`                                           | Пользователь базы         |
| `POSTGRES_PASSWORD` | обязательна                                          | Пароль пользователя       |

### RabbitMQ

| Переменная          | По умолчанию   | Описание                                                    |
| ------------------- | -------------- | ----------------------------------------------------------- |
| `RABBITMQ_HOST`     | `rabbitmq`     | Хост брокера                                                |
| `RABBITMQ_PORT`     | `5672`         | Порт AMQP                                                   |
| `RABBITMQ_USERNAME` | `normocontrol` | Пользователь                                                |
| `RABBITMQ_PASSWORD` | обязательна    | Пароль, без символов, требующих экранирования в URL         |

### S3

| Переменная      | По умолчанию        | Описание                                                  |
| --------------- | ------------------- | --------------------------------------------------------- |
| `S3_URL`        | `http://minio:9000` | Адрес хранилища для `api`, со схемой                      |
| `S3_ENDPOINT`   | `minio:9000`        | Адрес того же хранилища для `checker`, без схемы          |
| `S3_SECURE`     | `false`             | `true`, если `checker` подключается к хранилищу по HTTPS  |
| `S3_REGION`     | `us-east-1`         | Регион                                                    |
| `S3_BUCKET`     | `normocontrol`      | Бакет                                           |
| `S3_ACCESS_KEY` | обязательна         | Ключ доступа                                              |
| `S3_SECRET_KEY` | обязательна         | Секретный ключ |

## Требования к инфраструктуре

- PostgreSQL 16 или новее, пустая база. Схему создаёт `api` при старте.
- RabbitMQ 4 или новее. Очереди `document.request` и `document.result`
  создаются автоматически.
- S3-совместимое хранилище с заранее созданным бакетом.
