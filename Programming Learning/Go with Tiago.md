https://12factor.net/ talks about the 12 factor app. 12 factors that can be applied to apps written in any programming language

REST (Representative State Transfer)

Uniform interface: states that all API requests for same resource should look same, no matter where the request comes from (mobile, web, ...)

Stateless means the request has all information for the server to carry out its action. for a stateful request, the server has some cache or context about the requester, which it uses to fetch data. This doesn't work well for distributed systems under a load balancer as a new request to a different server may not have enough context to process the request.

Always rely on abstraction and not the concrete implementation [an interface comes in handy here]
```main.go
type Truck struck { id string }
func (t *Truck) processTruck(truck) {}

func main() {
	truck := []Truck{}
	
	processTruck(truck) // the function relies on concrete implementation of Truck (an anti-pattern)
}
```