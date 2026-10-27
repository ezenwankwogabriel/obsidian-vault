validates a candidates ability to design solution based on AWS Well-Architected Framework

### AWS Global Infrastructure
> AWS Regions
> AWS Availability Zones
> AWS Data Centers
> AWS Edge Locations / Point of Presence

### Factors determining chose of AWS Region
- Compliance with data governance and legal requirements: They may want the data to be local to the region. data never leaves a region without your explicit permission
- Proximity to customers: reduced latency
- AVailable Services within a region: some new services and features aren't available in every region. so we tend to use on in another region
- Pricing: varies by region

### AWS Available Zones  (AZ)
Each region has many AZ (min: 3, max: 6)
Each AZ has one or more discrete data centers with redundant power, networking and connectivity
They are separated from each other in case of disasters
Connected with high bandwidth, low latency networking, forming a region

### AWS Point of Presence (Edge locations)
Amazon has 400+ point of presence (400+ edge locations & 10+ regional caches) in 90+ cities across 40+ countries
This helps to deliver content to users with lower latency (choosing closest one you you)

*AWS has Global services (IAM, Route53, CloudFront, WAF) as well as Region-Scoped services (EC2, Elastic Bean, Lambda, Rekognition, KMS,  ...)*

### IAM: Users & Groups
Root account created by default, shouldn't be used/shared
**Groups can contain users, not other groups**
users don't have to belong to a group, and users can belong to multiple groups

### IAM: Permissions
- Users & groups can be assigned JSON document called *policies*
- These policies define the permissions for the users (so they don't explore more than the given resource and burn cash)
- Always apply least privilege principle: don't give more permissions than a user needs

### MFA devices options in AWS
- Virtual MFA devices (Google authenticator, Authy)
- Universal 2nd Factor (U2F) Security key (Yubikey by Yubico (3rd party)) [Physical device]. supports multiple token using a single security key
- Hardware key fob MFA Device (provided by Gemalto (3rd party)) 
- Hardware Key Fob MFA Device for AWS GovCloud(US) [Provided by SurePassID (3rd Party)]

IAM Roles for Services
- Some AWS Service will need to perform actions on your behalf
- To do so, we assign permissions to AWS services with IAM Roles

### IAM Security Tools
- IAM Credentials Report (account-level)
	- report lists all account's users and status of their credentials
- IAM Access Advisor (user-level)
	- shows service permission granted to user and when those services were last accessed
> IAM > Credential Report > Download credential report

### IAM guidelines and best practices
- Do not use root account except for AWS account setup
- one user = one aws user
- assign user to groups & assign permissions to group
- create strong password policy
- use and enforce use of MFA
- create and use roles for giving permissions to aws services
- audit permissions of your account using IAM Credential report & IAM Access Report