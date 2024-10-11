# Основы DevOps и DataOps | ЛБ 2 | Докеризация приложений

## Цель и задачи лабораторной работы

Цель лабораторной работы: Изучить процесс контейнеризации приложения и проблемы, возникающие в этом процессе

Задачи лабораторной работы:

1. Контейнеризировать приложение и обратиться к нему по протоколу http
2. Сохранить состояние приложения и задать параметры работы через переменные окружения
3. Добавить реверс-прокси приложения с самоподписанными сертификатами для обращений по протоколу https

## Установка Docker


```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

## Задание 1 | Запуск приложения

```bash
# Установка зависимостей
pip install -r requirements.txt

# Запуск приложения
uvicorn main:app
```

Примеры запросов:
```bash
curl --silent http://127.0.0.1:8000/tasks/ | jq

curl --silent -X POST http://127.0.0.1:8000/tasks/ --data '{ "title": "Прочитать книгу", "description": "Закончить чтение книги по FastAPI", "completed": fals
e }' | jq
```

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

```bash
sudo docker run hello-world
```

```bash
sudo usermod -aG docker $USER

sudo systemctl restart docker

su - $USER
```

## Задание 2 | Докеризация приложения

Dockerfile:
```Dockerfile
FROM python:3.12

COPY requirements.txt /requirements.txt
RUN  pip install -r /requirements.txt && \
     rm /requirements.txt

WORKDIR /app
COPY src/*.py .

CMD ["uvicorn", "main:app", "--host", "0.0.0.0"]
```

Подготовка образа:
```bash
docker build . -t {image_tag}
```

Запуск образа:
```bash
docker run -it --rm {image_tag}
```
*Проблемы с подключением к приложению и сохранением его состояния*

Публикация образа:
```bash
docker push {image_tag}
```

## Задание 3 | Исправление запуска контейнера приложения

```bash
docker run -it --rm -p 8080:8000 -v $(pwd)/src/test.db:/app/test.db {image_tag}
```

Изменение порта приложения:
```Dockerfile
ENV PORT 8080

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "$PORT"]
```

## Задание 4 | Добавление реверс прокси

Создание самоподписанных сертификатов
```bash
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -sha256 -days 365 -nodes
```

Запуск:
```bash
docker compose up
```

## Задание 5 | Добавленеие версионнсти API

```python
import os

PATH_PREFIX = os.environ.gen("PATH_PREFIX", "v1")

@app.get(f"/{PATH_PREFIX}/tasks/", response_model=list[schemas.Task])
```

## Содержание отчёта

1. Цель и задачи лабораторной работы
2. Задание 1 | Запуск приложения
3. Задание 2 | Докеризация приложения, указать выявленные проблемы взаимодействия с контейнером и пути их решения
4. Задание 3 | Исправление запуска контейнера приложения, указать какие модификации потребовались для их реализации
5. Задание 4 | Запуск реверс прокси и описание docker-compose файла мульти-контейнерного запуска приложения
6. Задание 5 | Добавление версионности API и названия приложения в path приложения (версия по вариантам)