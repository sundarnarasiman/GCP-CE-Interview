# Security & Compliance: 50 Practice Q&A for GCP Customer Engineer

This document contains 50 rapid-fire questions and answers focused strictly on Security, Compliance, IAM, Data Protection, and Zero Trust architectures in Google Cloud.

## Section 1: Resource Hierarchy & IAM (Q1-Q10)
**Q1: What are the four levels of the Google Cloud Resource Hierarchy?**
A: Organization -> Folders -> Projects -> Resources (like VMs or Buckets).

**Q2: How does policy inheritance work in GCP?**
A: Policies are inherited **downward**. If a user is granted "Compute Admin" at the Folder level, they have Compute Admin access to every Project and VM inside that Folder. Deny policies explicitly block inherited access.

**Q3: A developer has "Storage Object Viewer" on a Project, but "Storage Object Admin" on a specific bucket inside that project. What is their effective access to that bucket?**
A: **Storage Object Admin**. IAM policies are a union of parent and resource-level bindings; the less restrictive policy always applies unless explicitly denied.

**Q4: What is the principle of least privilege in GCP IAM?**
A: Never use Primitive Roles (Owner, Editor, Viewer). Always use **Predefined Roles** (e.g., Compute Instance Admin) or **Custom Roles** tailored to the exact permissions needed.

**Q5: What is a Service Account?**
A: An identity used by an application or a VM, rather than an individual end-user, to make programmatic API calls to Google services securely.

**Q6: A VM needs to read from a private Cloud Storage bucket. Is it better to put a JSON Service Account key on the VM or attach the Service Account to the VM explicitly?**
A: Always **attach the Service Account directly to the VM**. Storing long-lived JSON keys on disks is a massive security risk. GCP automatically rotates the temporary credentials for attached service accounts under the hood.

**Q7: A third-party SaaS tool needs temporary access to your GCP environment. Do you create a Service Account key for them?**
A: No. Use **Workload Identity Federation**. It allows external identities (like AWS IAM or Azure AD) to impersonate a GCP Service Account without creating or managing long-lived static JSON keys.

**Q8: What is the risk associated with "Service Account User" role?**
A: It allows a user to indirectly gain higher privileges. If a user is given "Compute Admin" and "Service Account User", they can create a VM running a Service Account with "Editor" permissions, SSH into the VM, and essentially become an "Editor" of the project.

**Q9: How do you prevent a specific IAM permission (like "storage.buckets.delete") from being granted anywhere in the organization, even by Project Owners?**
A: Use IAM **Deny Policies**. Deny policies override any allow policy, regardless of the resource hierarchy.

**Q10: An enterprise uses Active Directory on-premises. How do they synchronize those employee identities to Google Cloud?**
A: Using **Google Cloud Directory Sync (GCDS)** to sync users and groups to Cloud Identity (or Google Workspace), which acts as the Identity Provider (IdP) for GCP IAM.

## Section 2: Data Protection & Encryption (Q11-Q20)
**Q11: How is data encrypted at rest in Google Cloud by default?**
A: Every single piece of data stored in GCP is **encrypted at rest by default** using AES-256 without any customer action required. Google manages the cryptographic keys.

**Q12: A highly regulated customer requires cryptographic assurance that Google cannot access their data without their permission. They want to manage the key rotation themselves. What feature?**
A: **Customer-Managed Encryption Keys (CMEK)** via Cloud Key Management Service (KMS).

**Q13: An even stricter defense contractor refuses to store the root key in the cloud at all. They want to hold the key on their on-premises HSM. What feature?**
A: **Customer-Supplied Encryption Keys (CSEK)** or External Key Management (EKM). They inject the key dynamically during API calls; Google does not persist it.

**Q14: Where does Cloud KMS store the keys automatically to achieve a higher FIPS 140-2 Level 3 compliance?**
A: In **Cloud HSM** (Hardware Security Modules) clusters.

**Q15: A health insurance customer accidentally uploaded 1 million SSNs in a messy CSV file to a Cloud Storage bucket. What automated tool can scan for and mask PII/PHI in GCP data?**
A: **Cloud Data Loss Prevention (DLP)** or Sensitive Data Protection.

