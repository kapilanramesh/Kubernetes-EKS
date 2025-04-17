 **ELK Stack** (Elasticsearch, Logstash, and Kibana) does — in a **clear and practical way**:

---

### ✅ **E → Elasticsearch**
> **Job:** Stores, searches, and analyzes logs.

- Acts like a **powerful search engine + database**.
- All your logs are finally sent and stored here.
- You can **query and filter** logs lightning-fast.

🧠 Think of it as:  
📦 A smart warehouse where all logs are stored and organized.

---

### ✅ **L → Logstash**
> **Job:** Collects, processes, and forwards logs to Elasticsearch.

- It **ingests logs** from different sources (apps, servers, containers).
- Can **filter, clean, enrich** logs (e.g., remove passwords, add tags).
- Sends cleaned logs to **Elasticsearch**.

🧠 Think of it as:  
🧹 A cleaning and routing machine for raw log data.

---

### ✅ **K → Kibana**
> **Job:** Visualizes logs stored in Elasticsearch.

- You can **view logs in a UI**, create dashboards, charts, alerts.
- Helps DevOps/SRE teams **spot issues quickly**.
- Works directly on top of Elasticsearch.

🧠 Think of it as:  
📊 A beautiful dashboard that helps you see what’s going on.

---

### 🧩 Flow in 1 Line:

`Logstash` → (cleans + sends logs) → `Elasticsearch` → (stores logs) → `Kibana` (visualizes logs)

---
