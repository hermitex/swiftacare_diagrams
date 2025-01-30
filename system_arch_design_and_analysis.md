
# **📌 Swiftacare System Load & Scalability Analysis**
---

## **🔍 1. Market Data & Growth Assumptions**
To design a **scalable, high-performance** system, we must estimate **mental health patient volume, service demand, and system growth over time**.  

### **📊 Canadian Mental Health Service Usage (Industry Data)**
- **15% of Canadians (~5.5M people) use mental health services annually** *(Canadian Institute for Health Information, 2016–2017)*.
- **17% of Canadians accessed mental health services in 2023** *(Polling Data, 2023)*.
- **One-third of mental health patients use public healthcare**, while **two-thirds seek private therapy or online platforms like Swiftacare**.
- **Unmet needs:** **24.9% of people with a mental disorder report unmet or partially met needs** *(StatsCan, 2022)*.
- **Female patients are 30% more likely to access mental health services than males**.

### **💡 Implications for Swiftacare**
- **Total Addressable Market (TAM) in Canada** = **5.5M patients annually**.
- **We will NOT serve all patients at MVP stage** → Instead, a **realistic market capture**:
  - **MVP (Year 1 - 2025):** **5,000 early adopters**.
  - **Year 3 - 2027:** **50,000 active users**.
  - **Year 5 - 2029:** **250,000 active users**.

---

## **📈 2. Expected User Growth Over 5 Years**
| **Year**  | **Patients (Active Users)** | **Total Users (Including Therapists, Admins, Staff)** | **Sessions/Month (Avg. 2 per Patient)** |
|----------|----------------------|-----------------------------|---------------------------|
| **MVP (Year 1 - 2025)**  | **5,000**  | **7,000**  | **10,000**  |
| **Year 2 - 2026**  | **15,000**  | **20,000**  | **30,000**  |
| **Year 3 - 2027**  | **50,000**  | **75,000**  | **100,000**  |
| **Year 4 - 2028**  | **100,000**  | **150,000**  | **200,000**  |
| **Year 5 - 2029**  | **250,000**  | **400,000**  | **500,000**  |

---

## **⚡ 3. Monthly API Requests & System Load (Without Billing)**
Every action in Swiftacare generates **API requests** (logins, scheduling, messaging, session notes, etc.).

### **🔹 Monthly API Requests (MVP Year 1)**
| **Feature**                 | **Avg. Actions per Month** | **Est. Requests Per Action** | **Total Requests per Month** |
|-----------------------------|--------------------------|--------------------------|--------------------------|
| **User Logins** (Clients, Providers) | 7,000 users x 20 logins | **1 API call per login** | **140K** |
| **Booking & Rescheduling Appointments** | 10K appointments | **5 API calls (create, update, confirm, notify, log)** | **50K** |
| **Session Reminders** (SMS, Email, Push) | 10K appointments | **3 API calls per reminder cycle** | **30K** |
| **Therapist-Client Messaging** | 2K active patients x 20 messages | **1 API call per message** | **40K** |
| **Clinical Notes & Documentation** | 10K sessions | **2 API calls per session (create, update)** | **20K** |
| **File Uploads & Sharing (Reports, Documents)** | 4K file uploads | **1 API call per file** | **4K** |
| **Dashboard & Reports (Admin, Org Staff, Providers)** | 50,000 users x 10 reports | **1 API call per report** | **500K** |

**📊 Total API Requests Per Month (MVP Year 1):** **234K API calls**  
**📈 Estimated Peak RPS (Requests Per Second):** **~2 RPS (low traffic)**  

---

## **📊 4. Scaling API Requests Over 5 Years**
| **Year**  | **Total Users** | **Sessions/Month** | **Estimated Monthly API Requests** |
|----------|-------------|----------------|--------------------------------|
| **MVP (Year 1 - 2025)**  | **7,000**  | **10,000**  | **234K**  |
| **Year 2 - 2026**  | **20,000**  | **30,000**  | **750K**  |
| **Year 3 - 2027**  | **75,000**  | **100,000**  | **4.5M**  |
| **Year 4 - 2028**  | **150,000**  | **200,000**  | **9M**  |
| **Year 5 - 2029**  | **400,000**  | **500,000**  | **18M**  |

📢 **Target system capacity:** **18M API calls per month (~60 RPS at peak).**  

---

## **🗄 5. Data Storage & Read-Write Analysis**
| **Data Type**          | **Reads** | **Writes** | **Storage Growth (GB/Month)** | **Best Optimization** |
|----------------------|--------|--------|--------------------|------------------|
| **User Profiles & Auth Data** | High | Low | **2GB** | Redis Cache for fast auth |
| **Appointments** | High | Medium | **20GB** | PostgreSQL Indexing |
| **Messages & Chat Logs** | High | High | **50GB** | MongoDB + TTL Expiry |
| **Clinical Notes** | Medium | High | **100GB** | MongoDB Sharding |
| **Files (Reports, Docs)** | Low | Medium | **500GB+** | Object Storage (S3) |

💡 **Database Strategy**
- **Redis Cache** → **For frequent reads (auth, reports, dashboard analytics).**
- **PostgreSQL** → **Structured data (appointments, users).**
- **MongoDB** → **Flexible, high-volume data (messages, logs, clinical notes).**
- **S3/Blob Storage** → **Documents, scanned reports, medical images.**

