✅ Commonly Used kubectl Commands
📄 Get Information
kubectl get – List resources (pods, services, deployments, etc.)

kubectl describe – Detailed info about a resource

kubectl logs – View logs of a pod

kubectl top – Show resource usage (CPU, memory)

🚀 Create & Modify Resources
kubectl create – Create a resource from command or file

kubectl apply – Apply changes from a manifest (preferred for GitOps)

kubectl edit – Edit a running resource in-place

kubectl delete – Delete resources

🔧 Debug & Interact
kubectl exec -it <pod> -- <cmd> – Run a command in a container

kubectl port-forward – Forward local port to pod/service (as discussed)

kubectl cp – Copy files between your machine and a pod

kubectl attach – Attach to a running container (like docker attach)

kubectl proxy – Start a local proxy to access the Kubernetes API

📦 Deployment & Exposure
kubectl run – Start a pod (often replaced by apply)

kubectl expose – Expose a resource as a new service

kubectl rollout – Manage deployment rollouts (status, undo, etc.)

🧠 Cluster-Level Operations
kubectl config – View or set kubeconfig settings

kubectl version – Show client/server version

kubectl cluster-info – Show cluster endpoints

kubectl api-resources – List all available resource types

kubectl auth – Check access rules (e.g., can-i)
