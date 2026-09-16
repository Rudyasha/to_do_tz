# Todo · Тестирование приложения

QA-документация в стиле [разбора инцидента](https://rudyasha.github.io/incident/task-1/): Material for MkDocs, русский поиск, светлая и тёмная темы.

**Сайт:** https://rudyasha.github.io/to_do_tz/

Материалы: приоритеты и риски, план Infinite Scroll, 10 API-кейсов, 8 кейсов фильтрации и 3 баг-репорта. Источник — предоставленный автором текст; новые прогоны тестов приложения не проводились.

## Локальный просмотр

```sh
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
mkdocs serve
```

Проверка сборки: `mkdocs build --strict`.

## Публикация

GitHub Actions собирает и публикует документацию после push в `main`. В Settings → Pages должен быть выбран источник GitHub Actions.

Тексты находятся в `docs/`, навигация — в `mkdocs.yml`, стили — в `docs/assets/styles.css`.
