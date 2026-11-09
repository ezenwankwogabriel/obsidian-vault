The ability to stop & restart an EC2 instance depends on its root device type

| Feature                   | EBS-backed                       | Instance-store-backed |
| ------------------------- | -------------------------------- | --------------------- |
| Stop / Start              | Yes                              | No                    |
| Reboot                    | Yes                              | Yes                   |
| Root device               | EBS volume                       | Local instance store  |
| Data survives stop        | Yes                              | Cannot be stopped     |
| Data survives termination | Depends on `DeleteOnTermination` | No                    |

1. EBS-backed instances
- OS is stored on an EBS volume.
- EBS persists independently of the EC2 compute host.
- Stopping releases compute resources but preserves EBS volumes.
- Can be restarted later, potentially on different underlying hardware.

2. Instance-store-backed instances
- OS is stored on temporary local disks attached to the physical host.
- Cannot be stopped and restarted.
- Can be rebooted or terminated.
- Instance-store data is lost when the instance is terminated or the underlying host fails.

### We have two volumes: Root volume and additional volume

An EC2 instance can have multiple storage volumes. Could be EBS, or Instance Store

1\. Think of the root as the operating system disk
Just like your windows has a:
- Drive C: Windows OS
- Drive D: Your document
- An external USB drive: temp files

An EC2 works similarly

2\. EBS backed EC2
Root Volume is EBS (50GB) -> linux 
App data (document) is EBS (100GB)
Instance Store is 500GB (temp cache)

You can restart since the OS is on EBS. The additional instance store data is lost

3\. Instance store backed EC2
Root Volume is an Instance store (50GB) -> linux
App data (document) is EBS (100GB)
This is an instance-stored-backed instance

You cannot stop this instance because the OS is an Instance Store, which is a local storage. Having an EBS volume doesn't make it EBS-backed
