# Kubernetes app deployment

This project contains a Kubernetes deployment setup for a sample application stack composed of:

- a web application
- a MySQL database
- RabbitMQ
- Memcached
- an ingress entrypoint for external access
- a secret and persistent volume for database persistence

The manifests in this repository are meant to be applied to a Kubernetes cluster and provide a simple multi-service application environment.

## Project structure

- `appdeploy.yaml` — main application deployment
- `appservice.yaml` — service exposing the web application
- `dbdeploy.yaml` — MySQL deployment
- `dbservice.yaml` — service exposing the database
- `dbvolume.yaml` — persistent volume claim for MySQL data
- `rmqdeploy.yaml` — RabbitMQ deployment
- `rmqservice.yaml` — service exposing RabbitMQ
- `mcdeploy.yaml` — Memcached deployment
- `mcservice.yaml` — service exposing Memcached
- `appsecret.yaml` — Kubernetes Secret storing application credentials
- `ingress.yaml` — ingress routing traffic to the app service

## Architecture overview

The application stack is designed around a typical layered setup:

- The frontend/web application is deployed as a pod and exposed through a Kubernetes Service.
- MySQL stores application data in a PVC-backed volume.
- RabbitMQ is used as the messaging service.
- Memcached is used as an in-memory cache.
- The application waits for database, RabbitMQ, and Memcached dependencies to become available before it starts fully.
- The Ingress routes incoming traffic to the app service.

## Components

### 1. Application deployment

The app deployment uses the image:

- `vprocontainers/vprofileapp`

It exposes port `8080` and includes init containers that wait for the dependencies to be reachable:

- MySQL on `vprodb:3306`
- RabbitMQ on `vpromq01:5672`
- Memcached on `vprocache01:11211`

### 2. Database deployment

The database deployment uses:

- `vprocontainers/vprofiledb`

It mounts the database storage on `/var/lib/mysql` and uses a secret value for the MySQL root password:

- secret: `app-secret`
- key: `db-password`

The database service is named `vprodb` and exposes port `3306`.

### 3. RabbitMQ deployment

The RabbitMQ deployment uses the official `rabbitmq` image.

It exposes port `5672` and uses secret values for credentials:

- `RABBITMQ_DEFAULT_PASS` from `app-secret` key `rmq-password`
- `RABBITMQ_DEFAULT_USER` set to `guest`

The service is named `vpromq01`.

### 4. Memcached deployment

The Memcached deployment is based on the `memcached` image and exposes port `11211` through the service `vprocache01`.

### 5. Secret

The secret manifest stores sensitive values in Base64-encoded form:

- `db-password`
- `rmq-password`

This is a sample configuration and should be replaced with proper secret management in production environments.

### 6. Persistent storage

The MySQL database uses a PersistentVolumeClaim named `db-persistent-volume-claim` with:

- storage: `2Gi`
- access mode: `ReadWriteOnce`
- storage class: `standard`

### 7. Ingress

The ingress config routes host `kubernetes.docker.internal` to the app service on port `8080`. You can change the hostname to your cluster hostname.

## Prerequisites

Before deploying, ensure that:

- a Kubernetes cluster is available
- `kubectl` is installed and configured
- an ingress controller such as NGINX is installed if you want the ingress resource to work properly
- the target namespace is created if you use one

## Deployment steps

Apply the manifests in the following order:

```bash
kubectl apply -f appsecret.yaml
kubectl apply -f dbvolume.yaml
kubectl apply -f dbdeploy.yaml
kubectl apply -f dbservice.yaml
kubectl apply -f rmqdeploy.yaml
kubectl apply -f rmqservice.yaml
kubectl apply -f mcdeploy.yaml
kubectl apply -f mcservice.yaml
kubectl apply -f appdeploy.yaml
kubectl apply -f appservice.yaml
kubectl apply -f ingress.yaml
```

Or:
```bash
kubectl apply -f .
```

## Verify the deployment

Check that the pods are running:

```bash
kubectl get pods
```

Check the services:

```bash
kubectl get svc
```

Check the ingress:

```bash
kubectl get ingress
```

Inspect logs if needed:

```bash
kubectl logs <pod-name>
```

## Accessing the application

Once the ingress is active, the app can typically be accessed through:

```text
http://kubernetes.docker.internal/
```

If your cluster uses a different domain or local DNS setup, replace the host value in `ingress.yaml` accordingly.

## Notes

- The manifests are a sample Kubernetes setup and may require adjustments depending on your environment.
- Secrets are stored in manifest files for demonstration purposes; in production, prefer external secret managers or encrypted secret storage.
- Some resources may require cluster-specific configuration such as storage classes, ingress class names, or networking setup.

## Cleanup

To remove the deployed resources:

```bash
kubectl delete -f ingress.yaml
kubectl delete -f appservice.yaml
kubectl delete -f appdeploy.yaml
kubectl delete -f mcservice.yaml
kubectl delete -f mcdeploy.yaml
kubectl delete -f rmqservice.yaml
kubectl delete -f rmqdeploy.yaml
kubectl delete -f dbservice.yaml
kubectl delete -f dbdeploy.yaml
kubectl delete -f dbvolume.yaml
kubectl delete -f appsecret.yaml
```

## Author

BlackMagic-Master
Szymon G.