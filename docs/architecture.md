# Архітектура агента

## 1. Які проблеми вирішує агент

Агент `build-engineer` допомагає відтворювано створити кросплатформний Rust
Hello World-проєкт і перевірити його в GitHub Actions. Він координує каркас
Cargo, бібліотечну функцію та виконуваний файл, тест `basic_addition`,
платформні скрипти збірки й CI для Windows, Linux і macOS.

Інструкції також задають безпечну роботу з наявними файлами: перевірку перед
зміною, ідемпотентність, явне повідомлення про конфлікти та помилки й
заборону видалення чи небезпечних змін Git.

## 2. Skills і їхня відповідальність

| Skill | Відповідальність |
|---|---|
| `project-scaffold` — `.github/skills/project-scaffold/SKILL.md` | Створення початкового каркаса Rust-проєкту: `src/lib.rs`, `src/main.rs`, потрібних каталогів і README, якщо його ще немає. |
| `build-and-test` — `.github/skills/build-and-test/SKILL.md` | Налаштування `Cargo.toml`, інтеграційного тесту, генерація `Cargo.lock`, створення та перевірка скриптів збірки для Unix-подібних систем і Windows. |
| `github-actions` — `.github/skills/github-actions/SKILL.md` | Створення та перевірка workflow GitHub Actions для Linux, Windows і macOS, який запускає наявні CI-скрипти й публікує артефакт збірки. |

## 3. Commands для користувача

Команди доступні у Copilot Chat як prompt-файли в `.github/prompts/`:

| Command | Призначення |
|---|---|
| `/git-init` | Перевірити наявний Git-репозиторій і створити або доповнити `.gitignore`. |
| `/create-project` | Створити каркас проєкту за skill `project-scaffold`. |
| `/create-build` | Налаштувати Cargo, тест і кросплатформні скрипти за skill `build-and-test`. |
| `/create-actions` | Налаштувати GitHub Actions workflow за skill `github-actions`. |
| `/check` | У режимі лише читання перевірити структуру, збірку, тести, workflow та Git-ігнорування й видати статус `PASSED` або `FAILED`. |
| `/init` | Оркеструвати початкове налаштування та фінальну перевірку в заданій послідовності. |

## 4. Дані між етапами

Інформація передається як параметри й результати файлів між послідовними
етапами:

1. **`project_name` → `src/`:** для цього проєкту `project_name=hello`.
   Skill `project-scaffold` створює `src/lib.rs` із `add` та `src/main.rs`,
   який друкує привітання і використовує бібліотеку.
2. **`src/` → `Cargo.toml`:** skill `build-and-test` бере наявні шляхи до
   вихідного коду та описує package `hello`, бібліотечний target `hello_lib`
   у `src/lib.rs` і binary target `hello` у `src/main.rs`.
3. **`Cargo.toml` → `ci.sh` / `ci.bat`:** скрипти використовують Cargo
   manifest для release-збірки й тестів, а потім копіюють виконуваний файл у
   `dist/`. Unix-скрипт — `ci.sh`; Windows-скрипт — `ci.bat`.
4. **`ci.sh` / `ci.bat` → `ci.yml`:** workflow викликає відповідний скрипт
   для runner-платформи. Він не дублює команди Cargo або rustc, а після
   успішного скрипту завантажує `dist/` як артефакт.

Кожен етап перевіряє передумови попереднього й передає наступному файли, а не
неявний стан процесу.

## 5. Обробка помилок оркестратором `init`

`/init` виконує команди строго послідовно: `/git-init`, `/create-project`,
`/create-build`, `/create-actions`, `/check`. Після кожного кроку він
перевіряє результат і переходить далі лише після успіху. Якщо крок або його
перевірка завершується помилкою, оркестратор зупиняється, не запускає наступні
кроки й звітує про назву кроку, причину та необхідне виправлення. Невиконані
кроки позначаються `NOT RUN`. Загальний успіх дозволений лише тоді, коли
останній `/check` завершився статусом `PASSED`.

## 6. Відповідність файлів ролям

| Роль | Розташування | Приклади |
|---|---|---|
| Agent | `.github/agents/` | `build-engineer.agent.md` задає роль, область відповідальності, правила й критерії завершення агента. |
| Skills | `.github/skills/<skill-name>/` | `project-scaffold/SKILL.md`, `build-and-test/SKILL.md`, `github-actions/SKILL.md` описують спеціалізовані процедури. |
| Commands | `.github/prompts/` | `git-init.prompt.md`, `create-project.prompt.md`, `create-build.prompt.md`, `create-actions.prompt.md`, `check.prompt.md`, `init.prompt.md` визначають доступні команди та їхню оркестрацію. |

## 7. Формат налаштувань

У проєкті використано формат налаштувань VS Code Copilot: custom agents
(`.agent.md`), prompt files (`.prompt.md`) та agent skills (`SKILL.md`) у
`.github/`.
