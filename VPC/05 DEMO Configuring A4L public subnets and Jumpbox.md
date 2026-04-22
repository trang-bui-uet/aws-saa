1. Create IGW
2. Attach IGW to VPC
3. Create Route Table
4. Associate Route Table with Subnets
5. Edit Route -> Add a route -> 0.0.0.0/0 mean any traffic which is not destined for VPC will sent to IGW

6. Create EC2 with public IPv4 address and connect to it.