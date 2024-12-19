# Продвинутая установка :id=intro :priority=8

Под Windows поддерживается только установка через [Docker](/ru/getting-started/).

## Системные требования :id=requirements

Для работы Cattr, ресурсы вашего сервера должны соответствовать минимальным критериям:

- Оперативная память: не менее 3Gb
- Хранилище: не менее 10Gb зарезервированного дискового пространства
- Mariadb > 10.7 or Percona Server for Mysql > 8.0.28
- PHP: >= 8.0 (мы рекомендуем 8.2)
- Node: = 18
- LibGD >= 2
- Yarn
- Composer и cURL необходимы для функционирования приложения
- Веб-сервер, мы рекомендуем Nginx >= 1.22
- OS Linux: __Мы рекоммендуем использовать Ubuntu версии 22.04 и выше или Debian версии 11 и выше__

### Модули PHP

Данные модули необходимы для функционирования приложения:

- php82
- php82-gd
- php82-openssl
- php82-pdo_mysql
- php82-zip
- php82-mysqli
- php82-intl
- php82-session
- php82-exif
- php82-curl
- php82-xml
- php82-fileinfo
- php82-tokenizer
- php82-simplexml
- php82-dom
- php82-xmlwriter
- php82-xmlreader
- php82-iconv
- php82-posix
- php82-pecl-redis
- php82-pecl-apcu
- php82-pecl-swoole
- php82-pcntl

В apt-based системах, вы можете установить эти модули следующим образом:

```bash
# Use ~ondrej PPA
sudo add-apt-repository ppa:ondrej/php

# Install PHP and required modules
sudo apt install php8.2-{bcmath,bz2,intl,gd,mbstring,mysql,zip,fpm,curl,xml}
```

## Установка :id=installation

!> Если вы не чувствуете себя достаточно уверенно, то рекомендуем упрощённую установку, описанную в главе «[Начало работы](/en/getting-started/)».

1. Установите необходимые зависимости (перечисленные в разделе «системные требования»).

2. Загрузите Backend и Frontend монорепозиторий по ссылке ниже:

