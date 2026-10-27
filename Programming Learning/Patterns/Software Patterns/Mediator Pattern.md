*instead of objects talking directly to each other, they talk through a mediator*

### without a mediator
```
type OrderService struct {
	payment PaymentService
	email EmailService
	inventory InventoryService
}

func (o *OrderService) CreateOrder() {
	o.inventory.Reserve()
	o.payment.Charge()
	o.email.SendConfirmation
}
```
OrderService knows directly about three other services. As application grows, coupling increases

### with a mediator
```
                 Mediator
               /    |    \
              ↓     ↓     ↓
           Order  Payment Email
           
type Mediator interface {
	Send(message any) error
}

func (o *OrderService) CreateOrder() error {
	return o.mediator.Send(CreateOrderCommand) {
		OrderId: "123"
	}
}
```

### Complete small example
```
package main

import "fmt"

type CreateOrderCommand struct {
	OrderID string
	Amount  int
}

type CreateOrderHandler struct{}

func (h CreateOrderHandler) Handle(cmd CreateOrderCommand) error {
	fmt.Println("Creating order:", cmd.OrderID)
	fmt.Println("Amount:", cmd.Amount)

	return nil
}

type Mediator struct {
	createOrderHandler CreateOrderHandler
}

func (m Mediator) Send(cmd CreateOrderCommand) error {
	return m.createOrderHandler.Handle(cmd)
}

func main() {
	mediator := Mediator{
		createOrderHandler: CreateOrderHandler{},
	}

	err := mediator.Send(CreateOrderCommand{
		OrderID: "order-123",
		Amount: 500,
	})

	if err != nil {
		panic(err)
	}
}
```