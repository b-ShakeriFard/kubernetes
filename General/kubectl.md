## Main Commands for kubernetes:

### 1. Inspect & view
- get → list resources (kubectl get pods)
- describe → detailed info (kubectl describe pod mypod)
- logs → container logs (kubectl logs mypod)
- top → resource usage (needs metrics-server)

### 2. Create & update
- create → new resource from CLI or file (kubectl create deploy)
- apply → apply config file declaratively
- replace → replace from file
- edit → edit live resource in $EDITOR
- set → update fields (e.g., set image, set resources)
- scale → adjust replicas
- annotate → add/update annotations
- label → add/update labels
- patch → apply partial changes

### 3. Delete & clean up
- delete → delete resources
- drain → safely evict pods from a node
- cordon / uncordon → mark node unschedulable / schedulable

### 4. Interact with pods/containers
- exec → run command inside container
- attach → attach to running process
- port-forward → forward local port to pod/service
- cp → copy files to/from containers
- proxy → run local proxy to API server
- run → start a pod (test/dev)
- debug → create ephemeral debug container (>=1.18)

### 5. Cluster & config
- cluster-info → info about cluster
- api-resources → list resource types
- api-versions → list API versions
- config → manage kubeconfig
- auth → check access/RBAC (kubectl auth can-i)
- version → client/server version
- explain → docs for resources (kubectl explain pod.spec.containers)

### 6. Rollout & workloads
- rollout → manage deploy/DS/SS rollouts
- status
- history
- undo
- autoscale → HPA (horizontal pod autoscaler)

## 🔹 Common global flags (work almost anywhere)

- `-n, --namespace <ns>` → target namespace
- `-o yaml|json|wide|name` → output formats
- `--kubeconfig <file>` → custom kubeconfig
- `--context <name>` → pick context
- `--dry-run=client|server` → preview only
- `--force` → force some operations (delete, replace)
- `--grace-period=0` → immediate termination
- `--timeout=<duration>` → set operation timeout
- `--field-selector` → filter by resource fields
- `-l, --selector` → filter by labels

 ## 🔹 Commands admins use often
- `kubectl get events --sort-by=.lastTimestamp`
- `kubectl explain <resource> --recursive`
- `kubectl auth can-i create pods -n kube-system`
- `kubectl config view --minify | grep namespace:`
- `kubectl rollout undo deploy/<name> `

## In Short

- Top-level verbs = get, create, apply, delete, edit, describe, logs, exec, port-forward, rollout, scale, set, patch, label, annotate, auth, config, explain, top, drain, cordon, uncordon.
- With subcommands for some (like rollout, set, config).
- And flags that can be applied almost anywhere.
