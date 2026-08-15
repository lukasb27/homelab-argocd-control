# homelab-argocd-control

App-of-apps root for **core platform** infra — long-lived, human-reviewed
infrastructure like the CNPG operator, the Postgres cluster backing
Backstage's own database, and Backstage itself. Not application/service
environments — see
[application-argocd-control](https://github.com/lukasb27/application-argocd-control)
for the high-churn, machine-driven, per-service Applications instead. See
[the ADR](docs/two-argocd-repos-adr.md) for why these are two separate repos,
not one.

## Bootstrap

One-time only, since ArgoCD has no way to discover a new repo except being
told about it directly:

```
kubectl apply -f main.yaml
```

`main.yaml` lives at the repo root, deliberately outside `apps/` — the root
Application it defines watches `apps/`, so if its own manifest lived inside
that folder it would try to manage (and endlessly re-sync) itself.

After that, never `kubectl apply` anything else in `apps/` by hand — push to
this repo and the root Application (automated sync, prune, selfHeal) picks up
changes on its own.

## Manual prerequisites (not managed by ArgoCD)

These exist as raw cluster state, not tracked in git. They need to be
recreated by hand if the cluster (or these namespaces) is ever rebuilt.

- **`backstage-tls` Secret** (namespace `backstage`) — a `mkcert`
  development certificate for `backstage.example`, referenced by the Ingress
  in `backstage-app`'s `backstage-values.yaml` (`extraDeploy`). Deliberately
  kept out of git: this repo is public, and the cert's private key shouldn't
  be. Recreate with:

  ```
  mkcert -install
  mkcert backstage.example
  kubectl create secret tls backstage-tls \
    --cert=backstage.example.pem \
    --key=backstage.example-key.pem \
    -n backstage
  ```
