# M1 · Бутстрап мобільного застосунку

Каркас мобільного застосунку на Expo: стек, API-шар, рядки інтерфейсу, варіанти збірки, тести й CI. Бізнес-функцій тут немає — лише те, на чому будуються M2–M12.

Критерії приймання — у [юзер-сторі](user-stories.md).

## Обсяг

- Expo (dev builds), Expo Router, TypeScript strict, структура `mobile/` у монорепо ([B1](../01-backend-bootstrap/structure.md)).
- TanStack Query з persist-кешем, Zustand, FormatJS, власні UI-примітиви на токенах, темна тема.
- react-hook-form + zod зі схемами з OpenAPI.
- `expo-secure-store`, Sentry.
- API-шар: заголовки, помилки, екран оновлення на `426`.
- Пакет `ui_strings` з кешем і вбудованим запасним варіантом ([B4](../04-i18n/ui-strings.md)).
- Варіанти `development`, `testing`, `staging`, `production`.
- Jest, RNTL, Maestro, CI, EAS Build.
- Екрани каркаса: ping, перемикач теми, оновлення, тестовий краш, бібліотека примітивів.

**Не входить**: вхід, `401` і refresh ([M2](../../roadmap.md#мобільний-застосунок)), офлайн-сховище й черга операцій ([M6](../../roadmap.md#мобільний-застосунок), за [B14](../14-offline-sync/client.md)), прапорці, `feature.*` і OTA ([M10](../../roadmap.md#мобільний-застосунок)).

## Документи

- [Стек](stack.md) — навігація, стан, i18n, UI-примітиви й токени, форми, сховище, крашлітика.
- [API й варіанти](api-and-variants.md) — API-шар, `ui_strings`, варіанти застосунку, мінімальні ОС.
- [Тести й CI](testing-and-ci.md) — Jest, RNTL, Maestro, іменування AC, CI, EAS Build.
- [Юзер-сторі](user-stories.md) — US-MBOOT-1…7.

## Ключові рішення

| Рішення | Відкинуто | Чому |
|---|---|---|
| Expo Router | React Navigation напряму | Діплінки й universal links «з коробки»; під капотом той самий React Navigation |
| TanStack Query + persist у MMKV | Лише онлайн-кеш | Решта застосунку показує кеш без мережі ([UX](../../mobile-ux.md)) |
| Офлайн-сутності окремо в SQLite | Усе в кеші запитів | Журнал операцій B14 потребує власного сховища |
| Zustand | Лише React context | Мінімум коду, без зайвих перерендерів |
| FormatJS | i18next-icu | ICU — рідний формат `ui_strings`, без адаптерів |
| Власні примітиви на токенах | NativeWind, Tamagui, Paper | Токени стануть основою дизайн-системи; повний контроль над UX-вимогами |
| zod зі схем OpenAPI | Ручні схеми | Валідація клієнта збігається з контрактом |
| Sentry RN SDK | Окремий SDK під GlitchTip | Той самий SDK працює з обома; вибір у P2 не блокує |
| Варіант `testing` для автотестів | Окремий хостований QA | Детерміновані e2e без зайвої інфраструктури |
| Maestro для e2e | Detox | Прості YAML-флоу, стабільний на Expo |
