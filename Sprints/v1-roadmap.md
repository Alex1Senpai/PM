---
type: note
title: "Parent App v1 — 12-week roadmap (25 May → 15 Aug 2026)"
funnel-stage: "3-Parent-App"
status: living
owner: Bekhruz
created: 2026-05-25
updated: 2026-05-25
target-ship: 2026-08-15
---

# Parent App v1 — 12-week roadmap

> Календарный план до жёсткого дедлайна 15 августа 2026. 12 недель, 6 двухнедельных спринтов. Команда: Alex (middle, full-time) + Timur (intern, ~50%). Управляющий принцип: каждый спринт обязан закрыть один риск или поставить одну поверхность.
>
> Этот документ — **не план задач**, а **карта последовательности**. Конкретика по задачам живёт в `Funnel/3-Parent-App/Tasks/T-NNN-*.md` и в файлах спринтов `Sprints/S-NNN-*.md`.

---

## TL;DR

| Sprint | Daty | Цель | Что мы доказываем |
|---|---|---|---|
| **S-001** | 25 май – 7 июн | Foundation | Flutter-проект собирается на iOS и Android. SMS-провайдер выбран. D-008 + D-009 закрыты. |
| **S-002** | 8 – 21 июн | Backend bootstrap | Dart-сервер с Postgres работает. Реальный SMS-флоу в auth. 1C-API специфицирован. Toddle API исследован. |
| **S-003** | 22 июн – 5 июл | Payments backend | Webhook от Click и PayMe принимаются, идемпотентность работает, push в 1C идёт. Reconciliation cron крутится. |
| **S-004** | 6 – 19 июл | Payments UI + Attendance | Родитель может оплатить через приложение → чек → запись в истории. Attendance read-only из Toddle. |
| **S-005** | 20 июл – 2 авг | Passes + Messaging + Meals | Три оставшиеся поверхности интегрированы. К концу спринта приложение функционально готово. |
| **S-006** | 3 – 15 авг | Push + Beta + Polish | FCM/APNs работают. Внутренний бета на 10–20 семьях. Багфикс. Готовы к 1 сентября. |

**Жёсткий дедлайн запуска: 15 августа.** Между 15 августа и 1 сентября — 2 недели на исправление багов и обучение бухгалтерии.

---

## Принципы планирования

1. **Discovery всегда впереди разработки.** Если SDK/API не исследован — это блокер, не задача. Timur ведёт discovery, Alex строит на её результатах.
2. **Скорость > красота.** Внутренний UI можно перерисовать в v1.1. Платёжный flow должен работать с первого билда.
3. **Только Alex закрывает критический путь.** Timur не пишет код, от которого зависят сроки. Его работа — discovery, контракты, тесты, документация. Это снижает риск интерна на критическом пути.
4. **Каждая поверхность v1 имеет одного владельца на спринт.** Никаких «оба работают над платежами». Один пишет, второй ревьюит и пишет тесты.
5. **Honest budget.** Капасити срезается на 20% от того, что кажется возможным. Никаких stretch-задач, пока коммитментная работа реалистична.

---

## Критический путь до 15 августа

```
T-001 (Alex) Flutter+auth scaffolding [S-001]
   └─→ T-007 Backend scaffolding [S-002]
         └─→ T-011 Data model design [S-002]
               └─→ T-012/013 Payment integrations [S-003]
                     └─→ T-014 Payment UI in Flutter [S-004]
                           └─→ T-019 Push notifications [S-006]
                                 └─→ T-020 Internal beta [S-006]
```

Это позвоночник. Любая задержка здесь = задержка релиза. Всё остальное (attendance, passes, messaging, meals) — параллельные ветви, которые можно пожертвовать или сдвинуть в v1.1 если позвоночник опаздывает.

**Что отрезаем первым, если опаздываем:**
1. Meals (легко закрывается «фото меню в Telegram» в первые недели)
2. Messaging (можно временно оставить Telegram-каналы школы)
3. Hikvision passes (родители проживают без этого, у них уже есть звонок учителя)

**Что не отрезаем:** auth, payments, attendance. Без них приложение не имеет смысла.

---

## S-001 — Foundation (25 май – 7 июн)

**Цель:** К концу спринта Flutter-проект собирается на iOS и Android, выбран SMS-провайдер и платёжная стратегия (агрегатор vs прямой Click+PayMe).

**Что должно закрыться:**
- T-001 Flutter scaffolding + auth (моковая) — Alex, large, *уже в работе*
- T-002 SMS provider discovery — Timur, medium, *уже в работе*
- T-003 Review contract: пункт о штрафе 0.4%/день — Timur, small
- T-004 Discovery: payment aggregator vs Click+PayMe direct (закрывает D-009) — Timur, medium

**Что должно появиться в Decisions/:**
- D-008 — Server framework на Dart (Serverpod / Shelf / Conduit) — *Alex, в рамках T-001/T-005*
- D-009 — Прямая интеграция vs агрегатор — *Bekhruz, после T-004*

