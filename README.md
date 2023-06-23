# Kонтейнеры и CI/CD для Kittygram
(Проект создан в рамках учебного курса Яндекс.Практикум)

## Описание проекта

Kittygram — социальная сеть для обмена фотографиями любимых питомцев, где можно добавить нового котика на сайт или изменить существующего, а также просмотреть записи других пользователей.
Проект состоит из бэкенд-приложения на Django и фронтенд-приложения на React.

Цель проекта :

- настроить запуск проекта Kittygram в контейнерах;
- настроить автоматическое тестирование и деплой этого проекта на удалённый сервер.

## Стэк технологии

- Python 3.9
- Django 3.2.3
- Django REST Framework 3.12.4
- PostgreSQL
- Docker
- CI/CD
- GitHub Actions
- Gunicorn 20.1.0
- Nginx 1.18.0

## Инструкция по запуску

1. Клонируйте репозиторий на сервер:
    ```
    git clone git@github.com:UMAtaullin/kittygram_final.git
    ```
2. Создайте файл .env и заполните его своими данными:
    ```bash
    POSTGRES_DB= [имя_базы_данных]
    POSTGRES_USER=[имя_пользователя_базы]
    POSTGRES_PASSWORD=[пароль_к_базе]
    DB_HOST=[имя_хоста]
    DB_PORT=[порт_соединения_к_базе]
    ```

### Создание Docker-образов

1.  Замените username на ваш логин на DockerHub:
    ```bash
    cd frontend
    docker build -t username/kittygram_frontend .
    cd ../backend
    docker build -t username/kittygram_backend .
    cd ../nginx
    docker build -t username/kittygram_gateway .
    ```

2. Загрузите образы на DockerHub:
    ```bash
    docker push username/kittygram_frontend
    docker push username/kittygram_backend
    docker push username/kittygram_gateway
    ```

### Деплой на сервере

1. Подключитесь к удаленному серверу
    ```bash
    ssh -i путь_до_файла_с_SSH_ключом/название_файла_с_SSH_ключом имя_пользователя@ip_адрес_сервера
    ```

2. Создайте на сервере директорию kittygram через терминал
    ```bash
    mkdir kittygram
    ```

3. Установка docker compose на сервер:
    ```bash
    sudo apt update
    sudo apt install curl
    curl -fSL https://get.docker.com -o get-docker.sh
    sudo sh ./get-docker.sh
    sudo apt-get install docker-compose-plugin
    ```

4. В директорию kittygram/ скопируйте файлы docker-compose.production.yml и .env:
    ```bash
    scp -i path_to_SSH/SSH_name docker-compose.production.yml username@server_ip:/home/username/kittygram/docker-compose.production.yml
    ```

5. Запустите docker compose в режиме демона:
    ```bash
    sudo docker compose -f docker-compose.production.yml up -d
    ```

6. Выполните миграции, соберите статические файлы бэкенда и скопируйте их в /backend_static/static/:
    ```bash
    sudo docker compose -f docker-compose.production.yml exec backend python manage.py migrate
    sudo docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic
    sudo docker compose -f docker-compose.production.yml exec backend cp -r /app/collected_static/. /backend_static/static/
    ```

7. На сервере в редакторе nano откройте конфиг Nginx:
    ```bash
    nano /etc/nginx/sites-enabled/default
    ```

8. Измените настройки location в секции server:
    ```bash
    location / {
        proxy_set_header Host $http_host;
        proxy_pass http://127.0.0.1:9000;
    }
    ```

9. Проверьте работоспособность конфига и перезапустите Nginx:
    ```bash
    sudo nginx -t
    sudo service nginx reload
    ```

### Настройка CI/CD

1. Файл workflow уже написан. Он находится в директории
    ```bash
    kittygram/.github/workflows/main.yml
    ```

2. Для адаптации его на своем сервере добавьте секреты в GitHub Actions:
    ```bash
    SECRET_KEY          # стандартный ключ, который создается при старте проекта

    DOCKER_USERNAME     # имя пользователя в DockerHub
    DOCKER_PASSWORD     # пароль пользователя в DockerHub
    HOST                # ip_address сервера
    USER                # имя пользователя
    SSH_KEY             # приватный ssh-ключ (cat ~/.ssh/id_rsa)
    PASSPHRASE          # кодовая фраза (пароль) для ssh-ключа

    TELEGRAM_TO         # id телеграм-аккаунта (можно узнать у @userinfobot, команда /start)
    TELEGRAM_TOKEN      # токен бота (получить токен можно у @BotFather, /token, имя бота)
    ```

## Примеры запросов

1. Создание пользователя:
    ```bash
    [POST]https://<ваш домен>/api/users/
    {
    "username": "Name",
    "password": "password"
    }
    ```

2. Получение списка всех котиков:
    ```bash
    [GET]https://<ваш домен>/api/cats/
    ```

## Автор
Атауллин Урал, студент Яндекс практикумa.
