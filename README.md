# notes-k8s-platform

[notes-api](../notes-api) split into services and deployed to Kubernetes — first as plain **manifests** (Deployments, Services, Ingress, ConfigMap, Secret, StatefulSet + PVC), then packaged as a **Helm chart** with per-environment values.

```
Ingress (nginx, TLS) ──► frontend (nginx static)  ──► api (Flask)  ──► postgres (StatefulSet, PVC)
                                                            └────────► redis
```

## Layout
| Path | Purpose |
|---|---|
| `manifests/00-namespace.yaml` | `notes` namespace |
| `manifests/10-config.yaml` | ConfigMap (non-secret settings) and Secret (DB password) |
| `manifests/20-postgres.yaml` | StatefulSet + headless Service + PVC |
| `manifests/30-redis.yaml` | Deployment + ClusterIP Service |
| `manifests/40-api.yaml` | Deployment (2 replicas, probes, resources) + Service |
| `manifests/50-frontend.yaml` | Deployment + Service |
| `manifests/60-ingress.yaml` | one host, `/` → frontend, `/api` → api |
| `charts/notes/` | the same, as a Helm chart with values per environment |

## Apply
```bash
kubectl apply -f manifests/
kubectl -n notes get pods,svc,ingress
kubectl -n notes logs deploy/api -f
kubectl -n notes exec -it deploy/api -- curl -s localhost:8000/healthz
```
## Helm
```bash
helm install notes charts/notes -n notes --create-namespace -f charts/notes/values-dev.yaml
helm upgrade notes charts/notes -n notes --set api.image.tag=1.1.0
helm rollback notes 1 -n notes
```

## Useful kubectl
`get -o wide`, `describe`, `logs -f --previous`, `exec -it`, `port-forward svc/api 8000:80`,
`rollout status|history|undo`, `top pods`, `explain deployment.spec.strategy`.

