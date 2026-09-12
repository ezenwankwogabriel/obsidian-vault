A good architecture is important, otherwise it becomes slower and more expensive to add new capabilities in the future.

How to design software
Avoid over engineering and intentional progress

Ask two questions:
- what happens if the server dies
	- we want to not loose data, hence we create a db
- what happens when you want to scale
	- can have two servers

The need arises for more than one server, we start to get to a distributed architecture per say. They don't need to talk to each other, but we need a way to know which server is in use (serving the user). 
This is called a load balancer (acts as the balancer for traffics between the servers) mostly using what we call Round Robbin algorithm

When you think of scaling, you now think either horizontal(adding more machine) or vertical scaling(adding more compute(ram/cpu/storage)). 
Load balances allow you to autoscale (can have a minimum server, and can add more as demand needs)
Now demand is going up, we are adding more engineering teams, and requests are getting slow. The need for microservices comes up. Some customer mentioned slow uploads, we move this to it's own service. (identify a pinpoint and move it to a service)

Api gateway comes in when we now need to collate these. we now have a microservice, and many services running. how would users know where to get to (that the files requires an initial auth and profile service before hitting the files service). 
This gives rise to a virtual private cloud (VPC). where the gateway is the single point of entry for communicating with our system.

![[Pasted image 20260907092918.png]]

We previously had the loadbalancer after the gateway, but we also would rather want to scale the load balancer, so the gateway doesn't become a single point of failure




Things to look into practically:
servers 
scaling
load balancing (weighted round-robin, sticky sessions, least connections)
	sticky sessions where if a user goes to a particular server, best to redirect other of its traffic to same server
have some go services, then set them up with ngnix to load balance traffic between them using either of rr or sticky session
Api Gateway: receives request and redirect to the right service
Caching
CDN
