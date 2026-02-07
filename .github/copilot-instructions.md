# Helm From Scratch – AI Guide

## Architecture Snapshot
- Single Helm chart under [charts/myapp](charts/myapp) deploys every core Kubernetes primitive (Deployment, Service, ConfigMap, Secret, DaemonSet, CronJob, Job) to prove end-to-end workflows without extra microservices.
- Templating helpers in [charts/myapp/templates/_helpers.tpl](charts/myapp/templates/_helpers.tpl) centralize release naming, labels, and namespace overrides; always reuse these helpers for new manifests.
- All workloads consume the same ConfigMap/Secret pair ([configmap.yaml](charts/myapp/templates/configmap.yaml), [secret.yaml](charts/myapp/templates/secret.yaml)) via `envFrom` and projected volumes so any change to `values.config` or `values.secret` fans out automatically.

## Key Configuration Points
- Default image is BusyBox (`image.repository/tag` in [values.yaml](charts/myapp/values.yaml)); override to `hashicorp/http-echo` when you need an HTTP responder—remember to also update `deployment.containerPort` and possibly `deployment.command`.
- Workload toggles: `deployment.enabled`, `daemonset.enabled`, `cronjob.enabled`, `job.enabled` gate template rendering; disable unused components rather than deleting YAML.
- `namespaceOverride` lets you pin resources to a fixed namespace independent of `--namespace` during `helm upgrade --install`.
- `serviceAccount.create` is `false`; set it to `true` (and optionally `serviceAccount.name`) before referencing custom RBAC.

## Workflows & Commands
- Standard loop (from [README.md](README.md)): `cd charts/myapp && helm lint .` before every install/upgrade, then `helm upgrade --install myapp . --namespace dev --create-namespace`.
- Capture operational evidence for assignments in [outputs/*.txt](outputs) (install, upgrade, history, rollback). Keep these logs updated whenever behavior changes.
- Use overrides via `--set` / `--set-json` for quick experiments (example in README for swapping BusyBox with http-echo). Prefer `values.yaml` edits for long-lived defaults so they are code-reviewed.
- Post-install validation relies on `kubectl get all -n <ns>` plus targeted `kubectl get configmap/secret -n <ns>`; include `kubectl exec` checks when verifying mounted files under `/config` and `/secret`.

## Template Patterns to Preserve
- Each manifest pins both `metadata.namespace` and `spec.template.metadata.labels` using helper outputs; new templates must follow the same naming to keep `helm uninstall` predictable.
- Deployment/DaemonSet share common pod settings (volumes, labels, env) defined inline; when adding sidecars or probes, touch both templates or extract shared snippets into `_helpers.tpl`.
- CronJob/Job images and commands are fully data-driven under `.Values.cronjob.*` and `.Values.job.*`; avoid hardcoding logic in templates so QA can flip behavior through values only.
- Services only expose a single `http` port; if you add more ports, also update any consumers relying on `service.targetPort` defaults.

## Collaboration Notes
- Keep comments and strings ASCII; Kubernetes manifests here are intentionally minimal for clarity.
- When introducing new resources, document their purpose in [charts/myapp/README.md](charts/myapp/README.md) and mirror that summary in the root [README.md](README.md) so future learners understand why it exists.
- Ensure every change is reproducible on Minikube: test with `helm lint`, `helm template` (if you need a dry run), then record the resulting `helm upgrade` output in `outputs/`.
