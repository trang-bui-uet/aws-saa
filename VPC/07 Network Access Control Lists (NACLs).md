	 - Traditional Firewall available within AWS VPCs
- Every subnet has an associated Network ACL
- NACLs filter traffic crossing the subnet boundary INBOUND and OUTBOUND
- Connections within a subnet aren't impacted by NACLs
- NACLs contain INBOUND rules and OUTBOUND rules
	- Inbound rules match traffic **enter** the subnet
	- Outbound rules match traffic **leaving** the subnet
- NACLs are stateless (mean Rules required for both request and response part of every communication)
- Start from lowest rule number, if all rule un match -> Rule * Deny (If nothing else match, then traffic will be denied)
- It evaluates traffic against each individual rule until it find a match and then processing stop (If deny rule come first then allow rule might never be processed)



**Default NACL**
- A VPC created within a default NACL (all traffic allow)

**Custom NACL**
- Custom NACLs are created for a specific VPC
- Initially they're associated with no subnets.
- They only have one rule on both inbound and outbound rule, which is the default deny
- If you associate this customer NACL with any subnets all traffic will be denied.

**Network Access Control Lists (NACL)**
- **Stateless** - REQUEST and RESPONSE seen as different
- Only impacts **data crossing subnet boundary**
- NACLs can **explicit ALLOW** and **DENY**
- Only allow use IPs/CIDR, port & protocols - no logical AWS resources
- NACLs can not be assigned to AWS resources - **only subnets**
- Very often use together with Security Group to add explicit DENY (Bad IPs/Nets)
- Generally you use Security Group to allow traffic and you use NACLs to deny traffic.
- Each subnet can have **one NACL** (Default or Custom)
- A NACL can be associate with **MANY Subnet**