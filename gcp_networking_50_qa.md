# Network & Hybrid Connectivity: 50 Practice Q&A for GCP Customer Engineer

This document contains 50 rapid-fire questions and answers focused strictly on Networking and Hybrid Connectivity, the most critical domain for a Platform CE.

## Section 1: VPC Architecture & Foundations (Q1-Q10)
**Q1: What is the defining architectural difference between a GCP VPC and an AWS VPC?**
A: A GCP VPC is a **global** resource spanning all regions, while an AWS VPC is restricted to a single region. This means GCP subnets in Tokyo and London can communicate over internal Google network routing without peering.

**Q2: What is a Subnet in GCP?**
A: A subnet is a **regional** resource. It is an IP address range carved out of a VPC that exists entirely within a single region (but spans all zones in that region).

**Q3: Can subnets in the same VPC have overlapping IP ranges?**
A: No. Subnet IP ranges across all regions inside a single VPC must be globally unique.

**Q4: A customer wants centralized control over network security, while allowing diverse project teams to spin up VMs. What network design is best?**
A: **Shared VPC**. A central "Host Project" manages the VPC, Subnets, and Firewalls, while "Service Projects" simply attach their VMs to those subnets.

**Q5: Two different companies acquired each other. They each have their own GCP Organization but need their databases to communicate on internal IPs. Solution?**
A: **VPC Network Peering**. It connects two distinct VPCs (even across different organizations) natively without VPN overhead.

**Q6: Is VPC Peering transitive? (If A connects to B, and B connects to C, can A talk to C?)**
A: No. VPC peering is strictly non-transitive. A and C must explicitly peer with each other.

**Q7: Can two peered VPCs have overlapping IP ranges?**
A: No. The IP ranges must be completely disjoint.

**Q8: What is Cloud NAT used for?**
A: Network Address Translation. It allows VMs without public IP addresses to securely reach out to the internet (e.g., to download OS updates) without allowing incoming internet traffic.

**Q9: Does Cloud NAT provision a virtual machine in the background like a traditional NAT Gateway?**
A: No, it is entirely software-defined and distributed by the Andromeda SDN, meaning it cannot become a bottleneck or a single point of failure.

**Q10: Can a customer bring their own public IP addresses (BYOIP) to Google Cloud?**
A: Yes. Google Cloud supports BYOIP, allowing customers to provision and use their own registered IPv4 address prefixes for Google Cloud resources.

## Section 2: Hybrid Connectivity (VPN & Interconnect) (Q11-Q20)
**Q11: When should a customer choose Cloud VPN over Cloud Interconnect?**
A: When they need encrypted traffic (IPsec) fast, over the public internet, and have lower bandwidth requirements (up to 3 Gbps per tunnel). 

**Q12: What is the SLA of an HA VPN?**
A: **99.99%** SLA, provided you configure two active tunnels across two separate interfaces on the GCP side to dual customer gateway interfaces.

**Q13: What protocol does Cloud Router use to exchange routes dynamically?**
A: **BGP** (Border Gateway Protocol). This ensures that if the customer adds a subnet on-prem, GCP automatically learns the route without manual static updates.

**Q14: A customer requires 10 Gbps and absolutely cannot route traffic over the public internet due to military compliance. Solution?**
A: **Dedicated Interconnect**. It provides a physical, private cross-connect directly from the customer's router to Google's edge at a Colocation facility.

**Q15: What if the customer cannot physical reach a Google Colocation facility?**
A: They should use **Partner Interconnect**, leveraging a supported third-party service provider (ISP) to provision the private circuit.

**Q16: Can Cloud Interconnect traffic be encrypted?**
A: By default, Interconnect is private but *not* automatically encrypted via IPsec. For strict compliance, use **HA VPN over Cloud Interconnect** (IPsec inside the physical pipe) or MACsec for Dedicated Interconnect.

