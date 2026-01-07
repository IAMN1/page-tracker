## Увстановка зависемостей

```
python -m pip install --editable ".[dev]"
python -m pip freeze --exclude-editable > constraints.txt
```

---

## Запуск конетейнеров Docker

```
docker start redis-server
```

---

## Запуск flask приложения

```
flask --app page_tracker.app run
```

---

## Тесты

- Пример запуска тестов на распределенных серверах
```
python -m pytest -v ./tests/e2e/ \
    --flask-url http://127.0.0.1:5000 \
    --redis-url redis://127.0.0.1:6379
```

---

## Статический анализ кода

### проверка на ошибки
- black:
```
python -m black ./src --check
```

- isort:
```
python -m isort ./src --check
```

- flake8:
```
python -m flake8 ./src
```

### исправление найденных ошибок

- black:
```
python -m black ./src
```

- isort:
```
python -m isort ./src
```

- flake8
```
python -m flake8 ./src
```

### Оценка кода

- pylint:
```
python -m pylint ./src
```

### Проверка кода на уязвимости

- bandit:
```
python -m bandit -r ./src
```

## Сборка докер образа приложения Flask

```
docker build -t page-tracker .
```