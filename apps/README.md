# apps/

Each file in this directory is an ArgoCD `Application` manifest. The root Application (applied manually once,
see `../bootstrap/root-app.yaml`) watches this whole directory — drop a new Application manifest here, `git push`,
and ArgoCD deploys it. Delete one and ArgoCD prunes it (the root Application is configured with
`prune: true`).

Empty at the end of Week 2 — that's correct, nothing has been onboarded to GitOps yet beyond the bootstrap itself.

Planned additions, one per week, per the project plan:

- Week 3 — `kube-prometheus-stack.yaml`, `loki.yaml`
- Week 4 — `finance-app.yaml` (once the image exists somewhere ArgoCD/k3s can pull it from)
- Week 5 — `sealed-secrets.yaml`, `tailscale-operator.yaml`
- Week 6 — `argo-rollouts.yaml`
- Week 8 — `vaultwarden.yaml`

Each of these will be its own Application resource pointing at either a Helm chart (via `source.chart` +
`source.repoURL` set to the chart's Helm repo) or a path elsewhere in this repo (for the finance app's own
manifests, once it exists).
