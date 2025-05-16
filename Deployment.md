
# **[OCI Private DNS deployment guide](#)**
## **An OCI Open LZ [Addon](#) to Tailor and Optimize Your DNS configuration**

&nbsp; 

**Table of Contents**

- [Overview](#Overview)</br>

&nbsp;

## **Overview**
This guide provides step-by-step instructions for configuring and deploying Private DNS on top of existing Hub & Spoke architecture. While it uses the [Hub A model](https://github.com/oci-landing-zones/oci-landing-zone-operating-entities/blob/master/addons/oci-hub-models/hub_a/readme.md) as a reference, the process is applicable to all [Hub models](https://github.com/oci-landing-zones/oci-landing-zone-operating-entities/blob/master/addons/oci-hub-models/readme.md). 
The components highlighted in the architecture diagram below will be implemented as part of this process.

<img src="images/dns-arch.png" width="900" height="value">


## **Steps**

**Step 1**. Deploy [OCI Open Landing Zone One-OE](https://github.com/oci-landing-zones/oci-landing-zone-operating-entities/tree/master/blueprints/one-oe).

**Step 2**. Update the Network Configuration JSON to include the below presented objects (note: full configuration of the NSGs are available in the template). Alternatively, you can download and use the **Private DNS Network json template**  for the Hub A model (light version - without OCI Network Firewalls).

**Private DNS Network json template** consist of the following additional objects, which are added into Network Configuration:

- **1st object for Hub**: NSG (Network Security Group) configuration, which then is attached to the Hub DNS Listener. This allows required Ingress and Egress DNS traffic flow with Spoke DNS Forwarders.


                            "NSG-FRA-LZP-HUB-DNS-KEY": {
                                "display_name": "nsg-fra-lzp-hub-dns",
                                "egress_rules": {
                                    "egress_dns_prod_udp": {
                                        "description": "Egress to Prod DNS endpoint: UDP, Stateless",
                                        "src_port_max": 53,
                                        "src_port_min": 53,
                                        "dst": "10.0.67.10/32",
                                        "dst_type": "CIDR_BLOCK",
                                        "protocol": "UDP",
                                        "stateless": true
                                        ...
                            }


- **2nd object for Hub**: Configuration of Associated Private views, Forwarder and Listener in the Hub VCN. 
  OCIDs of the DNS views should be obtained from the OCI console or CLI, after the VCNs have been deployed in Step 1. An OCID for the DNS view in OCI console can be found in **Networking -> DNS management -> Private views -> Private view information**.

                        "dns_resolver": {
                            "display_name": "vcn-fra-lzp-hub",
                            "attached_views": {
                            "DNS-HUB": {
                                "existing_view_id": "ocid1.dnsview.oc1..." 
                                },
                            "DNS-P-PROJECTS": {
                                "existing_view_id": "ocid1.dnsview.oc1..." 
                                },
                            "DNS-PP-PROJECTS": {
                                "existing_view_id": "ocid1.dnsview.oc1..." 
                                }
                            },
                            "resolver_endpoints": {
                                "RESOLVER_HUB_ENDPOINT_FORWARDER_1": {
                                    "enpoint_type"      : "VNIC",
                                    "is_forwarding"     : "true",
                                    "is_listening"      : "false",
                                    "forwarding_address": "10.0.5.10",
                                    "name"              : "dns_forwarder_fra_hub",
                                    "subnet"            : "SN-FRA-LZP-HUB-DNS",
                                    "nsg"               : null
                                    },
                                "RESOLVER_HUB_ENDPOINT_LISTENER_1" : {
                                    "enpoint_type"     : "VNIC",
                                    "is_forwarding"    : "false",
                                    "is_listening"     : "true",
                                    "listening_address": "10.0.5.11",
                                    "name"             : "dns_listener_fra_hub",
                                    "subnet"           : "SN-FRA-LZP-HUB-DNS",
                                    "nsg"              : ["NSG-FRA-LZP-HUB-DNS-KEY"]
                                }
                            }
                        }


- **1st object for Spoke**: NSG configuration, which then is attached to the Prod DNS Forwarder. This allows required Ingress and Egress traffic for DNS communication with Hub DNS Listener.

                            "NSG-LZP-P-PROJECTS-DNS-KEY": {
                                "display_name": "nsg-lzp-p-projects-dns",
                                "egress_rules": {
                                    "egress_dns_udp": {
                                        "description": "Egress to Hub DNS endpoint: UDP, Stateless",
                                        "dst_port_max": 53,
                                        "dst_port_min": 53,
                                        "dst": "10.0.5.11/32",
                                        "dst_type": "CIDR_BLOCK",
                                        "protocol": "UDP",
                                        "stateless": true
                                        ...
                            }


- **2nd object for Spoke**: DNS resolver configuration for Forwarder and Forwarding Rules. 

                        "dns_resolver": {
                            "display_name": "vcn-fra-lzp-p-projects",
                            "attached_views": {},
                            "rules" : [
                                {
                                "action"                : "FORWARD",
                                "destination_address"   : ["10.0.5.11"],
                                "source_endpoint_name"  : "RESOLVER_P_ENDPOINT_FORWARDER_1",
                                "qname_cover_conditions": ["oraclevcn.com"]
                                },
                                {
                                "action"                : "FORWARD",
                                "destination_address"   : ["10.0.5.11"],
                                "source_endpoint_name"  : "RESOLVER_P_ENDPOINT_FORWARDER_1",
                                "qname_cover_conditions": ["oraclecloud.com"]
                                },
                                {
                                "action"                : "FORWARD",
                                "destination_address"   : ["10.0.5.11"],
                                "source_endpoint_name"  : "RESOLVER_P_ENDPOINT_FORWARDER_1",
                                "qname_cover_conditions": ["oci.customer-oci.com"]
                                }
                              ],
                            "resolver_endpoints": {
                                "RESOLVER_P_ENDPOINT_FORWARDER_1": {
                                    "enpoint_type"      : "VNIC",
                                    "is_forwarding"     : "true",
                                    "is_listening"      : "false",
                                    "forwarding_address": "10.0.67.10",
                                    "name"              : "dns_forwarder_fra_p",
                                    "subnet"            : "SSN-FRA-LZP-P-INFRA",
                                    "nsg"               : ["NSG-LZP-P-PROJECTS-DNS-KEY"]
                                }
                            }
                        }

The same configuration applies to both Prod and Pre-Prod environments, with the only difference being that each Spoke DNS Forwarder requires a unique IP address. Ensure all required values, such as IP addresses, are adjusted to match your specific deployment.

**Step 3**. Run the stack with the updated Network Configuration, which now includes the Private DNS objects.


&nbsp; 

#### License
Copyright (c) 2025 Oracle and/or its affiliates.

Licensed under the Universal Permissive License (UPL), Version 1.0.

See [LICENSE](/LICENSE.txt) for more details.
