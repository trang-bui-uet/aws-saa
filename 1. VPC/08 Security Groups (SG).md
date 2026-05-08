- STATEFUL - detect response traffic automatically
- That mean Allowed Request = Allowed Response
- No explicit DENY - only ALLOW or implicit DENY
- can not block specific Bad actors -> use NACLs to explicit deny.
- Support IPs/CIDR and logical AWS resource
- Including other Security Group and Itself 
- Not attached to instances or subnet, they actually attached to specific Elastic Network Interface (ENI).

 **(SG) Logical References**
 - Ex: Reference the web security group within the application security group
 - So it reference anything which has this security group
 ![[Pasted image 20260409211739.png]]
**(SG) Self Reference**
- Giúp các server cùng nhóm (cùng SG) giao tiếp dễ dàng với nhau.
- Không cần quan tâm IP thay đổi bao nhiêu lần.
- Khi scale up/down (thêm bớt server), rule vẫn tự động đúng.
- Rất hay dùng trong kiến trúc multi-tier: Web tier ↔ App tier, App servers với nhau, Database replication, clustering, v.v.
![[Pasted image 20260409212956.png]]
