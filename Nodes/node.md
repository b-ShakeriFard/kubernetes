# Node management

Most basic command:
```bash
kubectl get nodes -o wide
```

Contents <br>
[Jump to:](#) <br> 
• [Cheatsheet](#cheatsheet) <br>
• [Tips](#) <br>
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
> 💡 **Tip:** 
Typical maintenance flow: cordon → drain → (do work) → uncordon.
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
