## 'kubectl' CLI Structure

```bash
kubectl [command] [TYPE] [NAME] [flags]
```

[command] → verbs like get, describe, create …
[TYPE] → resource type (pods, deployments, svc, cm, secrets …).
[NAME] → specific resource name.
[flags] → global flags (-n, -o yaml, etc.) or command-specific flags (--replicas=3)


## 📄 Get Information <br>
```bash
kubectl get – List resources (pods, services, deployments, etc.)
<br>
kubectl describe – Detailed info about a resource

kubectl logs – View logs of a pod

kubectl top – Show resource usage (CPU, memory)
```

## 🚀 Create & Modify Resources <br>
```bash
kubectl create | Create a resource from command or file

kubectl apply | Apply changes from a manifest (preferred for GitOps)

kubectl edit – Edit a running resource in-place

kubectl delete – Delete resources
```

# 🔧 Debug & Interact
```bash
kubectl exec -it <pod> -- <cmd> – Run a command in a container

kubectl port-forward – Forward local port to pod/service (as discussed)

kubectl cp – Copy files between your machine and a pod

kubectl attach – Attach to a running container (like docker attach)

kubectl proxy – Start a local proxy to access the Kubernetes API
```

# 📦 Deployment & Exposure
```bash
kubectl run – Start a pod (often replaced by apply)

kubectl expose – Expose a resource as a new service

kubectl rollout – Manage deployment rollouts (status, undo, etc.)
```

# 🧠 Cluster-Level Operations
```bash
kubectl config – View or set kubeconfig settings

kubectl version – Show client/server version

kubectl cluster-info – Show cluster endpoints

kubectl api-resources – List all available resource types

kubectl auth – Check access rules (e.g., can-i)
```

## Main Commands for kubernetes:

1. Inspect & view

get → list resources (kubectl get pods)

describe → detailed info (kubectl describe pod mypod)

logs → container logs (kubectl logs mypod)

top → resource usage (needs metrics-server)

2. Create & update

create → new resource from CLI or file (kubectl create deploy)

apply → apply config file declaratively

replace → replace from file

edit → edit live resource in $EDITOR

set → update fields (e.g., set image, set resources)

scale → adjust replicas

annotate → add/update annotations

label → add/update labels

patch → apply partial changes

3. Delete & clean up

delete → delete resources

drain → safely evict pods from a node

cordon / uncordon → mark node unschedulable / schedulable

4. Interact with pods/containers

exec → run command inside container

attach → attach to running process

port-forward → forward local port to pod/service

cp → copy files to/from containers

proxy → run local proxy to API server

run → start a pod (test/dev)

debug → create ephemeral debug container (>=1.18)

5. Cluster & config

cluster-info → info about cluster

api-resources → list resource types

api-versions → list API versions

config → manage kubeconfig

auth → check access/RBAC (kubectl auth can-i)

version → client/server version

explain → docs for resources (kubectl explain pod.spec.containers)

6. Rollout & workloads

rollout → manage deploy/DS/SS rollouts

status

history

undo

autoscale → HPA (horizontal pod autoscaler)
