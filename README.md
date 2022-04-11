# Kubernetes
**
findEmptyNamespaces.sh: Loop over all namespaces in a cluster and find empty ones.

getPodsTopCSV.sh: Get a pod's cpu and memory usage (optionally per container) written as CSV formatted file.

getResourcesCSV.sh: Get all pods resources requests, limits and actual usage per container in a CSV format with values normalized.

getRestartingPods.sh: Get all pods (all or single namespace) that have restarts detected in one or more containers. Formatted in CSV.

podReady.sh: Simple script to check if pod is really ready. Check status is 'Running' and that all containers are ready. Returns 0 if ready. Returns 1 if not ready.

getNodesLoadCSV.sh: Traverse over the kube-proxy pods to get the nodes load average and number of CPUs in a CSV format. Will also mark high load node with big YES in the output.
