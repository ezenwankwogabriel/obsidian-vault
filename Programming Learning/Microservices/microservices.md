Typically we use monolithic server

Containing all we need to run *all* features in our application from the routing to middlewares to business logic and then database

![[Pasted image 20260913114844.png]]

Meanwhile a microservice contains all those needed to implement ***one feature*** of **`our`** application.
Each services might have it's own database

How do they communicate:
- using a standard JSON/REST
- RPC - faster than REST
- gRPC - faster than RPC in most cases
- Using a queue

Advantages:
- Microservices are much easier to scale, you can scale the part with more load only
- you could have these services in docker, and put them in k8 and have as many clusters as needed (making it easier to scale and also to maintain)
Drawback:
- Harder to write
- Data management between services
![[Pasted image 20260913114936.png]]


Communication strategies btw services
Question: when would you choose sync over async (event driven) communication
- Sync: services communicate with each other using direct requests
	![[Pasted image 20260909152228.png]]
- Async: services communicate with each other using events
	first approach
	- using an Event Bus, accessible to every service
	- services emit events/receive events from the event bus
	- you send an event, await a response, and send other events
	- this shares similar concerns with a sync comm approach

	Second approach
	- define the exact goal for the service
	- introduce a db that serves this exact purpose, mapping ids, and other required details

What we'll build
 - Broker: an optional single point of entry to micro services. Receive request from frontend and fire to different microservices
 - Authentication: Postgres
 - Logger: Mongodb
 - Main: sends mail with specific template
 - Listener: consumes messages in RabbitMQ and initiates a process

- We'd start communication using RestAPI with JSON as transport
- Sending & Receiving using RPC, then gRPC, and 
- initiating and responding to events using advanced message queuing protocol (amqp) using RabbitMQ as backend


### Setting up a broker service
- setup a microservice, the broker service, running in docker.

![[Pasted image 20260916110450.png]]
User initiates request to broker, which is sent to the MQ - rabbit mq (db) an ampq service, this is then broadcast to the listener


