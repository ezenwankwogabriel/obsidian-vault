For CLB(deprecated) this is called Connection Draining
For ALB or NLB it is called Deregistration Delay

Gives some time for instances to complete "in-flight requests" while the instance is de-registering or unhealthy.
Once drained, the ELB would stop sending new requests to the EC2 instance which is de-registered
