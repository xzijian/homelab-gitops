# monitoring/dashboards/

Every file here is a plain Kubernetes `ConfigMap` manifest, applied directly by the
`grafana-dashboards` ArgoCD Application (`../../apps/grafana-dashboards.yaml`) -- no Helm chart, no
templating, just YAML files in a directory. Grafana's sidecar (enabled in
`apps/kube-prometheus-stack.yaml`'s values, under `grafana.sidecar.dashboards`) watches the cluster
for ConfigMaps labeled `grafana_dashboard: "1"` and loads whatever JSON it finds inside, no Grafana
restart required -- it polls on its own, usually picking a new one up within about a minute.

## Why this exists

Without this, a dashboard imported or built in the Grafana UI lives only in Grafana's own PVC.
That PVC is fine day-to-day, but it isn't reproducible the way everything else in this repo is --
it wouldn't survive a deliberate `terraform destroy` + rebuild, and it's not something a portfolio
reviewer looking at the git history would ever see. Committing the dashboard JSON here closes that
gap: `git push` becomes the actual source of truth for dashboards too, same rule as every other
workload in this repo.

## Workflow

1. **Community dashboard:** in Grafana, Dashboards -> New -> Import, paste the numeric ID from
   grafana.com/grafana/dashboards (or upload its JSON directly), pick the `Prometheus` datasource
   already configured, Import. The project plan calls for 1-2 of these -- a Node Exporter host
   dashboard (grafana.com dashboard ID `1860`, "Node Exporter Full") is a solid first pick, since
   node-exporter is already running as part of kube-prometheus-stack.

   **Custom dashboard:** build it from scratch in the Grafana UI instead of importing an ID --
   this is the hands-on part the project plan calls out separately, worth doing deliberately
   rather than skipping to another import.

2. Either way, once it looks right: dashboard settings (gear icon, top right) -> **JSON Model** ->
   copy the entire JSON block.

3. Copy `_template.yaml.example` in this directory to a real filename (drop `.example`, e.g.
   `node-exporter-full.yaml`), and paste the JSON in as the value of the `<name>.json` key,
   keeping it indented under `data:` (a `|` block scalar -- indentation is what marks it as one
   literal string, so keep every line of the pasted JSON indented at least as far as the `{` on
   the first line).

4. `git add`, commit, `git push`. `grafana-dashboards` picks up the new ConfigMap on its next
   sync (or trigger a manual refresh in the ArgoCD UI), and the Grafana sidecar loads it shortly
   after.

## Verifying it worked

- `kubectl get configmap -n monitoring -l grafana_dashboard=1` should list the ConfigMap
- The dashboard should appear in Grafana's Dashboards list without you touching the UI again
- `kubectl logs -n monitoring -l app.kubernetes.io/name=grafana -c grafana-sc-dashboard` (the
  sidecar container) is the first place to look if a dashboard doesn't show up -- same "check the
  sidecar/controller logs, not just the UI toast" habit as the `argocd-repo-server` lesson from
  Week 2
