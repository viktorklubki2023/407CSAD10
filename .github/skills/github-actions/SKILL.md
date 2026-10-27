---
name: github-actions
description: "Застосовуй, коли потрібно створити або перевірити GitHub Actions CI workflow для Rust-проєкту на Linux, Windows і macOS."
---

# GitHub Actions

Створи або перевір GitHub Actions workflow для кросплатформної збірки Rust
проєкту. Workflow має лише запускати наявні скрипти CI та завантажувати
зібраний артефакт; команди Cargo або rustc напряму у workflow не додавай.

## Очікуваний вихід

Шлях workflow: `.github/workflows/ci.yml`.

- Тригери `push` і `pull_request` мають працювати для гілок `develop` та
  `master`.
- Має бути одна job із matrix:

  ```yaml
  strategy:
    fail-fast: false
    matrix:
      os: [ubuntu-latest, windows-latest, macos-latest]
  ```

- Job має використовувати `runs-on: ${{ matrix.os }}`.
- Кроки job:
  1. Checkout репозиторію через `actions/checkout@v4`.
  2. Для не-Windows runner запусти лише `./ci.sh` із `shell: bash`.
  3. Для Windows runner запусти лише `ci.bat` із `shell: cmd`.
  4. Завантаж `dist/` через `actions/upload-artifact@v4`, з унікальним ім'ям
     `hello-${{ matrix.os }}` та `if-no-files-found: error`.
- У `run` кроках workflow не повинно бути команд `cargo` чи `rustc` — лише
  виклик відповідного скрипту.
- Не додавай облікові дані, значення для автентифікації чи інші чутливі дані.

## Правила роботи

- Перед створенням чи зміною workflow перевір, чи файл уже існує, та прочитай
  його вміст.
- Не перезаписуй наявний файл мовчки. Якщо він відповідає вимогам, залиш без
  змін; якщо відрізняється, покажи конфлікт і збережи наявний вміст.
- Працюй ідемпотентно: повторний запуск не дублює тригери, jobs або кроки.
- Переконайся, що `ci.sh` існує та має executable bit у Git. Якщо його не
  позначено виконуваним, повідом про проблему і вкажи на необхідність
  `git update-index --chmod=+x ci.sh`; не приховуй помилку запуску.
- Не створюй і не змінюй CI-скрипти, якщо це не входить до запиту.

## Перевірки

1. Перевір синтаксис YAML за допомогою `actionlint`, якщо він доступний;
   інакше використай Python із PyYAML, якщо пакет встановлено. Не встановлюй
   залежності лише заради цієї перевірки.
2. Перевір, що workflow містить `ubuntu-latest`, `windows-latest` і
   `macos-latest`, `fail-fast: false`, обидва тригери та вказані гілки.
3. Перевір умови платформних кроків, shell і точні виклики `./ci.sh` та
   `ci.bat`.
4. Перевір налаштування `actions/upload-artifact@v4`, ім'я
   `hello-${{ matrix.os }}`, шлях `dist/` і `if-no-files-found: error`.
5. Переглянь усі `run` кроки: вони не повинні викликати Cargo чи rustc
   напряму.
6. Переконайся, що файл workflow не містить згадок облікових даних,
   паролів чи секретних значень.
7. Повідом результат кожної перевірки; не оголошуй роботу успішною, якщо
   будь-яка обов'язкова умова не виконана.

Типова помилка: `ci.sh` не має права на виконання. Перевір executable bit у Git
індексі; без нього крок `./ci.sh` може завершитися помилкою на Unix-подібному
runner.

## Приклад

Вхід:

```text
Потрібен GitHub Actions CI для Rust-проєкту на develop і master.
Наявні ci.sh та ci.bat.
```

Очікуваний результат: `.github/workflows/ci.yml` має одну matrix job для
`ubuntu-latest`, `windows-latest` і `macos-latest`, запускає відповідний
наявний CI-скрипт і завантажує `dist/` як артефакт `hello-${{ matrix.os }}`.
YAML валідний, прямі команди Cargo/rustc та чутливі значення у workflow відсутні.
