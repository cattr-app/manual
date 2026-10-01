# Начало работы  :id=intro :priority=9
Эта глава описывает установку серверной части приложения с использованием Docker для контейнеризации приложения.

?>Если Вы пользователь, то этого делать не нужно, достаточно [скачать клиент](ru/?id=if-youre-an-employee) и ввести в него данные учетной записи, полученные у администратора.

## Минимальные требования к серверной части  :id=requirements

### При базовой установке

* OS: 
  * Linux. __Мы рекоммендуем использовать Ubuntu версии 22.04 и выше или Debian версии 11 и выше__
  * Windows (10 или 11)
* RAM: не менее 3Гб
* Storage: не менее 10Гб зарезервированного свободного места
* Docker: >= 20.10
* Docker Compose v2

Для разработки из исходного кода см. требования в разделе [Расширенная установка](ru/advanced/).


## Установка на Linux Debian, Ubuntu, Alt :id=installation-linux-deb

?> Для разработки из исходного кода см. раздел [Расширенная установка](ru/advanced/?id=intro).

### Установка docker

#### Для Ubuntu и Debian
Выполните в терминале следующие команды в следующем порядке:


Создайте не root пользователя с правами sudo:
```bash
adduser cattr
usermod -aG sudo cattr
```

Залогиньтесь в новосозданного юзера и установите docker:
```bash
sudo apt update
sudo apt install apt-transport-https ca-certificates curl software-properties-common
```

#### Для Alt
Создайте не root пользователя с расширенными правами, для этого нужно его добавить в группу `wheel`. Эта группа даёт пользователю доступ к запуску команды `su -`, чтобы выполнять команды с повышенными правами:  

Выполняйте следующие команды от `root` пользователя, чтобы переключиться на него, выполните команду `su -`.
```bash
apt-get update
/usr/sbin/adduser cattr
# Установите пароль
/usr/sbin/passwd cattr
# Добавьте пользователя в wheel группу
/usr/sbin/usermod -aG wheel cattr
```

Установите docker и docker compose:
```bash
# Установка docker
apt-get install docker-engine
# Добавьте пользователя в docker группу
/usr/sbin/usermod -aG docker cattr
# Запуск соответствующей службы
systemctl enable --now docker
# Перезагрузка
reboot
```
```bash
# Установка docker compose
apt-get install docker-compose-v2
```
```bash
systemctl status docker # проверьте статус docker службы
docker info # посмотрите информацию об установленном docker
docker compose version # проверьте установку docker compose
```

#### Для Ubuntu

```bash

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

#### Для Debian

```bash

curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
#### Для Ubuntu и Debian

```bash
sudo apt update
apt-cache policy docker-ce # убедитель что docker будет установлен из Docker репозитория вместо стандартного репозитория Ubuntu
sudo apt install docker-ce
sudo systemctl status docker # проверьте статус
```

Добавьте пользователя cattr в docker группу:

```bash
sudo usermod -aG docker cattr
su - cattr # примените изменения
groups # проверьте что группа добавлена
docker info # посмотрите информацию об установленном docker
```

Установите docker compose:
```bash
mkdir -p ~/.docker/cli-plugins/
curl -SL https://github.com/docker/compose/releases/download/v2.30.3/docker-compose-linux-x86_64 -o ~/.docker/cli-plugins/docker-compose

chmod +x ~/.docker/cli-plugins/docker-compose
docker compose version # проверьте установку
```

#### Для Ubuntu, Debian и Alt

Создайте директорию для серверного приложения Кэттр и войдите в неё, выполняйте команды от пользоватеся `cattr`:

```bash
su cattr
cd /home/cattr
mkdir cattr-app
cd cattr-app
```

Далее нужно выбрать вариант установки с использованием https (для случая когда кэттр будет использоваться для работы) или http (если cattr используется только для тестирования, или при кластерной установке, когда этот функционал берет на себя обвязка кластера ingress/api gateway).

### Только HTTP установка, смотрите HTTPS ниже 

