Task about how to design IP Plan for a business.
Create custom VPC

**VPC Considerations**
- What **size** should VPC be..
- Are there any networks we can't use... (prevent overlap and duplicate range)
- VPC's, Cloud, On-premises, Partners & Vendors
- Try to predict **the future**...
- VPC structure - Tiers & Resiliency (Availability) Zones. Network often split, and part of this network assigned to each of these zones.

**More considerations**
- VPC minimum /28 (16 IP), maximum /16 (65536 IP)
- Personal preference for the 10.x.y.z range 
- Avoid common ranges - avoid future issues (suggest 10.16)
- Suggest 2 range of network per region being used per account.
- 3US, 2EU -> 5*2 - assume 4 account -> total 40 ranges.

**IP Range to avoid**
- 192.168.10.0/24
- 10.0.0.0/16 (AWS)
- 172.31.0.0/16 (Azure)
- Google 10.128.0.0/9

**VPC Sizing**

| VPC Size    | Netmask | Subnet Size | Hosts/Subnet | Subnet/VPC | Total IPs |
| ----------- | ------- | ----------- | ------------ | ---------- | --------- |
| Micro       | /24     | /27         | 27           | 8          | 216       |
| Small       | /21     | /24         | 251          | 8          | 2008      |
| Medium      | /19     | /22         | 1019         | 8          | 8152      |
| Large       | /18     | /21         | 2043         | 8          | 16344     |
| Extra large | /16     | /20         | 4091         | 16         | 65456     |
- You should answer:
	- How many **subnets** will you need?
	- How many **IPs total**? How many **per subnet**?

**VPC Structure**
- Services use subnet
- VPC services run from within subnets
- Subnet located in one AZ
- How many AZ your VPC will use.
- Each tier has it's own subnet in each AZ
- AZ-A, AZ-B, AZ-C, Future
- ->Four web subnets, four app subnets, four database subnets and four spares (Total 16 subnet)
- So a /16 VPC split into 16 subnets /20.

**Proposal**
- Animals4life could become **huge** global entity
- Use 10.16 -> 10.127 range (avoid Google)
- 