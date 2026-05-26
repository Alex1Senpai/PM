---
type: task
id: T-008
title: "Заменить моковый AuthService реальной интеграцией с выбранным SMS-провайдером"
spec: "Funnel/3-Parent-App/Specs/payments.md"
funnel-stage: "3-Parent-App"
status: pending
priority: high
assignee: Alex
estimate: medium
created: 2026-05-25
target-done: 2026-06-21
code_repo: "github.com/thebekhruz/core-mobile-app — server/lib/integrations/sms.dart + client/lib/services/auth_service.dart"
commit_initial: ""
commit_final: ""
---

# T-008 — Заменить моковый AuthService реальной интеграцией с выбранным SMS-провайдером

## Контекст
T-001 уже сделал AuthService в Flutter с моковой отправкой OTP. T-002 (Timur) к началу S-002 принёс выбор SMS-провайдера (Eskiz / PlayMobile / другой). T-007 (Alex) к середине S-002 поднял работающий backend.

Эта задача соединяет три провода: Flutter-клиент → наш backend → SMS-провайдер → телефон родителя → код возвращается в приложение → User создаётся в нашем backend.

Авторизация — фундамент. Без неё ни одна последующая задача (платежи, attendance) не работает.

**Спецификация:** [[../Specs/payments]] (косвенно — auth для всего)
**Связанные решения:**
- [[../Decisions/D-004-auth-model]] — Phone OTP + биометрия + мульти-роль
- [[../Decisions/D-006-backend-stack]] — Dart backend
**Стадия воронки:** 3-Parent-App

## Что нужно сделать

### Backend (server/)

1. **SMS-интеграция в `lib/integrations/sms.dart`:**
   - Класс `SmsProvider` (abstract) с методом `Future<bool> send(String phone, String message)`
   - Реализация `EskizSmsProvider` (или какого провайдера выбрали в T-002) — конкретный REST-клиент
   - Креды через env vars (`SMS_API_KEY`, `SMS_SENDER_ID`)
   - Тайм-аут 10с, ретраи 2 раза с exponential backoff

2. **OTP-сервис в `lib/services/otp_service.dart`:**
   - `Future<String> requestOtp(String phone)` — генерирует 6-значный код, сохраняет в Redis или Postgres-таблице `otp_codes` с TTL 5 минут, отправляет через `SmsProvider`, возвращает request_id для трекинга
   - `Future<User?> verifyOtp(String phone, String code)` — проверяет код, если ок → создаёт/находит User, возвращает; если нет → null
   - Rate limit: один номер не может запросить >3 OTP за час, >10 за день

3. **Auth endpoints в `lib/api/v1/auth.dart`:**
   - `POST /api/v1/auth/otp/request` body `{phone:String}` → `{request_id:String, ok:true}` или `{error:String, ok:false}`
   - `POST /api/v1/auth/otp/verify` body `{phone:String, code:String}` → `{token:String, user:User}` или 401
   - JWT или opaque session token (Alex решает в D-008-level subdecision)
   - Все ответы — JSON, ошибки — со стандартным error envelope

4. **Migration: таблицы `users`, `otp_codes`, `sessions`:**
   - `users`: id (uuid), phone (unique), role (enum, default 'parent'), full_name (nullable в v1), created_at, verified_at, status (active/disabled)
   - `otp_codes`: phone, code (hashed), created_at, expires_at, used_at (nullable)
   - `sessions`: token (hashed), user_id, device_id, created_at, expires_at, last_used_at, revoked

5. **Тесты:**
   - Unit на OtpService с mocked SmsProvider
   - Integration на полный flow: request → verify → token → /me endpoint возвращает user

### Client (client/)

6. **Заменить моковый AuthService в Flutter** на реальный REST-клиент к нашему backend:
   - `requestOtp(phone)` → POST `/auth/otp/request`
   - `verifyOtp(phone, code)` → POST `/auth/otp/verify`, при успехе сохраняет token в secure storage + User
   - Обработка ошибок (rate limit, неверный код, истёкший код) — понятные сообщения на русском

7. **Биометрический повторный вход** (уже частично в T-001) — теперь использует token из secure storage, делает GET `/api/v1/me` для проверки валидности сессии, при 401 → отправляет на экран OTP.

## Определение готовности (Definition of Done)
- [ ] Все 4 endpoint работают в staging
- [ ] SMS реально приходит на телефон Alex'а / Timur'а / Bekhruz при request
- [ ] Введённый код проходит verify, возвращается token, в `users` появляется запись
- [ ] Повторный запрос OTP в течение часа — после 3-го раза получает 429
- [ ] Истёкший код (старше 5 минут) — возвращает понятную ошибку
- [ ] Flutter-приложение делает полный flow без mock'ов: ввёл номер → пришёл SMS → ввёл код → попал на Home с реальным User
- [ ] Биометрия после первого входа работает без повторного SMS
- [ ] Unit и integration тесты зелёные в CI
- [ ] README backend обновлён: какие env vars нужны для SMS, как тестировать локально без реальной отправки (mock provider toggle)
- [ ] Файл задачи закоммичен через PR, статус в frontmatter = `done`, раздел «Завершение» заполнен

## Что уже есть / на что можно опираться
- T-001 — моковый AuthService и UI экранов Login/OTP/Biometry в Flutter
- T-002 — выбран SMS-провайдер, есть API ключ
- T-007 — backend живой, БД работает, миграции применяются
- D-004 — описывает все требования к auth

## План тестирования
- **Ручные проверки:**
  - Реальный звонок: Alex вводит свой номер, получает SMS, входит
  - Тот же тест на iOS симуляторе и Android эмуляторе (SMS — на физический телефон в любом случае)
  - Спам-тест: запросить OTP 5 раз подряд → должен прилететь 429
  - Истечение: запросить OTP, подождать 6 минут, попробовать verify → ошибка
- **Автотесты:**
  - Backend unit: OtpService.requestOtp с mock SMS — записывает в БД, не падает
  - Backend integration: полный flow через test HTTP-клиент
  - Flutter widget tests: проверка обработки 429, неверного кода, истёкшего кода
- **Граничные случаи:**
  - SMS-провайдер вернул 500 → пользователь видит «попробуйте позже», в логах backend есть запись
  - Провайдер вернул успех, но SMS не пришёл (известная проблема операторов) → у пользователя кнопка «получить заново» через 60с
  - Пользователь сменил SIM, запрашивает OTP с того же номера → создаёт новую сессию, не создаёт нового User

## Зависимости / блокируется
- Зависит от: T-007 (backend bootstrap) — нужен живой сервер с БД
- Зависит от: T-002 (SMS provider discovery) — нужен выбор провайдера и ключи
- Зависит от: T-001 (Flutter scaffolding + auth screens) — нужен UI и AuthService для замены
- **Блокирует:** все последующие задачи, требующие авторизованного пользователя (то есть всё)

## Вне scope
- Никаких ролей кроме `parent` в v1 (но поле `role` уже в схеме — D-004)
- Никакого password-reset / email-recovery — D-004 фиксирует, что восстановление при смене номера идёт через школу, не self-service
- Никакого 2FA сверх SMS — биометрия — это «локальный 2FA» на устройстве

## Завершение
> (пусто до завершения)

## История изменений
- 2026-05-25 — создано
