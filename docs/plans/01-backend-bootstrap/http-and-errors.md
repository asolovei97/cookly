# HTTP і помилки

Контракт API, кодогенерація, формат помилок і правила, спільні для всіх ендпоінтів.

## Контракт

- **OpenAPI 3.0**: oapi-codegen підтримує 3.1 лише частково.
- Файли розбиті за доменами в `contracts/`, збираються в один через **Redocly CLI**, перевіряються **Redocly lint**.
- Конвенції: префікс `/v1`, JSON у camelCase, час у RFC 3339 (UTC), ідентифікатори — UUIDv7.

## Кодогенерація

| Що | Інструмент | Куди |
|---|---|---|
| Go-сервер (типи, strict-інтерфейс) | oapi-codegen | `backend/api/` |
| TS-клієнт | openapi-typescript + openapi-fetch | `packages/api-client/` |
| Доступ до БД | sqlc | `backend/internal/<домен>/db/` |

- Згенерований код **комітиться**, у `.gitattributes` він позначений `linguist-generated=true`, тож у рев'ю згортається.
- Конфлікт злиття в згенерованому коді вирішується повторною генерацією (`make gen`).
- CI перевіряє, що після `make gen` немає змін.

## Формат помилок

`Content-Type: application/problem+json` (за мотивами RFC 9457):

```json
{
  "code": "auth.invalid_token",
  "message": "token signature is invalid",
  "requestId": "01J…",
  "errors": [
    { "pointer": "/displayName", "code": "auth.invalid_name", "message": "…" }
  ]
}
```

- `code` — стабільний, у форматі `<домен>.<причина>`. Клієнт показує локалізований текст з `ui_strings` за кодом.
- `message` — англійською, для розробників.
- `errors[]` — помилки окремих полів, `pointer` — JSON Pointer.

## Базові коди

| Ситуація | HTTP | `code` |
|---|---|---|
| Тіло не розбирається як JSON | 400 | `request.malformed` |
| Запит не відповідає контракту | 422 | `request.invalid` |
| Невідомий маршрут | 404 | `request.not_found` |
| Метод не підтримується | 405 | `request.method_not_allowed` |
| Немає або невалідний access-токен | 401 | `auth.unauthorized` |
| Необроблена помилка або panic | 500 | `internal.error` |

## Безпека за замовчуванням

- У контракті глобально задано `bearerAuth`. Публічні операції явно позначаються `security: []`.
- oapi-codegen кладе в контекст маркер `BearerAuthScopes` для операцій із `security`. Auth-middleware за ним визначає, що операція захищена, без окремого списку маршрутів.
- У B1 токенів ще немає, тож будь-яка захищена операція повертає `401 auth.unauthorized`. У B2 middleware почне перевіряти токени.

## Request ID

- Клієнтський `X-Request-ID` приймається, лише якщо він ≤ 64 символів і складається з `[A-Za-z0-9-_]`. Інакше генерується новий.
- Повертається в заголовку `X-Request-ID` і в полі `requestId` помилок, пишеться в кожен лог.

## Ping

`GET /v1/ping` (`security: []`) → `200 {"status": "ok"}`.

## Коректна зупинка

Після `SIGTERM` сервер перестає приймати нові з'єднання й чекає, поки завершаться запити, які вже виконуються (з таймаутом з конфігу).
