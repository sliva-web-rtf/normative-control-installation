# Обновление с v1.0.0 до v2.0.0

## `.env`

- Установите `IMAGE_TAG=v2.0.0`.
- Замените `OIDC_ADMIN_EMAILS` на `OIDC_OWNER_EMAILS` и укажите через запятую
  адреса пользователей, которым нужна роль владельца организации.
- Задайте `OIDC_INSTITUTION_NAME`, например `OIDC_INSTITUTION_NAME=УрФУ`.
  В режиме OIDC название вуза обязательно.

## `docker-compose.yml`

В `services.api.environment` замените `OIDC_ADMIN_EMAILS` на следующие строки.

```yaml
OIDC_OWNER_EMAILS: ${OIDC_OWNER_EMAILS:-}
OIDC_INSTITUTION_NAME: ${OIDC_INSTITUTION_NAME:?OIDC_INSTITUTION_NAME must be set}
```

При запуске API миграция V26 переименовывает `faculties` в `institutes`
и `users.faculty_id` в `users.institute_id`.
