# kubectl — Master Cheatsheet

> Command syntax: `kubectl [command] [TYPE] [NAME] [flags]`
>
> - **command**: verb (get, describe, apply, delete…)
> - **TYPE**: resource (pods, deploy, svc, cm, secret…)
> - **NAME**: specific resource name
> - **flags**: global or command-specific options

## Contents
- [Quick start](#quick-start)
- [Visual map of commands](#visual-map-of-commands)
- [Common patterns](#common-patterns)
- [Core verbs (by task)](#core-verbs-by-task)
- [Global flags](#global-flags)
- [Output & filtering](#output--filtering)
- [Resource type shortnames](#resource-type-shortnames)
- [Troubleshooting flow](#troubleshooting-flow)

---

## Quick start
```bash
kubectl get pods -A
kubectl describe pod <name> -n <ns>
kubectl logs -f <pod> [-c <container>]
kubectl exec -it <pod> -- sh
kubectl apply -f manifest.yaml
kubectl delete -f manifest.yaml
```

```ermaid
flowchart LR
  A[Inspect] -->|list| G[get]
  A -->|details| D[describe]
  A -->|metrics| T[top]
  A -->|events| E(events)

  B[Change] -->|create| C(create)
  B -->|apply| AP(apply)
  B -->|patch| P(patch)
  B -->|edit| ED(edit)
  B -->|delete| DEL(delete)
  B -->|label/annotate| LA(label/annotate)
  B -->|set image/resources| S(set)

  R[Rollouts] --> RS(rollout status)
  R --> RH(rollout history)
  R --> RU(rollout undo)
  R --> SC(scale)
  R --> AU(autoscale)

  W[Workload I/O] --> L(logs)
  W --> X(exec)
  W --> PF(port-forward)
  W --> AT(attach)
  W --> CP(cp)
  W --> RUN(run)
  W --> DBG(debug)

  K[Cluster/Config] --> CI(cluster-info)
  K --> AR(api-resources)
  K --> AV(api-versions)
  K --> EX(explain)
  K --> AUTH(auth can-i)
  K --> CFG(config)
  K --> VER(version)
  K --> DR(drain)
  K --> CO(cordon/uncordon)
```
## Common patterns
```bash
# Inspect everything in a namespace
kubectl get all -n <ns>

# Generate YAML without creating (great for learning)
kubectl create deploy web --image=nginx --dry-run=client -o yaml

# Apply a directory of manifests
kubectl apply -f k8s/

# Label select
kubectl get pods -l app=web -o wide

# Wait for readiness
kubectl wait --for=condition=Ready pod -l app=web --timeout=60s

# Delete by label
kubectl delete pod -l app=web
```

## Core Tasks
```bash
kubectl get <type> [name] [-A] [-o wide|yaml|json]
kubectl describe <type> <name>
kubectl logs [-f] <pod> [-c <container>]
kubectl top pod|node
kubectl get events --sort-by=.lastTimestamp
```

### Create & update
```bash
kubectl create <type> ...
kubectl apply -f <file|dir>
kubectl replace -f <file> [--force]
kubectl edit <type>/<name>
kubectl patch <type>/<name> --type=json -p='[{"op":"replace","path":"/spec/replicas","value":3}]'
kubectl set image deploy/web app=nginx:1.25.5
kubectl set resources deploy/web --limits=cpu=500m,memory=256Mi
kubectl label ns dev tier=backend --overwrite
kubectl annotate pod/web owner=behroox
```

### Delete & node operations
```bash
kubectl delete <type>/<name>            # or -l selector
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data
kubectl cordon <node> && kubectl uncordon <node>
```

### Interact with Pods/containers
```bash
kubectl exec -it <pod> [-c <container>] -- sh
kubectl attach -it <pod> [-c <container>]
kubectl port-forward pod/<pod> 8080:80
kubectl cp <ns>/<pod>:<path> <local>   # and reverse
kubectl run tmp --image=busybox --restart=Never -- sleep 3600
kubectl debug <pod> -it --image=busybox    # ephemeral debug container
```

### Rollout & scaling
```bash
kubectl rollout status deploy/web
kubectl rollout history deploy/web
kubectl rollout undo deploy/web [--to-revision=N]
kubectl scale deploy/web --replicas=4
kubectl autoscale deploy/web --min=2 --max=5 --cpu-percent=70
```

### Cluster & config
```bash
kubectl cluster-info
kubectl api-resources
kubectl api-versions
kubectl explain deployment.spec.template.spec.containers --recursive
kubectl auth can-i create pods -n kube-system
kubectl config get-contexts
kubectl config use-context <name>
kubectl version --short
```
