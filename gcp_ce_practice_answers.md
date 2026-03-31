# GCP Platform Customer Engineer: Detailed Scenario Answers

This document contains the detailed answers to the practice scenarios, highlighting the specific **GCP Differentiators** and **Trade-offs** you should mention during your interview.

---

## 1. Infrastructure Modernization & SAP

**Scenario 1: Lift & Shift VM Migration (1,000 VMs, 6-month deadline)**
*   **Recommended Solution:** **Google Cloud VMware Engine (GCVE)** or Migrate to Virtual Machines (GCE).
*   **Why:** Given the strict 6-month deadline and small Ops team, they do not have the time to refactor code or adopt containers. GCVE allows a pure "lift and shift"—they vMotion the VMs directly into Google's physical data centers running the VMware stack. This requires zero application changes and zero staff retraining on GCP networking. 
*   **Trade-offs:** GCVE is an Infrastructure-as-a-Service (IaaS) solution. They are still responsible for patching the guest operating systems and managing the monolithic architecture. The long-term Phase 2 plan should be to modernize those VMs into containers using GKE to reduce operational overhead.

**Scenario 2: SAP HANA Maintenance**
*   **Recommended Solution:** Compute Engine (GCE) **Live Migration**.
*   **Why:** Google Cloud is uniquely capable of migrating a running Virtual Machine from one physical host to another physical host *without dropping connections or requiring a reboot*. For SAP HANA—which keeps massive amounts of data entirely in RAM and can take hours to reboot after a server patch—Live Migration is a massive differentiator that functionally eliminates host-level maintenance downtime.

---

## 2. Networking & Hybrid Connectivity

**Scenario 3: Secure, Dedicated Hybrid Connectivity**
*   **Recommended Solution:** **Dedicated Interconnect** configured for a 99.99% SLA.
*   **Why:** The customer requires low-latency, massive bandwidth, and no public internet routing. Cloud VPN routes over the public internet (violating compliance). Partner Interconnect relies on third-party ISPs. Dedicated Interconnect provides a physical, private cross-connect directly into Google's edge.
*   **The Architecture:** To achieve the strict 99.99% SLA, you must provision 4 Dedicated Interconnect connections spanning 2 different Metropolitan areas, connecting to dual redundant routers on the customer's premise. 

**Scenario 4: Global VPCs vs Regional VPCs**
*   **Recommended Solution:** GCP **Global VPCs** and **Global L7 External HTTP(S) Load Balancing**.
*   **Why:** Unlike AWS and Azure where a VPC is restricted to a single region, a GCP VPC is a global resource (subnets are regional). This means a microservice in Tokyo and a microservice in New York can exist on the same VPC and communicate securely via internal IP addresses natively—no complex Transit Gateways or peering required.
*   **Load Balancing:** The Global LB provides a single Anycast IP address. Users in the UK and users in Japan hit the *same* IP address but are efficiently routed to the closest healthy backend at Google's edge.

---

## 3. Security & Compliance

**Scenario 5: Insider Threat & Data Exfiltration**
*   **Recommended Solution:** **VPC Service Controls (VPC SC)**.
*   **Why:** Standard IAM only determines *who* can access data. VPC SC determines *from where* they can access it. By placing BigQuery and Cloud Storage inside a VPC SC security perimeter, Google Cloud automatically blocks any API calls originating from outside the approved corporate IP range. Even if an employee steals valid IAM credentials, the request will be rejected if they attempt it from their home Wi-Fi.

**Scenario 6: Public API DDoS & WAF Protection**
*   **Recommended Solution:** **Cloud Armor** combined with Global HTTP(S) Load Balancing.
*   **Why:** Cloud Armor operates at Google's network edge. It serves as a Web Application Firewall (WAF) to block OWASP Top 10 vulnerabilities (like SQL injection) and instantly absorbs massive, volumetric DDoS attacks before that malicious traffic ever reaches the customer's backend compute resources.

---

## 4. Application Modernization & Apigee

