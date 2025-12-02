

---

# ✅ **1. Architecture (Final Working Model)**

### **Flow**

```
                   ┌───────────────────────────────┐
                   │     GOOGLE MANAGED SERVICES    │
                   │ (Cloud SQL / Storage / BigQuery)│
                   └───────────▲─────────────────────┘
                               │ PSC (Private Services Access)
                               │
                     (Automatic VPC Peering)
                               │
     ┌─────────────────────────▼─────────────────────────┐
     │                 YOUR CUSTOM VPC                   │
     │              shopglobal-vpc (global)             │
     │   Subnet (us-central1): 10.0.1.0/24 (private)    │
     └───────────▲───────────────────────────────▲──────┘
                 │                               │
          VM Instances                      Cloud SQL
          via Private IP                     (No public IP)
```

---

# ✅ **2. Enable All Required APIs**

Run this first:

```sh
gcloud services enable \
    compute.googleapis.com \
    servicenetworking.googleapis.com \
    sqladmin.googleapis.com
```

---

# ✅ **3. Create Custom VPC (Global)**

```sh
gcloud compute networks create shopglobal-vpc \
    --subnet-mode=custom \
    --bgp-routing-mode=regional
```

---

# ✅ **4. Create Subnet (Regional)**

Example: **us-central1**

```sh
gcloud compute networks subnets create app-subnet \
    --network=shopglobal-vpc \
    --region=us-central1 \
    --range=10.0.1.0/24 \
    --enable-private-ip-google-access
```

---

# ✅ **5. Create Firewall Rules**

### **Allow internal traffic**

```sh
gcloud compute firewall-rules create allow-internal \
    --network=shopglobal-vpc \
    --allow=tcp,udp,icmp \
    --source-ranges=10.0.0.0/8
```

### **Allow SSH (optional)**

```sh
gcloud compute firewall-rules create allow-ssh \
    --network=shopglobal-vpc \
    --allow=tcp:22 \
    --source-ranges=0.0.0.0/0 \
    --target-tags=ssh-access
```

---

# ✅ **6. Create Cloud Router**

Required for Private Service Access.

```sh
gcloud compute routers create shopglobal-router \
    --network=shopglobal-vpc \
    --region=us-central1
```

---

# ✅ **7. Allocate an IP Range for Private Service Access**

(Mandatory for Cloud SQL Private IP)

```sh
gcloud compute addresses create google-managed-services-range \
    --global \
    --purpose=VPC_PEERING \
    --addresses=10.100.0.0 \
    --prefix-length=16 \
    --network=shopglobal-vpc
```

---

# ✅ **8. Create Private Service Connection (PSC)**

```sh
gcloud services vpc-peerings connect \
    --service=servicenetworking.googleapis.com \
    --network=shopglobal-vpc \
    --ranges=google-managed-services-range \
    --project=$(gcloud config get-value project)
```

### ✔ Verify peering:

```sh
gcloud compute networks peerings list --network=shopglobal-vpc
```

You must see:

```
PEERING_NAME: servicenetworking-google
STATE: ACTIVE
```

---

# ✅ **9. Create PostgreSQL Cloud SQL (Private IP Only)**

```sh
gcloud sql instances create shopglobal-postgres \
    --database-version=POSTGRES_15 \
    --tier=db-custom-4-16384 \
    --region=us-central1 \
    --network=shopglobal-vpc \
    --no-assign-ip \
    --availability-type=REGIONAL \
    --backup-start-time=02:00 \
    --maintenance-window-day=SUN \
    --maintenance-window-hour=3 \
    --storage-type=SSD \
    --storage-size=100GB \
    --storage-auto-increase
```

**This will work ONLY if peering is ACTIVE.**

---

# ✅ **10. Create User + Database**

```sh
gcloud sql users create appuser \
    --instance=shopglobal-postgres \
    --password=Abhi@123
```

```sh
gcloud sql databases create appdb \
    --instance=shopglobal-postgres
```

---

# ✅ **11. Connect from VM via Private IP**

Get private IP:

```sh
gcloud sql instances describe shopglobal-postgres \
    --format="get(ipAddresses[0].ipAddress)"
```

On your VM:

```sh
psql "host=<PRIVATE_IP> user=appuser dbname=appdb password=Abhi@123"
```

---

# ✅ **12. Verification Steps**

### **Check VPC**

```sh
gcloud compute networks list
```

### **Check Subnets**

```sh
gcloud compute networks subnets list --network=shopglobal-vpc
```

### **Check PSA Peering**

```sh
gcloud compute networks peerings list --network=shopglobal-vpc
```

### **Check SQL instance**

```sh
gcloud sql instances list
```

---