Создайте файл .env рядом с docker-compose.yml. Сгенерируйте уникальный ключ приложения командой openssl rand -base64 32 и укажите ее результат после префикса base64: в APP_KEY. Для каждого параметра DB_PASSWORD, DB_ROOT_PASSWORD и APP_ADMIN_PASSWORD задайте отдельный пароль; команда openssl rand -hex 32 создаст значение, которое можно вставить в файл напрямую. Не публикуйте и не добавляйте .env в Git. В APP_URL укажите адрес, по которому пользователи будут открывать сервер; для HTTPS используйте адрес с https.

~~~dotenv
APP_URL=http://your-server.example.com
APP_KEY=base64:REPLACE_WITH_RANDOM_KEY
DB_PASSWORD=REPLACE_WITH_RANDOM_PASSWORD
DB_ROOT_PASSWORD=REPLACE_WITH_ANOTHER_RANDOM_PASSWORD
APP_ADMIN_EMAIL=admin@example.com
APP_ADMIN_PASSWORD=REPLACE_WITH_RANDOM_PASSWORD
APP_ADMIN_NAME=Admin
~~~

Создайте файл `docker-compose.yml` со следующим содержимым:

~~~yaml
services:
  app:
    image: ghcr.io/cattr-app/server:latest
    restart: unless-stopped
    ports:
      - "80:8080"
    depends_on:
      db:
        condition: service_healthy
    volumes:
      - backend_storage:/opt/cattr/app/storage
    environment:
      APP_URL: "${APP_URL:?Set APP_URL in .env}"
      APP_ENV: production
      APP_DEBUG: "false"
      APP_KEY: "${APP_KEY:?Set APP_KEY in .env}"
      DB_CONNECTION: mysql
      DB_HOST: db
      DB_PORT: "3306"
      DB_DATABASE: cattr
      DB_USERNAME: cattr
      DB_PASSWORD: "${DB_PASSWORD:?Set DB_PASSWORD in .env}"
      APP_ADMIN_EMAIL: "${APP_ADMIN_EMAIL:?Set APP_ADMIN_EMAIL in .env}"
      APP_ADMIN_PASSWORD: "${APP_ADMIN_PASSWORD:?Set APP_ADMIN_PASSWORD in .env}"
      APP_ADMIN_NAME: "${APP_ADMIN_NAME:-Admin}"

  db:
    image: docker.io/percona:8.0
    restart: unless-stopped
    environment:
      MYSQL_DATABASE: cattr
      MYSQL_USER: cattr
      MYSQL_PASSWORD: "${DB_PASSWORD:?Set DB_PASSWORD in .env}"
      MYSQL_ROOT_PASSWORD: "${DB_ROOT_PASSWORD:?Set DB_ROOT_PASSWORD in .env}"
    cap_add:
      - SYS_NICE
    volumes:
      - database:/var/lib/mysql
    healthcheck:
      test: ['CMD', 'mysqladmin', 'ping', '-h', 'localhost']
      timeout: 20s
      retries: 10

volumes:
  backend_storage:
  database:
~~~

### HTTPS установка

Если вы хотите использовать собственный домен. Вам нужно установить nginx и подключить ssl сертификаты.

Вы можете установить nginx сами или создать следующий `docker-compose.yml`:

~~~yaml
services:
  app:
    image: ghcr.io/cattr-app/server:latest
    restart: unless-stopped
    expose:
      - "8080"
    depends_on:
      db:
        condition: service_healthy
    volumes:
      - backend_storage:/opt/cattr/app/storage
    environment:
      APP_URL: "${APP_URL:?Set APP_URL in .env}"
      APP_ENV: production
      APP_DEBUG: "false"
      APP_KEY: "${APP_KEY:?Set APP_KEY in .env}"
      DB_CONNECTION: mysql
      DB_HOST: db
      DB_PORT: "3306"
      DB_DATABASE: cattr
      DB_USERNAME: cattr
      DB_PASSWORD: "${DB_PASSWORD:?Set DB_PASSWORD in .env}"
      APP_ADMIN_EMAIL: "${APP_ADMIN_EMAIL:?Set APP_ADMIN_EMAIL in .env}"
      APP_ADMIN_PASSWORD: "${APP_ADMIN_PASSWORD:?Set APP_ADMIN_PASSWORD in .env}"
      APP_ADMIN_NAME: "${APP_ADMIN_NAME:-Admin}"

  db:
    image: docker.io/percona:8.0
    restart: unless-stopped
    environment:
      MYSQL_DATABASE: cattr
      MYSQL_USER: cattr
      MYSQL_PASSWORD: "${DB_PASSWORD:?Set DB_PASSWORD in .env}"
      MYSQL_ROOT_PASSWORD: "${DB_ROOT_PASSWORD:?Set DB_ROOT_PASSWORD in .env}"
    cap_add:
      - SYS_NICE
    volumes:
      - database:/var/lib/mysql
    healthcheck:
      test: ['CMD', 'mysqladmin', 'ping', '-h', 'localhost']
      timeout: 20s
      retries: 10

  webserver:
    image: nginx:alpine
    container_name: webserver
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/conf.d/:/etc/nginx/conf.d/:ro
      - ./nginx/certs:/etc/nginx/certs:ro

