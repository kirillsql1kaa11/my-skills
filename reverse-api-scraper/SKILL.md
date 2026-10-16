---
name: reverse-api-scraper
description: >-
  Регламентирует реверс-инжиниринг недокументированных и приватных веб/мобильных API:
  анализ сетевого трафика, воспроизведение авторизации и заголовков, расчет контрольных
  подписей и обход базовых ограничений. Активируется при создании парсеров, загрузчиков
  и интеграций со сторонними сервисами без публичного API.
---

# Регламент реверс-инжиниринга API (reverse-api-scraper)

Данный навык определяет порядок исследования приватных эндпоинтов, извлечения токенов авторизации и реализации надежных клиентов для взаимодействия с ними.

## Этап 1. Перехват и анализ сетевого трафика

1. Анализ запросов через DevTools (вкладка Network) или прокси:
   - Экспорт целевого запроса в формате cURL.
   - Определение обязательных заголовков: `User-Agent`, `Referer`, `Origin`, `Accept`, `Authorization`, `Cookie`.
   - Изоляция минимального набора заголовков, необходимого для успешного ответа сервера.

2. Анализ структуры параметров:
   - URL query параметры (наличие `timestamp`, `sign`, `signature`, `token`).
   - Тело запроса (JSON, form-urlencoded, Protobuf, multipart).

---

## Этап 2. Расчет клиентских подписей и токенов

Если запрос требует формирования динамической подписи:
1. Поиск функции хеширования в исходниках клиентского бандла:
   - Поиск по ключевым словам: `signature`, `sign`, `secret`, `createHmac`, `md5`, `sha256`.
2. Извлечение статической соли или клиентского секрета.
3. Воспроизведение алгоритма формирования строки для подписи (конкатенация URL, тела и текущего времени).

Пример реализации на Node.js:

```javascript
import crypto from "node:crypto";

function generateApiSignature(path, params, secret) {
  const sortedQuery = Object.keys(params)
    .sort()
    .map((k) => `${k}=${params[k]}`)
    .join("&");

  const rawString = `${path}?${sortedQuery}`;
  return crypto.createHmac("sha256", secret).update(rawString).digest("hex");
}
```

---

## Этап 3. Эмуляция клиентских сессий

1. Сохранение и автоматическое обновление cookies (`Set-Cookie`) между запросами.
2. Использование актуальных User-Agent профилей современных браузеров.
3. Поддержка сжатия (`Accept-Encoding: gzip, deflate, br, zstd`).

Пример выполнения запроса с обработкой ошибок:

```javascript
async function requestWithRetry(url, options, maxRetries = 3) {
  let attempt = 0;
  while (attempt < maxRetries) {
    try {
      const response = await fetch(url, options);
      if (response.status === 429) {
        const retryAfter = Number(response.headers.get("Retry-After")) || Math.pow(2, attempt);
        await new Promise((resolve) => setTimeout(resolve, retryAfter * 1000));
        attempt++;
        continue;
      }
      if (!response.ok) {
        throw new Error(`HTTP error: ${response.status}`);
      }
      return await response.json();
    } catch (err) {
      attempt++;
      if (attempt >= maxRetries) throw err;
      await new Promise((resolve) => setTimeout(resolve, 1000 * attempt));
    }
  }
}
```

---

## Этап 4. Потоковая выгрузка больших файлов и медиа

При скачивании медиапотоков (аудио, видео, архивы):
1. Использовать потоки (`stream.Readable.fromWeb(response.body)`), а не загрузку всего файла в оперативную память.
2. Проверять целостность через заголовок `Content-Length` или контрольную сумму.
3. Поддерживать докачку через заголовок `Range: bytes=X-`.

---

## Чек-лист безопасности и чистоты кода

- Все секреты и токены передаются через переменные окружения, а не зашиты в репозиторий.
- Код не содержит комментариев.
- Реализована задержка между запросами для предотвращения блокировки по IP.
- Запросы проверяют статус ответа и обрабатывают HTTP 401/403/429.
