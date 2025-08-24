# Service management

Most basic command:
```bash
kubectl expose deployment web --port=80 --target-port=80 --name=web-svc
```

### Contents:

- CheatSheet (#CheatSheet)
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

> **Tip:**
Prefer named container ports and reference them in targetPort; this avoids breakage when you change port numbers.

### Examples

1) ClusterIP (default)

```bash
apiVersion: v1
kind: Service
metadata:
  name: web-svc
  labels: { app: web }
spec:
  selector: { app: web }    # must match Pod labels
  type: ClusterIP
  ports:
    - protocol: TCP
      port: 80                # service port (stable virtual IP)
      targetPort: 8080        # matches container port 

```

2) NodePort (direct node access)

```bash
apiVersion: v1
kind: Service
metadata: { name: web-nodeport }
spec:
  selector: { app: web }
  type: NodePort
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
      nodePort: 30080         # optional; else auto-assign from 30000-32767

```
3) LoadBalancer (cloud)

```bash
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
  clusterIP: 10.0.171.239
  type: LoadBalancer
status:
  loadBalancer:
    ingress:
    - ip: 192.0.2.127
```

4) Headless Service (for StatefulSets)

```bash
apiVersion: v1
kind: Service
metadata: { name: db }
spec:
  clusterIP: None               # headless
  selector: { app: postgres }
  ports:
  - name: pg
    port: 5432
    targetPort: 5432
```

> **Warnings:** 

1) Selectors are immutable for existing Services (practical rule): if you change matching labels, you may need to recreate the Service or update Pod labels.

2) No endpoints? No traffic. If a Service has 0 endpoints, check labels and pod readiness; only Ready Pods are added (unless publishNotReadyAddresses: true).

3) NodePort opens a port on every node; ensure firewalls allow it and mind the default range 30000–32767.

4) LoadBalancer requires a provider (cloud LB or MetalLB on bare metal). Without one, it behaves like a ClusterIP plus a pending status.

5) targetPort must match a containerPort (number or name) that actually exists in the Pod template.

## TroubleShoot
