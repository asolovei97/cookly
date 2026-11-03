# Структура

Як організовано монорепо і бекенд-код.

## Монорепо

```
backend/              — Go-модуль: API і воркер
contracts/            — OpenAPI-контракт, розбитий за доменами
packages/api-client/  — згенерований TS-клієнт (openapi-typescript + openapi-fetch)
admin/                — React (з'явиться в A1)
mobile/               — Expo (з'явиться в M1)
docs/
pnpm-workspace.yaml, Makefile, compose.yaml
```

JS-частина — **pnpm workspaces**. *Відкинуто*: bun — працює як пакетний менеджер, але в монорепо з React Native більше проблем з нативними модулями і Metro; deno — RN не підтримується.

## Бекенд

```
backend/
├── cmd/api/            — HTTP API, команда migrate
├── cmd/worker/         — воркер River
├── api/                — згенерований з контракту код (типи, strict-інтерфейс)
├── migrations/         — goose, спільні на всю БД, вбудовані через embed
└── internal/
    ├── platform/       — config, db, httpx, log, observability, queue
    ├── auth/           — Principal, хелпери контексту, middleware (у B1 лише заглушка)
    └── <домен>/        — household, catalog, recipes…
```

Пакети організовані **за доменами**, а не за шарами.

## Домен

На прикладі `household`:

| Файл | Роль |
|---|---|
| `handler.go` | Реалізує свою частину strict-інтерфейсу: HTTP ↔ сервіс |
| `service.go` | Бізнес-правила, транзакції |
| `store.go` | Обгортка над кодом sqlc |
| `queries.sql` | SQL для sqlc; згенерований код — у підпакеті `db/` |
| `errors.go` | Коди помилок `household.*` |
| `*_test.go` | Тести, зокрема `TestAC_HH_…` |

- Домени **не читають таблиці один одного** напряму, а звертаються до сервісу іншого домену через вузький інтерфейс.
- Сервер API — композиція обробників доменів, яка реалізує згенерований strict-інтерфейс.

## Контекст і Principal

- Auth-middleware перевіряє токен і кладе `Principal` у контекст запиту під неекспортованим ключем.
- Пакет `auth` дає типізовані хелпери `auth.User(ctx) (Principal, bool)` і `auth.MustUser(ctx) Principal`.
- Strict-обробник отримує `ctx` першим аргументом, тож користувач читається одним рядком:
  ```go
  func (h *Handler) GetMe(ctx context.Context, req api.GetMeRequestObject) (api.GetMeResponseObject, error) {
      me := auth.MustUser(ctx)
      ...
  }
  ```
- Той самий `ctx` іде в сервіс і sqlc, а slog-хендлер дописує з нього `request_id` і `user_id` у кожен лог.
- У B1 `Principal` ще не заповнюється; токени з'являться в B2.

## Конфіг

- Лише змінні оточення (12-factor), парсинг `caarlos0/env`, файл `.env` для локальної розробки.
- `ENV` = `development` | `staging` | `production`; інше значення — помилка.
- Валідація на старті: помилковий конфіг → процес завершується з помилкою.
- Для `staging` і `production` діють суворіші перевірки: налаштування, дозволені лише для розробки, там заборонені.

## Версія Go

Остання стабільна, зафіксована в `go.mod` через `toolchain`. Інструменти (oapi-codegen, sqlc, goose, golangci-lint, air) зафіксовані через `go tool`.
