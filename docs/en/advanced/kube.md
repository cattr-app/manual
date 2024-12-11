# Installation in a Kubernetes Cluster :id=kube :priority=8

To install Cattr in a Kubernetes cluster, you will need a Helm client installed. If you don't have it, you can install it using the [instructions on the official Helm website](https://helm.sh/docs/intro/install/).

Minimum cluster requirements:
- Ingress Controller (e.g., Nginx Ingress Controller)
- Persistent Volume support for data storage
- Secret support for storing secrets
- ConfigMap support for storing configuration

Minimum resource requirements:
- 2 vCPU
- 2 GB RAM
- 10 GB disk space

Operation has been tested on Kubernetes cluster version 1.30.5. The application may work on earlier versions, but this is not guaranteed.

## Step 1. Adding the Helm Repository

Repository address: https://git.amazingcat.net/api/v4/projects/469/packages/helm/stable

```bash
helm repo add cattr https://git.amazingcat.net/api/v4/projects/469/packages/helm/stable
helm repo update
```

## Step 2. Installing Cattr

```bash
helm install cattr cattr/cattr-server
```

If you want to change the default parameters (recommended, otherwise a startup error may occur), you need to create a `values.yaml` file and pass it to the installation command:

```bash
helm install cattr cattr/cattr-server -f values.yaml
```

Description of the `values.yaml` file:

```yaml
mysql:
  # Install the database within the cluster
  asChart: false

  # Database authentication
  auth:
    database: "cattr"
    username: "cattr"
    password: "password"
  
  # Other parameters for running the database can be specified here
  # More details can be found at https://github.com/bitnami/charts/blob/main/bitnami/mysql/README.md

app:
  env:
    # All values from https://github.com/cattr-app/server-application/blob/main/.env.example are supported
    # Database host
    DB_HOST: "master.mysql-sync.svc.cluster.local"
    # Database port
    DB_PORT: "3306"
    # Database name
    DB_DATABASE: "cattr"
    # Database username
    DB_USERNAME: "cattr"
    # Database user password
    DB_PASSWORD: "password"
  # Application key. It is recommended to leave this value empty.
  # Then the chart will generate a key automatically on the first installation.
  # More details can be found at https://laravel.com/docs/11.x/encryption#configuration
  key: ""
  # Number of application replicas to run
  replicas: 1
  # Number of stored application revisions
  revisionHistoryLimit: 2
  # Application environment
  environment: "production"
  persistence:
    screenshots:
      # Create PVC for storing screenshots
      enabled: "true"
      # Name of an existing PVC
      existingClaim: ""
      # StorageClass for PVC
      storageClass: ""
      # Access mode for PVC
      accessModes:
        - ReadWriteMany
      # PVC size
      size: 10Gi
    attachments:
      # Create PVC for storing files
      enabled: "true"
      # Name of an existing PVC
      existingClaim: ""
      # StorageClass for PVC
      storageClass: ""
      # Access mode for PVC
      accessModes:
        - ReadWriteMany
      # PVC size
      size: 10Gi
  service:
    # Type of service to create
    type: ClusterIP
    # IP for ClusterIP type service
    clusterIP: ""
    # IP for LoadBalancer type service
    loadBalancerIP: ""
    # External traffic policy
    externalTrafficPolicy: Cluster
    # Port for NodePort type
    nodePort: 80
    # Service port
    port: 80
  image:
    # Application image registry
    registry: registry.git.amazingcat.net
    # Application image name
    repository: cattr/core/app
    # Application image tag
    tag: v4.0.0-RC49
    # Image pull policy
    pullPolicy: IfNotPresent

ingress:
  # Enable Ingress
  enabled: true
  # Ingress host
  host: "cattr.ingress.cluster.local"
  # Ingress class
  class: "nginx"
```

## Mechanics

The Helm template has a dependency on MySQL. By default, the database is installed within the cluster. However, it is recommended to use an external MySQL server. To do this, set the `mysql.asChart` parameter to `false`.

When installing the application, Helm will create a Secret with the database connection details, as well as a ConfigMap to store other data.  A Deployment with the specified number of replicas, a Service to access the application, and an Ingress to access the Service will also be created.

The application exports some of its metrics in Prometheus format on port 80 at the path `/actuator/promehteus`, which can be used for monitoring by external systems.

## Update

The application supports updates via Helm. To do this, run the command:

```bash
helm upgrade cattr cattr/cattr-server -f values.yaml
```
