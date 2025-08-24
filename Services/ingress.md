## Ingress management

Most basic command:

```bash
# Creates a single host rule mapping "/" to a Service
kubectl create ingress web --class=nginx \
  --rule="app.example.com/=web-svc:80"
```

## Contents:

- CheatSheet
- Tips
- Examples
- Warnings
- Troubleshoot

## CheatSheet

```bash
# List & inspect
kubectl get ingress
kubectl describe ingress web
kubectl get ingressclass
kubectl get svc -n ingress-nginx   # find LB/NodePort of your controller

# Create (quick)
kubectl create ingress web --class=nginx \
  --rule="app.example.com/=web-svc:80"

# TLS add (secret must exist or be issued by cert-manager)
kubectl annotate ingress web cert-manager.io/cluster-issuer=letsencrypt-prod
kubectl edit ingress web
kubectl delete ingress web
```

> **Tip:**

In Kubernetes networking.k8s.io/v1, always set pathType (commonly Prefix) and prefer ingressClassName over legacy annotations.

## Examples:

1) Basic host & path routing (ClusterIP backend)

```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web
spec:
  ingressClassName: nginx
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-svc
            port:
              number: 80
```
