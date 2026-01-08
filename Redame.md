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
### запуск контейнеров с помощью docker compose
```
docker compose up -d
```

---
## Сквозное тестирование сервисов

- Вариант 1 - тестирование с помощью локально:
```
python -m pytest web/tests/e2e/ \
        --flask-url http://localhost:5000 \
        --redis-url redis://localhost:6379
```
`Важно, чтобы все сервисы пробрасывали порты на localhost!`

- Вариант 2 - сквозное тесирование с помощью профилированного сервиса

`в docker-compose.yml добавляем сервис:`
```
...
test-service:
    profiles:
      - testing
    build:
      context: ./web
      dockerfile: Dockerfile.dev
    environment:
      REDIS_URL: "redis://redis-service:6379"
      FLASK_URL: "http://web-service:5000"
    networks:
      - backend-network
    depends_on:
      - redis-service
      - web-service
    command: >
      sh -c 'python -m pytest tests/e2e/ -vv
      --redis-url $$REDIS_URL
      --flask-url $$FLASK_URL'

...
```
`Тогда для тестирования нужно запустить его вместе с остальными сервисами, с помощью команды:`
```
docker compose --profile testing up -d
```
`После чего проверяем работу test-service по логам:`
```
docker compose logs test-service
```
`Пример вывода логов:`
```
test-service-1  | ============================= test session starts ==============================
test-service-1  | platform linux -- Python 3.13.11, pytest-9.0.2, pluggy-1.6.0 -- /home/customuser/venv/bin/python
test-service-1  | cachedir: .pytest_cache
test-service-1  | rootdir: /home/customuser
test-service-1  | configfile: pyproject.toml
test-service-1  | plugins: timeout-2.4.0
test-service-1  | collecting ... collected 1 item
test-service-1  | 
test-service-1  | tests/e2e/test_app_redis_http.py::test_should_update_redis PASSED        [100%]
test-service-1  | 
test-service-1  | ============================== 1 passed in 0.13s ===============================
```

---


