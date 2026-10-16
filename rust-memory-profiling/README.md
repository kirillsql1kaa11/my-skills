# rust-memory-profiling

Профилирование памяти Rust-приложений: аллокации кучи, утечки, оптимизация аллокатора.

**Инструмент:** Google Antigravity
**Триггер:** анализ потребления ресурсов, оптимизация аллокаций, устранение утечек памяти

## Что делает

- Запрещает включать профилировщик аллокаций в релизный бинарник — только под отдельным Cargo feature-флагом
- Показывает настройку `dhat` и `mimalloc` в `Cargo.toml` с `dhat-heap` фичей и `debug = 1` в релизном профиле
- Даёт интеграцию `#[global_allocator]` с переключением между dhat и mimalloc по cfg-флагу
- Описывает замеры: запуск `cargo run --release --features dhat-heap`, файл `dhat-heap.json`, просмотр во вьюере dhat
- Объясняет, что смотреть: пиковое потребление кучи, точки churn, не освобождённую память
- Приводит паттерны снижения аллокаций: `&str` и `Cow` вместо `String`, `Vec::with_capacity`, `SmallVec`/`ArrayVec`, переиспользование буферов
- Завершается чек-листом: тесты проходят с флагом и без, в релизе по умолчанию mimalloc

## Установка

```powershell
Copy-Item -Recurse rust-memory-profiling "$env:USERPROFILE\.gemini\config\skills\"
```
