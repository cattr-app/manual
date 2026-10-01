# Установка в Kubernetes :id=kube :priority=8

Helm-чарт серверной части находится в каталоге .helm репозитория GitHub. Для установки нужны Helm 3, кластер Kubernetes, StorageClass или заранее созданные PersistentVolumeClaim, а для публичного доступа — Ingress Controller.

## Получение чарта

Клонируйте текущий репозиторий сервера:

~~~bash
git clone https://github.com/cattr-app/server-application.git
cd server-application
~~~

Создайте файл values.yaml. Явно укажите образ из GHCR: в чарте по умолчанию пока задан прежний registry. Перед установкой замените примеры паролей и не добавляйте файл в Git.

~~~yaml
image:
  registry: ghcr.io
  repository: cattr-app/server
  tag: latest # Для production лучше указать тег релиза.

cattr:
  appUrl: https://cattr.example.com
  appKey: "" # Если значение пустое, чарт сгенерирует ключ.

database:
  host: "" # Пустое значение использует MySQL, установленный чартом.
  name: cattr
  username: cattr
  password: REPLACE_WITH_A_LONG_RANDOM_PASSWORD
  rootPassword: REPLACE_WITH_ANOTHER_RANDOM_PASSWORD

mysql:
  enabled: true

ingress:
  enabled: true
  className: nginx
  hostname: cattr.example.com
  tls:
    - secretName: cattr-tls
      hosts:
        - cattr.example.com
~~~

Создайте TLS-secret cattr-tls в namespace cattr или укажите в values.yaml имя и домен, которые используете. Чтобы открыть приложение без Ingress, задайте ingress.enabled: false и настройте Service подходящего типа или собственный gateway.

## Установка и обновление

Установите или обновите релиз:

~~~bash
helm upgrade --install cattr ./.helm \
  --namespace cattr \
  --create-namespace \
  --values values.yaml
~~~

Чтобы обновить чарт, получите последние изменения репозитория и повторите команду:

~~~bash
git pull
helm upgrade --install cattr ./.helm \
  --namespace cattr \
  --values values.yaml
~~~

По умолчанию чарт устанавливает MySQL. Для внешнего сервера MySQL задайте mysql.enabled: false и укажите database.host, database.port, database.name, database.username, database.password и database.rootPassword в соответствии с конфигурацией чарта.

Kubernetes Service принимает подключения на порту 80 и передает их контейнеру приложения на порт 8080. Метрики Prometheus доступны по пути /actuator/prometheus на Service.

Текущие сведения о релизах публикуются в [репозитории сервера](https://github.com/cattr-app/server-application) и [пакете образа](https://github.com/cattr-app/server-application/pkgs/container/server).
