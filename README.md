
# **[OCI Private DNS configuration](#)**
## **An OCI Open LZ [Addon](#) to Tailor and Optimize Your DNS configuration**

&nbsp; 

**Table of Contents**

- [1. Overview](#Overview)</br>
- [2. OCI Private DNS resources](#OCI-Private-DNS-resources)</br>
- [3. VCN DNS Resolver query processing order](#VCN-DNS-Resolver-query-processing-order)</br>
- [4. Single-Region: Private DNS configuration view](#1-Single-Region-Private-DNS-configuration-view)</br>
  - [4.1. Single-Region: Private DNS animation](#Single-Region-Private-DNS-animation)</br>
- [5. Multi-Region: Private DNS configuration view](#2-Multi-Region-Private-DNS-configuration-view)</br>
  - [5.1. Multi-Region: Private DNS animation](#Multi-Region-Private-DNS-animation)

&nbsp;

## **Overview**
This configuration enables centralized management of private DNS zones within a Hub VCN, controlled by a central network team. All DNS configurations, including zone creation and record updates, are streamlined through a single management plane. This ensures consistency, simplifies administration and eliminates the complexity of managing DNS across multiple environments and OCI regions.

This document provides configuration views for the following use cases:
- Single-Region Deployment: Private DNS configuration view and animation for a Hub and Spoke architecture within one OCI region.
- Multi-Region Deployment: Private DNS configuration view and animation for a setup where Hub & Spoke VCNs are interconnected via Remote Peering Connection (RPC).


&nbsp;

### OCI Private DNS resources

| Resource | Description |
| - | - |
| VCN Resolver | A VCN dedicated private DNS resolver contains the configuration that serves responses to DNS queries within the VCN. The resolver listens on 169.254.169.254 by default. |
| Private Zones | Private zones contain DNS data only accessible from within a VCN, such as private IP addresses. |
| Private Views | A private DNS view is a collection of private zones, and these are:<br>• **Default Private View** - a dedicated/default view for VCN Resolver.<br>• **Associated Private Views** - the private views from other VCNs, added into VCN Resolver. |
| Resolver Endpoints | There are two types of endpoints:<br>• **Listening endpoint** - allows the DNS Resolver to answer DNS queries coming from outside the VCN, such as on-prem systems and other resolvers.<br>• **Forwarding endpoint** - allows the DNS resolver to query a remote DNS as defined in the Forwarding rules. |
| Forwarding Rules | Rules are used to answer queries that aren't answered by a resolver's views and the queries that match the rule condition will be handled by the rule. If no rules match, the query will be resolved from Internet DNS. |

&nbsp;

### VCN DNS Resolver query processing order
The VCN DNS resolver processes queries in the priority order presented below. It attempts to resolve each query by sequentially checking the configured options. After an answer is provided, no further items are evaluated, even if the answer is negative.

&nbsp;

<img src="images/resolver-order.png" width="500" height="value">

&nbsp;

## **1. Single-Region: Private DNS configuration view**
&nbsp;
Configuration details:
  - Hub VCN consists of the following resources and components: 
    - Forwarding (**hub_dns_forwarder**) and Listening (**hub_dns_listener**) endpoints.
    - Hub VCN Resolver has Associated private views for Hub and Spokes VCNs. As a result, it contains all DNS data/records for the three VCNs and can resolve any FQDN within the Hub and Spoke architecture.
  - Spoke VCN Resolvers include
    - Forwarding endpoints (**p_dns_forwarder** and **pp_dns_forwarder** respectively).
    - Conditional forwarding rules, that directs queries for **oraclevcn.com**, **oraclecloud.com** and **oci.customer-oci.com** domains to the **hub_dns_listener** for resolution.

<img src="images/one-region.png" width="900" height="value">

&nbsp;

#### DNS configuration with On-Premise connectivity.
In addition to the above configuration, the following setup includes forwarding rules for On-Premises DNS zones in both the Spoke VCNs and the Hub VCN. These rules direct queries to a Network Load Balancer (NLB), which serves as the target for forwarding. The On-Premises DNS servers are configured as backends for the DNS NLB.

<img src="images/onprem.png" width="900" height="value">


&nbsp;

&nbsp;

## Single-Region: Private DNS animation

These animations illustrate the DNS query and response within Hub & Spoke, and covers the following scenarios:

#### Scenario 1: DNS resolution within the same Spoke VCN
- The **web01-p.ssnpweb.vcnprod.oraclevcn.com** instance in prod Spoke VCN performs a nslookup to retrieve an IP address of the **db01-p.ssnpdb.vcnprod.oraclevcn.com** database instance, which is located in the same Spoke VCN but in a different subnet.
- Prod VCN resolver evaluates the query based on the [VCN DNS Resolver query processing order](#VCN-DNS-Resolver-query-processing-order) as follows:<br>
   •1• **Associated Private Views** - Since no Private view is associated with the Prod VCN resolver, it proceeds to the next step.<br>
   •2• **Default Private View** - The default private view contains a DNS record for the database, so the resolver retrieves the record.
- The Prod VCN resolver returns the DNS response to **web01-p**.

<img src="images/withinspoke.gif" width="800" />

&nbsp;

#### Scenario 2:. Spoke to Spoke DNS resolution
- The **web01-p.ssnpweb.vcnprod.oraclevcn.com** in prod Spoke VCN performs a nslookup to retrieve the IP address of **web02-pp.ssnppweb.vcnpreprod.oraclevcn.com**, located in the preprod Spoke VCN.
- Prod VCN resolver evaluates the query based on the [VCN DNS Resolver query processing order](#VCN-DNS-Resolver-query-processing-order) as follows:<br>
   •1• **Associated Private Views** - Since no Private view is associated with the Prod VCN resolver, it moves to the next option.<br>
   •2• **Default Private View** – This view contains only DNS records specific to the Prod VCN. Since the resolver does not have a record for **web02-pp**, it proceeds to the next step.<br>
   •3• **Forwarding Rules** – The resolver identifies a forwarding rule for **oraclevcn.com**, directing the query to the **hub_dns_listener** through the **p_dns_forwarder** in the Prod Spoke VCN.
- The Hub VCN Resolver has all DNS data/records from the **Associated Private views**, processes the query and returns the DNS response to **web01-p** in the Prod Spoke VCN.

<img src="images/spoke2spoke.gif" width="800" />

&nbsp;

## **2. Multi-Region: Private DNS configuration view**
Configuration details:
  - Each Hub VCN in a given region consists of the following resources and components:
    - Forwarding (**hub_dns_forwarder-(1/2)**) and Listening (**hub_dns_listener-(1/2)**) endpoints.
    - The Hub VCN Resolver is associated with Private views for both the Hub and Spoke VCNs, allowing it to store and resolve all DNS records across the three VCNs within the Hub and Spoke architecture.
    - Conditional Forwarding Rules in the Hub VCN resolvers ensure that region-specific DNS zones are forwarded to the appropriate region.
  - Spoke VCN Resolvers include:
    - Forwarding Endpoints.
    - Conditional forwarding rules that direct queries for **oraclevcn.com**, **oraclecloud.com** and **oci.customer-oci.com** domains to the respective  **hub_dns_listener-(1/2)** for resolution.
  
&nbsp;
<img src="images/multi-region.png" width="900" height="value">

&nbsp;

&nbsp;

## Multi-Region: Private DNS animation
**Spoke to Spoke DNS Resolution across different regions:**

- The **web01-p.ssnpweb.vcnprodregion1.oraclevcn.com** in Spoke VCN (Region-1) performs nslookup to get an IP address of the **web02-p.ssnpweb.vcnprodregion2.oraclevcn.com** located in Region-2 inside Prod VCN.
- Prod VCN resolver evaluates the query based on the [VCN DNS Resolver query processing order](#VCN-DNS-Resolver-query-processing-order) as follows:<br>
    ⓵ **Associated Private Views** - Since no private view is associated with the Prod VCN Resolver, it proceeds to the next step.<br>
    ⓶ **Default Private View** - This view contains only DNS records specific to the local VCN. Since the resolver does not have a record for **web02-p** in Region-2, it moves to the next step.<br>
    ③ **Forwarding Rules** – A forwarding rule for **oraclevcn.com** is found, directing the DNS query to **hub_dns_listener-1** through **p_dns_forwarder-1** endpoint, and subsequently to the Hub VCN Resolver in Region-1. 
- A Hub VCN Resolver in Region-1 evaluates the query based on the [VCN DNS Resolver query processing order](#VCN-DNS-Resolver-query-processing-order) as follows:<br>
    ⓵ **Associated Private Views** - These views do not contain records for the **vcnprodregion2.oraclevcn.com** subdomain, so it moves to the next step.<br>
    ⓶ **Default Private View** - This view contains only region-specific VCN records, so it proceeds to the next step.<br>
    ⓷ **Forwarding Rules** - A rule for **vcnprodregion2.oraclevcn.com** is found, forwarding the DNS query to **hub_dns_listener-2** via **hub_dns_forwarder-1**. Which then sends it to the Hub VCN Resolver in Region-2. 
- Hub VCN Resolver in Region-2 has the required DNS records from all the **Associated Private views**, and returns the IP address of **web02-p** as the DNS response.

<img src="images/multi-region.gif" width="1000" />

&nbsp;

> [!NOTE]
>OCI services such as Autonomous Databases, Oracle Analytics, Streaming, Object Storage, and others support Private Endpoints. These services have automatically generated, publicly resolvable DNS records within Oracle-owned public domains, such as:
>- oraclecloud.com
>- oci.customer-oci.com
>
> When a Private Endpoint is created for one of these services, an additional DNS record is automatically added to the Default Private View of the specific VCN where the endpoint's subnet resides. This allows the private IP address of the endpoint to be resolved within the VCN.<br>
> For simplicity, these domains are not explicitly depicted in Forwarding Rules within the animations but are included in the configuration views.

&nbsp;

#### Summary
This Private DNS configuration in a Hub and Spoke architecture ensures that all VCN-internal and Internet-specific DNS queries are handled by their respective VCN Resolvers. Meanwhile, Oracle-specific domains, On-Premises zones, and custom-created domains are handled and managed by the Hub VCN Resolver. This approach optimizes DNS management and supports a consistent, scalable architecture across OCI environments, in both single-region and multi-region deployments.



&nbsp; 

# License

Copyright (c) 2025 Oracle and/or its affiliates.

Licensed under the Universal Permissive License (UPL), Version 1.0.

See [LICENSE](/LICENSE.txt) for more details.
