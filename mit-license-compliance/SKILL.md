---
name: mit-license-compliance
description: >-
  Проверяет лицензионную совместимость сторонних зависимостей с лицензией MIT,
  выявляет несовместимые или вирусные лицензии (GPL, AGPL) и формирует отчеты атрибуции
  для проектов на Rust, C#, JavaScript и Python. Активируется при подготовке релизов,
  добавлении новых пакетов и аудите открытого исходного кода.
---

# Регламент проверки лицензий (mit-license-compliance)

Данный навык регулирует процесс аудита зависимостей на совместимость с лицензией MIT, предотвращает попадание ограничивающих (copyleft) лицензий в кодовую базу и формирует отчеты об используемых сторонних библиотеках.

## Матрица совместимости

### Допустимые лицензии
- MIT
- Apache-2.0
- BSD-2-Clause / BSD-3-Clause
- ISC
- 0BSD / Unlicense / CC0-1.0

### Запрещенные лицензии (требуют изоляции или замены)
- GPL-2.0 / GPL-3.0
- AGPL-3.0
- SSPL
- BSL-1.1
- Любые несвободные или non-commercial лицензии

---

## Аудит по языкам и стекам

### 1. Rust

Шаг 1. Установка и запуск аудита через cargo-deny:
```powershell
cargo install cargo-deny --locked
cargo deny check licenses
```

Шаг 2. Быстрый вывод таблицы лицензий:
```powershell
cargo license
```

---

### 2. JavaScript / Node.js

Шаг 1. Проверка дерева пакетов на соответствие белому списку:
```powershell
npx license-checker --onlyAllow "MIT;Apache-2.0;BSD-2-Clause;BSD-3-Clause;ISC;0BSD;Unlicense;CC0-1.0"
```

Шаг 2. Формирование сводного отчета по типам лицензий:
```powershell
npx license-checker --summary
```

---

### 3. C# / .NET

Шаг 1. Установка глобального инструмента проверки лицензий NuGet:
```powershell
dotnet tool install --global dotnet-project-licenses
```

Шаг 2. Сканирование решения и выгрузка списка:
```powershell
dotnet-project-licenses -i . -u
```

Шаг 3. Фильтрация пакетов с лицензиями, отличными от разрешенного списка:
```powershell
dotnet-project-licenses -i . --allowed-license-types "MIT;Apache-2.0;BSD-2-Clause;BSD-3-Clause;ISC"
```

---

### 4. Python

Шаг 1. Установка утилиты проверки:
```powershell
pip install pip-licenses
```

Шаг 2. Формирование сводной таблицы:
```powershell
pip-licenses --summary
```

Шаг 3. Поиск пакетов с несовместимыми лицензиями:
```powershell
pip-licenses --fail-on "GPL;AGPL;LGPL"
```

---

## Генерация файла LICENSE

Каждый открытый репозиторий должен содержать корневой файл `LICENSE` следующего вида:

```text
MIT License

Copyright (c) 2026 Kirill Sqweezy (kirillsql1kaa11)

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

## Действия при обнаружении несовместимой зависимости

1. Проверить возможность замены библиотеки на аналог с лицензией MIT/Apache-2.0.
2. Проверить, является ли зависимость исключительно dev-инструментом (не поставляемым в собранный бинарный файл или бандл).
3. Если замена невозможна, вынести несовместимый функционал в отдельный сервис или внешний плагин с изоляцией по сетевому протоколу или CLI.