**Q17: What is the required architectural setup for a 99.99% SLA on Dedicated Interconnect?**
A: Four (4) dedicated connections split across Two (2) metropolitan areas (e.g., two connections in Chicago, two in Ashburn), terminating on dual redundant routers on-prem.

**Q18: What is the Cloud VPN bandwidth limit per tunnel?**
A: Typically 1.5 Gbps to 3 Gbps depending on the routing configuration, but multiple ECMP (Equal-Cost Multi-Path) tunnels can be grouped for higher aggregate bandwidth.

**Q19: Can a customer use static routing with HA VPN?**
A: No. HA VPN strictly requires BGP dynamic routing via Cloud Router.

**Q20: What GCP tool provides centralized management of hybrid connectivity, treating GCP, AWS, and on-prem networks as a single logical network?**
A: **Network Connectivity Center (NCC)**.

## Section 3: Load Balancing & Edge Networking (Q21-Q30)
**Q21: How many IP addresses does a Global External HTTP(S) Load Balancer use?**
A: One single **Anycast IP address** worldwide.

**Q22: A customer in Tokyo reaches the Global Load Balancer IP. Where does their traffic enter Google's network?**
A: At the closest Google Global Network edge point of presence (PoP) to Tokyo. It travels the rest of the way on Google's private fiber, not the public internet.

**Q23: A gaming customer needs to balance raw UDP traffic (not HTTP) globally. What Load Balancer should they choose?**
A: **Global TCP/UDP Proxy Load Balancer** OR if it’s regional, the **External Passthrough Network Load Balancer**. Global HTTP(S) is only for L7 traffic.

**Q24: A customer wants to cache static content (images/CSS) at Google's edge. What service integrates natively with the Global L7 Load Balancer?**
A: **Cloud CDN**.

**Q25: What is Cloud Armor?**
A: GCP's Web Application Firewall (WAF) and DDoS protection service, integrating directly with the Global HTTP(S) Load Balancer at the edge.

**Q26: Can you place a Load Balancer *inside* a VPC for internal microservice communication?**
A: Yes, use the **Internal HTTP(S) Load Balancer** (L7) or **Internal TCP/UDP Load Balancer** (L4). They use private RFC 1918 IPs.

**Q27: A customer is using GKE and wants to balance traffic directly to specific containers (Pods) rather than the underlying VMs. What must they enable?**
A: **VPC-native clusters** (Alias IPs) and Network Endpoint Groups (**NEGs**).

**Q28: What is a Serverless Network Endpoint Group (Serverless NEG)?**
A: It allows the Global Load Balancer to route traffic directly to serverless backends like *Cloud Run, App Engine, or Cloud Functions*.

**Q29: What is the difference between Premium and Standard Tier networking?**
A: **Premium Tier** routes traffic onto Google's global fiber backbone as quickly as possible. **Standard Tier** routes traffic mostly over the public internet, entering Google's network only at the region where the workload resides. (Standard Tier is cheaper but unpredictable).

**Q30: Can a Global Load Balancer fail over traffic from a region that goes down (e.g., us-east1) to a healthy region (e.g., us-west1) automatically?**
A: Yes. Because it's a global service, it natively performs health checks and seamlessly redirects traffic to the closest healthy region without manual DNS intervention.

## Section 4: Private Services & API Access (Q31-Q40)
**Q31: A customer has a VM with no public IP. They want to upload a file to Cloud Storage. How can they do this securely?**
A: Enable **Private Google Access (PGA)** on the subnet. It directs API traffic over Google's internal network.

**Q32: Is Private Google Access a subnet-level or VPC-level setting?**
A: It is a **Subnet-level** configuration.

**Q33: How does Private Service Connect (PSC) differ from Private Google Access?**
A: PSC allows you to access Google services (or published 3rd-party SaaS services) through a dedicated, local internal IP address created directly inside your VPC. PGA routes to the public IPs of Google APIs (like 199.36.153.8/30) via custom routing rules.