**Scenario 7: Containerization & Vendor Lock-In**
*   **Recommended Solution:** **Google Kubernetes Engine (GKE)** and **Anthos**.
*   **Why GKE:** Kubernetes is open-source, directly addressing the "vendor lock-in" concern. Application containers can run anywhere.
*   **Why Anthos:** Because they have mixed environments (GCP + on-prem), Anthos provides a single pane of glass. Using Anthos Config Management, they can push a single Git repository of security policies out to every cluster simultaneously, ensuring consistency everywhere.

**Scenario 8: API Monetization for Third Parties**
*   **Recommended Solution:** **Apigee API Management**.
*   **Why:** GCP's standard API Gateway is a lightweight proxy meant for internal routing (e.g., routing to Cloud Run). Apigee is a full-lifecycle Enterprise API platform. It specifically provides API monetization (billing third-parties per request), custom Developer Portals, deep traffic analytics, and advanced quotas that the customer requires.

---

## 5. Databases & Data Management

**Scenario 9: Global ACID Multiplayer Database**
*   **Recommended Solution:** **Cloud Spanner**.
*   **Why:** The customer needs three things: Relational mapping, strict ACID consistency, and massive global horizontal scale. Cloud SQL maxes out around 64TB and is regional. Cloud Spanner horizontally scales infinitely across the globe without sacrificing consistency.
*   **How it works:** Spanner relies on **TrueTime**, Google's proprietary infrastructure of atomic clocks and GPS receivers in every data center, establishing a globally synchronized clock to sequence database transactions perfectly.

**Scenario 10: High-Throughput Time-Series Sensor Data**
*   **Recommended Solution:** **Cloud Bigtable**.
*   **Why:** Cloud SQL will immediately bottleneck on writes. Firestore is optimized for mobile document syncing. Bigtable is a wide-column NoSQL database natively designed for single-digit millisecond latency and millions of writes per second. It is the exact backend Google uses for Google Earth and Google Analytics.

---

## 6. Data Analytics & Business Intelligence

**Scenario 11: Unified Batch and Stream Pipeline**
*   **Recommended Solution:** **Cloud Dataflow** (using Apache Beam).
*   **Why:** Dataproc (managed Spark/Hadoop) often requires maintaining two separate codebases or processing paths for batch and streaming. Dataflow uses the Apache Beam model, allowing developers to write the pipeline logic *once* and execute it interchangeably over both streaming messages (Pub/Sub) and batch files (GCS). It is fully serverless.

**Scenario 12: Serverless Data Warehousing**
*   **Recommended Solution:** **BigQuery**.
*   **Why:** BigQuery completely decouples its compute engine (Dremel) from its storage tier (Colossus). When a query is run, BigQuery instantly provisions thousands of ephemeral compute nodes in the background, scans the required storage, returns the result, and immediately destroys the nodes. The customer never provisions servers; they only pay for the exact terabytes scanned.

---

## 7. AI Infrastructure & Cloud AI

**Scenario 13: Cost-Effective Custom LLM Training**
*   **Recommended Solution:** **Cloud TPUs (Tensor Processing Units)** on Vertex AI Custom Training.
*   **Why:** TPUs are Google's custom-designed foundational ASICs explicitly built to accelerate the massive matrix-multiplication operations required by Neural Networks. For training massive LLMs from scratch, they frequently provide a significantly higher price-to-performance ratio than standard NVIDIA GPUs.

**Scenario 14: Adding GenAI without ML Engineers**
*   **Recommended Solution:** **Vertex AI Gemini Foundation Models** combined with **Vector Search (RAG Architecture)**.
*   **Why:** Because they lack ML engineers, they should absolutely not train their own models. They can instead pass user queries to the pre-trained Gemini API. To ensure the model knows about their specific store products (and doesn't hallucinate), they can use Retrieval-Augmented Generation (RAG). Vector Search retrieves the relevant product details from their catalog and injects it into the Gemini prompt dynamically.
