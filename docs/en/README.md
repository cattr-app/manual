# Getting started  :id=intro

Cattr is an open-source time tracking solution, designed to be flawlessly integrated with your infrastructure. Superpowered with features like built-in screenshot capture and activity detection, it's a great instrument to boost your team's performance straight to the top.

## Minimal requirements  :id=requirements
* CPU: 2 core
* RAM: 2 GB
* HDD/SSD: 5 GB reserved space (it is highly recommended)
* Docker: >= 18.09

## Installation  :id=installation

?> If you're experienced system administrator, you can use the [Advanced install](en/advanced/?id=intro) manual

To install Cattr core execute the following command and wait until Catt will setup (it can take some time):

```shell
docker volume create cattr-db && \
docker volume create cattr-screenshots && \
docker volume create cattr-ssl && \
docker run -d -it --restart on-failure:10 -p 80:80 -p 443:443 --name cattr \
-v cattr-db:/var/lib/mysql \
-v cattr-screenshots:/app/backend/storage/app/uploads/screenshots \
-v cattr-ssl:/etc/letsencrypt \
-e FRONTEND_DOMAIN="YOUR_FRONTEND_DOMAIN" \
-e BACKEND_DOMAIN="YOUR_BACKEND_DOMAIN" \
-e ADMIN_NAME="YOUR_NAME" \
-e ADMIN_MAIL="mail@example.com" \
-e ADMIN_PASSWORD="SUPER_PASSWORD" \
-e HTTPS="HTTPS_STATE" \
amazingcat/cattr
```

Don't forget yo use the correct params for Cattr installation:
- `YOUR_FRONTEND_DOMAIN` domain name the Frontend part will use
- `YOUR_BACKEND_DOMAIN` domain name the Backend part will use
- `YOUR_NAME` administrator account's name 
- `mail@example.com` administrator account's email
- `SUPER_PASSWORD` administrator account's password 
- `HTTPS_STATE` should be `true` or `false` (if you set it as `true`, the Cattr's domains will have the https certificate issued by [Let's Encrypt](https://letsencrypt.org))

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

?> If you bumped into error that wasn't described above, feel free to ask a question in our [community](https://community.cattr.app).

## What's next?  :id=next

After you finish installing and configuring the Frontend and Backend modules, you will be able to login with the credentials you provided for Administrator user. Once you log in, you'll be able to create projects, tasks, and assign them to the new users.

?>You can read about how to do it all in [Create project](ru/workflow/?id=project), [Create task](ru/workflow/?id=task) and [Create user](ru/users/?id=create) sections.
