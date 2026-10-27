This allows you start a secure shell on your EC2 and on-premises servers without having SSH access, bastion host or SSH keys needed
This means no port 22 needed (better security)

### How it works
The EC2 instance has a SSM Agent. This is connected to the Session Manager.
The user accesses the EC2 instance through the Session manager.

Supports linux, macOS, and Windows. and can send session log data to S3 or CloudWatch logs

## System Manager -- Run command
This lets you remotely execute commands on one or more EC2 instances without using SSH, opening inbound ports, or logging in interactively

Run Command = execute commands remotely at scale

For an EC2 instance to receive a Run Command instruction, you'd need:
- SSM Agent installed and running
- IAM permissions
- Connectivity to Systems Manager

use-cases:
	Install software or apply patches
	restart a service
	run shell scripts, colect logs, ...

Does not need SSH

example: `sudo systemctl restart nginx` on multiple linux EC2 instances in one operation