volumes:
  backend_storage:
  database:
~~~

Создайте директории `nginx/conf.d` и `nginx/certs` для сервиса nginx

```bash
mkdir -p nginx/conf.d nginx/certs
```

Создайте `nginx.conf` файл в директории `nginx/conf.d` и поместите в него контент указанный ниже:

```nginx
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    location / {
      # измените домен для редиректа
        return 301 https://your-domain.example.com$request_uri;
    }
}
server {
    listen 443 ssl;
    listen [::]:443 ssl;
    http2 on;
    # измените домен
    server_name your-domain.example.com;
    # используйте правильный путь к вашим сертификатам
    ssl_certificate /etc/nginx/certs/fullchain.pem;
    ssl_certificate_key /etc/nginx/certs/privkey.pem;

    location / {
        proxy_set_header X-Real-IP  $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_set_header X-Real-Port $server_port;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;

        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_pass http://app:8080;
    }
}
```

Поместите сертификат и ключ в файлы nginx/certs/fullchain.pem и nginx/certs/privkey.pem.

Данные приложения и базы сохраняются в именованных томах Docker, поэтому создавать каталоги на хосте и менять права на них не нужно.

### Запускаем приложение 
Теперь вы можете запустить приложение запустив команду `docker compose up -d` в папке, в которой находится файл docker-compose.yml, первый запуск может занять несколько минут, пока база данных инициализируется.

![результат запуска docker compose up -d](../../assets/en/getting-started/up.png)

 Первый запуск может занять несколько минут, пока база данных инициализируется. Проверьте состояние в Docker Desktop или командой docker compose ps.

По завершении запуска приложения cattr сервер начнет отзываться по адресу http://localhost

Для входа используйте адрес и пароль администратора, указанные в файле .env.


?>Если что то пошло не так - [Отладка и возможные ошибки](ru/getting-started/?id=debug-and-errors)

## Установка на Windows  :id=installation-windows

### Установка docker

