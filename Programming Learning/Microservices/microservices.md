Typically we use monolithic server

Containing all we need to run `**all**` features in our application from the routing to middlewares to business logic and then database

Meanwhile a microservice contains all those needed to implement one feature of **`our`** application.

Challenges with microservices: Data management between services

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