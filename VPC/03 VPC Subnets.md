**VPC Subnets**
- AZ Resilient (Bền vững trong AZ nếu AZ lỗi thì toàn bộ subnets sẽ down)
- A subnetwork of a VPC - **within a particular AZ**
- 1 Subnet => 1 AZ, 1AZ => 0+ Subnets
- IPv4 CIDR is a subset of the VPC CIDR
- Cannot overlap with other subnets
- Optional IPv6 CIDR (/64 subset of /56 VPC - space for 256 subset /64)
- Subnets can communicate with other subnets in the VPC

**Subnet IP Addressing**
- Reserved IP addresses (5 in total) (5 địa chỉ IP bị dành riêng)
- 10.16.16.0/20 (10.16.16.0 => 10.16.31.255)
- **Network** Address (10.16.16.0) represents the network.
- **VPC Router** - Network + 1 - 10.16.16.1 
- **Reserved DNS** - Network + 2 - 10.16.16.2
- **Reserved Future Use** - Network + 3 - 10.16.16.3
- **Broadcast Address** - 10.16.31.255
- For Instance inside VPC
	- For every VPC, there's a DHCP option set, that's linked to it and that cannot be changed
	- Auto Assign Public IPv4
	- Auto Assign IPv6

**Demo - Create multi tier VPCs subnets**
- 