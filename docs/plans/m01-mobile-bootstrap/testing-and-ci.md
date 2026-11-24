# Тести й CI

Як перевіряються мобільні пункти і як збираються застосунки.

## Рівні тестів

| Рівень | Інструмент | Що |
|---|---|---|
| Логіка й хуки | **Jest** | API-шар, мапінг помилок, форматування, сховища |
| Компоненти | **React Native Testing Library** | Примітиви, екрани з моками API |
| E2E | **Maestro** | Основні флоу на варіанті `testing` проти бекенду в Compose |

- Test-first: AC мобільного пункту пишеться тестом до реалізації.
- Моки API — з типів `packages/api-client`, щоб змінений контракт ламав тести.

## Іменування AC

- Кожен AC мобільного пункту покривається тестом з назвою **`AC_<US>_<N>_...`**: `it('AC_MBOOT_2_1_...')` у Jest/RNTL або файл флоу `AC_MBOOT_2_1_....yaml` у Maestro.
- UX-чекліст ([mobile-ux](../../mobile-ux.md#чекліст-для-m-пункту)) — частина AC кожного M-пункту: ліміти тапів перевіряються Maestro-флоу.

## CI (GitHub Actions)

На кожен PR, що зачіпає `mobile/` чи `packages/`:

- `pnpm lint`, `pnpm typecheck`, Jest з покриттям;
- перевірка, що всі ключі текстів з коду є у вихідних `i18n/mobile/*.json` ([B4](../04-i18n/ui-strings.md#цикл-ключів));
- перевірка, що згенерований клієнт і схеми zod актуальні (як `make gen` у [B1](../01-backend-bootstrap/tooling-and-ci.md)).

**Maestro в CI** — з [M2](../../roadmap.md#мобільний-застосунок), коли з'явиться перший флоу: збірка `testing`, емулятор, бекенд у Compose з `fakeidp` і сидом.

## EAS Build

| Профіль | Коли | Куди |
|---|---|---|
| `development` | Вручну | Розробникам (dev client) |
| `testing` | У CI для Maestro | Лише CI |
| `staging` | Кожен merge у `main` | TestFlight / Internal testing |
| `production` | Тег `mobile-v*` | App Store / Google Play |

- Версія застосунку — semver з тегу; build number росте автоматично.
- Source maps — у Sentry для `staging` і `production`.
- Секрети підпису — в EAS, не в репозиторії.
