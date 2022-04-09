# Service Mesh / Istio: Study

** Service Mesh: An Extra layer in the cluster to monitor and modify in real time the app trafics, as well as level up the security and confiability of the ecosystem

** Istio: It is an opensource project that implements Service Mesh, independently of the language or technology (it works with kubernetes, apache mesos, nomad, etc.)

** Istio creates a proxy inside each POD to receive and to send data between pods. Thorough a Istio control panel layer (in a specific POD), it controls all the proxys

** In this study, we are going to use the k3d to simulate the kubernetes cluster

1) Install k3d through chocolatey (if you are using windows)
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
