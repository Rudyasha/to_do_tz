# API: создание и изменение задач

В исходном материале описана подготовка Postman-коллекции с помощью AI и ручная проверка сценариев через Postman и DevTools. Файл коллекции и результаты прогонов к материалу не приложены.

!!! note "Контракт и допущения"
    Ограничение длины 255 символов и строгая валидация полей не подтверждены требованиями. Ожидания 400/422 в соответствующих кейсах условные: поведение тестового API следует фиксировать отдельно от дефектов приложения. Идентификаторы кейсов перенумерованы последовательно.


## TC-A01. POST /todos: создание задачи с валидными данными


**Техника:** Positive / Contract testing.


### Request


```json
{
  "userId": 1,
  "title": "Buy milk",
  "completed": false
}
```


### Ожидаемый результат

- HTTP 201 Created;
- response содержит id, userId, title, completed;
- id сгенерирован сервером, тип number;
- userId = 1;
- title = "Buy milk";
- completed = false.

**Приоритет:** Critical



## TC-A02. POST /todos: минимальная длина title = "A"


**Техника:** Boundary Value Analysis.


### Request


```json
{
  "userId": 1,
  "title": "A",
  "completed": false
}
```


### Ожидаемый результат

- HTTP 201 Created;
- id сгенерирован;
- title = "A";
- остальные поля соответствуют request.

**Приоритет:** Medium


**Заметка:** Граничная проверка применима после подтверждения минимальной длины 1.



## TC-A03. POST /todos: title со спецсимволами и Unicode


**Техника:** Equivalence Partitioning.


### Request


```json
{
  "userId": 1,
  "title": "Купить молоко #1 <> & \"test\"",
  "completed": false
}
```


### Ожидаемый результат

- HTTP 201 Created;
- строка в response совпадает с request;
- Unicode и спецсимволы не повреждены;
- userId и completed не изменены.

**Приоритет:** Medium


**Заметка:** Ожидание применимо, если специальные символы разрешены требованиями.




## TC-A04. POST /todos: максимальная длина title, 255 и 256 символов


**Техника:** Boundary Value Analysis.


**Предусловие:** максимальная длина не определена требованиями. Для проверки используется допущение maxLength = 255.


### Шаги

1. Отправить title длиной 255 символов.
1. Отправить title длиной 256 символов.

### Ожидаемый результат

- Для 255:
- HTTP 201 Created;
- title возвращается полностью;
- строка не обрезается.
- Для 256:
- ожидается HTTP 400 или 422, если ограничение 255 поддерживается backend.
- Если оба запроса возвращают 201, фиксируется, что API не применяет предполагаемое ограничение длины.

**Приоритет:** Medium

Это exploratory boundary check: граница 255 не подтверждена требованиями.



## TC-A05. POST /todos: completed = null


**Техника:** Type validation / Negative testing.


### Request


```json
{
  "userId": 1,
  "title": "Buy milk",
  "completed": null
}
```


### Ожидаемый результат для строгого контракта

- HTTP 400 или 422;
- Todo с completed = null не создается.
- Применимо при обязательном boolean-поле completed.

**Приоритет:** High



## TC-A06. POST /todos: неверный тип completed = "false"


**Техника:** Type validation.


### Request


```json
{
  "userId": 1,
  "title": "Buy milk",
  "completed": "false"
}
```


### Ожидаемый результат

- HTTP 400 или 422 для типизированного API;
- строковое значение "false" не должно интерпретироваться как корректный boolean.
- Если API возвращает 201, фиксируется отсутствие schema validation.

**Приоритет:** High




## TC-A07. POST /todos: клиент передает id = null


**Техника:** Contract / Field ownership testing.


### Request


```json
{
  "id": null,
  "userId": 1,
  "title": "Task with null id",
  "completed": false
}
```


### Ожидаемый результат

- клиентский id = null не должен использоваться как ID созданной задачи;
- при успешном создании сервер возвращает собственный числовой id;
- либо API отклоняет передачу server-managed поля с HTTP 400/422;
- успешная сущность не должна иметь id = null.

**Приоритет:** High



## TC-A08. POST /todos: полностью пустой JSON {}


**Техника:** Negative testing / Required fields validation.


### Request


```json
{}
```


### Ожидаемый результат для продуктового API

- HTTP 400 Bad Request или 422 Unprocessable Entity;
- задача не создается;
- server-generated id не должен означать создание валидной Todo без title, userId и completed.
- Если JSONPlaceholder возвращает 201 Created, это фиксируется как существенное ограничение тестового API: обязательные поля и бизнес-контракт на стороне сервера не валидируются.

**Приоритет:** High


## TC-A09. DELETE /todos/{id} -> GET /todos/{id}: проверка persistence после удаления


**Техника:** CRUD consistency, Integration Testing.


**Предусловие:** задача с id = 1 доступна через API.


### Шаги

1. Отправить:
1. DELETE /todos/1
1. Проверить response.
1. После успешного DELETE отправить:
1. GET /todos/1
1. Проверить response GET.

### Ожидаемый результат

- DELETE:
- HTTP 200 OK;
- DELETE завершается без server error.
- Последующий GET:
- JSONPlaceholder возвращает HTTP 200 OK;
- задача с id = 1 снова присутствует;
- userId, id, title, completed соответствуют исходному объекту.

**Приоритет:** High


## TC-A10. PUT /todos/{id}: изменение completed = false -> true


**Техника:** State Transition Testing, Contract Testing.


**Предусловие:** существует задача:


```json
{
  "userId": 1,
  "id": 1,
  "title": "delectus aut autem",
  "completed": false
}
```


### Шаги

1. Отправить PUT /todos/1:

```json
{
  "userId": 1,
  "id": 1,
  "title": "delectus aut autem",
  "completed": true
}
```

1. Проверить status code и response body.

### Ожидаемый результат

- HTTP 200 OK;
- id = 1;
- userId = 1;
- title = "delectus aut autem" не изменился;
- completed изменился с false на true;
- типы всех полей соответствуют контракту;
- изменено только поле completed.

**Приоритет:** Critical


