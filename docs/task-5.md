# Баг-репорты

<div class="suite-summary" markdown>

**3 баг-репорта · 1 Critical · 2 Major**

Обновление новой задачи · сохранение статуса · удаление

</div>

!!! info "Статус материалов"
    Ниже оформлены наблюдения из предоставленного текста. В рамках подготовки этой документации дефекты не воспроизводились; к BUG-001 приложен предоставленный скриншот. Исходный код приложения и вложения к BUG-002 и BUG-003 не предоставлены.

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

## BUG-002 · Статус задачи изменяется в UI при неуспешном PUT /todos/{id}

<div class="case-tags"><span class="case-tag negative">Severity: Major</span><span class="case-tag">Chrome · macOS</span><span class="case-tag neutral">По предоставленным наблюдениям</span></div>

### Окружение

Google Chrome, macOS, localhost:4200

### Предусловие

В списке присутствует Active задача. DevTools открыт.

### Шаги воспроизведения

1. Открыть фильтр Active > В DevTools переключить Network в Offline.
1. Установить checkbox у Active задачи.
1. Проверить PUT /todos/{id}.
1. Проверить Active, Completed и All.

<div class="result-panel actual" markdown>

### Фактический результат

- PUT завершается ошибкой сети, но completed изменяется локально до получения успешного response. Задача может исчезнуть из Active или отображаться как Completed, хотя сервер не подтвердил изменение. Сообщение об ошибке и rollback отсутствуют.

</div>

<div class="result-panel expected" markdown>

### Ожидаемый результат

- Если PUT не выполнен успешно, приложение не должно отображать изменение как сохраненное. Предыдущее значение completed должно быть восстановлено либо пользователь должен получить понятное состояние ошибки.

</div>

### Вложения

- Network с failed PUT.
- Скриншот задачи после неуспешного запроса.

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