---

## **🚀 6. Infrastructure Scaling Plan**
| **Year**  | **Backend Infra** | **DB Infra** | **Storage & Caching** |
|----------|------------------|-------------|-------------------|
| **MVP (Year 1 - 2025)**  | **1 API Server (FastAPI, 4 CPUs, 16GB RAM)** | **Single PostgreSQL + MongoDB** | **S3 for files, Redis for caching** |
| **Year 2 - 2026**  | **2 API Servers (Auto-scaling)** | **Read Replicas for PostgreSQL** | **CDN for file storage** |
| **Year 3 - 2027**  | **Kubernetes, Load Balancers** | **Separate Read/Write DBs** | **Autoscaling Redis Cache** |
| **Year 4 - 2028**  | **Horizontally Scaling API Layer** | **MongoDB Sharding** | **More aggressive DB partitioning** |
| **Year 5 - 2029**  | **Enterprise-Scale Clusters** | **Multi-region DBs** | **Full-scale CDN, cold storage optimization** |

---

### **📌 Bottlenecks and Constraints in Swiftacare MVP & Growth Scaling**  
To ensure a **high-performance and scalable system**, we need to analyze potential **bottlenecks** and **constraints** that could arise during **MVP** and **growth phases**.  

---

## **1️⃣ Bottlenecks in Swiftacare MVP (Year 1)**  

| **Component** | **Potential Bottleneck** | **Impact** | **Mitigation Strategy** |
|--------------|-------------------|------------|-------------------|
| **Authentication (OAuth2, RBAC)** | Too many authentication calls hitting the database directly. | Increased response time for logins. | **Use Redis caching** for authentication sessions. |
| **Appointments API (PostgreSQL)** | High concurrent requests on **peak days (Tue-Thu)**. | Slow appointment creation & retrieval. | **Indexing & read replicas** for PostgreSQL. |
| **Messaging Service (MongoDB)** | High volume of therapist-client messages. | Latency in retrieving chat logs. | **MongoDB TTL Indexing** to auto-expire old messages. |
| **Session Reminders (Email, SMS, Push)** | Sending large numbers of reminders at the same time. | Notification delays. | **Batch & async processing** using a queue system (Kafka, Celery). |
| **File Uploads & Storage (S3, MongoDB)** | Large institutions storing **bulk patient reports**. | Increased read/write latency. | **Use Object Storage (S3) with CDN caching**. |

**MVP Scaling Approach:**
- **Use Redis for auth caching** (reduces DB calls by ~80%).
- **Index PostgreSQL tables for fast queries**.
- **Sharding for high-volume collections in MongoDB**.
- **Kafka for handling bulk appointment reminders asynchronously**.

---

## **2️⃣ Growth Constraints & Scaling Plan (Year 2 - 5)**  
As the system grows, **certain constraints will require proactive solutions**.

### **🔹 Constraint 1: API Request Load Growth**
- **By Year 5, API calls will reach ~18M/month (~60 RPS at peak).**
- **Constraint:** A single API server may fail under high load.
- **Solution:**  
  - **Horizontal scaling:** Add API instances dynamically.
  - **Load balancers (Nginx, AWS ALB) to distribute requests.**
  - **Reduce DB calls by caching frequently accessed data.**

### **🔹 Constraint 2: PostgreSQL Database Limits**
- **By Year 3, PostgreSQL will handle 4.5M API calls/month.**
- **Constraint:** Increased queries cause slower reads/writes.
- **Solution:**  
  - **Partitioning appointment data by month/year.**
  - **Read replicas for better read performance.**
  - **Use Redis as a cache layer for frequently accessed queries.**

### **🔹 Constraint 3: MongoDB Scaling Issues**
- **Messaging, session logs, and analytics are high-volume.**
- **Constraint:** Single MongoDB instance will become a bottleneck.
- **Solution:**  
  - **Sharding large collections (Messages, Logs).**
  - **TTL indexes to auto-delete old messages.**
  - **Separate read/write nodes for analytics queries.**

### **🔹 Constraint 4: File Storage & Large Uploads**
- **By Year 3, file storage will exceed 10TB.**
- **Constraint:** High storage costs and slow retrieval for large files.
- **Solution:**  
  - **Move to Cloud Object Storage (AWS S3, Google Cloud Storage).**
  - **Use CDN for caching frequently accessed reports.**
  - **Auto-archive old reports into cold storage.**

### **🔹 Constraint 5: System Security & Compliance**
- **Government institutions require strict data governance.**
- **Constraint:** Need for enterprise-level security & audits.
- **Solution:**  
  - **RBAC & role-based permissions per institution.**
  - **Audit logging of all user actions.**
  - **TLS encryption for data in transit, AES encryption for sensitive stored data.**

---

## **3️⃣ Performance Optimization & Future-Proofing**
| **Component** | **Optimization Strategy** |
|--------------|--------------------------|
| **Frontend Load Time** | Lazy loading, Next.js static page rendering. |
| **Database Reads** | Redis caching for frequent queries. |
| **Database Writes** | Partitioning & sharding (MongoDB). |
| **API Performance** | Load balancer & auto-scaling API servers. |
| **File Storage** | Object storage (S3) + CDN caching. |

---

