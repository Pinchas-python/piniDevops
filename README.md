# Helm From Scratch Assignment

This repository demonstrates a fully hand-authored Helm chart (`charts/myapp`) that deploys every required Kubernetes primitive (Deployment, Service, DaemonSet, CronJob, Job, ConfigMap, Secret) using lightweight Docker Hub images such as `busybox:1.36` or `hashicorp/http-echo:0.2.3`.

## Repository Structure & Resource Purpose

| Path | Purpose |
| ---- | ------- |
| `charts/myapp/templates/deployment.yaml` | Runs the main BusyBox/http-echo pod replicas and mounts ConfigMap/Secret data. |
| `charts/myapp/templates/service.yaml` | ClusterIP Service fronting the Deployment for stable networking. |
| `charts/myapp/templates/daemonset.yaml` | BusyBox daemon agent scheduled on every node to prove ConfigMap/Secret mounting throughout the cluster. |
| `charts/myapp/templates/cronjob.yaml` | Periodic maintenance task (default `*/5 * * * *`) for lifecycle demos. |
| `charts/myapp/templates/job.yaml` | One-off batch Job used to illustrate reruns and `backoffLimit`. |
| `charts/myapp/templates/configmap.yaml` | Injects non-sensitive settings (mode, log level) into workloads. |
| `charts/myapp/templates/secret.yaml` | Stores sensitive credentials consumed by Deployment/DaemonSet. |
| `outputs/*.txt` | Captured terminal output for install, upgrade, history, and rollback commands. |

## Prerequisites
- Kubernetes cluster (Minikube works great)
- kubectl configured to talk to the cluster
- Helm 3 installed on your path

## Usage

```sh
cd charts/myapp
helm lint .
helm upgrade --install myapp . \
  --namespace dev \
  --create-namespace
```

Swap in the BusyBox or HashiCorp http-echo image by overriding values:

```sh
helm upgrade --install myapp . \
  --namespace dev \
  --set image.repository=hashicorp/http-echo \
  --set image.tag=0.2.3 \
  --set deployment.containerPort=5678 \
  --set-json 'deployment.command=["/http-echo","-listen=:5678","-text=Hello from Helm"]'
```

Check resources:

```sh
kubectl get all -n dev
kubectl get configmap -n dev
kubectl get secret -n dev
```

## Verification Steps

1. Run `helm upgrade --install` as above and copy its full terminal output into `outputs/helm-install.txt`.
2. Capture the next upgrade (after changing `values.yaml`, e.g., `image.tag=1.37`) in `outputs/helm-upgrade.txt`.
3. Execute `helm history myapp -n dev` and `helm rollback myapp <revision> -n dev`, storing each output in `outputs/helm-history.txt` and `outputs/helm-rollback.txt` respectively.
4. Confirm ConfigMap/Secret data reach the pods with `kubectl exec` (optional) or by inspecting mounted files under `/config` and `/secret`.

Keeping these verification logs in version control satisfies the assignment requirement for a reproducible Helm workflow end to end.
