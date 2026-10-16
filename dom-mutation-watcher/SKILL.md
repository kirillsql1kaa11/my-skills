---
name: dom-mutation-watcher
description: >-
  Регламентирует эффективное отслеживание изменений DOM-дерева в браузерных расширениях
  и веб-клиентах с помощью MutationObserver: батчинг мутаций, debounce, фильтрация селекторов,
  предотвращение бесконечных циклов и очистка ресурсов. Активируется при модификации
  динамических интерфейсов и разработке клиентских расширений.
---

# Регламент отслеживания изменений DOM (dom-mutation-watcher)

Данный навык устанавливает стандарты использования `MutationObserver` для надежного и высокопроизводительного перехвата динамических элементов страницы в браузерных расширениях и клиентских скриптах.

## Ключевые требования

1. Никогда не выполнять тяжелые вычисления или запросы к DOM (`querySelectorAll`) синхронно внутри колбэка мутаций.
2. Сужать область наблюдения (`target`), избегая наблюдения за всем `document` или `document.body`, если целевой контейнер известен.
3. Ограничивать параметры наблюдения: использовать `attributeFilter` при отслеживании атрибутов, не включать `subtree: true` без необходимости.
4. Предотвращать бесконечные циклы: если обработчик модифицирует отслеживаемый DOM, временно отключать наблюдатель или помечать измененные узлы флагом `data-*`.
5. Обязательно освобождать ресурсы через `disconnect()` при выгрузке страницы или смене маршрута.

---

## Архитектура пакетной обработки с защитой от троттлинга

Реализация перехватчика с группировкой событий через `requestAnimationFrame`:

```typescript
export interface WatcherOptions {
  targetSelector: string;
  itemSelector: string;
  onAdded: (element: HTMLElement) => void;
}

export class DomBatchWatcher {
  private observer: MutationObserver | null = null;
  private pendingNodes: Set<HTMLElement> = new Set();
  private rafId: number | null = null;

  constructor(private readonly options: WatcherOptions) {}

  public start(): void {
    const root = document.querySelector(this.options.targetSelector) || document.body;

    this.observer = new MutationObserver((mutations) => {
      let hasNewNodes = false;

      for (const mutation of mutations) {
        if (mutation.type !== "childList") continue;

        mutation.addedNodes.forEach((node) => {
          if (node.nodeType !== Node.ELEMENT_NODE) return;

          const el = node as HTMLElement;
          if (el.matches(this.options.itemSelector)) {
            this.pendingNodes.add(el);
            hasNewNodes = true;
          }

          const children = el.querySelectorAll<HTMLElement>(this.options.itemSelector);
          children.forEach((child) => {
            this.pendingNodes.add(child);
            hasNewNodes = true;
          });
        });
      }

      if (hasNewNodes && this.rafId === null) {
        this.rafId = requestAnimationFrame(() => this.flush());
      }
    });

    this.observer.observe(root, {
      childList: true,
      subtree: true
    });
  }

  private flush(): void {
    this.rafId = null;
    const batch = Array.from(this.pendingNodes);
    this.pendingNodes.clear();

    for (const node of batch) {
      if (!node.isConnected) continue;
      this.options.onAdded(node);
    }
  }

  public stop(): void {
    if (this.observer) {
      this.observer.disconnect();
      this.observer = null;
    }
    if (this.rafId !== null) {
      cancelAnimationFrame(this.rafId);
      this.rafId = null;
    }
    this.pendingNodes.clear();
  }
}
```

---

## Защита от бесконечного цикла мутаций

Паттерн маркировки элементов перед их модификацией:

```typescript
function injectBadge(node: HTMLElement): void {
  if (node.dataset.injected === "true") {
    return;
  }

  node.dataset.injected = "true";

  const badge = document.createElement("span");
  badge.className = "custom-client-badge";
  badge.textContent = "Active";

  node.appendChild(badge);
}
```

---

## Ожидание появления единичного элемента

Паттерн промисификации для гарантированного поиска элемента без таймеров `setInterval`:

```typescript
export function waitForElement<T extends HTMLElement>(selector: string, timeoutMs = 10000): Promise<T> {
  return new Promise((resolve, reject) => {
    const existing = document.querySelector<T>(selector);
    if (existing) {
      return resolve(existing);
    }

    const timer = setTimeout(() => {
      observer.disconnect();
      reject(new Error(`Timeout waiting for ${selector}`));
    }, timeoutMs);

    const observer = new MutationObserver(() => {
      const el = document.querySelector<T>(selector);
      if (el) {
        clearTimeout(timer);
        observer.disconnect();
        resolve(el);
      }
    });

    observer.observe(document.body, {
      childList: true,
      subtree: true
    });
  });
}
```

---

## Чек-лист проверки

- Не используется периодический опрос DOM через `setInterval`.
- Обработчик не инициирует повторные мутации без защитного флага или `dataset`.
- В кодовой базе отсутствуют комментарии.
- Наблюдение останавливается вызовом `stop()` или `disconnect()` при деактивации скрипта.
