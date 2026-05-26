---
type: task
id: T-005
title: "Discovery + Decision D-008 — конкретный server-framework на Dart (Serverpod / Shelf / Conduit)"
spec: "Funnel/3-Parent-App/Specs/payments.md"
funnel-stage: "3-Parent-App"
status: pending
priority: urgent
assignee: Alex
estimate: medium
created: 2026-05-25
target-done: 2026-06-10
code_repo: "Funnel/3-Parent-App/Architecture/ — для прототипов"
commit_initial: ""
commit_final: ""
---

# T-005 — Decision D-008: какой server-framework на Dart выбираем для backend

## Контекст
D-006 зафиксировал: backend пишем на Dart с PostgreSQL. Но **конкретный server-framework на Dart** оставлен Алексу — это D-008. Сейчас D-008 не написан, и без него Алекс не может начать S-002 (Backend bootstrap).

Это **самое важное архитектурное решение S-002**. От него зависит:
- Скорость разработки backend (Serverpod даёт generated client для Flutter — экономия недель; Shelf даёт минимум магии — больше кода руками)
- Operational complexity (что деплоить, как мониторить, как мигрировать БД)
- Hire-ability в будущем (всё ниша, но насколько ниша)

Алекс делает прототип, выбирает, пишет D-008. Не «исследует месяц» — за 5 дней принимает решение и фиксирует.

**Спецификация:** [[../Specs/payments]]
**Связанные решения:**
- [[../Decisions/D-005-client-tech-stack]] — Flutter
- [[../Decisions/D-006-backend-stack]] — Dart на сервере + Postgres
**Стадия воронки:** 3-Parent-App

## Что нужно сделать

1. **Сделать короткий обзор трёх вариантов** (по полдня каждый, не больше):
   - **Serverpod** — самый Flutter-first, generated client, встроенный auth/push/storage. Минусы: молодой проект, больше магии, навязывает архитектуру.
   - **Shelf** — минимальный HTTP-фреймворк от Dart team. Прозрачный, гибкий. Минусы: всё руками, никакой генерации.
   - **Conduit** (бывший Aqueduct) — middleweight, явный, ORM встроен. Минусы: проект имеет долгие периоды без обновлений.

2. **Сделать «hello world» прототип на каждом из 3 вариантов** в новой временной папке (не в репозитории Parent App). Каждый прототип должен иметь:
   - Один GET endpoint `/api/v1/health` → возвращает JSON `{"status":"ok","ts":<unix>}`
   - Один POST endpoint `/api/v1/auth/otp/request` → принимает `{phone:String}`, возвращает `{ok:true}` (моковый)
   - Connection к локальному Postgres (docker-compose файл с одним сервисом postgres)
   - Базовый logging
   - README с командой запуска

   Цель — почувствовать скорость и эргономику. Не строить production.

3. **Для каждого варианта оценить:**
   - Сколько строк кода ушло на прототип
   - Сколько времени заняло (часов, честно)
   - Где «трение» — что было неприятно, что бы не хотелось делать каждый день
   - Что Serverpod-generated client даёт vs ручная синхронизация моделей

4. **Написать Decision-файл `D-008-server-framework.md`** в [[../Decisions/]]. Использовать `_Templates/Decision.md`. Включить:
   - Контекст
   - 3 рассмотренных варианта со своими плюсами/минусами
   - Выбор с обоснованием
   - Принимаемые риски
   - Открытые подвопросы (например, выбор ORM/migration tool — D-012)

5. **Дать Bekhruz прочитать D-008.** Bekhruz не будет переоценивать твою техническую оценку — он только проверит, что причины выбора понятны и риски названы. Если Bekhruz задаёт вопрос, на который ты не подумал — обнови D-008.

6. **Удалить прототипы** после фиксации решения (или коммитнуть в `Architecture/spike-server-framework/` с тегом spike — не основной код).

## Определение готовности (Definition of Done)
- [ ] Прототипы на 3 frameworks существовали и работали (можно показать)
- [ ] `D-008-server-framework.md` написан в `Decisions/` по шаблону `_Templates/Decision.md`
- [ ] Bekhruz прочитал D-008 и принял (status: `accepted` в frontmatter)
- [ ] Раздел «Открытые подвопросы» в D-008 заполнен (минимум: выбор migration tool, выбор hosting)
- [ ] `_Hub.md` обновлён: D-008 добавлен в список решений со статусом ✅ accepted
- [ ] Файл задачи закоммичен через PR, статус в frontmatter = `done`, раздел «Завершение» заполнен

## Что уже есть / на что можно опираться
- D-005 и D-006 фиксируют ограничения (Dart + Postgres)
- T-001 идёт параллельно — там Flutter-клиент, который потом будет общаться с этим backend
- Не нужно учиться чему-то новому — Алекс уже Dart-эксперт

## План тестирования
- **Ручные проверки:**
  - `curl localhost:8080/api/v1/health` для каждого прототипа → JSON ответ
  - `curl -XPOST localhost:8080/api/v1/auth/otp/request -d '{"phone":"+998901234567"}'` → `{"ok":true}`
  - Прототип переподключается к Postgres после restart docker
- **Граничные случаи:**
  - Что происходит при невалидном JSON в POST?
  - Что в логах при ошибке подключения к Postgres?
  - Hot reload работает? (Скорость итерации важна.)

## Зависимости / блокируется
- Не блокируется (Алекс уже умеет Dart)
- **Блокирует:** T-007 (Backend scaffolding) — без D-008 нет чем scaffold'ить

## Вне scope
- Не нужно строить production-ready backend — только прототипы
- Не нужно решать вопрос migration tool (D-012) или hosting (D-011) — это отдельные decisions
- Не нужно ставить рекорд по чистоте кода в прототипах — это spikes, не production

## Завершение
> (пусто до завершения)

## История изменений
- 2026-05-25 — создано
