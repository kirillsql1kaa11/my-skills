# ui-ux-pro-max

Дизайн-интеллект для UI/UX на web, мобильных и десктопе: локальная база знаний с поисковым движком. Агент применяет её при проектировании, сборке, ревью и починке интерфейсов.

**Инструмент:** opencode
**Триггер:** автоматически при задачах про структуру UI, визуальные решения, паттерны взаимодействия и качество UX

## Что внутри

| Данные | Объём |
| :--- | ---: |
| Стили | 79 (50 активных) |
| Продуктовые палитры и профили-рассуждения | 192 |
| Шрифтовые пары | 74 |
| UX-правила | 119 |
| Иконки | 105 |
| GSAP-пресеты | 17 |
| Типы графиков | 25 |
| Гайды по стекам | 22 |

Стеки: React, Next.js, Vue, Svelte, Astro, Nuxt, Angular, Laravel, SwiftUI, React Native, Flutter, Jetpack Compose, html-tailwind, shadcn, Three.js, JavaFX, WPF, WinUI, Avalonia, Uno, UWP.

## Возможности

- Генерация дизайн-системы под продукт: паттерн лендинга, стиль, палитра, типографика, эффекты, антипаттерны
- Сохранение дизайн-системы в проект: `design-system/<слаг>/MASTER.md` и переопределения страниц
- Точечные поиски по доменам: `product`, `style`, `color`, `typography`, `google-fonts`, `chart`, `ux`, `landing`, `icons`, `gsap`, `react`, `web`
- Диалеры `--variance`, `--motion`, `--density` для настройки смелости, анимации и плотности макета
- Приоритеты правил с проверками и антипаттернами: доступность, тач, производительность, стиль, раскладка, типографика, анимация, формы, навигация, графики
- Финальный чеклист перед сдачей UI: иконки, контраст, safe-area, доступность

## Структура каталога

```text
ui-ux-pro-max/
├── SKILL.md                      # инструкция для агента
├── scripts/
│   └── search.py                 # поисковый движок
├── data/                         # базы данных (CSV), в том числе по стекам
└── references/
    ├── quick-reference.md        # полный перечень UX-правил
    └── pro-rules.md              # правила полировки и финальный чеклист
```

## Использование

Движок запускается из каталога скилла; внешних зависимостей нет, сеть не используется. Нужен Python 3.x.

```powershell
py "scripts\search.py" "beauty spa wellness service" --design-system -p "Serenity Spa"
py "scripts\search.py" "glassmorphism dark" --domain style
py "scripts\search.py" "chip badge overflow nowrap" --stack html-tailwind
```

## Установка

```powershell
Copy-Item -Recurse ui-ux-pro-max "$env:USERPROFILE\.config\opencode\skills\"
```
