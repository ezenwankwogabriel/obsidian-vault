This is the way you structure a software system.
- its major components
- how these components communicate
- important decisions that determine how the system behaves and grows
[Deciding how to organize and connect pieces of a system before or while you build it]

Imagine an e-commerce app
```
                    E-commerce System
                           │
          ┌────────────────┼────────────────┐
          ↓                ↓                ↓
       Frontend         Backend          Database
          │                │                │
       React          API/Services       PostgreSQL
                           │
              ┌────────────┼────────────┐
              ↓            ↓            ↓
           Users        Orders       Payments
```

Architecture answers:
- should frontend and backend be separate?
- should the backend be one application or several microservices?
- how sould services communicate: HTTP, gRPC, events?
- where should authentication happen?
- where should business logic live
- which database should we use?
- should we use redis for caching
- what happens if the payment service goes down
- how do we scale when there are 10million users
- how do we deploy and monitor everything

Architecture exists at different levels
1. System architecture: Here you're deciding what the major building blocks are
```
Mobile App ──→ API ──→ Services ──→ Database
                         │
                         ├──→ Redis
                         ├──→ Payment Provider
                         └──→ Message Queue
```
  2. Application architecture: how you organize the internals of an individual application
  ```
HTTP Handler
     ↓
Service / Use Case
     ↓
Repository
     ↓
Database 
  ```
  Here you hear things like: 
  - layered architecture
  - clean architecture
  - hexagonal architecture
  - onion architecture
  - mvc

3. Component/module architecture
	How you divide the code itself
	```
	orders/
    handler.go
    service.go
    repository.go
    model.go

	users/
    handler.go
    service.go
    repository.go
    model.go
	```

There isn't one best architecture. Let's take a look at the many parts of system deployment and communication


