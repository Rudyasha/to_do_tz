# Тест кейсы для Сценария Б

<div class="suite-summary" markdown>

**All · Active · Completed**

8 кейсов · фильтрация и согласованность состояния

</div>

Проверки состояния интерфейса и соответствующих API-запросов. Переключение фильтров предполагается локальным; это ожидание необходимо сверить с требованиями.

<div class="test-case" markdown>

## TC-B01. GET /todos: проверка данных, используемых для фильтрации

<div class="case-tags"><span class="case-tag positive">Positive / Consistency</span><span class="case-tag priority">Приоритет: Critical</span></div>

**Техника:** API Contract Testing.

**Предусловие:** приложение открывается с чистого состояния.

### Шаги

1. Выполнить загрузку списка.
1. Проверить GET /todos?_limit=10.
1. Сопоставить response со списком All.

### Ожидаемый результат

- HTTP 200 OK;
- response содержит 10 объектов;
- каждый объект содержит id, userId, title, completed;
- completed имеет тип boolean;
- список All содержит все объекты из response без потерь и дублей.

</div>

<div class="test-case" markdown>

## TC-B02. Фильтр Active по completed = false

<div class="case-tags"><span class="case-tag positive">Positive / Consistency</span><span class="case-tag priority">Приоритет: High</span></div>

**Техника:** Equivalence Partitioning.

**Предусловие:** первоначальный GET завершился HTTP 200 OK и содержит задачи с разными значениями completed.

### Шаги

1. Выбрать Active.
1. Сопоставить отображенные задачи с исходным GET response.

### Ожидаемый результат

- отображаются только объекты с completed = false;
- объекты с completed = true отсутствуют;
- id, userId, title исходных объектов не изменяются;
- дополнительный GET при переключении фильтра не отправляется.

</div>

<div class="test-case" markdown>

## TC-B03. Фильтр Completed по completed = true

<div class="case-tags"><span class="case-tag positive">Positive / Consistency</span><span class="case-tag priority">Приоритет: High</span></div>

**Техника:** Equivalence Partitioning.

**Предусловие:** первоначальный GET завершился HTTP 200 OK.

### Шаги

1. Выбрать Completed.
1. Сопоставить результат с полем completed исходного response.

### Ожидаемый результат

- отображаются только объекты с completed = true;
- completed = false отсутствуют;
- id, userId, title не изменяются;
- дополнительный HTTP-запрос не отправляется.

</div>

<div class="test-case" markdown>

## TC-B04. PUT /todos/{id}: переход completed = false -> true

<div class="case-tags"><span class="case-tag positive">Positive / State transition</span><span class="case-tag priority">Приоритет: Critical</span></div>

**Техника:** State Transition Testing.

**Предусловие:** задача:

```json
{
  "id": 1,
  "userId": 1,
  "title": "delectus aut autem",
  "completed": false
}
```

### Шаги

1. Изменить задачу на Completed.
1. Проверить PUT /todos/1.
1. Проверить фильтры Active, Completed и All.

### Ожидаемый результат

- HTTP 200 OK;
- response содержит completed = true;
- id = 1, userId = 1, title не изменяются;
- задача исчезает из Active;
- появляется в Completed;
- остается в All в единственном экземпляре.

</div>

<div class="test-case" markdown>

## TC-B05. PUT /todos/{id}: обратный переход completed = true -> false

<div class="case-tags"><span class="case-tag positive">Positive / State transition</span><span class="case-tag priority">Приоритет: Critical</span></div>

**Техника:** State Transition Testing.

**Предусловие:** задача имеет completed = true.

### Шаги

1. Снять отметку Completed.
1. Проверить PUT.
1. Проверить Active и Completed.

### Ожидаемый результат

- HTTP 200 OK;
- completed = false;
- id, userId, title не изменены;
- задача появляется в Active;
- исчезает из Completed;
- в All остается одна запись с тем же id.

</div>

<div class="test-case" markdown>

## TC-B06. DELETE /todos/{id}: удаление Completed-задачи и консистентность фильтров

<div class="case-tags"><span class="case-tag positive">Positive / Consistency</span><span class="case-tag priority">Приоритет: High</span></div>

**Техника:** CRUD Consistency Testing.

### Предусловие

задача существует;

completed = true;

задача присутствует в All и Completed.

### Шаги

1. Отправить DELETE /todos/{id} через действие Delete.
1. Проверить response.
1. Проверить All и Completed.

### Ожидаемый результат

- HTTP 200 OK;
- из клиентского списка удаляется только задача с указанным id;
- задача отсутствует в All;
- задача отсутствует в Completed;
- остальные задачи не изменены и не удалены.

</div>

<div class="test-case" markdown>

## TC-B07. DELETE -> GET того же id: проверка ограничения JSONPlaceholder

<div class="case-tags"><span class="case-tag neutral">Integration / Persistence</span><span class="case-tag priority">Приоритет: Medium</span></div>

**Техника:** Integration / Persistence Testing.

**Предусловие:** существует задача с известным id.

### Шаги

1. Отправить DELETE /todos/1.
1. Проверить response.
1. Отправить GET /todos/1.

### Ожидаемый результат

- DELETE возвращает HTTP 200 OK;
- последующий GET возвращает HTTP 200 OK;
- исходная задача снова возвращается API.
- Результат фиксируется как особенность JSONPlaceholder: DELETE имитируется и не сохраняется реально.

</div>

<div class="test-case" markdown>

## TC-B08. Последовательность All -> Active -> Completed -> All не изменяет данные

<div class="case-tags"><span class="case-tag positive">Positive / Consistency</span><span class="case-tag priority">Приоритет: Medium</span></div>

**Техника:** State Consistency Testing.

**Предусловие:** GET вернул набор задач с completed = true и false.

### Шаги

1. Сохранить исходные id, userId, title, completed.
1. Переключить All -> Active -> Completed -> All.
1. Проверить Network и конечный список.

### Ожидаемый результат

- при переключении фильтров PUT/POST/DELETE не отправляются;
- новые GET также не отправляются;
- id, userId, title, completed не меняются;
- после возврата в All набор соответствует исходному GET;
- нет потерянных или продублированных объектов.

</div>