**Риски:**
- SMS-провайдеры в Узбекистане медленно отвечают на запросы тестовых ключей → закладываем 5 рабочих дней.
- Если Timur не может получить контракт у юриста — T-003 встаёт.

Полная детализация → [[S-001-foundation]]

---

## S-002 — Backend bootstrap (8 – 21 июн)

**Цель:** К концу спринта работает Dart-сервер с PostgreSQL, реальный SMS-флоу в auth заменяет моковый, и Timur приносит готовую discovery 1C-API и Toddle API.

**Что должно закрыться:**
- T-005 D-008 Server framework decision (Alex) — small (одно решение + 1 PR декларирующий выбор)
- T-007 Backend scaffolding: проект Dart-сервера, health check, Postgres, миграции, CI/CD — Alex, large
- T-008 Real SMS integration: замена мокового AuthService на выбранного провайдера — Alex, medium
- T-006 Discovery 1C API: что доступно, как авторизоваться, формат данных — Timur (с участием 1C-программиста школы), medium
- T-009 Discovery Toddle API: какие данные о посещаемости, какой контракт — Timur, medium

**Что должно появиться в Decisions/:**
- D-011 — Hosting backend (DigitalOcean App Platform / Render / Lightsail) — *Alex*
- D-012 — Migration tool для Postgres (drift / postgres / serverpod_orm) — *Alex*

**Риски:**
- 1C-программист недоступен → T-006 встаёт. Контр-мера: Bekhruz договаривается о слоте в начале спринта.
- Serverpod не подходит → нужно переключиться на Shelf, теряем 2–3 дня.

---

## S-003 — Payments backend (22 июн – 5 июл)

**Цель:** К концу спринта родитель (теоретически — через curl или Postman) может сделать тестовый платёж через выбранный платёжный путь, webhook приходит на наш backend, запись в Postgres появляется, push в 1C проходит, в 1C запись отображается.

**Что должно закрыться:**
- T-011 Data model design: User, Family, Child, Charge, Payment, Receipt — Alex, medium
- T-012 Payment integration (зависит от D-009: либо Click+PayMe прямо, либо агрегатор) — Alex, large
- T-013 Webhook receiver + idempotency + push-to-1C — Alex, large
- T-NNN Daily reconciliation cron + logging — Alex, medium *(если успеваем — иначе в S-004)*
- T-010 Discovery Hikvision API: события входа/выхода, формат — Timur, medium

**Что должно появиться в Specs/:**
- `Specs/attendance.md` — финальная версия (закрыто после T-009)
- `Specs/passes-hikvision.md` — финальная версия (закрыто после T-010)
- *Bekhruz пишет — это spec, не код*

**Риски:**
- Click/PayMe долго оформляют мерчант-аккаунт школы → нужно начать переговоры в S-002.
- 1C push-механизм оказывается несуществующим → срочный fallback на ручной CSV-импорт в первые недели.

---

## S-004 — Payments UI + Attendance (6 – 19 июл)

**Цель:** К концу спринта родитель с тестовым аккаунтом может открыть приложение, увидеть баланс, оплатить через приложение, получить чек. Attendance читается из Toddle и отображается в карточке ребёнка.

**Что должно закрыться:**
- T-014 Payment UI в Flutter: экран оплаты, выбор провайдера, история, чек — Alex, large
- T-015 Attendance read из Toddle (через backend) + UI — Alex, medium
- T-NNN Receipt PDF генерация + email — Alex/Timur, medium
- T-NNN Unit + widget тесты на payments flow — Timur, medium

**Что должно появиться в Specs/:**
- `Specs/messaging.md` — финальная версия — *Bekhruz*
- `Specs/meals-schedule.md` — финальная версия — *Bekhruz*
- `Specs/home.md` — финальная версия экрана главной — *Bekhruz*

**Риски:**
- Toddle API не отдаёт реалтайм-посещаемость → читаем с задержкой N часов, UX-предупреждение.
- Бухгалтерия не успевает завести merchant-аккаунт → откладываем real payments на S-005.

---

## S-005 — Passes + Messaging + Meals (20 июл – 2 авг)

**Цель:** К концу спринта приложение функционально полное. Все 6 поверхностей v1 работают end-to-end в staging.

**Что должно закрыться:**
- T-016 Hikvision pass events: интеграция backend + UI карточки «✅ Вошёл в школу 8:42» — Alex, large
- T-017 Messaging school→parent + базовый ответ родителя — Alex, large
- T-018 Meals: фото меню загружается ресепшеном, отображается в карточке — Alex (или Timur с ревью Alex), medium
- T-NNN Settings экран (профиль, выход, биометрия on/off) — Timur, small
- T-NNN End-to-end QA по всем 6 поверхностям — Timur, medium

**Риски:**
- Hikvision-сервер на стороне школы не отдаёт webhooks → переход на pull-режим, потеря real-time.
- Messaging spec не готов к началу спринта → встаёт на 3+ дня.

---

## S-006 — Push + Beta + Polish (3 – 15 авг)