**Q16: How does Cloud DLP protect data before it enters a data warehouse?**
A: It can be integrated into a Cloud Dataflow pipeline to instantly **tokenize** or **mask** the sensitive columns (like credit cards) in streaming data *before* it gets written to BigQuery.

**Q17: Where should a developer store API keys, database passwords, and TLS certificates securely in GCP?**
A: **Secret Manager**, which encrypts them securely and handles versioning and IAM access controls. (Never hardcode them in Git or environmental variables).

**Q18: What is Confidential Computing?**
A: Utilizing specialized hardware (AMD SEV) to ensure that data remains encrypted **in use** (in system RAM), protecting data even from the hypervisor or physical RAM attacks.

**Q19: A customer wants to prevent any developer from accidentally making a Cloud Storage bucket public. How is this enforced globally?**
A: By configuring an **Organization Policy Constraint** specifying that `storage.publicAccessPrevention` is "enforced" across the Folder or Org.

**Q20: What is the difference between an Identity (IAM policy) and a Resource Policy (Bucket Policy)?**
A: IAM policies say "User X can access Bucket Y." Bucket (or resource) policies say, "Bucket Y can be accessed by User X." GCP evaluates both for the final result.

## Section 3: NetSec & Zero Trust (BeyondCorp) (Q21-Q30)
**Q21: A malicious actor steals an employee's valid IAM credentials and tries to download a BigQuery dataset from a coffee shop Wi-Fi network. What stops them?**
A: **VPC Service Controls (VPC SC)**. It creates a defined security perimeter based on network attributes (like corporate IP ranges), blocking API access to managed services if the request originates outside the perimeter, regardless of IAM validity.

**Q22: How does an internal admin securely access an internal VM without VPN overhead for the entire company?**
A: **Identity-Aware Proxy (IAP)**. IAP handles authentication and authorization at the application layer.

**Q23: How does IAP TCP Forwarding work?**
A: It allows administrators to SSH or RDP into a private VM without assigning the VM a public IP address or setting up jump servers (bastion hosts), routing traffic securely through Google's edge proxies.

**Q24: What is Context-Aware Access?**
A: The core of Zero Trust. It grants access not just based on *who* you are, but the *context* of the request (e.g., "Are you on an enterprise device? Is it patched? Are you accessing from an approved IP?").

**Q25: A customer wants massive global DDoS protection and an OWASP Top 10 Web Application Firewall. Which GCP service?**
A: **Cloud Armor**, integrated directly with the Global HTTP(S) Load Balancer.

**Q26: What is a Cloud Armor "Adaptive Protection" rule?**
A: It uses machine learning models to analyze baseline traffic patterns to a specific web application, automatically detecting and proposing rules to block anomalous Layer 7 volumetric attacks.

**Q27: A customer runs sensitive workloads directly on Kubernetes (GKE). How do they restrict pod-to-pod communication?**
A: They enforce **Network Policies** in GKE. By default, all pods can talk to each other; Network Policies restrict traffic dynamically based on pod labels.

**Q28: How does an enterprise centrally prevent project owners from altering security rules within their own VPCs?**
A: By implementing **Hierarchical Firewall Policies** at the Folder or Organization level, which override any local VPC firewall rules.

**Q29: Can GCP firewalls block traffic based on specific URLs or Domain Names?**
A: The standard VPC firewall is a L3/L4 firewall (IPs & ports). Blocking L7 attributes (Domains/FQDNs) generally requires **Cloud Armor** (inbound), **Secure Web Proxy** (outbound), or a third-party NGFW.

**Q30: What handles certificate management for HTTPS load balancing?**
A: **Google-managed SSL certificates**, which automatically provision and renew TLS certificates for your domains at the load balancer without manual intervention.

## Section 4: Security Operations & Audit Logging (Q31-Q40)
**Q31: What is the "Single Pane of Glass" for security visibility, misconfigurations, and threat detection across an entire GCP Organization?**
A: **Security Command Center (SCC)**.

**Q32: What specific SCC tier or feature detects if a VM has suddenly started cryptomining?**
A: **Event Threat Detection** (part of the Premium tier), which analyzes VPC flow logs, DNS logs, and audit logs to find malware or cryptomining signatures.

**Q33: Can SCC scan container images for known vulnerabilities before they are deployed?**
A: Yes, via integration with the **Artifact Registry** vulnerability scanning.

