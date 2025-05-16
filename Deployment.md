
# **[OCI Private DNS deployment guide](#)**

## **Overview**
This guide provides step-by-step instructions for configuring and deploying OCI Private DNS on top of existing Hub & Spoke architecture. While it uses the [Hub A model](https://github.com/oci-landing-zones/oci-landing-zone-operating-entities/blob/master/addons/oci-hub-models/hub_a/readme.md) as a reference, the process is applicable to all [Hub models](https://github.com/oci-landing-zones/oci-landing-zone-operating-entities/blob/master/addons/oci-hub-models/readme.md).<br>
The components highlighted in the architecture diagram below will be implemented as part of this process.

<img src="images/dns-arch.png" width="900" height="value">


## **Configuration and deployment**

**Step 1**. Deploy [OCI Open Landing Zone](https://github.com/oci-landing-zones/oci-landing-zone-operating-entities/blob/master/blueprints/one-oe/runtime/one-stack/readme.md). 

**Step 2**. Update the Network Configuration json to include the below presented objects, or you can download and use the **Private DNS Network json template** for the Hub A model (light version - without OCI Network Firewalls).

**Private DNS Network json template** consist of the following four additional objects, which are added into Network Configuration:

- **1st object**: Network Security Group (NSG) configuration in the **Hub VCN**, attached to the Hub DNS Listener. This configuration enables the necessary Ingress and Egress DNS traffic flow with Spoke DNS Forwarders.


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

Note: full configuration of the NSGs are available in the **Private DNS Network json template**

- **2nd object**: Configuration of Private views, Forwarder and Listener in the **Hub VCN**.<br>
  The OCIDs for the Hub, Prod, and PreProd private views should be obtained from the OCI Console or CLI and specified as values for `"existing_view_id"`.<br>

                        "dns_resolver": {
                            "display_name": "vcn-fra-lzp-hub",
                            "attached_views": {
                            "DNS-HUB": {
                                "existing_view_id": "HUB PRIVATE VIEW OCID, e.g. ocid1.dnsview.oc1.eu-frankfurt-1.aaaaaaaaa4xr3d..." 
                                },
                            "DNS-P-PROJECTS": {
                                "existing_view_id": "PROD PRIVATE VIEW OCID, e.g. ocid1.dnsview.oc1.eu-frankfurt-1.aaaaaaaap4jtzm..." 
                                },
                            "DNS-PP-PROJECTS": {
                                "existing_view_id": "PREPROD PRIVATE VIEW OCID, e.g. ocid1.dnsview.oc1.eu-frankfurt-1.aaaaaaaakaxf5m..." 
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

  An OCID for the private view in OCI console can be found in **Networking -> DNS management -> Private views -> Private view information**.

- **3rd object**: NSG configuration in **Spoke VCN**, attached to the Prod DNS Forwarder. This enables the required Ingress and Egress DNS traffic flow to the Hub DNS Listener.

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


- **4th object**: DNS resolver configuration for Forwarder and Forwarding Rules in **Spoke VCN**. 

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

The same configuration applies to both Prod and PreProd environments, with the only difference being that each Spoke DNS Forwarder requires a unique IP address.<br>
Note: Ensure all required values, such as IP addresses, are adjusted to match your specific deployment.

&nbsp;

**Step 3**. Run the stack with the updated Network Configuration, which now includes the Private DNS objects.


&nbsp; 

#### License
Copyright (c) 2025 Oracle and/or its affiliates.

Licensed under the Universal Permissive License (UPL), Version 1.0.

See [LICENSE](/LICENSE.txt) for more details.
