install minikube using homebrew
install kubectl also likely using homebrew

```
	// start kubernetes cluster
	minikube start --nodes=2 
	
	// stop
	minikube stop
	
	// restart
	minikube start
	
	minikube status
	
	// to setup a pod
	// create a k8 folder and a yml file for the service (k8/mongo.yml)
	 kubectl apply -f k8/
	 
	 ```gives the below after the apply
	deployment.apps/mongo created
	service/mongo created
	```
	
	check this works by running `kubectl get pods`
	mongo-5db87d9c64-9q2q6   1/1     Running   0          2m
	
	get running services `kubectl get svc`
	NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
	kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP     146m
	mongo        ClusterIP   10.103.115.126   <none>        27017/TCP   4m32s
	
	get active deployments `kubectl get deployments`
	NAME    READY   UP-TO-DATE   AVAILABLE   AGE
	mongo   1/1     1            1           14m

	can now check the dashboard to see the pod
	`minekube dashboard`
	 
```
![[Pasted image 20260921144608.png]]


After setting up your clusters in kubernetes, they are not exposed to the outside world. there are three was to go about this.
- a node port
- load balancer
- ingress (used in production)

for test, we'd use load balancer, which would be like the broker service we had, handling connections to and out of the services from the client
initially the broker-service was run with 
```
# start a specific service
kubectl apply -f ~/files/k8/broker-service

# start a service as a loadbalancer
kubectl expose deployment broker-service --type=LoadBalancer --port=8080 --target-port=8080
```


## Ingress
An ingress is a means of getting to all of the microservices deployed inside the cluster.

It is a webserver, called NginX that'd handle requests from outside the cluster, examine them and determine where they need to go. (similar to caddy)

```
# enable addon with minikube
minikube addons enable ingress

# deploy a deployment file for the ingress
```


A production kubernetes cluster setup typically has
- multiple master & worker nodes
- separate virtual / physical machines

![[Pasted image 20260921122004.png]]

To test on a local machine, it'd be difficult if you don't have enough resource. for this, we use minikube.
one node cluster, where the master and worker processes run on one node. having docker pre installed. it creates a virtualbox on your laptop, and runs the nodes in that virtual box.
*a one node k8 cluster that runs in a virtual box in your laptop*
After this setup, you'd need some way to communicate with the cluster. and this is where *kubectl* comes in

### kubectl
a way to create pods, or other components on the node



A pod is deployed in k8. can have one or more containers/services. This is the basic uint in a kubernetes resource