---
type: note
title: "Карта кода — где живут репозитории и как они связаны с PM"
status: living
updated: 2026-05-22
---

# Карта кода — где живут репозитории и как они связаны с PM

> **PM-репозиторий держит планы.** Код живёт в отдельных репозиториях. Этот файл — мост: какие репозитории существуют, где они клонируются локально, и какой PM-task им соответствует.
> Документ живой. Обновляется по мере появления и закрытия репозиториев.

---

## Локальная конвенция

Все код-репозитории клонируются в **`~/Desktop/Career-Vault/Oxbridge/code/`** на машине Bekhruz.

```
~/Desktop/Career-Vault/                 ← Obsidian-vault (личный)
├── Oxbridge/                           ← Всё, что относится к Оксбриджу
│   ├── PM/                             ← PM-репозиторий (отдельный git, доступен команде через GitHub)
│   │   ├── README.md
│   │   ├── Funnel/...
│   │   └── code-repos.md               ← этот файл
│   ├── code/                           ← Локальные клоны код-репозиториев
│   │   ├── oxbridge-parent-app/        ← Parent App (Flutter + Dart monorepo)
│   │   ├── oxbridge-school-site/       ← School-site (текущий маркетинговый сайт)
│   │   └── oxbridge-crm/               ← CRM-интеграции (будущий)
│   ├── Marketing/                      ← Личные материалы (бренд, кампании, и т.д.)
│   ├── Strategy/                       ← Стратегические заметки
│   └── ...
├── Retro/                              ← Retro Milliy (отдельный проект)
└── ...
```

**Важно:** PM и `code/` находятся **внутри** `Oxbridge/`, но это **отдельные git-репозитории**. PM-репозиторий не видит код. Каждый код-репозиторий — свой git. Obsidian может листать код-файлы для удобства, но они не комитятся в PM.

**Граница данных:**
- Личные/стратегические материалы (Marketing, Strategy, и т.д.) **не попадают в PM** — это для команды разработки, а не для бизнес-данных.
- PM **может быть прочитан** из верхнего уровня (например, через скилл `/oxbridge`) — это полезно для стратегического контекста.

---

## Список репозиториев

| Репозиторий | Воронка | URL (GitHub) | Локальный путь | Владелец | Видимость | Статус |
|---|---|---|---|---|---|---|
| **core-mobile-app** | 3-Parent-App | https://github.com/thebekhruz/core-mobile-app | `~/Desktop/Career-Vault/Oxbridge/code/core-mobile-app/` | @thebekhruz (Alex добавляется) | Private 🔒 | Bootstrapped (README + .gitignore + .github/) |
| **remix-of-oxbridge** | 1-School-site | https://github.com/thebekhruz/remix-of-oxbridge | `~/Desktop/Career-Vault/Oxbridge/code/remix-of-oxbridge/` | @thebekhruz | Private 🔒 | Активный (Vite + React + TS + Vercel) |
| **oxbridge-crm** | 2-CRM | TBD (создаётся при старте CRM-работы) | `~/Desktop/Career-Vault/Oxbridge/code/oxbridge-crm/` | TBD | TBD | Будущий |

> **Примечание по именам:** репозиторий Parent App называется `core-mobile-app` (не `oxbridge-parent-app`) — оставлено место для будущих Flutter-приложений (Teacher App, Admin Dashboard) в той же монорепе под `apps/`.
>
> **Маркетинговый сайт** называется `remix-of-oxbridge` (исторически — построен на Vite + React + TypeScript, несмотря на название "remix").

---

## Структура core-mobile-app (monorepo)

По решениям [[Funnel/3-Parent-App/Decisions/D-005-client-tech-stack]] (Flutter) и [[Funnel/3-Parent-App/Decisions/D-006-backend-stack]] (Dart на сервере), Parent App — это **monorepo**. Один git-репозиторий с клиентом и сервером, плюс место для будущих Flutter-поверхностей.

**Текущая структура** (закоммичена в репозиторий):

```
core-mobile-app/
├── README.md                    ← Что это, как клонировать, workflow с PM (закоммичено)
├── .gitignore                   ← Flutter + Dart + macOS standard (закоммичено)
├── .github/                     ← Заглушка под будущие CODEOWNERS, PR template, CI workflows
├── apps/
│   └── parent/                  ← Flutter-приложение для родителей (создаётся в T-001)
│       ├── lib/
│       ├── ios/
│       ├── android/
│       ├── test/
│       └── pubspec.yaml
├── server/                      ← Dart backend (Serverpod или эквивалент, выбор в D-008)
│   ├── lib/
│   ├── bin/
│   ├── test/
│   └── pubspec.yaml
├── shared/                      ← Общие модели (User, Payment, Family, и т.д.)
│   ├── lib/
│   └── pubspec.yaml
├── infra/                       ← docker-compose, миграции БД, deploy-конфиги
└── docs/                        ← ER-диаграммы, sequence-диаграммы, README модулей
```

