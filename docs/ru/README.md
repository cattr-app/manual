# Начало работы  :id=intro

*тут идет краткое описание катра*

## Минимальные требования  :id=requirements
* CPU: 2 core
* RAM: 2 GB
* HDD/SSD: 5 GB зарезервированного свободного места (it is highly recommended)
* Docker: >= 18.09

## Установка  :id=installation

?> Если Вы опытный системный администратор, мы рекомендуем воспользоваться шагами установки из раздела [Установка для системных администраторов](ru/advanced/?id=intro)

Для установки Cattr откройте консоль и выполните команду:

```bash
docker run -d -it --restart on-failure:10 -p 80:80 -p 443:443\
-e FRONTEND_DOMAIN="YOUR_FRONTEND_DOMAIN" \
-e BACKEND_DOMAIN="YOUR_BACKEND_DOMAIN" \
-e ADMIN_NAME="YOUR_NAME" \
-e ADMIN_MAIL="YOUR_MAIL" \
-e ADMIN_PASSWORD="SUPER_PASSWORD" \
-e HTTPS="HTTPS_STATE"\
amazingcat/cattr
```

Не забудьте заменить параметры, с которыми будет запущен Cattr:
- `YOUR_FRONTEND_DOMAIN` на доменное имя, которое будет использовать Frontend-составляющая
- `YOUR_BACKEND_DOMAIN` на доменное имя, которое будет использовать Backend-составляющая
- `YOUR_NAME` на имя администратора, который будет создан в системе
- `YOUR_MAIL` на почтовый ящик администратора, который будет создан в системе
- `SUPER_PASSWORD` на пароль администратора, который будет создан в системе
- `HTTPS_STATE` на значение `true` или `false` (если установить значение `true`, то для используемых Cattr доменов будет выпущен https сертификат от [Let's Encrypt](https://letsencrypt.org))

## Часто возникающие ошибки  :id=errors

<details>
<summary>
docker: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
</summary>

**Убедитесь, что Docker запущен на машине, на которой Вы пытаетесь запустить Cattr и выполните команду еще раз.**
</details>

?> Если у Вас возникла ошибка, не описанная выше, то Вы всегда можете задать вопрос о ней на нашем [форуме](https://community.cattr.app).

## Что дальше?  :id=next

После запуска можно будет зайти по адресу, который был указан в качестве доменного имени Frontend-составляющей, авторизоваться с использованием заданных на предыдущем шаге учетных данных администратора и создать первые проекты, задачи и назначить их новым пользователям.

?>Как это сделать можно прочитать в разделах [Создание проекта](ru/workflow/?id=project), [Создание задач](ru/workflow/?id=task) и [Создание пользователя](ru/users/?id=create).
