# Kubernetes
***
findEmptyNamespaces.sh: Loop over all namespaces in a cluster and find empty ones.

getPodsTopCSV.sh: Get a pod's cpu and memory usage (optionally per container) written as CSV formatted file.

getResourcesCSV.sh: Get all pods resources requests, limits and actual usage per container in a CSV format with values normalized.

getRestartingPods.sh: Get all pods (all or single namespace) that have restarts detected in one or more containers. Formatted in CSV.

podReady.sh: Simple script to check if pod is really ready. Check status is 'Running' and that all containers are ready. Returns 0 if ready. Returns 1 if not ready.

getNodesLoadCSV.sh: Traverse over the kube-proxy pods to get the nodes load average and number of CPUs in a CSV format. Will also mark high load node with big YES in the output.

Commands
Kubectl
See all cluster nodes load (top)
kubectl top nodes
Get cluster events
# All cluster
kubectl get events

# Specific namespace events
kubectl get events --namespace=kube-system
Get all cluster nodes IPs and names
# Single call to K8s API
kubectl get nodes -o json | grep -A 12 addresses

# A loop for more flexibility
for n in $(kubectl get nodes -o name); do \
  echo -e "\nNode ${n}"; \
  kubectl get ${n} -o json | grep -A 8 addresses; \
done
See all cluster nodes CPU and Memory requests and limits
# With node names
kubectl describe nodes | grep -A 3 "Name:\|Resource .*Requests .*Limits" | grep -v "Roles:"

# Just the resources
kubectl describe nodes | grep -A 3 "Resource .*Requests .*Limits"
Using kube-capacity
There is a great CLI for getting a cluster capacity and utilization - kube-capacity.
Install as described in the installation section.
# Get cluster current capacity
kube-capacity

# Get cluster current capacity with pods breakdown
kube-capacity --pods

# Get cluster current capacity and utilization
kube-capacity --util

# Displaying available resources
kube-capacity --available

# Roll over all clusters in your kubectl contexts
for a in $(kubectl ctx); do echo -e "\n---$a"; kubectl ctx $a; kube-capacity; done

# Roll over all clusters in your kubectl contexts and get just summary of each cluster
for a in $(kubectl ctx); do echo -e "\n---$a"; kubectl ctx $a; kube-capacity| grep -B 1 "\*"; done
Get all labels attached to all pods in a namespace
for a in $(kubectl get pods -n namespace1 -o name); do \
  echo -e "\nPod ${a}"; \
  kubectl -n namespace1 describe ${a} | awk '/Labels:/,/Annotations/' | sed '/Annotations/d'; \
done
Forward local port to a pod or service
# Forward localhost port 8080 to a specific pod exposing port 8080
kubectl port-forward -n namespace1 web 8080:8080

# Forward localhost port 8080 to a specific web service exposing port 80
kubectl port-forward -n namespace1 svc/web 8080:80
Port forwarding
•	A great tool for port forwarding all services in a namespace + adding aliases to /etc/hosts is kubefwd. Note that this requires root or sudo to allow temporary editing of /etc/host.
# Port forward all service in namespace1
kubefwd svc -n namespace1
Extract and decode a secret's value
# Get the value of the postgresql password
kubectl get secret -n namespace1 my-postgresql -o jsonpath="{.data.postgres-password}" | base64 --decode
Copy secret from namespace1 to namespace2
kubectl get secret my-secret --namespace namespace1 -o yaml | sed "/namespace:/d" | kubectl apply --namespace=namespace2 -f -
Create an Ubuntu pod
A one liner to create an Ubuntu pod that will just wait forever.
# Create the pod
cat <<ZZZ | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: my-ubuntu-pod
spec:
  containers:
  - name: my-ubuntu-container
    image: ubuntu:20.04
    command:
    - 'bash'
    - '-c'
    - 'while true; do sleep 5; done'
ZZZ

# Shell into the pod
kubectl exec -it my-ubuntu-pod bash

