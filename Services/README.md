# Service management

Most basic command:
```bash
kubectl expose deployment web --port=80 --target-port=80 --name=web-svc
```

### Contents:

- CheatSheet
- Tips
- Examples
- Warnings
- Troubleshoot

### CheatSheet

List & inspect

```bash
kubectl get svc
kubectl get svc -o wide
kubectl describe svc web-svc
kubectl get endpoints web-svc
kubectl get endpointslice -l kubernetes.io/service-name=web-svc

# Create quickly from an existing workload
kubectl expose deployment web --port=80 --target-port=8080 --name=web-svc

# Port-forward to a Service (handy for local testing)
kubectl port-forward svc/web-svc 8080:80

# Edit / delete
kubectl edit svc web-svc
kubectl delete svc web-svc
```

{!Tip}
