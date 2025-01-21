# OCI Private DNS configuration for Hub & Spoke

&nbsp; 

### Overview
This addon is designed to provide central management of the OCI private DNS service from the Hub VCN and by central Network team.
This add-on is designed to allow the OCI Private DNS service to be managed centrally from the VCN of the hub, by a central team.

&nbsp;

### OCI Private DNS resources

| Resource name | Description |
| - | - |
| VCN Resolver | A VCN dedicated private DNS resolver contains the configuration that serves responses to DNS queries within the VCN. The resolver listens on 169.254.169.254 by default. |
| Private Zones | Private zones contain DNS data only accessible from within a VCN, such as private IP addresses. |
| Private Views | A private DNS view is a collection of private zones, and these are:<br>• **Default Private View** - a dedicated/default view for VCN Resolver.<br>• **Associated Private Views** - the private views from other VCNs, added into VCN Resolver. |
| Resolver Endpoints | There are two types of endpoints:<br>• **Listening endpoint** - allows the DNS Resolver to answer DNS queries coming from outside the VCN such as on-prem systems and other resolvers.<br>• **Forwarding endpoint** - allows the DNS resolver to query a remote DNS as defined in the Forwarding rules. |
| Forwarding Rules | Rules are used to answer queries that aren't answered by a resolver's views and the queries that match the rule condition will be handled by the rule. If no rules match, the query will be resolved from internet DNS. |

&nbsp;

### VCN Resolver order 
DNS responses by VCN Resolver are processed based on the below presented order:
&nbsp;

<img src="images/resolver-order.png" width="600" height="value">

&nbsp;

### One Region: Private DNS configuration view
&nbsp;
Configuration details:
  - Hub VCN consist of the following resources and components: 
    - Forwarding (**hub_dns_forwarder**) and Listening (**hub_dns_listener**) endpoints.
    - VCN Resolver has associated private views for Hub and Spokes VCNs, so it contains all DNS data/records of all three VCNs and can resolve any FQDN inside Hub and Spoke architecture.
  - Spoke VCNs have Forwarding (**p_dns_forwarder** and **pp_dns_forwarder** accordingly) endpoints, and conditional forwarding rules, which forwards **oraclevcn.com** domain specific queries to the **hub_dns_listener**.

<img src="images/one-region.png" width="900" height="value">



### Two/Multi Regions: Private DNS configuration view
&nbsp;
<img src="images/two-regions.png" width="900" height="value">

