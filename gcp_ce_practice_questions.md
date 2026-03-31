# GCP Platform Customer Engineer: RRK Practice Scenarios

This document contains scenario-based questions designed to test your architectural decision-making, trade-off analysis, and knowledge of GCP differentiators. For each scenario, remember to:
1. **Clarify constraints** (Budget, timeline, team expertise, SLAs).
2. **Propose a GCP-native solution.**
3. **Defend your choices** against alternatives (e.g., "Why GKE and not Cloud Run?").

---

## 1. Infrastructure Modernization & SAP
**Scenario 1:** A large retail customer has a physical data center lease expiring in 6 months. They have 1,000 VMware VMs running critical stateful legacy applications that their small Ops team manages. They want to move to the cloud but lack the resources to rewrite or containerize the applications before the move. 
*   **Question:** What migration strategy and specific GCP services would you recommend to get them out of their data center in time, and why? What are the long-term trade-offs of this approach?

**Scenario 2:** A global manufacturing company runs their core business on SAP HANA on-premises. They experience painful, hours-long downtime every quarter when patching the physical servers beneath the database. 
*   **Question:** How does Google Cloud uniquely solve this infrastructure maintenance issue for massive, memory-heavy SAP workloads? 

---

## 2. Networking & Hybrid Connectivity
**Scenario 3:** A financial institution requires a dedicated, highly available, and low-latency connection from their on-premises data center to GCP. They must transfer massive amounts of batch data nightly and cannot route any of this traffic over the public internet due to compliance.
*   **Question:** Would you recommend Cloud VPN, Partner Interconnect, or Dedicated Interconnect? How would you architect it for 99.99% availability?

**Scenario 4:** You are designing the network topology for a multi-national corporation operating in Tokyo, London, and New York. They want their microservices in different regions to communicate securely without complex network peering.
*   **Question:** Explain how GCP's Global VPC architecture differs from AWS/Azure in this scenario, and how you would utilize subnets and Global L7 Load Balancing.

---

## 3. Security & Compliance
**Scenario 5:** A healthcare provider is migrating sensitive patient records to Google Cloud Storage and BigQuery. The CISO is terrified of "insider threats"—specifically, an angry employee taking their valid IAM credentials, logging in from their home network, and downloading the entire database.
*   **Question:** How would you use VPC Service Controls (VPC SC) to mitigate this exact threat? How does this interact with standard IAM permissions?

**Scenario 6:** A customer is building a public-facing API for their mobile app. They have been victims of DDoS attacks and SQL injection attacks in the past. 
*   **Question:** Which GCP networking edge products would you deploy in front of their backend services to protect them?

---

## 4. Application Modernization & Apigee
**Scenario 7:** A media company is breaking apart a massive Java monolith into microservices. They want a container orchestration platform but have a strict requirement that they avoid "vendor lock-in." They also want to manage policy and security consistently across their GCP clusters and a few remaining on-premises clusters.
*   **Question:** Why is Google Kubernetes Engine (GKE) the right choice here, and how does Anthos fit into the multi-environment requirement? 

**Scenario 8:** A telecommunications company has hundreds of internal APIs. They want to expose a subset of these APIs to third-party developers, create a self-service developer portal, and charge developers based on the number of API calls they make.
*   **Question:** Why would you recommend Apigee over GCP's basic API Gateway for this use case?

---

## 5. Databases & Data Management
**Scenario 9:** A global gaming company is launching a massive multiplayer game. The player inventory database must be fiercely consistent (ACID compliant) so players don't accidentally duplicate high-value items, but it must also scale horizontally to handle millions of players globally on launch day. Cloud SQL cannot handle the scale.
*   **Question:** Which GCP database is designed exactly for this use case (Global + Relational/ACID + Horizontally Scalable), and what is the underlying technology (e.g., TrueTime) that makes it possible?

**Scenario 10:** An IoT company collects temperature sensor data from millions of devices every second. They need to write this data with single-digit millisecond latency and perform high-throughput time-series analysis on it. 
*   **Question:** Would you recommend Cloud SQL, Firestore, or Cloud Bigtable? Why?

---

## 6. Data Analytics & Business Intelligence
**Scenario 11:** A logistics company receives real-time streams of package tracking events (via Pub/Sub) and also has daily batch files of shipping manifests dropped into Cloud Storage. They want a single, unified processing pipeline to handle both stream and batch data before loading it into a data warehouse.
*   **Question:** Would you recommend Dataproc (Spark/Hadoop) or Dataflow (Apache Beam) for this pipeline? Why?

**Scenario 12:** A marketing agency wants to query petabytes of clickstream data without managing any servers, clusters, or indexes. They just want to write standard SQL and get answers in seconds.
*   **Question:** How does BigQuery's architecture (specifically the separation of compute and storage) allow for this Serverless data warehousing experience?

---

## 7. AI Infrastructure & Cloud AI
**Scenario 13:** A startup is training a massive, custom Large Language Model (LLM) from scratch using PyTorch. They are highly sensitive to compute costs and need specialized hardware that is more cost-effective than standard NVIDIA GPUs for training massive tensor operations.
*   **Question:** What proprietary Google Cloud hardware accelerators would you recommend they use with Vertex AI Custom Training?

**Scenario 14:** An e-commerce company wants to add a "conversational product search" feature to their website but they do not have a team of Ph.D. Machine Learning engineers. They just want to use state-of-the-art foundational models.
*   **Question:** How would you position Vertex AI (specifically the Gemini API and Vector Search for RAG) to help them build this quickly?