**Q34: What are the two primary types of Cloud Audit Logs?**
A: **Admin Activity logs** (always on, track any API call that modifies configuration) and **Data Access logs** (off by default due to high volume; track API calls that simply read data).

**Q35: Are Cloud Audit Logs immutable?**
A: Yes. No user, not even an Organization Administrator, can delete or alter Admin Activity audit logs.

**Q36: A customer generates 5 TB of logs daily and needs to store them for 7 years for compliance. Cloud Logging is too expensive for 7 years. Solution?**
A: Create a **Log Router Sink** at the Organization level to continuously export all logs to a Cloud Storage bucket using the Archive storage class for extremely low-cost, long-term retention.

**Q37: Can Log Router intercept and route logs to a third-party SIEM like Splunk?**
A: Yes, Log Router can continuously publish logs to a **Pub/Sub topic**, which the third-party SIEM natively subscribes to and ingests in real time.

**Q38: What is Google SecOps (formerly Chronicle)?**
A: Google's cloud-native **SIEM (Security Information and Event Management)** platform, built on the core infrastructure of Google Search to ingest petabytes of security telemetry in seconds without the indexing bottlenecks of traditional SIEMs.

**Q39: What is the purpose of VPC Flow Logs?**
A: They sample the IP traffic (source IP, destination IP, ports, bytes) flowing into and out of VMs in a VPC. Critical for network forensics and troubleshooting routing.

**Q40: A regulatory body requires proof that no physical Google engineer has accessed customer data. How does GCP provide this?**
A: **Access Transparency**. It generates near real-time audit logs whenever a Google administrator interacts with customer data or configurations for an authenticated support ticket.

## Section 5: Compliance & Organization Policies (Q41-Q50)
**Q41: A US Federal Government agency requires that all their cloud workloads physically remain within the continental United States, supported only by US citizens. Feature?**
A: **Assured Workloads**. It enforces data residency and personnel access controls natively without requiring the customer to migrate to a completely separate, physically isolated "GovCloud."

**Q42: What is the primary difference between GCP Assured Workloads and AWS GovCloud?**
A: Assured Workloads achieves compliance through logical controls and policies while running on the standard GCP commercial fiber network (getting access to the latest APIs immediately). AWS GovCloud is a separate, physically isolated environment that often lags behind commercial regions in capabilities.

**Q43: How does a customer instantly ensure that no developer can create a VM in a region outside of Europe?**
A: Implement an **Organization Policy Constraint**: `gcp.resourceLocations`, applied at the Organization or Folder level, listing only the `europe-*` regions as allowed.

**Q44: A company wants to enforce that no one can associate a public IP address with a VM, anywhere in the organization. How?**
A: Implement the Organization Policy Constraint: `compute.disableInternetNetworkEndpointGroup` and enforce internal IP requirements.

**Q45: What compliance standard is relevant for credit card data?**
A: **PCI DSS** (Payment Card Industry Data Security Standard).

**Q46: What compliance standard is relevant for US healthcare data (e.g., patient records)?**
A: **HIPAA** (Health Insurance Portability and Accountability Act). GCP supports HIPAA compliance by ensuring BAA (Business Associate Agreement) coverage for essentially all of its core services.

**Q47: If a GCP service is FedRAMP High compliant, what does that indicate?**
A: It indicates the service has passed the most stringent security requirements set by the US Federal Government for handling highly sensitive (but unclassified) cloud computing data.

**Q48: What is the "Shared Responsibility Model" in cloud security?**
A: The cloud provider (Google) is responsible for the security **OF** the cloud (physical data centers, network fiber, hypervisors). The customer is responsible for security **IN** the cloud (IAM policies, firewall rules, OS patching, application vulnerabilities). 

**Q49: How does the Shared Responsibility Model shift when using a Serverless product like Cloud Run compared to Compute Engine?**
A: The customer’s responsibility decreases. In Compute Engine, the customer must patch OS vulnerabilities and manage the network layer. In Cloud Run, Google handles the OS, container runtime, scaling, and network security; the customer only manages their application code and IAM access.

**Q50: What is Binary Authorization?**
A: A supply-chain security feature used primarily with GKE or Cloud Run. It enforces that only explicitly signed and verified container images (e.g., scanned by QA and approved by a CI/CD pipeline) are allowed to be deployed to production clusters, preventing rogue or untested images from executing.