Скачайте и установите Docker Desktop с [официального сайта](https://www.docker.com/).

![docker](../../assets/en/getting-started/docker.png)

Для работы Docker в Windows вам может потребоваться включить виртуализацию в BIOS и [установить WSL 2](https://learn.microsoft.com/ru-ru/windows/wsl/install). Подробно процесс установки описан [в руководстве пользователя Docker](https://docs.docker.com/desktop/setup/install/windows-install/).

### Подготовка к запуску cattr

Создайте директории вручную или выполните следующие команды в cmd или PowerShell:

```bash
cd c:\
mkdir cattr-server
cd cattr-server

```

Создайте в этом каталоге файл .env с переменными из примера выше и укажите APP_URL=http://localhost для локального доступа. Затем создайте файл Compose:

Создайте файл `docker-compose.yml` в папке c:\cattr-server\ со следующим содержимым:

~~~yaml
services:
  app:
    image: ghcr.io/cattr-app/server:latest
    restart: unless-stopped
    ports:
      - "80:8080"
    depends_on:
      db:
        condition: service_healthy
    volumes:
      - backend_storage:/opt/cattr/app/storage
    environment:
      APP_URL: "${APP_URL:?Set APP_URL in .env}"
      APP_ENV: production
      APP_DEBUG: "false"
      APP_KEY: "${APP_KEY:?Set APP_KEY in .env}"
      DB_CONNECTION: mysql
      DB_HOST: db
      DB_PORT: "3306"
      DB_DATABASE: cattr
      DB_USERNAME: cattr
      DB_PASSWORD: "${DB_PASSWORD:?Set DB_PASSWORD in .env}"
      APP_ADMIN_EMAIL: "${APP_ADMIN_EMAIL:?Set APP_ADMIN_EMAIL in .env}"
      APP_ADMIN_PASSWORD: "${APP_ADMIN_PASSWORD:?Set APP_ADMIN_PASSWORD in .env}"
      APP_ADMIN_NAME: "${APP_ADMIN_NAME:-Admin}"

  db:
    image: docker.io/percona:8.0
    restart: unless-stopped
    environment:
      MYSQL_DATABASE: cattr
      MYSQL_USER: cattr
      MYSQL_PASSWORD: "${DB_PASSWORD:?Set DB_PASSWORD in .env}"
      MYSQL_ROOT_PASSWORD: "${DB_ROOT_PASSWORD:?Set DB_ROOT_PASSWORD in .env}"
    cap_add:
      - SYS_NICE
    volumes:
      - database:/var/lib/mysql
    healthcheck:
      test: ['CMD', 'mysqladmin', 'ping', '-h', 'localhost']
      timeout: 20s
      retries: 10

volumes:
  backend_storage:
  database:
~~~

Для входа администратора используйте адрес и пароль из APP_ADMIN_EMAIL и APP_ADMIN_PASSWORD в файле .env.

### Запускаем приложение 

Теперь, находясь в папке c:\cattr-server\ запустите приложение командой 

```bash
docker compose up -d

```

![результат запуска команды будет выглядеть так](../../assets/en/getting-started/cattr-docker-compose-up-windows.png)


 Первый запуск может занять несколько минут, пока база данных инициализируется. Проверьте состояние в Docker Desktop или командой docker compose ps.

По завершении запуска приложения cattr сервер начнет отзываться по адресу http://localhost

Для входа используйте адрес и пароль администратора, указанные в файле .env.


?>Если что то пошло не так - [Отладка и возможные ошибки](ru/getting-started/?id=debug-and-errors)


### Отладка :id=debug-and-errors

Для просмотра логов запустите команду `docker compose logs -f`  
!> Если прошло несколько минут и похоже что процесс завис на базе данных, запустите `docker compose down` а потом снова запустите с помощью `docker compose up -d`
![docker compose logs -f](../../assets/en/getting-started/logs-db.png)

!> Подождите завершения всех миграций базы данных

![завершенные миграции базы данных](../../assets/en/getting-started/migrations-completed.png)

Наконец в логах Вы увидите сообщение "Server running" и сможете получить доступ к Кэттр по http или https протоколу в зависимости от ранее выбранного режима установки.
Например по http если ip вашего сервера 0.0.0.0, откройте в браузере http://0.0.0.0 или для https посетите ваш домен, например https://cattr.app
![логи работающей программы](../../assets/en/getting-started/running.png)




## Часто возникающие ошибки  :id=errors

<details>
<summary>
docker: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
</summary>

**Убедитесь, что Docker запущен на машине, на которой Вы пытаетесь запустить Cattr и выполните команду еще раз.**
</details>

<details>
<summary>
Error starting userland proxy: listen tcp 0.0.0.0:80 bind: address already in use.
</summary>

**Убедитесь, что Cattr не был запущен до этого момента и на машине, на которой Вы пытаетесь запустить Cattr не запущен http-сервер. Советуем обратиться к системному администратору.**
</details>

?> Если у Вас возникла ошибка, не описанная выше, то Вы всегда можете задать вопрос о ней в [Github Discussions](https://github.com/orgs/cattr-app/discussions) нашего проекта.

## Что дальше?  :id=next

После запуска можно будет зайти по адресу, который был указан в качестве доменного имени, авторизоваться с использованием заданных на предыдущем шаге учетных данных администратора и создать первые проекты, задачи и назначить их новым пользователям.

?>Как это сделать можно прочитать в разделах [Создание проекта](ru/workflow/?id=project), [Создание задач](ru/workflow/?id=task) и [Создание пользователя](ru/users/?id=create).
