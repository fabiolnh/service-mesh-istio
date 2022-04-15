# Service Mesh / Istio: Study

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
* Service Mesh: An Extra layer in the cluster to monitor and modify in real time the app trafics, as well as level up the security and confiability of the ecosystem

* Istio: It is an opensource project that implements Service Mesh, independently of the language or technology (it works with kubernetes, apache mesos, nomad, etc.)

* Istio creates a proxy (sidecar proxy) inside each POD to receive and to send data between pods. Thorough a Istio control panel layer (in a specific POD), it controls all the proxys

* In this study, we are going to use the k3d to simulate the kubernetes cluster. This tool is easy to redirect port, as we can see under. The tool "kind" (another tool to work with kubernetes) is not so easy to do that.

* Consistent Hash (Stick Session): when a user access one version, it will always access the same version. The load balancer will always forward to that version. If you use weight (percentage for each version), this feature will not work (istio issue. probably will be fixed later). Ways to do the consisten hash:
  1) httpHeaderName
  2) httpCookie
  3) UseSourceIp
  4) httpQueryParameterName

* Ingress Gateway -> Virtual Service -> Destination Rule

1) Ingress Gateway: release the input traffic. It connects to the Virtual Service
2) Virtual Service: Route the traffic (it is not the service from kubernetes), using the Service (from kubernetes) to forward it. It configurates all the proxies. Features:
  - Match: Ex: Match the URL, foward to some pod 
  - Retries: Ex: Retries the communication X times to other pod
  - Fault Injection: 
      Ex: I want to 70% of my services are OK and 30% not, to test the whole application. 
      Ex: wait 10s to request an http communitation
      Ex: return httpStatus 500 of a percentage of all the services
  - Timeout: Ex: Iif the answer of the call wait too long, cancel the call
  - Subsets (v1 and v2): destiny categories (DESTINATION RULES). (What happens with the traffic when it gets to the destiny)
        1) Selector
        You can create some destination rules to request some traffic to one and to another. 
        Ex: 30% to the first one version (v1) and 70% to the second one version (v2). 
        2) Load Balancer
        3) Locality: specific the location of the pod (Europe, Brazil, etc)
        4) Circuit Breaker: 
          Ex: If the connection from one pod to another stops to answer, waits X seconds and redirect to another one. And wait to connect again when it come back
          Ex: define a rule that when some microservice gets slow (an example), break the circuit and return an 500 httpStatus (this way we can free the microservice). This way is better than keep the connection. After some time, defined, we can release this microservice again.
```

To test:
```
while true ; do curl http://localhost:8000; echo; sleep 0.5; done;

or:

Use Fortio to execute requests:
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.13/samples/httpbin/sample-client/fortio-deploy.yaml
export FORTIO_POD=$(kubectl get pods -l app=fortio -o 'jsonpath={.items[0].metadata.name}')
kubectl exec "$FORTIO_POD" -c fortio -- fortio load -c 2 -qps 0 -t 200s -loglevel Warning http://nginx-service:8000
```

Annotations:
```
* We can create the VirtualService and the Destination Rule through the Kiali or configuring manually the yaml 
```