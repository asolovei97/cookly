# Інструменти й CI

Як розробник запускає проєкт локально і що перевіряє CI.

## Makefile

| Ціль | Дія |
|---|---|
| `make up` / `make down` | Підняти чи зупинити інфраструктуру в Docker (PostgreSQL, Redis, Garage) |
| `make up-obs` | Те саме плюс профіль `observability` |
| `make migrate` | Застосувати міграції |
| `make dev` | API і воркер нативно з hot reload (`air`) |
| `make run` | Те саме без hot reload |
| `make gen` | Уся кодогенерація: oapi-codegen, sqlc, TS-клієнт |
| `make lint` | golangci-lint, Redocly lint |
| `make test` | Тести (testcontainers-go) |

## Локальна розробка

- **Docker у development — лише для інфраструктури.** API і воркер запускаються нативно (`make dev`).
- Інструменти Go зафіксовані через `go tool`, тож окремо їх ставити не треба.
- JS-залежності — `pnpm install` у корені.

## Тести

- Юніт-тести і API-тести через `httptest` + згенерований клієнт на справжніх PostgreSQL і Redis.
- Інфраструктура тестів — **testcontainers-go**: тести самодостатні і однаково запускаються локально й у CI.
- Кожен AC з [юзер-сторі](user-stories/index.md) покривається тестом `TestAC_<US>_<N>_...`, наприклад `TestAC_BOOT_1_1_PingWithoutToken`.

## Docker-образ

- **Один образ** для `staging` і `production` з двома бінарниками: `api` і `worker`; команда запуску обирає, що запускати.
- Багатоетапна збірка, фінальний образ — distroless.

## CI (GitHub Actions)

Окремі job-и з фільтрами за шляхами:

| Job | Перевіряє |
|---|---|
| `backend` | golangci-lint, тести |
| `contracts` | Redocly lint, збирання контракту, `oasdiff` проти контракту попереднього релізу: несумісні зміни в `/v1` → помилка |
| `codegen` | Після `make gen` немає змін у git |
| `image` | Збирання Docker-образу (без пушу) |

Публікація образу в реєстр — у [P2](../../roadmap.md#платформа), після вибору хостингу.
