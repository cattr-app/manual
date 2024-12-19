# Advanced installation :id=intro :priority=8

In Windows, installation is only possible with [Docker](/en/getting-started/).

## System requirements :id=requirements

In order for your server to be able to work with our Core application, you'll need:

- Memory: at least 3Gb of RAM
- Storage: at least 10Gb of reserved disk space
- Mariadb > 10.7 or Percona Server for Mysql > 8.0.28
- PHP: >= 8.0 (we recommend 8.2)
- Node: = 18
- LibGD >= 2
- Yarn
- Composer and cURL are necessary to work with the Backend part
- A web-server, we recommend Nginx >= 1.22
- OS Linux: __We recommend using Ubuntu version 22.04 and higher or Debian version 11 or higher__.

### PHP modules

These modules are required for Core application functioning:

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

- Server application: [https://git.amazingcat.net/cattr/core/app](https://git.amazingcat.net/cattr/core/app)

3. Go to the directory of the project, execute the following command and follow the installation manager instructions:

```bash
composer install 
```

4. Execute the following commands in the project directory:

```
# Install dependencies
yarn install

# Build frontend application
yarn prod
```

5. Set up your web server so it could work with both Cattr backend and frontend modules

- HTTP root directory for Frontend part and Backend (API) part: `/app/public`

6. Creating a key for the project

```bash
php artisan key:generate
```

7. After connecting to the database via .env, run the following commands to perform migrations:
```bash
php artisan migrate
```

8. Configure statuses, priorities, and companies.

```bash 
php artisan db:seed --class=InitialSeeder
```

9. To create admin user run  
```bash
php artisan cattr:make:admin
```  
Use following creadentials to login:
```
admin@cattr.app
password
```

10.  Set up a cron job to run the command php82 /app/artisan schedule:run every minute.

Run the command to edit the cron jobs:

```bash
crontab -e
```

In the opened file, add the following line: 

```bash
* * * * * php82 /app/artisan schedule:run 
```
11. Start the WebSocket in the background and log the output to reverb.log:  
```bash
nohup php artisan reverb:serve > reverb.log 2>&1 &
```
12. Start the queue listener in the background, and all output, including errors, will be logged to queue.log: 

```bash
nohup php artisan queue:listen > queue.log 2>&1 &
```

## Configuration Examples :id=configuration-examples

You'll find web server configuration examples for Cattr bellow.

!> To make things easier to understand, examples below are demonstrating HTTP-only configuration.
In production environments, security is a key. Make sure that your production configuration will be
available only via HTTPS connection with modern ciphersuits, unless you have some extremely strong reasons not to do that.

### Configuration for nginx with single domain

Let's assume that Cattr should be installed to **cattr.acme.corp** without HTTPS with both frontend and backend on the same domain,
and paths to Cattr's Core apps are:

- **Frontend:**,**Backend:** /opt/server-application/app


Server block for nginx:

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
    #Increase the maximum POST request size for batch uploading.
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
    #Serves PHP scripts on the server
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
Configuration for nginx in nginx.conf:

```conf
# Nginx will operate as the www-data user
user www-data;
# The number of worker processes is selected automatically based on the number of processors on the server.
worker_processes auto;
pid /run/nginx.pid;
error_log /var/log/nginx/error.log;
include /etc/nginx/modules-enabled/*.conf;

events {
         # Each process can handle up to 8192 connections
        worker_connections 8192;
        # Processes will accept multiple connections at once.
        multi_accept on;
        # For scalable connection handling
        use epoll;
    }

http {

        ##
        # Basic Settings
        ##

        sendfile on;
        tcp_nopush on;
        # Disable TCP packet delays for immediate transmission.
        tcp_nodelay on;
        # Sets the timeout for persistent connections to 65 seconds
        keepalive_timeout 65;
        # Maximum of 2000 requests from a single connection
        keepalive_requests 2000;
        # Closes inactive connections
        reset_timedout_connection on;
        types_hash_max_size 2048;
        # Maximum request body size
        client_max_body_size 512;
        # Buffer settings for proxy requests
        proxy_buffer_size   128k;
        proxy_buffers   16 128k;
        proxy_busy_buffers_size   128k;
        # Server names hash size
        server_names_hash_bucket_size 64;
        # Disable server name use in redirects
        server_name_in_redirect off;
        # Disable Nginx version output in response headers
        server_tokens off;
        include /etc/nginx/mime.types;
        default_type application/octet-stream;

        ##
        # SSL Settings
        ##

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
        ssl_prefer_server_ciphers on;
        # List of ciphers for secure connections
        ssl_ciphers;
        # SSL session lifetime
        ssl_session_timeout 1h;
        # SSL session cache size of 50 MB
        ssl_session_cache shared:SSL:50m;
        # Disable SSL tickets for improved security
        ssl_session_tickets off;
        # Enable SSL stapling for certificate authenticity
        ssl_stapling_verify on;
        ssl_stapling on;
        # Add HSTS header
        add_header Strict-Transport-Security max-age=1576800;

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
        # Disable Gzip
        gzip_disable "msie6";
        # Minimum content length for compression
        gzip_min_length 20;
        gzip_vary on;
        gzip_proxied any;
        gzip_comp_level 6;
        gzip_buffers 16 8k;
        gzip_http_version 1.1;
        # Content types for compression
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