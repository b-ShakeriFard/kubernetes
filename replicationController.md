It is important to make sure labels and selectors contain correct information. Because one of the ket tasks of replication controller, is to monitor pods and restart them if one fails.

If you want to scale your application: use this command:

<b> kubectl scale --replicas=6 -f replicaset-definition.yaml </b>

or 

<b> kubectl scale --replicas=6 replicaset [type of resource] myapp-replicaset [NAME of the resource defined in metadata] </b>

other useful commands:

<b> kubectl create -f replicaset-definition.yaml </b>

<b> kubectl get replicaset </b>

<b> kubectl delete replicaset frpn-replicaset [name of the resource] </b>

<b> kubectl replace -f replicaset-definition.yaml </b>

<b> kubectl scale --replicas=6 -f replicaset-definition.yaml </b>

<b> kubectl edit replicaset my-new-replicaset-app-01 [Name of the Resource] </b>

