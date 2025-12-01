 **exact, practical, step-by-step steps** to verify:

### **1️⃣ If your firewall rule’s `source-range` is 0.0.0.0/0**

### **2️⃣ And whether it is ACTUALLY working (OPEN TO INTERNET)**

You will get both:

✔ **GCP-side verification** (official way)
✔ **External attack-style verification** (real-world test)

---

# ✅ **PART 1 — Verify if Firewall Rule Source Range = 0.0.0.0/0**

Use any ONE of these commands.

---

## **Step 1: Describe the Rule**

```bash
gcloud compute firewall-rules describe allow-internal
```

Look for:

```
sourceRanges:
- 0.0.0.0/0
```

If you see this → **open to everyone (internet)**.

---

## **Step 2: List the Rule with Filter**

```bash
gcloud compute firewall-rules list --filter="name=allow-internal"
```

Check output:

```
NAME            NETWORK         SRC_RANGES      ALLOW
allow-internal  shopglobal-vpc  0.0.0.0/0       tcp:0-65535,udp:0-65535,icmp
```

If `SRC_RANGES = 0.0.0.0/0` → **worldwide access**.

---

## **Step 3: Extract Only Source Range Value**

```bash
gcloud compute firewall-rules describe allow-internal \
  --format="value(sourceRanges)"
```

Expected output:

```
0.0.0.0/0
```

---

# 🟢 **Now Part 2 — VERIFY IF IT IS ACTUALLY WORKING**

Meaning: **Is your VM truly accessible from the internet?**

This is real-world verification.

---

# 🚨 Before testing, ensure:

### ✔ VM has **External Public IP**

### ✔ The VM has **NO target-tags**, or tags match the firewall rule

### ✔ The firewall rule is **INGRESS** (default)

---

# 🧪 **TEST 1 — Ping Test (For ICMP)**

From your laptop or mobile hotspot:

```bash
ping <YOUR_VM_EXTERNAL_IP>
```

If ICMP is allowed + source range is 0.0.0.0/0 → you will see replies:

```
64 bytes from <IP> icmp_seq=1 ttl=52 time=40 ms
```

If blocked → it will time out.

---

# 🧪 **TEST 2 — Port Scan from Your Laptop (Check Any TCP Port)**

Use `nmap`:

```bash
nmap <YOUR_VM_EXTERNAL_IP>
```

If your rule allows **all TCP ports** (0–65535):

You will see LOTS of open ports.

Example:

```
PORT      STATE SERVICE
22/tcp    open   ssh
80/tcp    open   http
5432/tcp  open   postgresql
...
```

This means:

### ⚠️ Your VM is globally exposed to the internet.

---

# 🧪 **TEST 3 — Try SSH from ANY NETWORK**

Try this from your home or mobile network:

```bash
ssh <username>@<YOUR_VM_EXTERNAL_IP>
```

If the rule is open:

✔ You will get SSH prompt
❌ If SSH port is not open → connection will fail

But if ALL ports are open (0–65535):

Even DB ports (5432/3306/1433) will be open.

---

# 🧪 **TEST 4 — Try Accessing VM’s Service using Browser**

Example: VM running Apache/Nginx:

```
http://<YOUR_VM_EXTERNAL_IP>
```

or if HTTPS:

```
https://<YOUR_VM_EXTERNAL_IP>
```

If the site opens → rule WORKS and is PUBLICLY reachable.

---

# 🧪 **TEST 5 — Using Online Port Checkers**

From external internet:

Go to:

[https://portchecker.co/](https://portchecker.co/)

Enter:

```
External IP: <YOUR_VM_EXTERNAL_IP>
Port: 22 or 80 or 5432 etc.
```

If it says **OPEN** → firewall is allowing it.

---

# 📌 FULL VERIFICATION CHECKLIST

| Check                | Command        | Expected Result          |
| -------------------- | -------------- | ------------------------ |
| Is source 0.0.0.0/0? | `describe`     | Yes = public             |
| Does ping work?      | `ping VM_IP`   | Replies = public         |
| Are ports open?      | `nmap VM_IP`   | Many open ports = public |
| Can I SSH?           | `ssh VM_IP`    | Works with open rule     |
| Browser access?      | `http://VM_IP` | Site loads = public      |
| Online port test?    | portchecker.co | OPEN = public            |


