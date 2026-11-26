# Обновление с v1.0.0 до v2.0.0

## `.env`

- Установите `IMAGE_TAG=v2.0.0`.
- Замените `OIDC_ADMIN_EMAILS` на `OIDC_OWNER_EMAILS` и укажите через запятую
  почты пользователей, которым нужна роль владельца организации с максимальным доступом.
- Задайте `OIDC_INSTITUTION_NAME`, например `OIDC_INSTITUTION_NAME=УрФУ`.

## `docker-compose.yml`

В `services.api.environment` замените `OIDC_ADMIN_EMAILS` на следующие строки.

```yaml
OIDC_OWNER_EMAILS: ${OIDC_OWNER_EMAILS:-}
OIDC_INSTITUTION_NAME: ${OIDC_INSTITUTION_NAME:?OIDC_INSTITUTION_NAME must be set}
```
