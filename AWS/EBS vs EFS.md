EBS = a hard drive attached to a server
	- One instance (except multi-attach io l/io 2)
	- locked at AZ level
	- gp2: IO increases if disk size increases
	- gp3 & io 1: can increase IO independently
	- To migrate an EBS across AZ, youd need to take a snapshot, and restore the snapshot to another AZ
	
EFS = a shared folder that multiple servers can access
- Mounting 100s of instances across AZ
- Only for linux instances
- higher price point
- can leverage storage tiers for cost savings

