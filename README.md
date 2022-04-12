# Service Mesh / Istio: Study

** Service Mesh: An Extra layer in the cluster to monitor and modify in real time the app trafics, as well as level up the security and confiability of the ecosystem

** Istio: It is an opensource project that implements Service Mesh, independently of the language or technology (it works with kubernetes, apache mesos, nomad, etc.)

** Istio creates a proxy (sidecar proxy) inside each POD to receive and to send data between pods. Thorough a Istio control panel layer (in a specific POD), it controls all the proxys

** In this study, we are going to use the k3d to simulate the kubernetes cluster. This tool is easy to redirect port, as we can see under. The tool "kind" (another tool to work with kubernetes) is not so easy to do that.

1) Install k3d through WSL2
2) Create the cluster 
```
k3d cluster create -p "8000:30000@loadbalancer" --agents 2

* 8000: local port
* 30000: Cluster port. It is going to call the 30000 port of the service
* --agents 2: nodes
```

3) Change the kube context if you are using another one:
```
kubectl config use-context k3d-k3s-default
```

4) With WSL2, download the Istio CLI as a form to easily apply the configuration to the kubernetes
``` 
istioctl install
```
5) To inject the proxy inside all the pods of a specific namespace (default, in this case):
```
kubectl label namespace default istio-injection=enabled

* It will affect only the new pods. So, delete the existing ones.
```
6) Install the addons (grafana, jaeger, kiali, promotheus):
```
kubectl apply -f *
* https://github.com/istio/istio/tree/master/samples/addons
* kubectl get pods -n istio-system (to show all the pods wunning the addons)
* to run the kiali dashboard: istioctl dashboard kiali
```

Some concepts:
```
1) Ingress Gateway: release the input traffic. It connects to the Virtual Service
2) Virtual Service: Route the traffic (it is not the service from kubernetes), using the Service (from kubernetes) to forward it. It configurates all the proxies. Features:
  - Match: Ex: Match de URL, foward to some pod 
  - Retries: Ex: Retries the communication X times to other pod
  - Fault Injection: Ex: I want to 70% of my services are OK and 30% not, to test the whole application
  - Timeout: Ex: Iif the answer of the call wait too long, cancel the call
  - Subsets (v1 and v2): destiny categories (DESTINATION RULES). (What happens with the traffic when it gets to the destiny)
        1) Selector
        You can create some destination rules to request some traffic to one and to another. 
        Ex: 30% to the first one version (v1) and 70% to the second one version (v2). 
        2) Load Balancer
        3) Locality: specific the location of the pod (Europe, Brazil, etc)
        4) Circuit Breaker: Ex: If the connection from one pod to another stops to answer, waits X seconds and redirect to another one. And wait to connect again when it come back
```
