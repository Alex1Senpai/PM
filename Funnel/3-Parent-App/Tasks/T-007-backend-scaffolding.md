---
type: task
id: T-007
title: "Bootstrap backend: Dart-сервер с Postgres, миграции, CI/CD, health check"
spec: "Funnel/3-Parent-App/Specs/payments.md"
funnel-stage: "3-Parent-App"
status: pending
priority: urgent
assignee: Alex
estimate: large
created: 2026-05-25
target-done: 2026-06-21
code_repo: "github.com/thebekhruz/core-mobile-app — папка server/ или отдельный repo (Alex решает в D-008)"
commit_initial: ""
commit_final: ""
---

# T-007 — Bootstrap backend: Dart-сервер с Postgres, миграции, CI/CD, health check

## Контекст
После T-005 (D-008 принят) и T-006 (1C API исследован) Алекс строит реальный backend. Это позвоночник всего, что происходит в S-003 и далее. Без работающего backend ни одна последующая задача (платежи, attendance, messaging) не может начаться.

Цель: к концу S-002 у нас есть **живой backend в продакшен-окружении** с health check, подключённый к managed Postgres, с автоматическим деплоем при пуше в `main`. Никакой бизнес-логики — только инфраструктура.

**Спецификация:** [[../Specs/payments]]
**Связанные решения:**
- [[../Decisions/D-006-backend-stack]] — Dart + Postgres
- [[../Decisions/D-008-server-framework]] — *появится после T-005*
**Стадия воронки:** 3-Parent-App

## Что нужно сделать

1. **Решить расположение backend-кода:**
   - Вариант A: добавить папку `server/` в `core-mobile-app` repo (моноре)
   - Вариант B: создать отдельный repo `core-mobile-app-backend`
   - Решение делает Алекс. Записать в D-008 или в отдельный мини-D.

2. **Инициализировать Dart-сервер** на выбранном фреймворке (D-008). Структура папок — стандарт фреймворка плюс:
   - `lib/api/v1/` — routes
   - `lib/services/` — business logic
   - `lib/db/` — models + migrations
   - `lib/integrations/` — 1C, SMS, payment provider, push
   - `lib/util/` — общие утилиты
   - `test/` — unit + integration tests

3. **Подключить PostgreSQL:**
   - Локально: `docker-compose.yml` с одним сервисом `postgres` (для разработки)
   - В продакшене: managed Postgres у выбранного hosting (D-011) — DigitalOcean Managed Postgres, AWS RDS, Render Postgres, или эквивалент
   - Connection через env vars (`DATABASE_URL`)

4. **Настроить миграции БД:**
   - Выбрать tool: `drift`, `postgres` + ручные SQL, `serverpod_orm` (если Serverpod) — это D-012
   - Создать первую миграцию: пустая, чтобы убедиться что toolchain работает
   - Команда `dart run migrate` локально применяет миграции

5. **Реализовать health check:**
   - `GET /api/v1/health` → JSON `{"status":"ok","version":<git-sha>,"db":"up"|"down","ts":<unix>}`
   - Должен быть честным: если БД недоступна — `db:"down"`, status 503
   - Никакой авторизации на health check (он используется монитором)

6. **Реализовать базовый middleware:**
   - Request logging (метод, путь, статус, время ответа)
   - Error handler (любая необработанная ошибка → JSON 500, не stacktrace в response)
   - CORS (для будущей dev-работы с Flutter web, если потребуется)

7. **Конфигурация через env vars:**
   - `DATABASE_URL`
   - `PORT` (default 8080)
   - `ENV` (`dev` | `staging` | `prod`)
   - Чтение через стандартный механизм Dart (`Platform.environment`)
   - Локально — `.env` файл (не коммитить, `.env.example` коммитить)

8. **CI через GitHub Actions:**
   - На каждый PR: `dart format --set-exit-if-changed`, `dart analyze`, `dart test`
   - Билд должен быть зелёный, чтобы PR можно было влить
   - При merge в `main` — автоматический деплой в продакшен hosting

9. **CD на выбранный hosting (D-011):**
   - Деплой триггерится при пуше в `main`
   - После деплоя — автоматический health check, если не отвечает 200 за 60с → откат
   - Логи доступны через hosting dashboard

10. **Документация:**
    - `server/README.md`: как запустить локально, как добавить миграцию, как деплоить
    - Endpoint catalog (пока — только `/health`)
    - Env vars каталог

## Определение готовности (Definition of Done)
- [ ] D-011 (hosting) принят и записан в `Decisions/`
- [ ] D-012 (migration tool) принят и записан в `Decisions/`
- [ ] Backend репозиторий/папка существует, README объясняет setup
- [ ] `docker-compose up` поднимает локальный Postgres
- [ ] `dart run` запускает сервер локально, `curl localhost:8080/api/v1/health` возвращает OK с `db:"up"`
- [ ] Миграция применена, видна в Postgres (`SELECT * FROM schema_migrations;`)
- [ ] CI настроен: PR с форматной/analyze/test ошибкой не вливается
- [ ] CD настроен: push в `main` деплоит автоматически
- [ ] `https://<production-url>/api/v1/health` отвечает 200 с `db:"up"`
- [ ] PR template и CODEOWNERS на backend repo/папке настроены
- [ ] Файл задачи закоммичен через PR, статус в frontmatter = `done`, раздел «Завершение» заполнен

## Что уже есть / на что можно опираться
- D-006 (стек), D-008 (framework), D-011 (hosting), D-012 (migrations) — все решены до начала
- T-001 параллельно идёт, AuthService там моковый — backend ещё не нужен ему
- Hosting account (DigitalOcean / Render / AWS) — Bekhruz обеспечивает доступ

## План тестирования
- **Ручные проверки:**
  - `curl localhost:8080/api/v1/health` — OK, `db:"up"`
  - Останови Postgres → `curl` возвращает 503 с `db:"down"`
  - Push PR с `print` или `TODO` в коде → CI красный
  - Push в `main` → деплой запускается → через минуту production health check OK
- **Автотесты:**
  - Unit-тест на health endpoint (mock DB up/down)
  - Integration test: поднять in-memory или ephemeral Postgres, прогнать health через реальное подключение
- **Граничные случаи:**
  - Что если DATABASE_URL пуст? → сервер не стартует, логирует понятную ошибку
  - Что если миграция падает? → сервер не стартует, логирует ошибку
  - Что при rollback hosting (если поддерживает)? → описать в README процедуру

## Зависимости / блокируется
- Зависит от: T-005 закрыт (D-008 принят)
- Зависит от: D-011 (hosting) и D-012 (migration tool) приняты Алексом
- **Блокирует:** T-008 (Real SMS integration), T-013 (Webhook + payment integration), T-015 (Attendance read из Toddle через backend)

## Вне scope
- Никакой бизнес-логики (платежи, attendance) — только инфраструктура
- Никаких auth endpoints — это T-008
- Никакого admin UI — backend только REST, никакого frontend в этой задаче
- Никаких production secrets в репозитории — только через env vars / hosting secrets manager

## Завершение
> (пусто до завершения)

## История изменений
- 2026-05-25 — создано
