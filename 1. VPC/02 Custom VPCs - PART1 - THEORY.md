How to build a complex, multi tier custom VPC step by step
Start simple and add more and more complexity

**VPCs**
- Regional service - All AZs in the region
- Allow you to create isolated network inside AWS
- Nothing **IN** or **OUT** without explicit configuration
	- Muốn ra internet phải cấu hình Route Table + IGW or NAT Gateway
- Flexible configuration - simple or multi-tier
- Hybrid networking - other cloud and on-premises
- **Default** or **Dedicated Tenancy**!

- IPv4 Private CIDR Blocks and Public IPs
- 1 Primary Private IPv4 CIDR Block
- ...Min /28 (16 IPs) Max /16 (65536 IPs)
- Optional secondary IPv4 Blocks (max 4 secondary IPv4 block) ( 5 Total = 1 Primary IPv4 CIDR + 4 Secondary IPv4 CIDR)
- Optional single assigned IPv6/56 CIDR block

**DNS in a VPC**
- AWS VPCs has fully feature of DNS
- Provided by R53
- VPC **Base IP + 2** address (If VPC is 10.0.0.0 then DNS IP will be 10.0.0.2)
- Two option critical for How DNS functions in a VPC
	- **enableDnsHostnames** - give instances DNS name.
	- **enableDnsSupport** - enable DNS resolution in VPC