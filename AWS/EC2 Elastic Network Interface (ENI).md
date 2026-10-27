This is a virtual network card that you attach to an EC2 instance so it can communicate over a network

Just as a laptop has a network card with an IP address, an EC2 instances uses ENI to connect to a VPC network

Can have:
- Private IPv4 addresses –– For comm inside the VPC
- Public IPv6 address - optionally through an Elastic IP or auto-assigned public IP
- MAC address –– a hardware-like network identifier
- Security groups: controlling inbound & outbound traffic
- Network interface & attachment information

In summary, it holds network identity and config, while the EC2 instance provides the computing power

![[Pasted image 20261014162929.png]]

| Exam clue                                 | Remember                                                        |
| ----------------------------------------- | --------------------------------------------------------------- |
| Virtual network card                      | ENI                                                             |
| Move a network identity between instances | Detach and reattach a secondary ENI                             |
| Attach an ENI to another EC2 instance     | Same Availability Zone                                          |
| Primary ENI                               | Cannot be detached from its instance                            |
| Multiple network interfaces               | Can provide separate network identities or network connectivity |

### Primary network interface
On launching an EC2, aws attaches a network interface to it. this is the primary network interface, called eth0 in many linux envs
- it is the main connection for the instance
- is a private ipv4 address
- cannot be detached from the instance

### Secondary network interface
An additional network interface beyond the primary ENI. Needed to:
- give an instance another network interface & private IP address
- connect an instance to an additional subnet in same VPC
- support network separation/specialized network configurations
- move network identity between instances during a failover

For example, a server might have a primary ENI for normal application traffic and a secondary ENI for a separate traffic path.

This helps preserve network identity, although it doesn't automatically transfer application data or guarantee that the application is ready.
