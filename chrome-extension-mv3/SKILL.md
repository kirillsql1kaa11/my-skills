---
name: chrome-extension-mv3
description: >-
  Регламентирует разработку браузерных расширений на Chrome Manifest V3:
  настройку manifest.json, жизненный цикл Service Worker, внедрение Content Scripts,
  организацию обмена сообщениями и сборку через Vite.
  Активируется при создании, модификации или отладке браузерных расширений.
---

# Регламент разработки расширений Chrome Manifest V3 (chrome-extension-mv3)

Данный навык устанавливает стандарты архитектуры, безопасности и межпроцессного взаимодействия при создании современных браузерных расширений на платформе Manifest V3.

## Ключевые требования платформы

1. Фоновые скрипты работают исключительно в формате Service Worker (без доступа к `window` и DOM).
2. Запрещено использование `eval()`, загрузки внешних скриптов с CDN и inline-скриптов в HTML.
3. Долгосрочное состояние сохраняется в `chrome.storage.local` или `chrome.storage.session`, так как Service Worker засыпает при бездействии.
4. Фоновые интервалы реализуются через `chrome.alarms`, а не `setInterval`.

---

## Базовая структура `manifest.json`

```json
{
  "manifest_version": 3,
  "name": "Extension Name",
  "version": "1.0.0",
  "description": "Описание расширения",
  "permissions": [
    "storage",
    "alarms",
    "activeTab",
    "scripting"
  ],
  "host_permissions": [
    "*://*.example.com/*"
  ],
  "background": {
    "service_worker": "background.js",
    "type": "module"
  },
  "content_scripts": [
    {
      "matches": ["*://*.example.com/*"],
      "js": ["content.js"],
      "run_at": "document_idle"
    }
  ],
  "action": {
    "default_popup": "popup.html",
    "default_icon": {
      "16": "icons/16.png",
      "48": "icons/48.png",
      "128": "icons/128.png"
    }
  }
}
```

---

## Архитектура компонентов и взаимодействие

### 1. Service Worker (`background.js`)

Паттерн инициализации периодических задач и слушателей событий:

```javascript
chrome.runtime.onInstalled.addListener(() => {
  chrome.alarms.create("sync-task", { periodInMinutes: 30 });
});

chrome.alarms.onAlarm.addListener((alarm) => {
  if (alarm.name === "sync-task") {
    executeBackgroundSync();
  }
});

async function executeBackgroundSync() {
  const data = await chrome.storage.local.get(["userSettings"]);
  if (!data.userSettings) return;
}
```

### 2. Обмен сообщениями между Content Script и Background

Отправка разового запроса из контентного скрипта:

```javascript
chrome.runtime.sendMessage({ action: "FETCH_DATA", payload: { id: 10 } }, (response) => {
  if (chrome.runtime.lastError) {
    return;
  }
  handleResult(response);
});
```

Прием и асинхронный ответ в Service Worker:

```javascript
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.action === "FETCH_DATA") {
    fetchResource(message.payload).then((data) => {
      sendResponse({ success: true, data });
    }).catch((error) => {
      sendResponse({ success: false, error: error.message });
    });
    return true;
  }
});
```

### 3. Внедрение в основной контекст страницы (Main World)

Если контентному скрипту требуется доступ к глобальным объектам страницы (`window.App`, WebSocket), скрипт внедряется с параметром `MAIN`:

В `manifest.json`:
```json
{
  "content_scripts": [
    {
      "matches": ["*://*.example.com/*"],
      "js": ["injected.js"],
      "world": "MAIN",
      "run_at": "document_start"
    }
  ]
}
```

---

## Сборка и организация рабочего окружения

Для модульной разработки на TypeScript/ES-модулях с поддержкой HMR рекомендуется сборщик Vite с плагином `@crxjs/vite-plugin`.

### Инициализация проекта

```powershell
npm create vite@latest my-extension -- --template vanilla-ts
cd my-extension
npm install @crxjs/vite-plugin@latest -D
```

### Конфигурация `vite.config.ts`

```typescript
import { defineConfig } from "vite";
import { crx } from "@crxjs/vite-plugin";
import manifest from "./manifest.json";

export default defineConfig({
  plugins: [crx({ manifest })],
  server: {
    port: 5173,
    strictPort: true,
    hmr: {
      port: 5173
    }
  }
});
```

### Сборка для публикации

```powershell
npm run build
```

Итоговый результат сборки находится в директории `dist/`, готовой для загрузки в `chrome://extensions` через режим разработчика.

---

## Чек-лист проверки расширения перед публикацией

- Манифест строго 3 версии (`"manifest_version": 3`).
- Запрошены только минимально необходимые разрешения (`permissions`).
- Отсутствуют вызовы `eval` и `new Function`.
- В Service Worker нет обращений к `window`, `document` и глобальным переменным в предположении, что они сохраняются между событиями.
- Все асинхронные обработчики `onMessage` возвращают `true`.
- Ошибки API проверяются через `chrome.runtime.lastError`.