# Delete the pod once done
kubectl delete pod my-ubuntu-pod
Start a shell in a temporary pod
Note - Pod will terminate once exited
# Ubuntu
kubectl run my-ubuntu --rm -i -t --restart=Never --image ubuntu -- bash

# CentOS
kubectl run my-centos --rm -i -t --restart=Never --image centos:8 -- bash

# Alpine
kubectl run my-alpine --rm -i -t --restart=Never --image alpine:3.10 -- sh

# Busybox
kubectl run my-busybox --rm -i -t --restart=Never --image busybox -- sh
Get formatted list of container images in pods
Useful for listing all running containers in your cluster
# Option 1 - just the container name
kubectl get pods -A -o jsonpath='{..containers[*].name}' | tr -s ' ' '\n'

# Option 2 - namespace, pod container images and tags
kubectl get pods -A -o=jsonpath='{range .items[*]}{.metadata.namespace},{.metadata.name},{.spec.containers[*].image}{"\n"}' | tr -s ' ' '\n'

# Option 3 - pod container images and tags
kubectl get pods -A -o=jsonpath='{..containers[*].image}' | tr -s ' ' '\n'
Look into a few more examples of listing containers
Get list of pods sorted by restart count
•	Option 1 for all pods (Taken from kubectl cheatsheet)
kubectl get pods -A --sort-by='.status.containerStatuses[0].restartCount'
•	Option 2 with a filter, and a CSV friendly output
kubectl get pods -A | grep my-app | awk '{print $5 ", " $1 ", " $6}'  | sort -n -r
Get current replica count on all HPAs (Horizontal Pod Autoscaling)
kubectl get hpa -A -o=custom-columns=NAME:.metadata.name,REPLICAS:.status.currentReplicas | sort -k2 -n -r
List non-running pods
kubectl get pods -A --no-headers | grep -v Running | grep -v Completed
Top Pods by CPU or memory usage
# Top 20 pods by highest CPU usage
kubectl top pods -A --sort-by=cpu | head -20

# Top 20 pods by highest memory usage
kubectl top pods -A --sort-by=memory | head -20

# Roll over all kubectl contexts and get top 20 CPU users
for a in $(kubectl ctx); do echo -e "\n---$a"; kubectl ctx $a; kubectl top pods -A --sort-by=cpu | head -20; done
Helm
Helm template
View the templates generated by helm install. Useful for seeing the actual templates generated by helm before deploying.
Can also be used for deploying the templates generated when cannot use Tiller
helm template <chart>
Debug helm install
•	Debug a helm install. Useful for seeing the actual values resolved by helm before deploying
helm install --debug --dry-run <chart>
Rolling restarts
Roll a restart across all resources managed by a Deployment, DaemonSet or StatefulSet with zero downtime
IMPORTANT: For a Deployment or StatefulSet, a zero downtime is possible only if initial replica count is higher than 1!
# Deployment
kubectl -n <namespace> rollout restart deployment <deployment-name>

# DaemonSet
kubectl -n <namespace> rollout restart daemonset <daemonset-name>

# StatefulSet
kubectl -n <namespace> rollout restart statefulsets <statefulset-name>
Metrics Server in Kubernetes on Docker Desktop for Mac
To get around issue with certificates in your local Docker Desktop Kubernetes
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
Edit the metrics-server deployment and inject --kubelet-insecure-tls to the args key:
spec:
  containers:
  - args:
    - --cert-dir=/tmp
    - --secure-port=443
    - --kubelet-insecure-tls
    - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
    - --kubelet-use-node-status-port
    - --metric-resolution=15s
  
Resources
  
Most of the code above is self experimenting and reading the docs. Some are copied and modified to my needs from other resources...
•	https://kubernetes.io/docs/reference/kubectl/cheatsheet/
•	https://medium.com/flant-com/kubectl-commands-and-tips-7b33de0c5476
•	https://github.com/robscott/kube-capacity

