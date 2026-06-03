---
type: task
id: T-007
title: "Внедрить Schema.org JSON-LD разметку на сайт для GEO (Generative Engine Optimisation)"
spec: "Funnel/1-School-site/_Hub.md"
funnel-stage: "1-School-site"
status: pending
priority: high
assignee: Timur
estimate: medium (1-3 дня)
created: 2026-06-03
target-done: 2026-06-10
code_repo: ""
commit_initial: ""
commit_final: ""
---

# T-007 — Внедрить Schema.org JSON-LD разметку на сайт для GEO

## Контекст

Сайт oxbridgeschool.uz не содержит ни одного блока структурированных данных Schema.org. Это означает, что AI-инструменты (ChatGPT, Perplexity, Google AI Overview, Claude) не могут надёжно извлекать факты о школе — они угадывают из прозы вместо того, чтобы читать структурированные данные. Google AI Overview напрямую вытягивает FAQPage и EducationalOrganization schema в ответы на поисковые запросы. Цель задачи — добавить JSON-LD разметку на ключевые страницы без изменения их визуала.

**Стадия воронки:** 1-School-site
**Эталонный пример JSON-LD:** `Oxbridge/Admissions/schema-org-example.json` — прочитай перед началом работы, там полные примеры всех блоков.

## Что нужно сделать

### Страница 1 — Главная (`/`)

1. Добавить JSON-LD блок `EducationalOrganization` — название, адреса обоих кампусов, телефон, email, URL, координаты, соцсети, возраст учеников.
2. В блок `EducationalOrganization` добавить брендинг-поля:
   - `description`: развёрнутое описание с брендовыми proof points — **точный текст в эталонном JSON** (`schema-org-example.json`). Не писать своими словами, не сокращать. Там: 98% поступления, 82% топ-300, 19% выше IB average, 90% без репетиторов, макс 24 ученика, 25 стран, философия школы.
   - `slogan`: "Воспитание целостной личности: от любознательности к уверенности"
   - `foundingDate`: "2018"
   - `foundingLocation`: Tashkent, Uzbekistan
   - `additionalProperty` с `schoolMotto`: "Scientia · Futurum · Hereditatem"
   - `knowsAbout`: ["International Baccalaureate", "IB Diploma Programme", "bilingual education", "private school Tashkent", "Russian language curriculum Uzbekistan"]
   - `keywords`: "IB школа Ташкент, частная школа Ташкент, международная школа Узбекистан, IB Diploma Ташкент, русская школа Ташкент"
3. Добавить JSON-LD блок `FAQPage` — взять 4 существующих вопроса из секции "Вопросы, которые не дают спать по ночам". Ответы — полные, не обрезанные.
4. Разместить все блоки в `<head>` или непосредственно перед `</body>` — не в теле контента.

### Страница 2 — Как поступить (`/how-to-apply`)

5. Добавить JSON-LD блок `FAQPage` — взять вопросы из существующего FAQ-аккордеона на странице (все раскрытые вопросы). Ответы должны быть полными, даже если в UI они скрыты за аккордеоном.

### Страница 3 — Стоимость обучения (`/tuition-fees`)

6. Добавить JSON-LD блок `FAQPage` с вопросами про оплату, если они есть на странице.
7. Добавить JSON-LD блок `OfferCatalog` — две программы (Futurum IB и Hereditum) с ценовыми диапазонами по кампусам.

### Страница 4 — Программные страницы (`/primary`, `/secondary-school`, `/high-school`, `/early-years`)

8. На каждой странице добавить `EducationalOccupationalProgram` — название программы, язык обучения, возрастной диапазон, аккредитация (IB World School где применимо).

### Финальная проверка

9. Проверить каждую страницу через [Google Rich Results Test](https://search.google.com/test/rich-results) — все блоки должны проходить без ошибок.
10. Проверить через [Schema.org Validator](https://validator.schema.org/) — без критических предупреждений.

## Определение готовности (Definition of Done)

- [ ] Главная: `EducationalOrganization` (включая брендинг-поля: slogan, foundingDate, schoolMotto, knowsAbout) + `FAQPage` добавлены, Rich Results Test — зелёный
- [ ] `/how-to-apply`: `FAQPage` добавлен, Rich Results Test — зелёный
- [ ] `/tuition-fees`: `FAQPage` + `OfferCatalog` добавлены, Rich Results Test — зелёный
- [ ] `/primary`, `/secondary-school`, `/high-school`, `/early-years`: `EducationalOccupationalProgram` добавлены
- [ ] Ни один блок не дублируется (один `<script type="application/ld+json">` на тип на страницу)
- [ ] Google Search Console не показывает новых ошибок структурированных данных через 48 часов после деплоя

## Что уже есть / на что можно опираться

- Эталонный JSON-LD с правильными значениями: `Oxbridge/Admissions/schema-org-example.json`
- Адреса кампусов: МУ — Oymariq ko'chasi, 10; YA — Uysozlar ko'chasi, 29
- Координаты: МУ 41.357280, 69.372671 (уже есть в meta-ICBM на сайте)
- Телефон: +998555552233
- Email: admissions@oxbridgeschool.uz
- Instagram: https://www.instagram.com/oisuzbekistan/
- Telegram: https://t.me/ois_uz_bot

## План тестирования

- **Ручные проверки:** после деплоя каждую страницу прогнать через Rich Results Test и Schema Validator
- **Граничные случаи:** убедиться что JSON-LD не ломает CSP заголовки; убедиться что аккордеон-ответы на FAQ попали в schema полностью, а не как пустые строки

## Зависимости / блокируется

- Нет. Задача независима — только правки в `<head>` / перед `</body>`.

## Вне scope

- Изменение визуального дизайна страниц
- Добавление новых страниц
- Review schema (`AggregateRating`) — нет достаточного количества верифицированных отзывов
- Product schema — не применимо к школе

## Завершение

> (пусто до завершения)

## История изменений
- 2026-06-03 — создано (Bekhruz / Claude)
