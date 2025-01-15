# Elixir Application Setup on Kubernetes

This guide provides detailed steps to set up the `sre` application in your Kubernetes cluster.

---

## Prerequisites

Before proceeding, ensure you have the following:

- A running Kubernetes cluster (e.g., Minikube, EKS, GKE, etc.).
- `kubectl` installed and configured to interact with your cluster.
- Helm installed for managing package installations.

---

## Installation Steps

### 1. Install PostgreSQL

Use the Bitnami PostgreSQL Helm chart to deploy PostgreSQL in the `stord` namespace:

```bash
NAMESPACE=stord

helm install sre-db oci://registry-1.docker.io/bitnamicharts/postgresql \
  --create-namespace \
  --namespace $NAMESPACE \
  --set auth.database=sre-technical-challenge \
  --set auth.postgresPassword=password
```

### 2. Verify PostgreSQL Deployment

Check if the PostgreSQL pods are running:

```bash
kubectl get pods -n stord
```

Confirm that the service is available:

```bash
kubectl get svc -n stord
```

Check PostgreSQL logs for readiness:

```bash
kubectl logs -n stord sre-db-postgresql-0
```

Verify the PostgreSQL connection:

```bash
kubectl port-forward svc/sre-db-postgresql 55432:5432
psql postgresql://postgres:password@127.0.0.1:55432/sre-technical-challenge
```

---

## Troubleshooting

### 1. Verify Connectivity to PostgreSQL

Run a debug pod and test PostgreSQL connectivity:

```bash
kubectl run debug --rm -it --restart=Never --image=busybox -n stord -- sh
```

Inside the pod, use the following commands:

```sh
nslookup sre-db-postgresql
nc -zv sre-db-postgresql 5432
```

---

## Install `sre-app`

Navigate to the `sre` application folder and install the Helm chart:

```bash
cd <path-to-sre-app-folder>
helm install sre-app . \
  --debug \
  --namespace $NAMESPACE \
  --values values.yaml
```

### 1. Verify Application Status

Check if the application pods are running:

```bash
kubectl get pods -n stord
```

### 2. Verify Application Functionality

Port-forward the application service and test the endpoints:

```bash
kubectl port-forward svc/sre-app-sre-technical-challenge 8080:80 -n stord
curl http://localhost:8080
```

---

## Cleanup

To delete all resources created in the `stord` namespace:

```bash
kubectl delete namespace stord
```

---
