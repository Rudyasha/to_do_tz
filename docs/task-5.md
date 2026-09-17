# Баг-репорты

<div class="suite-summary" markdown>

**3 баг-репорта · 2 Critical · 1 Major**

Обновление новой задачи · одинаковые идентификаторы · удаление

</div>

!!! info "Статус материалов"
    Ниже оформлены наблюдения из предоставленного текста. В рамках подготовки этой документации дефекты не воспроизводились; к BUG-001 и BUG-002 приложены предоставленные скриншоты. Исходный код приложения и вложения к BUG-003 не предоставлены.

<div class="test-case bug-report" markdown>

## BUG-001 · PUT новой таски возвращает 500, Edit и Complete не работают

<div class="case-tags"><span class="case-tag negative">Severity: Critical</span><span class="case-tag">Chrome · macOS</span><span class="case-tag neutral">PUT /todos/201</span></div>

### Окружение

Google Chrome, macOS, `localhost:4200`

### Предусловие

Приложение открыто, API JSONPlaceholder доступен.

### Шаги воспроизведения

1. Создать новую задачу, например с `title = "Test task"`.
2. Убедиться, что `POST /todos` завершился успешно и в response получен новый id таски.
3. Кликнуть Edit у созданной задачи → изменить title → Save.
4. Проверить Network / Console.

<div class="result-panel actual" markdown>

### Фактический результат

- Отправляется `PUT /todos/{id новой таски}`;
- сервер возвращает `HTTP 500 Internal Server Error`;
- изменение title не сохраняется корректно;
- аналогичная ошибка возникает при изменении completed у созданной задачи;
- в Console отображается `HttpErrorResponse`;
- response имеет `Content-Type: text/html; charset=utf-8`, а не структурированный JSON error response.

</div>

### Дополнительные данные

```yaml
access-control-allow-origin: http://localhost:4200
x-ratelimit-remaining: 995
x-powered-by: Express
```

**Вывод:** запрос доходит до backend, ошибка не связана с CORS или rate limit и возникает при серверной обработке `PUT /todos/201`.

<div class="result-panel expected" markdown>

### Ожидаемый результат

- Созданная задача должна поддерживать последующий Update;
- `PUT /todos/{id}` должен завершаться успешным ответом;
- обновлённые данные должны применяться к выбранной таске;
- при ошибке API пользователь должен получить корректное состояние ошибки.

</div>

### Вложения

<figure class="bug-attachment" markdown>

[![Список задач и Chrome DevTools: PUT /todos/201 возвращает 500 Internal Server Error](assets/images/bug-001-put-500.png)](assets/images/bug-001-put-500.png){ target="_blank" rel="noopener" }

<figcaption>BUG-001 · Ошибка PUT /todos/201 в Network<br>Нажмите на скриншот, чтобы открыть в полном размере.</figcaption>
</figure>

</div>

<div class="test-case bug-report" markdown>

## BUG-002 · POST /todos возвращает одинаковый `id=201` для разных созданных задач

<div class="case-tags"><span class="case-tag negative">Severity: Critical</span><span class="case-tag">Chrome · macOS</span><span class="case-tag neutral">POST /todos · id=201</span></div>

### Окружение

Google Chrome, macOS, `localhost:4200`

### Предусловие

Приложение открыто, JSONPlaceholder доступен.

### Шаги воспроизведения

1. Создать задачу с `title = "Task A"`.
2. Проверить response `POST /todos` → сохранить полученный `id`.
3. Создать задачу с `title = "Task B"` → проверить response `POST /todos`.
4. Сравнить `id` созданных задач.
5. Попробовать изменить `title` или `completed` одной из них.

<div class="result-panel actual" markdown>

### Фактический результат

- Обе разные задачи получают одинаковый `id = 201`;
- frontend хранит несколько различных объектов с одинаковым идентификатором;
- последующие Edit / Complete / Delete определяют задачу по `id`, поэтому операция не может однозначно определить нужный объект.

</div>

<div class="result-panel expected" markdown>

### Ожидаемый результат

- Каждая отдельная таска на клиенте должна иметь уникальный id;
- Update/Delete должны однозначно применяться только к выбранной таске.

</div>

### Вложения

<div class="attachment-gallery" markdown>

<figure class="bug-attachment" markdown>

[![Chrome DevTools: список запросов к ресурсу 201](assets/images/bug-002-network-201.png)](assets/images/bug-002-network-201.png){ target="_blank" rel="noopener" }

<figcaption><strong>01 · Network</strong><br>Запросы к ресурсу с идентификатором 201.</figcaption>
</figure>

<figure class="bug-attachment" markdown>

[![JSON response с полями title, completed, userId и id: 201](assets/images/bug-002-response-201.png)](assets/images/bug-002-response-201.png){ target="_blank" rel="noopener" }

<figcaption><strong>02 · Response</strong><br>Объект задачи с <code>id: 201</code>.</figcaption>
</figure>

</div>

<p class="attachment-hint">Нажмите на любой скриншот, чтобы открыть его в полном размере.</p>

</div>

<div class="test-case bug-report" markdown>

## BUG-003. Удаление задачи завершается HTTP 500 и задача остается в списке

<div class="case-tags"><span class="case-tag negative">Severity: Major</span><span class="case-tag">Chrome · macOS</span><span class="case-tag neutral">По предоставленным наблюдениям</span></div>

**Окружение:** Google Chrome, macOS, http://localhost:4200

**Предусловие:** в списке существует задача, доступная для удаления.

### Шаги воспроизведения

1. Открыть приложение > Создать задачу
1. Нажать Delete у задачи.
1. В DevTools открыть Network > Найти DELETE /todos/{id}.
1. Проверить status code и Console.

<div class="result-panel actual" markdown>

### Фактический результат

- сервер возвращает HTTP 500 Internal Server Error;
- в Console появляется HttpErrorResponse;

</div>

<div class="result-panel expected" markdown>

### Ожидаемый результат

- DELETE должен завершаться ожидаемым успешным статусом, для JSONPlaceholder обычно 200 OK;

</div>

### Вложения

- скриншот Network с DELETE /todos/{id} и 500;
- скриншот Console с HttpErrorResponse.

!!! note "Уточнение для BUG-003"
    HTTP 500 сам по себе не устанавливает причину ошибки. Для воспроизведения нужно зафиксировать фактический ID, URL и тело ответа, а также сравнить удаление исходной и вновь созданной задачи. Обработка ошибки в UI проверяется отдельно от доступности внешнего API.

</div>
