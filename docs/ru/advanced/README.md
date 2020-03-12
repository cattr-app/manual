# Установка для системных администраторов  :id=intro

## Минимальные требования  :id=requirements
* CPU: 2 core
* RAM: 2 GB
* HDD/SSD: 5 GB зарезервированного свободного места (it is highly recommended)
* PHP: >=7.4
* Node: >=10.14
* Npm рекомендуется для работы с Frontend-составляющей
* Composer необходим для работы с Backend-составляющей
* Nginx (мы рекомендуем использовать именно данный веб-сервер)

## Установка  :id=installation
1. Скачайте репозитории Frontend и Backend частей приложения
   * Backend: `https://github.com/cattr-app/backend-application.git`
   * Frontend: `https://github.com/cattr-app/frontend-application.git`
2. Перейдите в каталог с Backend-составляющей и введите `composer install && php artisan app:install` и следуйте инструкциям менеджера по установке
3. Перейдите в каталог с Frontend-составляющей
    1. В папке `app/etc` скопируйте файл `env.example.js` в файл `env.js`
    2. Установите следующие значения переменных в файлу `env.js`
        * `API_URL`: <ссылка на домен, на котором будет находиться Backend Cattr>
        * `API_VERSION`: 'v1'
        * `DEVELOPER_MODE`: 'package'
        * `LOCAL_BUILD`: false
    3. В корневой папке Frontend-составляющей введите <br> `NODE_ENV=production yarn install && yarn compile` <br> или <br> `NODE_ENV=production npm install && npm run compile`
4. Настройте ваш веб-сервер для работы с Cattr: создайте конфигурационные файлы как для Frontend, так и для Backend.
    * Каталог статических файлов Frontend-составляющей: `path/to/cattr/frontend/dist`
    * Каталог статических файлов Backend-составляющей: `path/to/cattr/backend/public`

?>Обратите внимание, что эти пути должны быть прописаны в качестве `root` для ___Nginx___ и в качестве `DocumentRoot` для ___Apache___

!> Если Backend составляющая находится на домене, отличном от Frontend домена Cattr, вам необходимо включить опцию `CORS_ENABLED=true` в конфигурации окружения Backend'а

## Примеры конфигурации  :id=configuration-examples

Ниже вы найдете примеры конфигурации различных веб-серверов для работы с Cattr.

### NGINX Config Sample

```nginx
# API Endpoint Configuration
server {
  listen 80;
  listen [::]:80;
  server_name api.example.co,;

  # Extend POST size
  client_max_body_size 256M;

  # Security headers
  add_header X-XSS-Protection 1;
  add_header X-Content-Type-Options nosniff;
  add_header Referrer-Policy "same-origin";
  add_header Upgrade-Insecure-Requests 1;
  add_header Content-Security-Policy upgrade-insecure-requests;
  add_header Strict-Transport-Security "max-age=31536000; preload;";

  # Serve static content
  root /srv/dev/backend/public;
  index index.php;

  # Routing and CSRF
  location / {
    try_files $uri $uri/ /index.php?$query_string;
    add_header 'Access-Control-Allow-Origin' 'https://api.example.com';
    add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS, PUT, DELETE';
    add_header 'Access-Control-Allow-Headers' '*';
    add_header 'Access-Control-Expose-Headers' '*';
  }

  # Handle PHP files
  location ~ \.php$ {
    fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
    include misc.d/fastcgi_params;
  }
}

# Frontend Configuration
server {
  listen 80;
  listen [::]:80;
  server_name example.com;
  client_max_body_size 256M;

  # Security headers
  add_header X-XSS-Protection 1;
  add_header X-Content-Type-Options nosniff;
  add_header Referrer-Policy "same-origin";
  add_header Upgrade-Insecure-Requests 1;
  add_header Content-Security-Policy upgrade-insecure-requests;
  add_header Strict-Transport-Security "max-age=31536000; preload;";

  # Serve static content
  root /srv/dev/frontend/dist;
  index index.html;

  # Routing rewrite
  location / {
    try_files $uri $uri/ /index.html;
  }

  # API proxy
  # At this particular example we're using "https://example.com/api" as an API endpoint
  # To prevent CORS issues
  location /api {
    rewrite ^/api/(.+)$ /$1 break;
    proxy_pass http://api.example.com;
  }

}
```
