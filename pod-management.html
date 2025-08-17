<h2> Pod Management </h2>

<h4> Create a Pod </h4>

# kubectl run <pod-name> --image=nginx --restart=Never
# Pod management

> Manage individual Pods for dev/test scenarios. For production, prefer Deployments/StatefulSets.

[Jump to:](#) [Cheatsheet](#cheatsheet) • [Examples](#kubectl-run-examples) • [YAML](#minimal-pod-yaml) • [Troubleshooting](#troubleshooting)

> [!TIP]
> Use `--dry-run=client -o yaml` to learn YAML from a one-liner.

## Cheatsheet
| Task | Command |
|---|---|
| List pods | `kubectl get pods` |
| Describe one | `kubectl describe pod <name>` |
| Watch | `kubectl get pods -w` |
| Shell into container | `kubectl exec -it <pod> -- sh` |
| Logs (follow) | `kubectl logs -f <pod> [-c <container>]` |
| Port-forward | `kubectl port-forward pod/<pod> 8080:80` |
| Delete (graceful) | `kubectl delete pod <pod>` |
| Generate YAML | `kubectl run web --image=nginx --dry-run=client -o yaml` |

## `kubectl run` examples
```bash
# Pod, not Deployment
kubectl run web --image=nginx --restart=Never

# Command + args
kubectl run jobby --image=busybox --restart=Never --command -- sh -c 'echo hi && sleep 10'

# With port + env + resources + labels
kubectl run api --image=nginx --restart=Never \
  --port=80 \
  --env=MODE=dev --env=API_URL=https://example.com \
  --requests=cpu=200m,memory=128Mi \
  --limits=cpu=500m,memory=256Mi \
  --labels=app=api,env=dev

