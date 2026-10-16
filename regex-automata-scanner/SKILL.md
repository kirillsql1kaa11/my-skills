---
name: regex-automata-scanner
description: >-
  Регламентирует построение высокопроизводительных движков многошаблонного поиска на Rust
  с использованием regex-automata и aho-corasick: компиляцию DFA, сканирование байтовых
  срезов, сопоставление сотен сигнатур за один проход и распараллеливание. Активируется
  при разработке сканеров секретов, парсеров логов и оптимизации строкового поиска.
---

# Регламент многошаблонного сканирования (regex-automata-scanner)

Данный навык устанавливает стандарты разработки систем высокоскоростного поиска по множеству регулярных выражений на языке Rust.

## Ключевые архитектурные требования

1. Запрещен последовательный запуск `Regex::is_match` по списку выражений в цикле.
2. Использовать `regex_automata::meta::Regex` или составной детерминированный конечный автомат (DFA).
3. Сканировать байтовые срезы `&[u8]`, избегая предварительной валидации UTF-8 для файлов произвольного формата.
4. Отключать захватывающие группы (captures) в цикле горячего сканирования; извлекать координаты только после факта совпадения.

---

## Конфигурация зависимостей `Cargo.toml`

```toml
[dependencies]
regex-automata = { version = "0.4", features = ["dfa-build", "hybrid", "nfa-pikevm"] }
aho-corasick = "1.1"
rayon = "1.10"
```

---

## Построение многошаблонного сканера

### 1. Компиляция мета-движка

```rust
use regex_automata::meta::Regex;
use regex_automata::PatternID;

pub struct SecretScanner {
    engine: Regex,
    rules: Vec<String>,
}

impl SecretScanner {
    pub fn new(patterns: &[&str]) -> Result<Self, regex_automata::meta::BuildError> {
        let engine = Regex::builder()
            .configure(Regex::config().utf8_empty(false))
            .build_many(patterns)?;

        let rules = patterns.iter().map(|s| s.to_string()).collect();
        Ok(Self { engine, rules })
    }

    pub fn scan_bytes(&self, haystack: &[u8]) -> Vec<(PatternID, usize, usize)> {
        self.engine
            .find_iter(haystack)
            .map(|m| (m.pattern(), m.start(), m.end()))
            .collect()
    }
}
```

---

## Оптимизация горячего пути

### 1. Предварительный фильтр через Aho-Corasick

Если правила содержат обязательные константные литералы (например, префиксы `ghp_`, `AKIA`, `eyJh`), перед запуском регулярного выражения применяется быстрый фильтр:

```rust
use aho_corasick::{AhoCorasick, MatchKind};

pub struct Prefilter {
    ac: AhoCorasick,
}

impl Prefilter {
    pub fn new(literals: &[&str]) -> Result<Self, aho_corasick::BuildError> {
        let ac = AhoCorasick::builder()
            .match_kind(MatchKind::Standard)
            .build(literals)?;
        Ok(Self { ac })
    }

    pub fn is_candidate(&self, haystack: &[u8]) -> bool {
        self.ac.find(haystack).is_some()
    }
}
```

### 2. Параллельная обработка файлов через Rayon

```rust
use rayon::prelude::*;
use std::path::PathBuf;

pub fn scan_files(scanner: &SecretScanner, files: &[PathBuf]) -> Vec<(PathBuf, usize)> {
    files
        .par_iter()
        .filter_map(|path| {
            let content = std::fs::read(path).ok()?;
            let matches = scanner.scan_bytes(&content);
            if matches.is_empty() {
                None
            } else {
                Some((path.clone(), matches.len()))
            }
        })
        .collect()
}
```

---

## Чек-лист проверки и профилирования

- Движок инициализируется один раз и повторно используется во всех потоках.
- Чтение файлов осуществляется напрямую в байтовые буферы без конвертации в `String`.
- В кодовой базе отсутствуют отладочные комментарии.
- Проведено бенчмарк-тестирование пропускной способности (МБ/с) через Criterion.
