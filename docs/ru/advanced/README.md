# Расширенная установка :id=intro :priority=8

В этом разделе описана сборка проекта из исходного кода для разработки. Для production-развертывания используйте [руководство по установке](/ru/getting-started/) или [инструкцию для Kubernetes](/ru/advanced/kube). Репозиторий сервера: [cattr-app/server-application на GitHub](https://github.com/cattr-app/server-application).

## Требования :id=requirements

Текущий исходный код сервера требует:

- PHP, совместимый с ограничением Composer ~8.2; production-образ использует PHP 8.3.
- Composer.
- Node.js 24.21, указанную в .nvmrc.
- Corepack и pnpm 12.5.1, зафиксированный в package.json.
- Базу данных MySQL-совместимого типа для локального запуска приложения.
- PHP-расширения, нужные проекту, включая GD, JSON, OpenSSL, PDO, ZIP и PDO MySQL при использовании MySQL.

## Локальная настройка для разработки :id=installation

Клонируйте репозиторий и установите PHP-зависимости:

~~~bash
git clone https://github.com/cattr-app/server-application.git
cd server-application
cp .env.example .env
composer install
php artisan key:generate
~~~

Укажите подключение к базе данных и APP_URL в .env. Перед созданием первой учетной записи администратора задайте уникальные APP_ADMIN_EMAIL и APP_ADMIN_PASSWORD и не используйте значения по умолчанию. Затем установите зависимости фронтенда с версиями, закрепленными в репозитории:

~~~bash
nvm use
corepack enable
pnpm install --frozen-lockfile
~~~

Подготовьте локальную базу данных и создайте учетную запись администратора:

~~~bash
php artisan migrate --seed --seeder=InitialSeeder
php artisan storage:link
php artisan cattr:make:admin
~~~

Запустите backend и frontend в отдельных терминалах:

~~~bash
php artisan serve
pnpm watch
~~~

Для задач, которым нужна фоновая обработка, запустите нужные процессы Laravel в дополнительных терминалах, например php artisan queue:work, php artisan reverb:start или php artisan schedule:work.

## Production-развертывание :id=configuration-examples

Используйте поддерживаемый образ контейнера и инструкции в разделе [Начало работы](/ru/getting-started/) или [Установка в Kubernetes](/ru/advanced/kube). Приведенные выше команды предназначены для локальной разработки и не настраивают production-веб-сервер.
