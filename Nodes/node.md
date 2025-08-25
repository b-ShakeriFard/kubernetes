# Node management

Most basic command:
```bash
kubectl get nodes -o wide
```

Contents <br>
[Jump to:](#) <br> 
• [Cheatsheet](#cheatsheet) <br>
• [Tips](#Tips) <br>
• [Examples](#minimal-pod-yaml) <br>
• [Warnings](#Warnings) <br>
• [Troubleshooting](#troubleshooting) <br>

# CheatSheet

### List & inspect 
| Command | explanation |
|---------|-------------|
|`kubectl get nodes`                            |# short view |
|`kubectl get nodes -o wide`                    |# OS, kernel, container runtime, IP |
|`kubectl describe node <node>`                 |# conditions, capacity/allocatable, taints |
|`kubectl top node`                             |# needs metrics-server |

### Schedule control 
| Command | explanation |
|---------|-------------|
| `kubectl cordon <node>`                       |# mark unschedulable (no new Pods) |
| `kubectl uncordon <node> `                    |# allow scheduling again |
| `kubectl drain <node> --ignore-daemonsets --delete-emptydir-data` | # evict Pods safely for maintenance |

### Labels (for selectors/affinity) 

| Command | explanation |
|---------|-------------|
| `kubectl label node <node> role=db`           | # add|
| `kubectl label node <node> role- `            | # remove key 'role' |
| Taints (for controlled placement) | - |
| `kubectl taint node <node> env=prod:NoSchedule` | - |
| `kubectl taint node <node> env-`               | # remove taint key 'env'|

### Quick views 
| Command | explanation |
|---------|-------------|
| `kubectl get nodes -L role`                   | # show label column |
| `kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints` | - |

# Tips

> [!TIP] 
> Typical maintenance flow: **cordon** → **drain** → **(do work)** → uncordon.
Pair labels (on nodes) with nodeSelector/affinity (in Pods) for intentional placement.

# Examples

- 1) Prepare a node for DB workloads
 
 ```bash
# Label the node and prevent general scheduling
kubectl label node worker-2 workload=db
kubectl taint node worker-2 db=true:NoSchedule
```
<br>
Pod spec (schedule only on that node and tolerate the taint):

<br>

```bash
spec:
  nodeSelector:
    workload: db
  tolerations:
  - key: "db"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
```

<br>

- 2) Safe node maintenance

```bash
kubectl cordon worker-1
kubectl drain worker-1 --ignore-daemonsets --delete-emptydir-data --grace-period=60 --timeout=10m
# ...patch/upgrade node...
kubectl uncordon worker-1
```

- 3) Quick condition/pressure checks

```bash
kubectl get node <node> -o jsonpath='{.status.conditions[*].type}{"\n"}{.status.conditions[*].status}{"\n"}'
kubectl describe node <node> | sed -n '/Conditions:/,$p' | head -n 40
```

- 4) Targeted scheduling (affinity, richer than nodeSelector)

```bash
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: workload
            operator: In
            values: ["api","web"]
```

<hr>

# Warnings

[!WARNING]
- kubectl drain can hang if PodDisruptionBudgets block evictions. Check kubectl get pdb -A.

- Draining control-plane nodes can disrupt the cluster; understand scheduling and static Pods before proceeding.

- Deleting a Node (kubectl delete node <node>) removes it from the API, not the machine. If re-provisioning, also reset kubelet/kubeadm on the host.

- Node objects are kubelet-managed; avoid editing fields other than labels and taints.

<hr>

# Troubleshoot

```bash
# 1) Health & pressure signals
kubectl describe node <node> | sed -n '/Conditions:/,$p'
# Look for: Ready, MemoryPressure, DiskPressure, PIDPressure, NetworkUnavailable

# 2) Capacity vs allocatable (scheduling headroom)
kubectl get node <node> -o jsonpath='{.status.capacity}{"\n"}{.status.allocatable}{"\n"}'

# 3) Pods stuck on node (why drain fails)
kubectl get pods -A --field-selector spec.nodeName=<node> -o wide
kubectl get pdb -A

# 4) Taints/labels sanity
kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints
kubectl get nodes -L role,workload

# 5) Quick metrics (needs metrics-server)
kubectl top node
```

[!TIP]
For deep host debugging, use ephemeral node debug:

```bash
kubectl debug node/<node> -it --image=busybox
# then (optional) chroot into host filesystem if your cluster supports it
```

