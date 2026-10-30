This is when a load balancer stops sending new requests to a target being removed or marked unhealthy, while allowing existing requests to finish within a configured timeout.

Example:
An EC2 instance is being replaced during development:
1\. The load balancer stops sending new requests to that instance
2\. Existing requests are allowed to complete
3\. When they finish - or the timeout expires - the instance is deregistered and can be shut-down.

*Connection Draining = stop new traffic, let existing requests finish*

For CLB(deprecated) this is called Connection Draining
For ALB or NLB it is called Deregistration Delay

Gives some time for instances to complete "in-flight requests" while the instance is de-registering or unhealthy.
Once drained, the ELB would stop sending new requests to the EC2 instance which is de-registered
