

# 🟦 **STEP 1 — Install Cloud SQL Auth Proxy**

### **Windows (EXE Download)**

Download here:
[https://cloud.google.com/sql/docs/postgres/connect-auth-proxy#install](https://cloud.google.com/sql/docs/postgres/connect-auth-proxy#install)

Then place the `.exe` file in:

```
C:\cloudsql\
```

Rename:

```
cloud-sql-proxy.exe
```

---

# 🟦 **STEP 2 — Login to GCP from your laptop**

Open **CMD** (NOT PowerShell):

```
gcloud auth login
```

This opens your browser → choose your Google Cloud account.

Then set your project:

```
gcloud config set project <PROJECT_ID>
```

---

# 🟦 **STEP 3 — Get INSTANCE CONNECTION NAME**

Run:

```
gcloud sql instances describe shopglobal-postgres \
    --format="value(connectionName)"
```

Example output:

```
novgcpuser0910dec-nclient:us-central1:shopglobal-postgres
```

Copy this!

---

# 🟦 **STEP 4 — Start Cloud SQL Proxy**

Go to your downloaded folder:

```
cd C:\cloudsql
```

Run:

```
cloud-sql-proxy.exe novgcpuser0910dec-nclient:us-central1:shopglobal-postgres --port=5432
```

You will see logs like:

```
Listening on 127.0.0.1:5432
Connection authorized for Cloud SQL instance...
```

This means your laptop has a **secure tunnel → Cloud SQL (Private IP)**.

---

# 🟦 **STEP 5 — Connect from psql**

If `psql` is installed, run:

```
psql "host=127.0.0.1 user=appuser dbname=appdb password=Abhi@123 port=5432"
```

🎯 **You are now inside Cloud SQL from laptop!!!**

---

# 🟩 OPTIONAL — Connect using pgAdmin

In pgAdmin → Register Server → Connection:

| Field    | Value     |
| -------- | --------- |
| Host     | 127.0.0.1 |
| Port     | 5432      |
| User     | appuser   |
| Password | Abhi@123  |
| Database | appdb     |

This will work because **Cloud SQL Proxy is doing secure forwarding**.

---

# 🔥 WHY THIS WORKS?

Cloud SQL Private IP normally cannot be accessed outside VPC.
But the **Auth Proxy** uses Google IAM → Cloud SQL Admin API → internal secure tunnel.

So no public IP is required.

---


