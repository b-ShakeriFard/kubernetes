# Deployment management

Most basic command:
```bash
kubectl create deployment web --image=nginx
```

# Content

[Jump to:] CheatSheet (#CheatSheet) Tips (#Tips)

```bash
# Create / Inspect
kubectl create deployment web --image=nginx
kubectl get deploy
kubectl describe deploy web
kubectl get rs -l app=web                         # ReplicaSets created by the Deployment

# Scale
kubectl scale deploy/web --replicas=3
kubectl autoscale deploy/web --min=2 --max=5 --cpu-percent=70   # HPA

# Update image (triggers rolling update)
kubectl set image deploy/web app=nginx:1.25.5
kubectl rollout status deploy/web
kubectl rollout history deploy/web
kubectl rollout undo deploy/web [--to-revision=1]

# Pause / resume rollouts (edit multiple fields safely)
kubectl rollout pause deploy/web
kubectl set resources deploy/web -c app --limits=cpu=500m,memory=256Mi
kubectl rollout resume deploy/web

# YAML workflow
kubectl create deploy api --image=nginx --dry-run=client -o yaml > deploy.yaml
kubectl apply -f deploy.yaml
kubectl delete deploy web
```

## Tips
[!TIPS]
