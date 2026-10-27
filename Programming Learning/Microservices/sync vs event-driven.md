Synchronous: I ask you to do something, and I wait for you to finish before it continue
Event driven: Something happened, i announce that it happened, and whoever cares can react to it independently.

*nuance* btw sync/event-driven and sync/async.
Event driven systems are often asynchronous
But an event driven system can contain synchronous operations too

Examples
```
		NORMAL FUNCTION
 # user clicks *Buy Product*
 Browser ––> order service ––> Database ––> Order service -–> Browser
 
 # key here: caller direclty asks the receiver to perform an operation
 order := CreateOrder(userId, productId) # waits for response (order)
```

```
		WHAT SYNCHRONOUS MEANS
# The caller's flow is dependent on the completion of the operation it called
result := database.CreateOrder(order)
sendConfirmation(result)
# result must be created before confirmation can be sent

**HTTPs are usually synchronous**
# browser sends request
# server receives and processes it, then responds -> 201 created

# in javascript, await expresses dependency
```

```
		IMAGINE YOU DON'T NEED TO WAIT
# suppose order is created, you need other things to do
- send email
- send sms
- update analytics, .... generate invoice
  
  
# one way is to wait for the http to do all of these (delays the user)

		WITH EVENT DRIVEN THINKING, WE CHANGE THIS
# The order service can say: "An Order was created" >> this is an event 
exampe {type: "orderCreated", orderId: "123", userId: "456"}

# it publishes that event to an event broker (e.g broker-service)
# *key*: it doesn't necessarily call each service for (email, analytic, invoice)
# instead it only announces the object with type: "ordercreated"
# any service listening for that event, get's it
This is *event driven architecture*
```

```
synchronous:
A ───── request ─────> B
A <──── response ───── B

event-driven
A ───── event ─────> Broker
                       |
             ┌─────────┼─────────┐
             ↓         ↓         ↓
             B         C         D
```

```
		COMMAND vs EVENT
a command says: Please do X -> CreateOrder, ChargeCreditCard
an event says: X already happened -> OrderCreated, PaymentCompleted
```

```
		WHERE RABBITMQ FITS
Order Service
     |
     | publish
     ↓
RabbitMQ
     |
     ├────────→ Email Service
     |
     ├────────→ Inventory Service
     |
     └────────→ Analytics Service
```

```
		SYNCRHONOUS VS ASYNCHROUNOUS

await sendEmail()
vs
queue.publish("sendEmail")
console.log("continue")

THESE OFTEN HAPPEN TOGETHER
User
 |
 | POST /orders
 ↓
Order API
 |
 | create order
 ↓
Database
 |
 | success
 ↓
Publish OrderCreated (RABBITMQ -> email, analytic, inventory)
 |
 ↓
Return 201

```

With synchronous, you have tight coupling. meanwhile eventdriven gives you a loose system
Event driven doesn't necessarily mean microservices. You can have an event-driven system inside one application.
Node.js has its own event emitter 
`emitter.emit("user.created", user)`
`emitter.on("user.created", handler)`

```
	EVENT DRIVEN ARCHI HAS 3 MAJOR PIECES
		   PRODUCER (service)
              |
              | event (action)
              ↓
          EVENT BUS (rabbitmq, kafka, nats, ...)
              |
       ┌──────┼──────┐
       ↓      ↓      ↓
   Consumer Consumer Consumer (service)
```