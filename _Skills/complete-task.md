---
skill: complete-task
purpose: Generate the "## Завершение" section of a Task.md from a git commit range
trigger: "/complete-task" or "generate completion for T-NNN"
language: дев-facing output is Russian (since task files are Russian)
---

# Complete-task — generate completion summary from git history

This skill produces the "## Завершение" section of a task file. The dev runs it when they finish work on a task. The output is reviewed by the dev, possibly edited, and pasted into the task file.

## Inputs the skill needs

The dev provides:
1. **Task ID** — e.g. `T-001`
2. **Path to task file** — e.g. `Funnel/3-Parent-App/Tasks/T-001-flutter-scaffolding-auth.md`
3. **Code repo URL + branch** — where the work lives
4. **`commit_initial` SHA** — from the task file frontmatter (set when dev started)
5. **`commit_final` SHA** — last commit on the branch
6. **Git log output** — the dev runs and pastes:
   ```bash
   git -C <code_repo_path> log --no-merges --pretty=format:"%h %s%n%b%n---" <commit_initial>..<commit_final>
   git -C <code_repo_path> diff --stat <commit_initial>..<commit_final>
   ```
   If commit SHAs are lost (rebase), use the fallback:
   ```bash
   git -C <code_repo_path> log --no-merges --pretty=format:"%h %s%n%b%n---" --grep="T-NNN"
   ```

## What the skill does

1. **Read the task file** at the provided path.
2. **Read the "Что нужно сделать" section** — these are the steps that were supposed to happen.
3. **Read the "Определение готовности" section** — these are the checks that prove done.
4. **Read the git log + diff stats** the dev provided.
5. **Cross-reference:** map each commit to a step or DoD item. Note items that don't have a corresponding commit (could be intentionally skipped or incomplete).
6. **Produce the completion summary** in this exact shape (Russian):

```markdown
## Завершение

**Что запущено:**
- {{1-line summary of each major piece that shipped, mapped to commits}}

**Где это лежит:**
- **Код:** {{code_repo URL + branch + final commit SHA}}
- **Deploy / staging URL:** {{if dev mentioned one in commit messages}}
- **Связанные файлы (по diff):** {{key file paths from --stat}}

**Что НЕ сделано из изначального плана (если что-то не сделано):**
- {{list items from "Что нужно сделать" that have no corresponding commit, OR mark as "всё сделано по плану"}}

**Сюрпризы и решения в процессе:**
- {{anything in commit messages that indicates a non-obvious design decision, refactor, or workaround}}

**Что нужно знать следующему разработчику:**
- {{actionable handoff notes: things the next person needs to be aware of (e.g. "env variable X required", "data migration Y must run first")}}
- {{if nothing notable, write "нет специфических заметок"}}

**Метрики:**
- Коммитов: {{count}}
- Файлов изменено: {{from --stat}}
- Строк +/-: {{from --stat}}
- Период работы: {{from first commit date to last commit date}}
```

## Rules for output quality

- **Не выдумывай.** Если в коммитах нет упоминания того или иного факта — не пиши, что это сделано.
- **Будь конкретен.** «Реализован экран авторизации» — нет. «Реализован экран ввода номера телефона с валидацией +998 и переход на OTP-экран при нажатии Получить код» — да.
- **Цитируй пути к файлам** из diff stat, если это помогает контексту: «Логика валидации — `lib/auth/phone_validator.dart`».
- **Если есть тесты — упомяни их явно:** «Добавлено 7 unit-тестов на AuthService».
- **Если в коммитах есть TODO, FIXME, или XXX** — упомяни в «Что не сделано».
- **Сюрпризы — самая важная секция.** Если разработчик пишет в commit message «решил использовать X вместо Y потому что Z» — это сигнал, что есть архитектурное решение, которое надо зафиксировать. Возможно, должно стать Decision-файлом.

## Когда AI должен пушбэкнуть и не генерировать

- Если `commit_initial` или `commit_final` отсутствуют, и `--grep` тоже не находит коммитов: ответь «Не нашёл коммитов для этой задачи. Возможно, ID задачи не использовался в commit messages. Проверь git log вручную или дай прямой список SHAs».
- Если в git log нет коммитов с описанием — только «WIP» или «fix» — ответь «Сообщения коммитов слишком неинформативны для генерации завершения. Дополни описание коммитов через `git commit --amend` или напиши завершение вручную».
- Если ни одно из DoD-условий не покрыто в diff — ответь «По коммитам не видно, что задача выполнена. Уверен, что это правильная пара commit_initial / commit_final?»

## Использование

Самый простой способ — открыть Claude (этого же или другого) и вставить следующий запрос:

```
Сгенерируй раздел «Завершение» для задачи T-NNN, используя _Skills/complete-task.md.

Файл задачи: <путь к Task.md>

Код-репозиторий: <URL + бранч>
commit_initial: <SHA>
commit_final: <SHA>

Git log:
<вставь сюда вывод git log>

Diff stat:
<вставь сюда вывод git diff --stat>
```

Claude вернёт готовый раздел «Завершение» — прочитай, поправь, вставь в файл задачи. Затем коммить и открывай PM PR.