**Цель:** К 15 августа приложение в App Store и Google Play в режиме TestFlight / Internal Testing. 10–20 семей школы пользуются им ежедневно. К 15 августа все P0/P1 баги закрыты.

**Что должно закрыться:**
- T-019 FCM + APNs push notifications: оплата, просрочка, новое сообщение — Alex, large
- T-020 Internal beta: подбор 10–20 семей, onboarding, feedback channel — Bekhruz + Alex, medium
- T-NNN Стор-листинги (скриншоты, описание, политика конфиденциальности) — Bekhruz + Alex, medium
- T-NNN Багфикс + полировка по результатам беты — Alex+Timur, ongoing

**Risk reserve:** последние 3 дня (12–15 авг) — буфер на непредвиденное. Никаких новых задач в эти дни. Только багфикс.

**Что НЕ делается в S-006:**
- Никаких новых фич
- Никаких архитектурных изменений
- Никаких «давайте всё-таки добавим X» от стейкхолдеров

---

## Сводная таблица задач

| ID | Название | Owner | Sprint | Status |
|---|---|---|---|---|
| T-001 | Flutter scaffolding + auth | Alex | S-001 | in progress |
| T-002 | SMS provider discovery | Timur | S-001 | in progress |
| T-003 | Contract review: пункт о штрафе | Timur | S-001 | pending |
| T-004 | Discovery: payment aggregator vs direct | Timur | S-001 | pending |
| T-005 | D-008 server framework decision | Alex | S-002 | pending |
| T-006 | Discovery: 1C API | Timur+1C-prog | S-002 | pending |
| T-007 | Backend scaffolding (Dart + Postgres + CI) | Alex | S-002 | pending |
| T-008 | Real SMS integration | Alex | S-002 | pending |
| T-009 | Discovery: Toddle API | Timur | S-002 | pending |
| T-010 | Discovery: Hikvision API | Timur | S-003 | not yet created |
| T-011 | Data model design | Alex | S-003 | not yet created |
| T-012 | Payment integration | Alex | S-003 | not yet created |
| T-013 | Webhook receiver + idempotency | Alex | S-003 | not yet created |
| T-014 | Payment UI Flutter | Alex | S-004 | not yet created |
| T-015 | Attendance: read из Toddle + UI | Alex | S-004 | not yet created |
| T-016 | Hikvision pass events | Alex | S-005 | not yet created |
| T-017 | Messaging school→parent | Alex | S-005 | not yet created |
| T-018 | Meals: upload + display | Alex/Timur | S-005 | not yet created |
| T-019 | Push notifications FCM/APNs | Alex | S-006 | not yet created |
| T-020 | Internal beta with 10–20 families | Bekhruz+Alex | S-006 | not yet created |

**Tasks created this session: T-003 → T-008** (детали в `Funnel/3-Parent-App/Tasks/`).
**Tasks T-010 → T-020:** появятся в Hub'е к началу соответствующих спринтов, по мере того как закрываются предшествующие спецификации.

---

## Что ещё должно произойти параллельно

### Specs, которые Bekhruz пишет
| Spec | Дедлайн | Зависит от |
|---|---|---|
| `Specs/attendance.md` | конец S-003 | T-009 (Toddle discovery) |
| `Specs/passes-hikvision.md` | конец S-003 | T-010 (Hikvision discovery) |
| `Specs/messaging.md` | конец S-004 | — |
| `Specs/meals-schedule.md` | конец S-004 | — |
| `Specs/home.md` | конец S-004 | все остальные specs (это шапка) |

### Decisions, которые ещё нужно закрыть
| ID | Title | Owner | Когда |
|---|---|---|---|
| D-008 | Server framework на Dart | Alex | S-001/S-002 |
| D-009 | Direct vs aggregator | Bekhruz (после T-004) | S-001 |
| D-011 | Hosting backend | Alex | S-002 |
| D-012 | Postgres migration tool | Alex | S-002 |
| D-013 | Push notification stack (FCM-only / OneSignal) | Alex | S-005 |
| D-014 | Student-parent-family data model source | Bekhruz+Alex | S-003 |

---

## Сигналы тревоги (не игнорировать)

| Сигнал | Что это значит | Что делать |
|---|---|---|
| К концу S-001 не выбран SMS-провайдер | T-002 встал | Bekhruz делает звонок Eskiz/PlayMobile напрямую |
| К концу S-002 backend не собирается локально | D-008 / hosting под вопросом | Перенести 2 дня S-003 на доделку backend |
| К концу S-003 webhook от Click не приходит | Платёжный путь не закрыт | Внутренний staging с моком + параллельная эскалация Click |
| К концу S-004 нет реального тестового платежа | Релиз 15 августа под вопросом | Срезаем v1: убираем messaging или meals |
| К S-005 spec'ов нет в Specs/ | Bekhruz не успевает писать | Привлечь второго мозга, или вырезать поверхность из v1 |

---

## История изменений
- 2026-05-25 — создано. Базис: _Hub.md + end-state-v1 + 8 принятых Decision'ов. Описывает 6 спринтов до 15 августа с критическим путём, рисками и сигналами тревоги.
