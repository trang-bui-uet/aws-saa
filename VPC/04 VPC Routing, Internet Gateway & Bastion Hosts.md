How routing work, Internet Gateway, How config VPC so that data exit to and enter from AWS public zone
- Routing
- Internet Gateway
- Architecture
- Bastion Host

**VPC Router**
- Every VPC has a VPC Router - Highly available device
- In every subnet .. **network+1** address
- Routes traffic between subnets
- Controlled by **route tables** each subnet has one
- A VPC has a **Main route table** - subnet default
- Destination match -> route to target (more specific, more priority, except local target always take priority)

**Internet Gateway**
- **Region resilient gateway** attached to a VPC
- 1 VPC = 0 or 1 IGW, 1 IGW = 0 or 1 VPC
- Internet Gateway run from the border of the VPC (AWS Public Zone)
- Gateways traffic between the VPC and the Internet or AWS Public Zone (S3...SQS...SNS...etc)
- It's managed gateway and so AWS handles performance

**Using an IGW**
1. Create IGW
2. Attach IGW to VPC
3. Create custom RT
4. Associate RT
5. Default Routes -> IGW
6. Subnet allocate IPv4

![[Pasted image 20260401172746.png]]

**Bastion Host/Jumpbox**
- An instance in a public subnet
- Incoming management connections arrive there
- Then access internal VPC resources
- Often the only way IN to a VPC