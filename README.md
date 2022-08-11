# simple example of running netperf in k8s cluster
Simple project to get started with latency testing in kubernetes cluster. Similar concept can be used with
different tools e.g. iperf, ping, ...

This is just a template project, feel free to update `.yaml` manifests to include node selectors, tolerations etc.

## setup
You can skip this setup if you are going to run tests on an existing cluster
- start kind cluster `kind create cluster --name netperf --config kind-config.yaml`
- build netperf docker image `docker build -t pete911/netperf:latest .`
- load netperf docker image to the cluster `kind load docker-image pete911/netperf:latest --name netperf`

## install
- install netperf deployments:
  - daemonset (one server, client on every node) `kubectl apply -f k8s-netperf-ds.yaml`
  - server and client as containers in a single pod `kubectl apply -f k8s-netperf-single-pod.yaml`

## test

### pod to pod
- get netperf server 
  - pod name, node name and pod IP `kubectl get pod -l app.kubernetes.io/name=netperf-server -o=custom-columns=NAME:.metadata.name,NODE:.spec.nodeName,IP:.status.podIP` 
  - or more info if needed `kubectl get pod -l app.kubernetes.io/name=netperf-server -o wide`
- get netperf clients
  - pod and node name `kubectl get pod -l app.kubernetes.io/name=netperf-client -o=custom-columns=NAME:.metadata.name,NODE:.spec.nodeName`
  - or more info if needed `kubectl get pod -l app.kubernetes.io/name=netperf-client -o wide`
- exec to a client pod (either on the same node as server or different) `kubectl exec -it <pod-name> -- sh` (replace `<pod-name>`)
- run netperf against the server `netperf -H <server-pod-IP> -l 100 -t TCP_RR -- -o min_latency,mean_latency,max_latency` (replace `<server-pod-IP>`)

### container to container
- get netperf pod `kubectl get pod -l app.kubernetes.io/name=netperf`
- exec to a client container `kubectl exec -it <pod-name> -c client -- sh` (replace `<pod-name>`)
- run netperf against the server `netperf -H 127.0.0.1 -l 100 -t TCP_RR -- -o min_latency,mean_latency,max_latency`

## cleanup
- daemonset
  - client `kubectl delete ds netperf-client`
  - server `kubectl delete deploy netperf-server`
- single pod deployment `kubectl delete deploy netperf`

If you used kind cluster (setup sections), run `kind delete cluster --name netperf` as well
