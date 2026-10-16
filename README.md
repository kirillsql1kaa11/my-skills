# my-skills

21 скилл для AI-агентов: [opencode](https://opencode.ai) и [Google Antigravity](https://antigravity.google).

9 скиллов работают в opencode, 12 — в Antigravity. Каждый лежит отдельным каталогом: сам навык, README с описанием и командой установки.

![skills](https://img.shields.io/badge/skills-21-2ea44f)
![opencode](https://img.shields.io/badge/opencode-9-0b7285)
![antigravity](https://img.shields.io/badge/antigravity-12-6741d9)
![license](https://img.shields.io/badge/license-MIT-eab308)

## Структура репозитория

```text
my-skills/
└── <имя-скилла>/
    ├── SKILL.md     # сам навык
    ├── README.md    # описание, когда применяется, установка
    └── ...          # вспомогательные файлы (данные, скрипты), если есть
```

## Скиллы для opencode

| Скилл | Назначение |
| :--- | :--- |
| [`cleanup-temp`](cleanup-temp) | Автоуборка временных файлов и каталогов после задачи |
| [`no-comments`](no-comments) | Код без комментариев, docstring и JSDoc |
| [`perf-check`](perf-check) | Оптимизация только через измерения: база → изменение → повторный замер |
| [`plain-talk`](plain-talk) | Краткие ответы без канцелярита и воды |
| [`project-brief`](project-brief) | Разведка незнакомого проекта: стек, команды, конвенции |
| [`responsive-basics`](responsive-basics) | Адаптивная вёрстка без развалов на узких и широких экранах |
| [`ru-commits`](ru-commits) | Русские коммиты и релизы: conventional commits, без эмодзи и шаблонов |
| [`tradeoff-say`](tradeoff-say) | Честный блок «Компромиссы» после значимых технических решений |
| [`ui-ux-pro-max`](ui-ux-pro-max) | Дизайн-интеллект: 79 стилей, 192 палитры, UX-правила и гайды по 22 стекам |

## Скиллы для Google Antigravity

| Скилл | Назначение |
| :--- | :--- |
| [`chrome-extension-mv3`](chrome-extension-mv3) | Разработка расширений Chrome на Manifest V3 |
| [`clean-documentation-readme`](clean-documentation-readme) | Регламент эталонного README для проектов |
| [`code-review-agent`](code-review-agent) | Структурированное код-ревью с уровнями критичности |
| [`dead-code-cleaner`](dead-code-cleaner) | Безопасное удаление мёртвого кода по стекам |
| [`dom-mutation-watcher`](dom-mutation-watcher) | MutationObserver: батчинг, защита от циклов, очистка ресурсов |
| [`frontend-design`](frontend-design) | Уникальная визуальная идентичность вместо шаблонных дефолтов |
| [`git-russian-commit`](git-russian-commit) | Регламент коммитов на русском языке без эмодзи |
| [`mit-license-compliance`](mit-license-compliance) | Аудит лицензий зависимостей на совместимость с MIT |
| [`regex-automata-scanner`](regex-automata-scanner) | Высокоскоростной многошаблонный поиск на Rust |
| [`reverse-api-scraper`](reverse-api-scraper) | Реверс приватных API: подписи, сессии, потоковая выгрузка |
| [`rust-memory-profiling`](rust-memory-profiling) | Профилирование памяти Rust: dhat, mimalloc, утечки |
| [`sqlite-dapper-csharp`](sqlite-dapper-csharp) | SQLite + Dapper в .NET: WAL, транзакции, тесты в памяти |

## Установка

Скиллы каждого инструмента кладутся в свой каталог. Достаточно скопировать папку нужного навыка из клонированного репозитория.

### opencode

```powershell
git clone https://github.com/kirillsql1kaa11/my-skills.git
Copy-Item -Recurse my-skills\cleanup-temp "$env:USERPROFILE\.config\opencode\skills\"
```

Все 9 скиллов разом:

```powershell
Get-ChildItem my-skills -Directory | Where-Object { Test-Path "$($_.FullName)\SKILL.md" } | Where-Object { $_.Name -in @("cleanup-temp","no-comments","perf-check","plain-talk","project-brief","responsive-basics","ru-commits","tradeoff-say","ui-ux-pro-max") } | Copy-Item -Destination "$env:USERPROFILE\.config\opencode\skills\" -Recurse -Force
```

### Google Antigravity

```powershell
Copy-Item -Recurse my-skills\chrome-extension-mv3 "$env:USERPROFILE\.gemini\config\skills\"
```

Все 12 скиллов разом:

```powershell
Get-ChildItem my-skills -Directory | Where-Object { Test-Path "$($_.FullName)\SKILL.md" } | Where-Object { $_.Name -in @("chrome-extension-mv3","clean-documentation-readme","code-review-agent","dead-code-cleaner","dom-mutation-watcher","frontend-design","git-russian-commit","mit-license-compliance","regex-automata-scanner","reverse-api-scraper","rust-memory-profiling","sqlite-dapper-csharp") } | Copy-Item -Destination "$env:USERPROFILE\.gemini\config\skills\" -Recurse -Force
```

Инструменты сами обнаруживают скиллы при следующем запуске: у каждого навыка из `SKILL.md` читаются имя и описание, а полный текст подгружается только при необходимости.

## Обновление

Источник истины — рабочие каталоги скиллов на машине автора (`~/.config/opencode/skills` и `~/.gemini/config/skills`). Изменения переносятся сюда вручную; репозиторий — витрина и точка распространения.

## Лицензия

Проект распространяется под условиями лицензии [MIT](LICENSE).

Скилл `frontend-design` содержит собственный файл условий — см. `frontend-design/LICENSE.txt`.
