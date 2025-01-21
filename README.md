# OCI Private DNS configuration for Hub & Spoke

&nbsp; 

### Overview
This addon is designed to provide central managment of the OCI private DNS service from the Hub VCN and by central Network team.

### OCI Private DNS resources

| Resource name | Description |
| - | - |
| VCN Resolver | A VCN dedicated private DNS resolver contains the configuration that serves responses to DNS queries within the VCN. The resolver listens on 169.254.169.254 by default. |
| Private Zones | Private zones contain DNS data only accessible from within a VCN, such as private IP addresses. |
| Private Views | A private DNS view is a collection of private zones. |
| Resolver Endpoints | There are two types of endpoints:<br>• Listening endpoint: allows the DNS Resolver to answer DNS queries coming from outside the VCN such as on-prem systems and other resolvers.<br>• Forwarding endpoint: allows the DNS resolver to query a remote DNS as defined in the Forwarding rules. |



&nbsp;


OCI Private DNS configuration in one region
<img src="images/one-region.png" width="900" height="value">



OCI Private DNS configuration two connected regions
<img src="images/two-regions.png" width="900" height="value">

