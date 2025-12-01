Below is the **complete firewall rule set** for the final architecture we designed:

✔ HTTPS Load Balancer
✔ GKE Cluster (nodes + pods)
✔ Cloud SQL PostgreSQL (private IP)
✔ BigQuery (no firewall needed—HTTPS 443 outbound only)
✔ Optional SSH

Everything is production-grade and aligned to **GCP best practices (Nov 2025)**.

---

# ⭐ **FINAL — ALL FIREWALL RULES (ONE GO, READY TO COPY–PASTE)**

Replace `shopglobal-vpc` with your VPC name if needed.

---

# ✅ **1. Allow HTTPS + HTTP (for Load Balancer → GKE)**

Backend services need GKE nodes to accept LB health checks + traffic.

```
gcloud compute firewall-rules create allow-http-https \
    --network=shopglobal-vpc \
    --allow=tcp:80,tcp:443 \
    --target-tags=gke-nodes \
    --description="Allow HTTP/HTTPS traffic from LB to GKE nodes"
```

---

# ✅ **2. Allow GKE Control Plane to Connect to Nodes**

Required for node health, kubelet, and API communication.

```
gcloud compute firewall-rules create allow-gke-control-plane \
    --network=shopglobal-vpc \
    --allow=tcp:10250,tcp:443 \
    --source-ranges=35.191.0.0/16,130.211.0.0/22 \
    --target-tags=gke-nodes \
    --description="Allow GKE master control plane traffic to node kubelet/API"
```

📌 **Google GKE Master IP ranges**:

* 35.191.0.0/16
* 130.211.0.0/22

---

# ✅ **3. Allow Private IP Communication (GKE → Cloud SQL PostgreSQL)**

Port 5432 is needed for PostgreSQL via private VPC.

```
gcloud compute firewall-rules create allow-cloudsql-postgres \
    --network=shopglobal-vpc \
    --allow=tcp:5432 \
    --source-tags=gke-nodes \
    --target-tags=cloudsql \
    --description="Allow private PostgreSQL access from GKE to Cloud SQL"
```

💡 **Notes:**

* `source-tags=gke-nodes` → only workloads inside GKE can reach DB
* `target-tags=cloudsql` → use this tag on Cloud SQL’s associated connector VM (if using Connector)
* If using **Private IP (recommended)** → this rule applies inside VPC

---

# ✅ **4. Allow GKE Nodes to Pull Images, Reach APIs (Outbound 443)**

This is not inbound firewall, but needed for GKE to function.

```
gcloud compute firewall-rules create allow-egress-https \
    --network=shopglobal-vpc \
    --allow=tcp:443 \
    --direction=EGRESS \
    --description="Allow GKE nodes to reach Google APIs (BigQuery, GCR, Artifact Registry)"
```

This allows GKE pods to reach:

* BigQuery API
* Container registry
* Cloud SQL Auth API
* Logging/Monitoring
* Pub/Sub
* etc.

---

# ✅ **5. Allow Internal Traffic (Node ↔ Node, Pod ↔ Service)**

```
gcloud compute firewall-rules create allow-internal \
    --network=shopglobal-vpc \
    --allow=tcp,udp,icmp \
    --source-ranges=10.0.0.0/16 \
    --description="Allow all internal VPC communication for GKE, SQL, services"
```

---

# ✅ **6. (Optional) Allow SSH from your public IP only**

```
gcloud compute firewall-rules create allow-ssh \
    --network=shopglobal-vpc \
    --allow=tcp:22 \
    --source-ranges=YOUR_PUBLIC_IP/32 \
    --target-tags=gke-nodes \
    --description="Restricted SSH access for maintenance"
```

Example:

```
--source-ranges=122.176.12.45/32
```

---

# ❌ **What You Do NOT Need**

(These are often misconfigured or mistakenly added)

### ❌ Cloud SQL public port 5432 open to internet

### ❌ BigQuery ports

(BigQuery uses HTTPS only → outbound 443 already covered)

### ❌ GKE master inbound access

Master is Google-managed & private.

---

# 🧩 TOTAL FIREWALL RULE SUMMARY (Final)

| Rule                    | Purpose                    | Ports            |
| ----------------------- | -------------------------- | ---------------- |
| allow-http-https        | LB → GKE                   | 80, 443          |
| allow-gke-control-plane | Master → Node              | 10250, 443       |
| allow-cloudsql-postgres | GKE → PostgreSQL           | 5432             |
| allow-egress-https      | GKE → BigQuery/Google APIs | 443              |
| allow-internal          | VPC internal               | ALL TCP/UDP/ICMP |
| allow-ssh (optional)    | Secure admin access        | 22               |

---
