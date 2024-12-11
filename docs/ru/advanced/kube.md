# Установка в кластер Kubernetes :id=kube :priority=8

Для установки Cattr в кластер Kubernetes вам потребуется установленный Helm клиент. Если у вас его нет, вы можете 
установить его с помощью [инструкций на официальном сайте Helm](https://helm.sh/docs/intro/install/).

Минимальные требования к кластеру:
- Наличие Ingress Controller (например, Nginx Ingress Controller)
- Поддержка Persistent Volume для хранения данных
- Поддержка Secret для хранения секретов
- Поддержка ConfigMap для хранения конфигурации

Минимальные требования к свободным ресурсам:
- 2 vCPU
- 2 Гб оперативной памяти
- 10 Гб дискового пространства

Работа была протестирована на кластере Kubernetes версии 1.30.5. Приложение может работать на более ранних версиях, 
однако это не гарантируется.

## Шаг 1. Добавление Helm репозитория

Адрес репозитория: https://git.amazingcat.net/api/v4/projects/469/packages/helm/stable

```bash
helm repo add cattr https://git.amazingcat.net/api/v4/projects/469/packages/helm/stable
helm repo update
```

## Шаг 2. Установка Cattr

```bash
helm install cattr cattr/cattr-server
```

Если вы хотите изменить параметры по умолчанию (рекомендуется, в противном случае возможна ошибка запуска), то 
необходимо создать файл `values.yaml` и передать его в команду установки:

```bash
helm install cattr cattr/cattr-server -f values.yaml
```

Описание файла `values.yaml`:

```yaml
mysql:
  # Установить базу данных в рамках кластера
  asChart: false

  # Авторизация в базе данных
  auth:
    database: "cattr"
    username: "cattr"
    password: "password"
  
  # Тут могут быть другие параметры для запуска базы данных 
  # Подробнее можно прочитать на https://github.com/bitnami/charts/blob/main/bitnami/mysql/README.md

app:
  env:
    # Поддерживаются все значения из https://github.com/cattr-app/server-application/blob/main/.env.example
    # Хост базы данных
    DB_HOST: "master.mysql-sync.svc.cluster.local"
    # Порт базы данных
    DB_PORT: "3306"
    # Имя базы данных
    DB_DATABASE: "cattr"
    # Имя пользователя базы данных
    DB_USERNAME: "cattr"
    # Пароль пользователя базы данных
    DB_PASSWORD: "password"
  # Ключ приложения. Рекомендуется оставить пустым значением. 
  # Тогда чарт при первой установке сгенерирует ключ самостоятельно. 
  # Подробнее можно прочитать на https://laravel.com/docs/11.x/encryption#configuration
  key: ""
  # Количество запускаемых реплик приложения
  replicas: 1
  # Количество хранимых ревизий приложения
  revisionHistoryLimit: 2
  # Среда выполнения приложения
  environment: "production"
  persistence:
    screenshots:
      # Создание PVC для хранения скриншотов
      enabled: "true"
      # Имя существующего PVC
      existingClaim: ""
      # StorageClass для PVC
      storageClass: ""
      # Режим доступа к PVC
      accessModes:
        - ReadWriteMany
      # Размер PVC
      size: 10Gi
    attachments:
      # Создание PVC для хранения файлов
      enabled: "true"
      # Имя существующего PVC
      existingClaim: ""
      # StorageClass для PVC
      storageClass: ""
      # Режим доступа к PVC
      accessModes:
        - ReadWriteMany
      # Размер PVC
      size: 10Gi
  service:
    # Тип создаваемого сервиса
    type: ClusterIP
    # IP Для сервиса типа ClusterIP
    clusterIP: ""
    # IP Для сервиса типа LoadBalancer
    loadBalancerIP: ""
    # Политика внешнего трафика
    externalTrafficPolicy: Cluster
    # Порт для типа NodePort
    nodePort: 80
    # Порт сервиса
    port: 80
  image:
    # Репозиторий с образом приложения
    registry: registry.git.amazingcat.net
    # Название образа приложения
    repository: cattr/core/app
    # Тег образа приложения
    tag: v4.0.0-RC49
    # Политика обновления образа
    pullPolicy: IfNotPresent

ingress:
  # Включение Ingress
  enabled: true
  # Хост Ingress
  host: "cattr.ingress.cluster.local"
  # Класс Ingress
  class: "nginx"
```

## Механика работы

Helm шаблон имеет зависимость от MySQL. По умолчанию, база данных устанавливается в рамках кластера. Однако 
рекомендуется использовать внешний MySQL сервер. Для этого необходимо установить параметр `mysql.asChart` в значение 
`false`.

При установке приложения, Helm создаст Secret с данными для подключения к базе данных, а также ConfigMap для 
хранения остальных данных.
Также будет создан Deployment с указанным количеством реплик, Service для доступа к приложению и Ingress для доступа к Service.

Приложение экспортирует некоторые свои метрики в формате prometheus на порт 80 по пути `/actuator/promehteus`, что 
можно использовать для мониторинга внешними системами.

## Обновление

Приложение поддерживает обновление через Helm. Для этого необходимо выполнить команду:

```bash
helm upgrade cattr cattr/cattr-server -f values.yaml
```
