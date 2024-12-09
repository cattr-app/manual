# Getting started :id=intro :priority=9
This article describes simplified installation using Docker.

## Minimal requirements  :id=requirements
* RAM: at least 3Gb
* Storage: at least 10Gb of reserved disk space
* Docker: >= 20.10
* Docker compose: >= 2.3.4

## Installation  :id=installation

?> If you have enough experience, you can consider non docker [installation](ru/advanced/?id=intro) (linux only)

### Install docker

#### Windows

Download and install Docker Desktop from the [official site](https://www.docker.com/).

![docker](../../assets/en/getting-started/docker.png)

For Docker to work in Windows you may need to enable virtualization in BIOS and [install WSL 2](https://learn.microsoft.com/en-us/windows/wsl/install). The installation process is described in details [in the Docker user manual](https://docs.docker.com/desktop/setup/install/windows-install/).

#### Linux

Run the following commands in the terminal:

```bash
# Create none root user with sudo privilages
adduser cattr
usermod-aG sudo cattr
exit
# login into newly created user
# install docker 
sudo apt update
sudo apt install apt-transport-https ca-certificates curl software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
apt-cache policy docker-ce # make shure docker will be installed from Docker repo instead of the default Ubutnu repo
sudo apt install docker-ce
sudo systemctl status docker # check status

# add user to docker group
sudo usermod -aG docker ${USER}
su - ${USER} # apply changes
groups # check if group added
docker info # see info of installed docker

# install docker compose
mkdir -p ~/.docker/cli-plugins/
curl -SL https://github.com/docker/compose/releases/download/v2.30.3/docker-compose-linux-x86_64 -o ~/.docker/cli-plugins/docker-compose

chmod +x ~/.docker/cli-plugins/docker-compose
docker compose version # verify installation

# create directory for cattr server application
cd /home/cattr
mkdir cattr-app
```

### HTTP only setup, see HTTPS below  
Create a `docker-compose.yml` file with the following content:
```yaml
version: '3.9'

services:
  app:
	  image: registry.git.amazingcat.net/cattr/core/app:0-grant
    restart: unless-stopped
    ports:
      - "80:80"
    depends_on:
      db:
        condition: service_healthy
    volumes:
      - ./storage:/app/storage
    networks:
       - default
#      - web
    environment:
      - DB_USERNAME=root
      - DB_PASSWORD=bP8T109h6BuL
      - APP_KEY=base64:lg1m/12MHBbBpiWTXjot98Q9MP/nSzPrvLEU2beD+2Y=
      # Admin user only created on initial run with the following credentials, change them if needed
      - APP_ADMIN_EMAIL=admin@cattr.app
      - APP_ADMIN_PASSWORD=password
      - APP_ADMIN_NAME=Admin

  db:
    image: percona:8.0
    restart: unless-stopped
    environment:
      - MYSQL_DATABASE=cattr
      - MYSQL_ROOT_PASSWORD=bP8T109h6BuL
    cap_add:
      - SYS_NICE
    volumes:
      - ./data:/var/lib/mysql
    healthcheck:
      test: ['CMD', 'mysqladmin', 'ping', '-h', 'localhost', '--password=bP8T109h6BuL', '-u', 'root']
      timeout: 20s
      retries: 10
```

### HTTPS setup

If you wish to use custom domain. You need to install and setup nginx with proper ssl certificates.

You can setup nginx by youself or create the following `docker-compose.yml`:

```yaml
version: '3.9'
services:
  app:
    image: registry.git.amazingcat.net/cattr/core/app:0-grant
    restart: unless-stopped
    depends_on:
      db:
        condition: service_healthy
    volumes:
      - ./storage:/app/storage
    networks:
      - default
    environment:
      - DB_USERNAME=root
      - DB_PASSWORD=bP8T109h6BuL
      - APP_KEY=base64:lg1m/12MHBbBpiWTXjot98Q9MP/nSzPrvLEU2beD+2Y=
      # Admin user only created on initial run with the following credentials, change them if needed
      - APP_ADMIN_EMAIL=admin@cattr.app
      - APP_ADMIN_PASSWORD=password
      - APP_ADMIN_NAME=Admin

  db:
    image: percona:8.0
    restart: unless-stopped
    environment:
      - MYSQL_DATABASE=cattr
      - MYSQL_ROOT_PASSWORD=bP8T109h6BuL
    cap_add:
      - SYS_NICE
    volumes:
      - ./data:/var/lib/mysql
    healthcheck:
      test: ['CMD', 'mysqladmin', 'ping', '-h', 'localhost', '--password=bP8T109h6BuL', '-u', 'root']
      timeout: 20s
      retries: 10


#Nginx Service
  webserver:
    image: nginx:alpine
    container_name: webserver
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:80"
    volumes:
      - ./nginx/conf.d/:/etc/nginx/conf.d/
      - ./nginx/certs:/etc/nginx/certs
    networks:
      - default
```

Create `nginx/conf.d` and `nginx/certs` directories for a nginx service

#### Windows

Create folders manually or run the following command in the cmd or PowerShelll:

```bash
mkdir nginx
mkdir nginx/conf.d
mkdir nginx/certs
```

#### Linux

```bash
mkdir -p nginx/conf.d nginx/certs
```

Create a `nginx.conf` file in the `nginx/conf.d` directory and put below content in it:

```nginx
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    location / {
      # change domain for redirect
        return 301 https://cattr.app$request_uri;
    }
}
server {
    listen 443 ssl;
    listen [::]:443 ssl;
    http2 on;
    # change domain
    server_name cattr.app www.cattr.app;
    # use correct path for certs
    ssl_certificate /etc/nginx/certs/live/cattr.app/fullchain.pem; 
    ssl_certificate_key /etc/nginx/certs/live/cattr.app/privkey.pem; 
    ssl_dhparam /etc/nginx/certs/live/cattr.app/ssl-dhparams.pem;

    location / {
        proxy_set_header X-Real-IP  $remote_addr;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header Host $host;
        proxy_set_header X-Real-Port $server_port;
        proxy_set_header X-Real-Scheme $scheme;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;

        proxy_pass http://app:80;
    }
}
```

Don’t foget to put you certificates in `nginx/certs` folder and make sure the path to the certificates is correct in the config file.

### Persist database data

#### Windows

```bash
mkdir data
```

#### Linux

```bash
# create directory to persist database data
mkdir data

# check which permissions to give using the following command
docker run --rm percona:8.0 id mysql 
# outputs: uid=1001(mysql) gid=1001(mysql) groups=1001(mysql)

# set directory permissions
sudo chown -R 1001:1001 ./data
```

### Launching the app
Now you can launch the app with `docker compose up -d` command in the folder where the docker-compose.yml file is located, first launch on a slow 4 core 2000MHz  server should take no more than 5 minutes.

![result of running docker compose up -d](../../assets/en/getting-started/up.png)

To see logs run `docker compose logs -f`  
!> If you have waited for several minutes and it seems like the process froze on db side, run `docker compose down` and than launch it again with `docker compose up -d`
![docker compose logs -f](../../assets/en/getting-started/logs-db.png)

!> Wait untill all database migrations are completed

![completed database migrations](../../assets/en/getting-started/migrations-completed.png)

Finally you should see the following message at the end "Server running" and be able to access Cattr by http or https protocol which depends on the chosen installation method, for example with http installation if your server’s ip is 0.0.0.0 go to http://0.0.0.0 or with https visit your domain, for example https://cattr.app 
![logs of running app](../../assets/en/getting-started/running.png)

## Common errors list  :id=errors

<details>
<summary>
docker: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
</summary>

**Make sure Docker is installed on the server you're trying to launch Cattr, and execute the command one more time**
</details>

<details>
<summary>
Error starting userland proxy: listen tcp 0.0.0.0:80 bind: address already in use.
</summary>

**Make sure Cattr is not running already, and there is no other web services installed on the server you're trying to launch Cattr on. We would suggest you consulting your system administrator for that matter.**
</details>

?> If you bumped into error that wasn't described above, feel free to ask a question in our [Github Discussions](https://github.com/orgs/cattr-app/discussions).

## What's next?  :id=next

After you finish installing, you will be able to login with the credentials you provided for Administrator user. Once you log in, you'll be able to create projects, tasks, and assign them to the new users.

?>You can read about how to do it all in [Create project](en/workflow/?id=project), [Create task](en/workflow/?id=task) and [Create user](en/users/?id=create) sections.
