# Продвинутая установка :id=intro :priority=8

## Системные требования :id=requirements
Для работы Cattr, ресурсы вашего сервера должны соответствовать минимальным критериям:
* Оперативная память: не менее 2Gb
* Хранилище: не менее 5Gb зарезервированного дискового пространства
* MySQL: >=8.0.19
* PHP: >=7.4
* Node: >=12
* Composer и cURL необходимы для функционирования приложения
* Веб-сервер, мы рекомендуем nginx

### Модули PHP
Данные модули необходимы для функционирования приложения:
* php-dom 
* php-bcmath
* php-bz2
* php-intl
* php-gd
* php-mbstring
* php-mysql
* php-zip
* php-fpm
* php-curl

В apt-based системах, вы можете установить эти модули следующим образом:
```bash
# Use ~ondrej PPA
sudo add-apt-repository ppa:ondrej/php

# Install PHP and required modules
sudo apt install php7.4-{bcmath,bz2,intl,gd,mbstring,mysql,zip,fpm,curl,xml}
```

## Установка  :id=installation
!> Если вы не чувствуете себя достаточно уверенно, то рекомендуем упрощённую установку, описанную в главе «[Начало работы](/en/getting-started/)».

1. Установите необходимые зависимости (перечисленные в разделе «системные требования»).

2. Загрузите исходный код Backend и Frontend модулей по ссылкам ниже:
  * Backend: [github.com/cattr-app/backend-application](https://github.com/cattr-app/backend-application)
  * Frontend: [github.com/cattr-app/frontend-application](https://github.com/cattr-app/frontend-application)

3. Перейдите в директорию с Backend-модулем, выполните следующую команду и следуйте указаниям установщика:
```bash
composer install && php artisan cattr:install
```

?> В течении установки, у вас будут запрошены учётные данные для администраторского доступа. Используйте их в дальнейшем для входа в систему.

4. Перейдите в директорию с Frontend-модулем:
  1. Перейдите в директорию `app/etc` и скопируйте файл `env.production.js` в `env.js`.
  2. Отредактируйте `env.js` и удостоверьтесь, что значения в нём соответствуют перечисленным ниже:
    * **API_URL:** полный URL-адрес Backend-приложения
    * **API_VERSION:** `v1`
    * **DEVELOPER_MODE:** `package`
    * **LOCAL_BUILD:** `false`
  3. Выполните следующие команды в директории с Frontend-модулем
     ```
     # Install dependencies
     npm install

     # Compile modules
     NODE_ENV=production npm run compile

     # Build frontend application
     NODE_ENV=production npm run build
     ```

5. Настройте ваш веб-сервер на работу с Cattr. Ниже указан пример папок, которые следует использовать как root-директории в Nginx или DocumentRoot-директории в Apache:
  * HTTP root directory для Frontend: `path/to/cattr-frontend-application/dist`
  * HTTP root directory для Backend (API): `path/to/cattr-backend-application/public`

?> Если Frontend и Backend модули располагаются на разных доменах (например, api.cattr.acme.corp для Backend и cattr.acme.corp для Frontend),  
то установите значение `CORS_ENABLED=true` в конфигурационном файле Backend-модуля (.env файл в папке Backend-модуля).

## Примеры конфигураций  :id=configuration-examples
Ниже вы найдете примеры различных конфигураций Cattr.

!> Чтобы облегчить понимание примеров, мы описали только HTTP-конфигурацию. Помните, что самое главное в реальных применениях — это безопасность.
Всегда используйте HTTPS с современными протоколами и наборами алгоритмов, если у вас нет крайней необходимости в использовании незащищённого HTTP.

### Конфигурация nginx с одним доменом
В этой конфигурации, Cattr устанавливается на один домен **cattr.acme.corp** и для Frontend, и для Backend.
В этом примере, в качестве путей для директорий Frontend и Backend используются следующие значения:
  - **Frontend:** /opt/frontend-application
  - **Backend:** /opt/backend-application

Конфигурационный файл Frontend-модуля (/opt/frontend-application/app/etc/env.js) должен содержать следующие значения:
```js
module.exports = {
  API_URL: 'http://cattr.acme.corp/api',
  API_VERSION: 'v1',
  DEVELOPER_MODE: 'package',
  LOCAL_BUILD: false
};
```

Конфигурация сервера для nginx:
```conf
server {
  listen 80;
  listen [::]:80;
  server_name cattr.acme.corp;

  # Serve frontend
  root /opt/frontend-application/dist;
  index index.html;

  # Setup redirection for direct frontend links
  # Cattr uses HTML5 History API for routing
  location / {
    try_files $uri $uri/ /index.html;
  }

  # Backend (API) configuration
  location /api {

    rewrite ^/api/(.+)$ /$1 break;
    try_files $uri $uri/ /api/index.php?$query_string;
    
    # Increase max POST size for batch intervals uploading
    client_max_body_size 64M;

    # Serve PHP scripts
    location ~ \.php$ {
      include fastcgi_params;
      fastcgi_param SCRIPT_FILENAME /opt/backend-application/public/index.php;
      fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
    }

  }

}
```