**Q34: A customer built a microservice in VPC-A and wants to sell it to a partner in VPC-B without exposing it to the internet or setting up full VPC peering. Solution?**
A: The customer can publish their service using **Private Service Connect (PSC)**. The partner creates an endpoint in VPC-B to privately consume it.

**Q35: A Cloud Run instance needs to query a highly secure Cloud SQL database running on private internal IPs. How?**
A: Use a **Serverless VPC Access connector** (or the newer Direct VPC Egress functionality). It creates a secure tunnel between the serverless environment and the VPC.

**Q36: Can on-premises servers (connected via Interconnect) utilize Private Google Access?**
A: Yes. This is called **Private Google Access for on-premises hosts**. You configure local DNS to resolve `restricted.googleapis.com` or `private.googleapis.com` and route traffic over the Interconnect.

**Q37: What is the DNS zone name used to access Google APIs privately without internet exposure?**
A: Typically `private.googleapis.com` (doesn't support VPC Service Controls) or `restricted.googleapis.com` (enforces VPC Service Controls).

**Q38: Can a customer access an internal Load Balancer from an on-premises network?**
A: Yes, as long as the on-premises network is connected via Cloud VPN or Interconnect and the routing is configured correctly.

**Q39: What is the primary use case of the "restricted.googleapis.com" domain?**
A: Accessing Google services that are supported by **VPC Service Controls**, ensuring data cannot leak to non-supported public APIs.

**Q40: Do internal Load Balancers require a proxy VM?**
A: No, the Internal TCP/UDP Load Balancer is entirely software-defined in the Andromeda hypervisor layer. The Internal HTTP(S) LB uses Envoy proxies managed continuously by Google under the hood.

## Section 5: Security, DNS, and Observability (Q41-Q50)
**Q41: What is Cloud DNS?**
A: A scalable, reliable, and managed authoritative Domain Name System (DNS) service with a 100% uptime SLA. It supports both public and private zones.

**Q42: What is DNS Peering in GCP?**
A: It allows a VPC to query private DNS records from a different peered VPC, or to seamlessly resolve internal `.local` domain names residing in an on-premises Active Directory via Interconnect.

**Q43: What is the difference between standard GCP Firewalls and Hierarchical Firewall Policies?**
A: Standard firewalls apply to a single VPC network. Hierarchical Firewalls apply centrally at the **Organization** or **Folder** level, superseding local project firewalls (e.g., allowing InfoSec to block SSH port 22 globally across the entire company).

**Q44: Are GCP Firewalls stateful or stateless?**
A: They are **stateful**. If you allow an incoming request on port 443, the firewall automatically permits the return reply traffic without needing an explicit egress rule.

**Q45: What is the default firewall behavior in a custom VPC?**
A: Implied **Allow Egress** (instances can talk to the internet) and Implied **Deny Ingress** (all incoming traffic is blocked).

**Q46: A customer is terrified of a rogue developer downloading petabytes of customer data from BigQuery to a personal AWS account. What specific network security tool prevents this data exfiltration?**
A: **VPC Service Controls (VPC SC)**.

**Q47: How does VPC SC work?**
A: It creates a logical "perimeter" around Google-managed service APIs (like BigQuery and GCS). It inspects the context of the API call—if the request does not originate from a trusted IP address or authenticated identity within the perimeter, it drops the request instantly.

**Q48: Can a perimeter span multiple projects?**
A: Yes, this is exactly what a **VPC Service Controls Service Perimeter** does.

**Q49: The operations team wants to see a topological map of all VMs, Interconnects, and Load Balancers and run connectivity tests between them. What product should they use?**
A: **Network Intelligence Center**.

**Q50: A specific network connection between a VM and an on-prem server is dropping over Interconnect. What specific tool in Network Intelligence Center can verify if a GCP firewall rule is causing the drop?**
A: **Connectivity Tests**. It mathematically analyzes the network topology and firewall rules (without sending actual packets) to prove whether a path is valid or blocked.
