## Работа с приложениями через терминал и docker 
### Увстановка зависимостей

```
python -m pip install --editable ".[dev]"
python -m pip freeze --exclude-editable > constraints.txt
```

---
### Запуск конетейнеров Docker

```
docker start redis-server
```

---
### Запуск flask приложения

```
flask --app page_tracker.app run
```

---
### Тесты

- Пример запуска тестов на распределенных серверах
```
python -m pytest -v ./tests/e2e/ \
    --flask-url http://127.0.0.1:5000 \
    --redis-url redis://127.0.0.1:6379
```

---
### Статический анализ кода
#### проверка на ошибки
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

#### исправление найденных ошибок

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

#### Оценка кода

- pylint:
```
python -m pylint ./src
```

#### Проверка кода на уязвимости

- bandit:
```
python -m bandit -r ./src
```
---
### Сборка докер образа приложения Flask

- с вервсионированием хэша-коммита в git
```
docker build -t page-tracker:$(git rev-parse --short HEAD) .
```

### Запуск контенеризированого приложения Flask
```
docker run -p 5000:5000 --name web-service page-tracker:a265dd6
```

---
### Создание сети docker-network
```
docker network create page-tracker-network
```

---
### Создаение volume для redis-данных
```
docker volume create redis-volume
```

---
### Запуск redis-контейнера
```
docker run -d \
            -v redis-volume:/data \
            --network page-tracker-network \
            --name redis-service \
            redis:latest
```

---
### Запуск контенеризированого приложения Flask
```
docker run -p 5000:5000 --name web-service page-tracker:a265dd6
```

### запуск контенеризированного приложения flask с указанием сети и redis
```
docker run -d \
            -p 5000:5000 \
            -e REDIS_URL=redis://redis-service:6379 \
            --network page-tracker-network \
            --name web-service \
            page-tracker:a265dd6
```

---
## Работа с приложениями через docker-compose


