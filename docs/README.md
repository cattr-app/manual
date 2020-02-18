# Getting started  :id=intro

*Project's short description goes here*

## Minimal system requirements  :id=requirements
In order for your server to be able to work with our Core application, you'll need:
* CPU: 2 core
* RAM: 2 GB
* HDD/SSD: 5 GB of reserved free space (it is highly recommended)
* PHP: >=7.2
* Node: >=10.14
* Yarn is recommended to work with the Frontend part
* Composer is necessary to work with the Backend part
* Nginx (we recommend to use this particular web server)

## Installation  :id=installation
!> If you're not a qualified system administrator, we'd recommend you to go through the installation process with using the Docker image

1. Download the repositories for the Frontend and Backend application parts. You can find them here:
   * Backend: `тут идет ссылка на бэк`
   * Frontend: `тут идет ссылка на фронт`
2. Go to the directory with the Backend part, execute the following command `composer install && php artisan app:install` and follow the installation manager's instructions.

?> You'll be asked to provide the credentials you're gonna use for Administrator account. Use them to log in after you finish installation.

3. Go to the directory with the Frontend part
    1. Go to the `app/etc` directory and copy the `env.example.js`'s containments to the `env.js` file.
    2. Edit the`env.js`'containments, so it had the following variables' values:
        * `API_URL`: <ссылка на домен, на котором будет находиться Backend Cattr>
        * `API_VERSION`: 'v1'
        * `DEVELOPER_MODE`: 'package'
        * `LOCAL_BUILD`: false
    3. In the Frontend directory execute the following command: <br> `NODE_ENV=production yarn install && yarn compile` <br> or <br> `NODE_ENV=production npm install && npm run compile`
4. Set up your web server so it could work with Cattr: create configuration files for both Frontend and Backend modules.
    * Static files directory for Frontend module: `path/to/cattr/frontend/dist`
    * Static files directory for Backend module: `path/to/cattr/backend/public`

?>Make sure that these directories should be added as `root` for ___Nginx___ and as `DocumentRoot` for ___Apache___

!> If the Backend module is located on a different Cattr domain rather than Frontend module, you'll need to enable the `CORS_ENABLED=true` option in the Backend's environment configuration.


## What's next?  :id=next

After you finish installing and configuring the Frontend and Backend modules, you will be able to login with the credentials you provided for Administrator user. Once you log in, you'll be able to create projects, tasks, and assign them to the new users.

?>You can read about how to do it all in [Create project](ru/workflow/?id=project), [Create task](ru/workflow/?id=task) and [Create user](ru/users/?id=create) sections.
 
---

## Configuration samples  :id=configuration-examples

Bellow you'll find the configuration examples for different web servers to work with Cattr.

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
