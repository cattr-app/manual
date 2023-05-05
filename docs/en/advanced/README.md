# Advanced installation :id=intro :priority=8

## System requirements :id=requirements

In order for your server to be able to work with our Core application, you'll need:

- Memory: at least 2Gb of RAM
- Storage: at least 5Gb of reserved disk space
- MySQL: >=8.0.19
- PHP: >=8.2
- Node: >=16
- Composer and cURL are necessary to work with the Backend part
- A web-server, we recommend nginx

### PHP modules

These modules are required for Core application functioning:

- php-dom
- php-bcmath
- php-bz2
- php-intl
- php-gd
- php-mbstring
- php-mysql
- php-zip
- php-fpm
- php-curl

For Ubuntu, these modules can be installed with this command:

```bash
# Use ~ondrej PPA
sudo add-apt-repository ppa:ondrej/php

# Install PHP and required modules
sudo apt install php8.2-{bcmath,bz2,intl,gd,mbstring,mysql,zip,fpm,curl,xml}
```

## Installation :id=installation

!> If you're not a qualified system administrator, we'd recommend you to go through the Docker installation guide in the «[Getting started](/en/getting-started/)» section

1. Install the neccessary depdendencies

2. Download the Frontend and Backend monorepo. You can find it here:

- Server application: [https://github.com/cattr-app/server-application](https://https://github.com/cattr-app/server-application)

3. Go to the directory of the project, execute the following command and follow the installation manager instructions:

```bash
composer install && php artisan cattr:install
```

?> You'll be asked to provide the credentials you're gonna use for Administrator account. Use them to log in after you finish installation.

4. Execute the following commands in the project directory:

   ```
   # Install dependencies
   yarn

   # Build frontend application
   yarn prod
   ```

!> Information below needs to be updated.

5. Set up your web server so it could work with both Cattr backend and frontend modules

- HTTP root directory for Frontend part: `path/to/cattr-frontend-application/dist`
- HTTP root directory for Backend (API) part: `path/to/cattr-backend-application/public`

?> If the backend module is located on a different domain rather than Frontend module, you'll need to enable the `CORS_ENABLED=true` option in the Backend's environment configuration (`.env` file).

## Configuration Examples :id=configuration-examples

You'll find web server configuration examples for Cattr bellow.

!> To make things easier to understand, examples below are demonstrating HTTP-only configuration.
In production environments, security is a key. Make sure that your production configuration will be
available only via HTTPS connection with modern ciphersuits, unless you have some extremely strong reasons not to do that.

### Configuration for nginx with single domain

Let's assume that Cattr should be installed to **cattr.acme.corp** without HTTPS with both frontend and backend on the same domain,
and paths to Cattr's Core apps are:

- **Frontend:** /opt/frontend-application
- **Backend:** /opt/backend-application

Frontend configuration (/opt/frontend-application/app/etc/env.js) should looks like this:

```js
module.exports = {
  API_URL: "http://cattr.acme.corp/api",
  GET_SCREENSHOTS_BY_ID: true,
};
```

Server block for nginx:

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