**`apps/` (а не просто `client/`):** оставляет место для будущих Flutter-приложений (Teacher App, Admin Dashboard) под `apps/teacher/`, `apps/admin/` без миграции структуры.

**Почему monorepo для Parent App:**
- Shared models (`shared/`) переиспользуются и в `client/` и в `server/` — это главное преимущество одного-языка-везде из D-006
- Атомарные коммиты: меняешь поле в модели → в одном коммите чинятся и клиент, и сервер
- Простой git workflow: один репо, один CI
- Bekhruz видит весь Parent App в одном месте

**Когда monorepo стал бы плохим выбором (escape hatch — пересмотреть, если):**
- CI билды становятся медленными (>15 минут) — рассмотреть разделение на client + server
- Команда растёт до 4+ разработчиков с разными ритмами деплоя — рассмотреть разделение
- Появляется third-party команда / подрядчик, которому нужен ТОЛЬКО код сервера — рассмотреть разделение
- Размер репозитория переваливает за 5GB — рассмотреть split (нереалистично для нашего scope)

Если эти условия наступят — открыть Decision-файл `D-NNN-split-parent-app-monorepo.md`, обосновать, мигрировать. Сейчас они **далеки** от реальности.

---

## Структура remix-of-oxbridge

Маркетинговый сайт. **Стек:** Vite + React + TypeScript + shadcn/ui + Tailwind + Bun. Деплой через Vercel.

Несмотря на название "remix" — фреймворк именно Vite + React, не Remix framework. Название историческое.

Текущее состояние требует cleanup-прохода (организационный долг от AI-builders Lovable/Trae): mixed package managers, root-level scripts, branch typo. Сейчас этот cleanup отложен — см. T-005 (Timur weekend sprint) и Funnel/1-School-site/_Hub для приоритетов.

---

## Структура oxbridge-crm (будущая)

Пока не создан. Когда работа по CRM ([[Funnel/2-CRM/_Hub]]) перейдёт в стадию реализации — этот репозиторий будет создан. Тогда здесь появится структура.

Скорее всего: интеграционный слой между UCM6304 (IP-телефония), amoCRM, и Parent App backend. Стек определится отдельным Decision-файлом.

---

## Конвенция бранчей и коммитов

См. README PM-репозитория, раздел «Как отметить работу выполненной». Краткая суть:

- Бранч: `task/T-NNN-короткое-описание` (например, `task/T-001-flutter-auth`)
- Все коммиты префиксированы ID задачи: `T-001: scaffold flutter project`
- PR в код-репозиторий содержит ссылку на PM-task: `Closes: PM/Funnel/3-Parent-App/Tasks/T-001-*.md`

---

## Как клонировать (для разработчиков)

```bash
mkdir -p ~/Desktop/Career-Vault/Oxbridge/code
cd ~/Desktop/Career-Vault/Oxbridge/code

# Parent App (когда репозиторий будет создан)
git clone <github-url-from-this-file> oxbridge-parent-app
cd oxbridge-parent-app
# Следуй README для настройки локального окружения

# School-site
git clone <github-url> oxbridge-school-site
```

PM-репозиторий уже клонирован у Bekhruz. Команда клонирует его так же:

```bash
git clone <github-url-PM> ~/Desktop/Career-Vault/Oxbridge/PM
```

---

## Связи с PM-задачами

Каждая код-related PM-задача указывает в frontmatter:
- `code_repo:` — URL репозитория + бранч
- `commit_initial:` — SHA первого коммита по этой задаче
- `commit_final:` — SHA последнего коммита

См. [[_Templates/Task]] и [[_Skills/complete-task]].

---

## Действия для Bekhruz (заполнить, когда выполнено)

- [x] Создать GitHub-репозиторий `core-mobile-app` (приватный) — **сделано 2026-05-25**
- [x] Внести URL в таблицу выше — **сделано**
- [x] Подтвердить URL `remix-of-oxbridge` — **сделано**
- [ ] Дать Алексу и Тимуру доступ как collaborators на `core-mobile-app` и `remix-of-oxbridge`
- [ ] Создать локальную папку `~/Desktop/Career-Vault/Oxbridge/code/` и клонировать репозитории туда (для себя)
- [ ] Настроить branch protection на `main` (Settings → Branches на каждом репо)
- [ ] Настроить repo settings: "Automatically delete head branches" + "Squash and merge only"

---

## История изменений
- 2026-05-22 — создано. Monorepo для Parent App зафиксировано. Локальный путь — `~/Desktop/Career-Vault/Oxbridge/code/`. URL'ы TBD.
