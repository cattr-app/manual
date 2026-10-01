# Advanced installation :id=intro :priority=8

This section documents a source checkout for development. For a production deployment, follow [Getting started](/en/getting-started/) or [Kubernetes installation](/en/advanced/kube). The server repository is [cattr-app/server-application on GitHub](https://github.com/cattr-app/server-application).

## Requirements :id=requirements

The current server source requires:

- PHP compatible with the Composer constraint ~8.2; the production runtime uses PHP 8.3.
- Composer.
- Node.js 24.21, as pinned in .nvmrc.
- Corepack and pnpm 12.5.1, as pinned in package.json.
- A MySQL-compatible database for a full local application setup.
- PHP extensions required by the project, including GD, JSON, OpenSSL, PDO, ZIP, and PDO MySQL when using MySQL.

## Local development setup :id=installation

Clone the repository and install the PHP dependencies:

~~~bash
git clone https://github.com/cattr-app/server-application.git
cd server-application
cp .env.example .env
composer install
php artisan key:generate
~~~

Set the database connection and APP_URL in .env. Also set a unique APP_ADMIN_EMAIL and APP_ADMIN_PASSWORD before creating the first administrator; do not rely on application defaults. Then install the frontend dependencies using the versions pinned by the repository:

~~~bash
nvm use
corepack enable
pnpm install --frozen-lockfile
~~~

Initialize the local database and create the administrator account:

~~~bash
php artisan migrate --seed --seeder=InitialSeeder
php artisan storage:link
php artisan cattr:make:admin
~~~

Run the backend and frontend development processes in separate terminals:

~~~bash
php artisan serve
pnpm watch
~~~

For workflows that need background processing, start the relevant Laravel worker processes in additional terminals, such as php artisan queue:work, php artisan reverb:start, or php artisan schedule:work.

## Production deployment :id=configuration-examples

Use the maintained container image and deployment instructions in [Getting started](/en/getting-started/) or [Kubernetes installation](/en/advanced/kube). The source-development commands above are intended for local development and do not configure a production web server.
