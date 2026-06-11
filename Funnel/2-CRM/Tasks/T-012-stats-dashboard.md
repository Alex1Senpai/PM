---
type: task
id: T-012
title: "Реализовать пятничный дашборд статистики: stats-модуль на сервере + виджет amoCRM"
spec: "Funnel/2-CRM/Specs/stats-dashboard.md"
funnel-stage: "2-CRM"
status: in_progress
priority: high
assignee: "Unassigned"
estimate: "large (3-5 дней)"
created: 2026-06-11
target-done: 2026-06-20
code_repo: "remix-of-oxbridge + amocrm-widget"
commit_initial: "ff5b52f"
commit_final: "0baec30 (remix-of-oxbridge), e0c4b76 (amocrm-widget)"
---

# T-012 — Реализовать пятничный дашборд статистики

## Контекст
Пятничный разбор команды приёма должен открываться в один клик внутри amoCRM и показывать работу команды в процентах: сквозная конверсия «заявка → зачислен», гигиена лидов/задач, строка на менеджера, сравнение кампусов МЮ/ЯШ, причины закрытия, медианное время в стадии. Расчёт — в существующем сервере `remix-of-oxbridge` (модуль `stats`), отображение — новый виджет amoCRM (`amocrm-widget`). Полные бизнес-правила — в спецификации.

**Спецификация:** [[Funnel/2-CRM/Specs/stats-dashboard.md]]
**Стадия воронки:** 2-CRM

## Что нужно сделать
1. Миграция `0007_stats_dashboard.sql`: таблицы `stats_weekly_snapshot`, `stats_cache`.
2. Экстракторы amoCRM (leads/tasks/users/pipelines/loss_reasons/events) с троттлером ≤7 rps и пагинацией.
3. Чистый модуль расчёта метрик #1–#10 (TDD на фикстурах).
4. Роуты `GET /stats/report`, `POST /stats/refresh` (Bearer `STATS_API_TOKEN`), `GET /stats/cron` (`requireOpsToken`) + крон в `vercel.json`.
5. Виджет amoCRM: manifest, script.js (AMD), i18n ru/en, рендер 4 блоков + ◀▶ недели + «⟳ обновить» + ⚠-маркеры, сборка в zip.
6. Тесты сервера зелёные; e2e-проверка вручную по чек-листу спеки.

## Определение готовности (Definition of Done)
- [ ] Все критерии приёмки спеки выполнены (см. spec → «Критерии приёмки»).
- [ ] `npm exec nx run server:test` зелёный, новые расчётные функции покрыты юнит-тестами.
- [ ] Виджет собирается в `widget.zip` одной командой.
- [ ] Цифры сходятся с ручной проверкой по 2–3 лидам.

## Что уже есть / на что можно опираться
- OAuth-клиент amoCRM: `server/src/services/amoCRMService.ts` (`getValidAccessToken()`), токены в Neon.
- Паттерны: `getPool()` (`server/src/utils/postgresPool.ts`), `requireOpsToken` (`middleware/opsAuth.ts`), `safeTokenEquals` (`middleware/auth.ts`), крон-пример `/ops/monitor` + `vercel.json`.
- Известные ID полей: кампус 996203 (MU 1223825 / ЯШ 1223823), программа 991841 — `server/src/config/amocrm.ts`.

## План тестирования
- **Ручные проверки:** открыть виджет → отчёт < 2 сек; создать/закрыть задачу в amoCRM → refresh → метрики #3/#4 изменились; сверить конверсию по 2–3 лидам.
- **Автотесты:** vitest-юниты на расчёт (фикстуры), supertest на auth роутов (401 без токена, 503 крон в prod без секрета), юнит на троттлер.
- **Граничные случаи:** нет снимка прошлой недели (WoW «—»), events недоступны (velocity со старой датой), пустая воронка, менеджер без лидов.

## Зависимости / блокируется
- Реальные `STATS_ENROLLED_STATUS_IDS` / `STATS_PIPELINE_IDS` от Bekhruz при деплое (до этого — автообнаружение по имени статуса).

## Вне scope
- Поэтапная воронка по менеджеру, запись в amoCRM, realtime, рассылки, отдельный сайт-дашборд (см. спеку).

## Завершение
> (пусто до завершения)

## История изменений
- 2026-06-11 — создано из спеки stats-dashboard (брейншторм Claude + Bekhruz).