- Server application: [https://git.amazingcat.net/cattr/core/app](https://git.amazingcat.net/cattr/core/app)

3. Перейдите в директорию с проектом, выполните следующую команду и следуйте указаниям установщика:

```bash
  composer install 
```

4. Выполните следующие команды в директории проекта

```
# Install dependencies
yarn install

# Build frontend application
yarn prod
```

5. Настройте ваш веб-сервер на работу с Cattr. Ниже указан пример папок, которые следует использовать как root-директории в Nginx или DocumentRoot-директории в Apache:

- HTTP root directory для Frontend и для Backend (API): `/app/public`

6. Создаем ключ для проекта

```bash
php artisan key:generate
```

7. После подключения по .env к базе выполните миграции

```bash
php artisan migrate
```

8. Настройте статусы, приоритеты, компании. 

```bash 
php artisan db:seed --class=InitialSeeder
```

9. Чтобы создать пользователя с правами администратора, запустите команду  
```bash
php artisan cattr:make:admin
```  
Используйте следующие доступы для входа:
```
admin@cattr.app
password
```

10. Настройте ваш cron задачи, которая будет запускать команду php82 /app/artisan schedule:run каждую минуту

Выполните команду для редактирования заданий cron:

```bash
crontab -e
```

В открывшемся файле добавьте следующую строку:

```bash
* * * * * php82 /app/artisan schedule:run 
```
11. Запускаем websocket в фоновом режиме и записываем выводы reverb.log
```bash
nohup php artisan reverb:serve > reverb.log 2>&1 &
```
12. Запускаем очереди в фоновом режиме и все выводы включая ошибки будут записаны в queue.log

```bash
nohup php artisan queue:listen > queue.log 2>&1 &
```


## Примеры конфигураций :id=configuration-examples

Ниже вы найдете примеры различных конфигураций Cattr.

!> Чтобы облегчить понимание примеров, мы описали только HTTP-конфигурацию. Помните, что самое главное в реальных применениях — это безопасность.
Всегда используйте HTTPS с современными протоколами и наборами алгоритмов, если у вас нет крайней необходимости в использовании незащищённого HTTP.

### Конфигурация nginx с одним доменом

В этой конфигурации, Cattr устанавливается на один домен **cattr.acme.corp** и для Frontend, и для Backend.
В этом примере, в качестве путей для директорий Frontend и Backend используются следующие значения:

- **Frontend:**,**Backend:** /opt/server-application/app

Конфигурация сервера для nginx:

```conf

  map $http_upgrade $type {   
    default "web"; 
    websocket "ws"; 
  }
server {
    listen 80 default;
    server_name server_name;

    root /var/www/app/public;
    index index.php;

    location @web { try_files $uri $uri/ @octane; }
    location @ws{

    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $http_host;
    proxy_cache_bypass $http_upgrade;
    proxy_redirect off;
    proxy_pass http://127.0.0.1:8080;

 }
  location @octane {
    proxy_send_timeout 300;
    proxy_read_timeout 300;
    #Увеличьте максимальный размер POST-запроса для загрузки интервальными партиями.
    client_max_body_size 512M;
    client_body_temp_path /tmp;

    proxy_set_header Host $http_host;
    proxy_set_header SERVER_PORT $server_port;
    proxy_set_header REMOTE_ADDR $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

    proxy_pass http://127.0.0.1:8090$uri?$query_string;
  }

     location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
    #Запускает PHP-скрипты на сервере
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }

    client_max_body_size 64M;
    error_log /var/log/nginx/your-project-error.log;
    access_log /var/log/nginx/your-project-access.log;
}

```

Конфигурация для nginx в nginx.conf:

```conf
#Nginx будет работать от имени пользователя www-data
user www-data;
#Количество процессов worker выбирается автоматически в зависимости от количества процессоров на сервере.
worker_processes auto;
pid /run/nginx.pid;
error_log /var/log/nginx/error.log;
include /etc/nginx/modules-enabled/*.conf;

events {
        # Каждый процесс может обслуживать до 8192 соединений
        worker_connections 8192;
        # Процессы будут принимать сразу несколько соединений за раз.
        multi_accept on;
        # Для масштабируемой обработки соединений
        use epoll;
    }

http {

        ##
        # Basic Settings
        ##

        sendfile on;
        tcp_nopush on;
        # Отключение задержек в пакетах TCP для мгновенной передачи.
        tcp_nodelay on;
        # Устанавливает время ожидания для поддерживаемых соеденений в течени 65 секунд
        keepalive_timeout 65;
        # Максимум 2000 запросов от одного соединения
        keepalive_requests 2000;
        # Закрытие неактивных соеденений
        reset_timedout_connection on;
        types_hash_max_size 2048;
        # Максимальный размер тела запроса
        client_max_body_size 512;
        #Настройка буфера для проксирования запросов
        proxy_buffer_size   128k;
        proxy_buffers   16 128k;
        proxy_busy_buffers_size   128k;
        #Размер хеша имен серверов
        server_names_hash_bucket_size 64;
        #Отключение использование имени сервера в редиректах
        server_name_in_redirect off;
        #Отключение вывода версии Nginx в заголовках ответа
        server_tokens off;
        include /etc/nginx/mime.types;
        default_type application/octet-stream;

        ##
        # SSL Settings
        ##

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
        ssl_prefer_server_ciphers on;
        #Список шифров для защиты соединений
        ssl_ciphers
        #Время жизни сесси SSL
        ssl_session_timeout 1h;
        #Кэширование SSL-сессий размером 50 MB
        ssl_session_cache shared:SSL:50m;
        #Отключение тикетов SSL для улучшения безопасности
        ssl_session_tikets off;
        #Включение SSL stapling для подлинности сертификатов
        ssl_stapling_verify on;
        ssl_stapling on;
        #Добавление заголовка HSTS
        add_header Strict-Transport-Security max-age=1576800

        ##
        # Logging Settings
        ##

        log_format main '$remote_addr - $remote_user [$time_local] "$request" '
      '$status $body_bytes_sent "$http_referer" '
      '"$http_user_agent" "$http_x_forwarded_for"';
        access_log /var/log/nginx/access.log;
        error_log /var/log/nginx/error.log error;
        ##
        # Gzip Settings
        ##

        gzip on;
        # Отключение Gzip
        gzip_disable "msie6";
        # Минимальная длина конента для сжатия
        gzip_min_length 20;
        gzip_vary on;
        gzip_proxied any;
        gzip_comp_level 6;
        gzip_buffers 16 8k;
        gzip_http_version 1.1;
        # Типы содержимого для сжатия
        gzip_types text/css text/x-component application/x-javascript;
        ##
        # Virtual Host Configs
        ##

         map $http_upgrade $connection_upgrade {
        default upgrade;
        '' close;
          }
        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;
      }

```

