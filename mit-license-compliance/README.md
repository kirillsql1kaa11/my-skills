# mit-license-compliance

Аудит лицензионной совместимости зависимостей с MIT: выявление GPL/AGPL, отчёты атрибуции, генерация файла LICENSE.

**Инструмент:** Google Antigravity
**Триггер:** подготовка релиза, добавление новых пакетов, аудит открытого исходного кода

## Что делает

- Задаёт матрицу совместимости:
  - допустимые: MIT, Apache-2.0, BSD-2/3-Clause, ISC, 0BSD, Unlicense, CC0-1.0
  - запрещённые: GPL-2.0/3.0, AGPL-3.0, SSPL, BSL-1.1, любые non-commercial
- Даёт команды аудита по стекам:
  - Rust: `cargo deny check licenses`, `cargo license`
  - JavaScript / Node.js: `npx license-checker --onlyAllow` и `--summary`
  - C# / .NET: `dotnet-project-licenses` с фильтром разрешённых типов
  - Python: `pip-licenses --summary`, `--fail-on "GPL;AGPL;LGPL"`
- Содержит эталонный текст `LICENSE` (MIT) для корня репозитория
- Описывает действия при обнаружении несовместимой зависимости: замена на MIT/Apache-аналог, проверка, что пакет — только dev-инструмент, изоляция в отдельный сервис или плагин

## Установка

```powershell
Copy-Item -Recurse mit-license-compliance "$env:USERPROFILE\.gemini\config\skills\"
```
