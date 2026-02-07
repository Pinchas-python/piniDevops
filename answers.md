# Answers

1. **Image choice**: Defaulted to `busybox:1.36` because it is ~1 MB, starts instantly, and supports `sh`, `echo`, and `sleep`, which keeps every resource lightweight. Students can override to `busybox:1.37`, `busybox:1.38`, or `hashicorp/http-echo` to practice upgrades.
2. **Config and secrets**: Plain key/value pairs are provided through a ConfigMap (`APP_MODE`, `LOG_LEVEL`) and Secret (`username`, `password`) rendered from `values.yaml`. The Deployment and DaemonSet both import them via `envFrom` and mount them as read-only volumes at `/config` and `/secret`, which proves out Part 5’s requirement.
3. **Helm focus**: Every manifest was written manually—no `helm create` scaffolding—so students can inspect how templates, helpers, and values interact while still leveraging Helm release tooling (`helm upgrade --install`, `helm history`, `helm rollback`).
4. **External chart comparison**: Adding the Bitnami repo (`helm repo add bitnami https://charts.bitnami.com/bitnami`) and installing `bitnami/nginx` shows how vendor charts expose far more values (ingress, autoscaling, TLS) compared with our minimalist chart. Overriding `--set image.tag=1.23.0 --set service.type=NodePort` keeps behavior comparable for local dev, but the Bitnami release also provisions extra objects (ConfigMaps, ServiceAccounts) automatically, which highlights the difference between curated upstream charts and handcrafted ones.
5. **Helm lifecycle**: The steady-state loop is `helm lint` (template validation) → `helm upgrade --install` (idempotent apply) → `kubectl get ...` (runtime verification) → `helm history` (audit revisions) → `helm rollback <rev>` (recover previous config) → `helm uninstall` when finished. Recording each command’s output in `outputs/*.txt` guarantees traceability.
6. **Why `helm upgrade --install`**: It first checks if the release exists; if not, it runs an install, otherwise it upgrades in place. That single command pattern avoids branch logic in scripts, works well in CI/CD pipelines, and ensures the chart can be re-applied safely with new values at any time.
7. **Template breakdown**:
	- `deployment.yaml`: Main stateless workload with replica count, BusyBox/http-echo container, envFrom wiring, and ConfigMap/Secret volume mounts.
	- `service.yaml`: Stable ClusterIP endpoint for the deployment pods; also supports NodePort via values.
	- `daemonset.yaml`: Per-node BusyBox agent that reuses the same configuration data, proving secrets/configs mount everywhere.
	- `cronjob.yaml`: Schedules a shell script every 5 minutes to show recurring automation.
	- `job.yaml`: One-off batch container for ad-hoc tasks; uses `backoffLimit` and optional TTL cleanup.
	- `configmap.yaml`: Emits plaintext configuration from values for the app to read.
	- `secret.yaml`: Emits sensitive key/value pairs (stringData) consumed by the workloads.
