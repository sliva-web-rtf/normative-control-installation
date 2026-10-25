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
cp .env.example .env   # заполните под себя, см. «Переменные окружения»
docker compose up -d --wait
```

Сервис доступен на `http://127.0.0.1:8080`.

## OIDC-клиент

Вход выполняется через Identity Provider по OpenID Connect. Требования к
клиенту:

- Authorization Code flow с PKCE `S256`, confidential-клиент с секретом;
- redirect URI: `<HOST_URL>/api/auth/oidc/callback`;
- post logout redirect URI: `<HOST_URL>/`;
- в ID token или userinfo передаётся claim `person` (допускается внутри claim
  `user`) с полями `guid`, `id`, `email`, `lastName`, `firstName` и
  необязательным `middleName`;
- необязательный claim `roles`: институт и группа берутся из роли с
  `userType: Student` (`instituteTitle`, `instituteId`, `groupTitle`,
  `groupId`).

## Переменные окружения

Все переменные перечислены в `.env.example`. Адреса PostgreSQL, RabbitMQ и S3
в нём указывают на сервисы `postgres`, `rabbitmq` и `minio` из
`docker-compose.yml`.

### Приложение

| Переменная               | В `.env.example`                  | Описание                                                    |
| ------------------------ | --------------------------------- | ----------------------------------------------------------- |
| `IMAGE_TAG`              | `v1.0.0`                          | Тег образов `frontend`, `api`, `checker`                    |
| `HOST_URL`               | `https://normocontrol.example.ru` | Публичный адрес сервиса без `/` в конце                     |
| `APP_BIND`               | `127.0.0.1`                       | Адрес, на котором слушает `gateway`                         |
| `APP_PORT`               | `8080`                            | Порт `gateway`                                              |
| `DOCUMENT_CHECK_WORKERS` | `1`                               | Число параллельных проверок в `checker`                     |
| `JWT_SECRET`             | пусто                             | Ключ подписи сессий, Base64 не короче 32 байт               |

### OIDC

| Переменная           | В `.env.example`                        | Описание                            |
| -------------------- | --------------------------------------- | ----------------------------------- |
| `OIDC_ISSUER_URI`    | `https://idp.example.ru/realms/example` | Issuer Identity Provider            |
| `OIDC_CLIENT_ID`     | пусто                                   | Идентификатор OIDC-клиента          |
| `OIDC_CLIENT_SECRET` | пусто                                   | Секрет OIDC-клиента                 |
| `OIDC_SCOPES`        | `openid,profile,email`                  | Запрашиваемые scope через запятую   |
| `OIDC_ADMIN_EMAILS`  | `admin@example.ru,ivanov@example.ru`    | Email администраторов через запятую |

### PostgreSQL

| Переменная          | В `.env.example`                                    | Описание                  |
| ------------------- | --------------------------------------------------- | ------------------------- |
| `POSTGRES_URL`      | `jdbc:postgresql://postgres:5432/normative-control` | JDBC-адрес базы для `api` |
| `POSTGRES_USER`     | `postgres`                                          | Пользователь базы         |
| `POSTGRES_PASSWORD` | пусто                                               | Пароль пользователя       |

### RabbitMQ

| Переменная               | В `.env.example`   | Описание                                |
| ------------------------ | ------------------ | --------------------------------------- |
| `RABBITMQ_HOST`          | `rabbitmq`         | Хост брокера                            |
| `RABBITMQ_PORT`          | `5672`             | Порт AMQP                               |
| `RABBITMQ_USERNAME`      | `normocontrol`     | Пользователь                            |
| `RABBITMQ_PASSWORD`      | пусто              | Пароль: только латинские буквы и цифры  |
| `RABBITMQ_REQUEST_QUEUE` | `document.request` | Очередь заданий на проверку             |
| `RABBITMQ_RESULT_QUEUE`  | `document.result`  | Очередь результатов проверки            |

### S3

| Переменная      | В `.env.example`    | Описание                                                 |
| --------------- | ------------------- | -------------------------------------------------------- |
| `S3_URL`        | `http://minio:9000` | Адрес хранилища для `api`, со схемой                     |
| `S3_ENDPOINT`   | `minio:9000`        | Адрес того же хранилища для `checker`, без схемы         |
| `S3_SECURE`     | `false`             | `true`, если `checker` подключается к хранилищу по HTTPS |
| `S3_REGION`     | `us-east-1`         | Регион                                                   |
| `S3_BUCKET`     | `normocontrol`      | Бакет                                                    |
| `S3_ACCESS_KEY` | `normocontrol`      | Ключ доступа                                             |
| `S3_SECRET_KEY` | пусто               | Секретный ключ                                           |

## Требования к инфраструктуре

- PostgreSQL 16 или новее, пустая база. Схему создаёт `api` при старте.
- RabbitMQ 4 или новее. Очереди из `RABBITMQ_REQUEST_QUEUE` и
  `RABBITMQ_RESULT_QUEUE` создаются автоматически.
- S3-совместимое хранилище с заранее созданным бакетом.
