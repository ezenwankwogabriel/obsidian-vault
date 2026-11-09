An ENI is a virtual network interface that has its own

Think of this as the network card on a physical computer which holds:
- Private IPv4 address
- MAC address
- Security groups
- Subnet association

### Primary ENI
On launching an EC2 instance, AWS creates/attaches a primary network interface. This is called eth0 (like we have in our system/switches/routers)

This interface cannot be detached while the instance is running or stopped
It is deleted when the instance is terminated
It also is responsible for the instance's primary private IPv4 addres

###  Secondary ENI
You can create additional network interfaces and attach them to an existing EC2 instance.
This secondary ENI can be moved between EC2 instances, provided the instances are in the same AZ and satisfy attachment requirements

### What happens when you attach a secondary ENI
Let's walk through an actual example.

You have:
- EC2-A in subnet 10.0.1.0/24.
- Secondary ENI with private IP 10.0.1.50.
- EC2-A is running your application.
You now attach the secondary ENI.

The instance has two network interfaces:
```
EC2-A
│
├── eth0 → Primary ENI
│          10.0.1.10
│
└── eth1 → Secondary ENI
           10.0.1.50
```

Both interfaces belong to the same instance, but they have separate network identities.

You can use them for different purposes, such as:
- Separating management and application traffic.
- Separating public-facing and internal traffic.
- Network appliances that forward traffic.
- High-availability failover.
One important caveat: attaching a second ENI doesn't automatically make the operating system or application use it. Network routing and application configuration may also be needed.

### How ENI failover works
Let's return to the Spot instance question.

You have:
- EC2-A: primary application instance.
- EC2-B: replacement instance.
- ENI-002: secondary ENI with private IP 10.0.1.50.
- DNS: dashboard.company.com → 10.0.1.50.

Now EC-A is terminated because AWS reclaims the Spot capacity. The automation attaches ENI-002 to EC2-B

The DNS record doesn't need to change because the private IP is preserved