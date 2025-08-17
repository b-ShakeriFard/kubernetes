# Node management

Most basic command:
```bash
kubectl get nodes -o wide
```

Contents
[Jump to:](#) [Cheatsheet](#cheatsheet) <br>
• [Tips](#) <br>
• [Examples](#minimal-pod-yaml) <br>
• [Warnings](#Warnings)
• [Troubleshooting](#troubleshooting)

### CheatSheet

 List & inspect <br>
`kubectl get nodes`                            # short view
`kubectl get nodes -o wide`                    # OS, kernel, container runtime, IP
`kubectl describe node <node>`                 # conditions, capacity/allocatable, taints
`kubectl top node`                             # needs metrics-server

# Schedule control <br>
kubectl cordon <node>                        # mark unschedulable (no new Pods)
kubectl uncordon <node>                      # allow scheduling again
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data
                                             # evict Pods safely for maintenance
# Labels (for selectors/affinity)
kubectl label node <node> role=db            # add
kubectl label node <node> role-              # remove key 'role'

# Taints (for controlled placement)
kubectl taint node <node> env=prod:NoSchedule
kubectl taint node <node> env-               # remove taint key 'env'

# Quick views
kubectl get nodes -L role                    # show label column
kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints

