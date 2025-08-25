# Deployment management

Most basic command:
```bash
kubectl create deployment web --image=nginx
```

# CheatSheet

```bash
kubectl get deploy
kubectl describe deploy web
kubectl scale deploy web --replicas=3
kubectl set image deploy web app=nginx:1.25.5
kubectl rollout status deploy web
kubectl rollout undo deploy web
```

[!TIP]
Use kubectl create deploy ... --dry-run=client -o yaml to generate YAML manifests quickly.

<hr>

## Example

```bash
kubectl create deployment api --image=nginx --replicas=2
kubectl expose deployment api --port=80 --target-port=80
```
[!WARNING]
If spec.selector.matchLabels doesn’t match the Pod template’s labels, your Deployment won’t manage any Pods.

## Troubleshoot

```bash
kubectl describe deploy web                # check Events
kubectl get rs                             # verify ReplicaSets
kubectl describe rs <name>                 # see why Pods aren’t created
kubectl rollout history deploy web         # check rollout versions
kubectl get events --sort-by=.lastTimestamp
```
