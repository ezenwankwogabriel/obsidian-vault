This is a message broker ----- It accepts & forwards messages. 

Can think of it as a post office: 
- you put the mail for posting in a post box 
- you can be sure that the letter carrier will eventually deliver the mail to your recipient
- you don't really have to wait for response from the recipient
Here, RabbitMQ is a post box, a post office, and a letter carrier.

Terminologies:
- Producer: A program that sends message (clients/servers)
- Queue: the post box here. messages are stored here. they are bound by the host's memory & disk limits (a large message buffer)
- Consumer: A program that mostly waits to receive message

NB: the producer, consumer & broker do not have to live on same host. An app can also be both a producer & consumer.

## Example for "hello world"
- a producer that sends a single message (P)
- a consumer that receives messages & prints them out (C)
`P ------> "hello" ------> C`

RabbitMQ speaks multiple protocol. For this example, AMQP 1.0 is used over same port as AMQP 0-9-1 client (amqp091-go)
`go get github.com/rabbitmq/amqp/rabbitmq-amqp-go-client`

### Sending [P -----> "hello"]
we'd call our publisher(sender) `send.go` and consumer `receive.go`. The publisher connects to RabbitMQ, sends a single message, then exit.

```send.go
package main

import (
	context
	log
	
	rmq "github.com/rabbitmq/rabbitmq-amqp-go-client/pkg/rabbitmqamqp"
)

// setup connection to the broker using env & connection
const brokerURI = "amqp://guest:guest@localhost:5672"

func main() {
	ctx := context.Background()
	env := rmq.NewEnvironment(brokerURI, nil)
	conn, err := env.NewConnection(ctx)
	if err != nil {
		log.Panicf("failed to connect to rabbitmq: %v", err)
	}
	defer func() {
		_ = env.CloseConnections(context.Background())	
	}()
}
```

Next we declare a quorum queue named "hello"
```
	_, err = conn.Management().DeclareQueue(ctx, &rmq.QuorumQueueSpecification{Name: "hello"})
	if err != nil {
		log.Panicf("Failed to declare a queue: %v", err)
	}
```
^^^ declaring  a queue is idempotent. only created if doesn't exist.

create a publisher and send a message
```
publisher, err := conn.NewPublisher(ctx, &rmq.QueueAddress{Queue: "hello"}) // publisher to declared queue "hello"
defer func() { _ = publisher.Close(context.Background()) }()

body := "Hello World!"  

res, err := publisher.Publish(ctx, rmq.NewMessage([]byte(body)))  

if err != nil {  
	log.Panicf("Failed to publish a message: %v", err)  
}

switch res.Outcome.(type) {  
case *rmq.StateAccepted:  
default:  
	log.Panicf("Unexpected publish outcome: %v", res.Outcome)  
}  

log.Printf(" [x] Sent %s\n", body)
```


### Receiving "hello" -----> C
Consumer listens for messages from RabbitMQ, unlike publisher which publishes a single message, we'll keep the consumer running to listen for messages & print them out

setup environment & connection, declare same queue then create consumer
```receive.go:
...
main() {
	.. setup connection like we did for publisher
	.. declare queue as same also
	
	consumer, err := conn.NewConsumer(ctx, "hello", nil)
	.. handle err
	
	log.Printf("[*] Waiting for messages. To exit press CTRL+C")
	for {
		delivery, err := consumer.Receive(ctx)
		if err != nil {
			if errors.Is(err, context.Canceled) {
				return
			}
			log.Panicf("failed to recieve message: %v", ,err)
		}
		msg := delivery.Message()
		var body string  

		if len(msg.Data) > 0 {  
			body = string(msg.Data[0])  
		}  
		log.Printf("Received a message: %s", body)  
		err = delivery.Accept(ctx)  
		
		if err != nil {  
		log.Panicf("Failed to accept message: %v", err)  
		}
	}
}
```
