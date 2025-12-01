
## **CLOUD PORTS — WHAT TO OPEN & WHEN TO OPEN (GCP Best Practices)**

### 🌍 **Public Web/App Ports (Safe for Internet)**

| Port            | Use Case                        |
| --------------- | ------------------------------- |
| **80**          | HTTP website                    |
| **443**         | HTTPS website                   |
| **8080**        | App servers (Tomcat/Node)       |
| **3000 / 5000** | API/dev servers (only dev/test) |
| **8443**        | API secure port                 |

---

### 🔐 **Admin Ports (High Risk — Restrict!)**

| Port     | Purpose     | Best Practice       |
| -------- | ----------- | ------------------- |
| **22**   | SSH         | Restrict to your IP |
| **3389** | RDP         | Restrict to your IP |
| **5601** | Kibana      | Private only        |
| **9090** | Prometheus  | Private only        |
| **9000** | Admin tools | Private only        |

---

### 🗄️ **Database Ports (NEVER Public)**

| Port      | Database   |
| --------- | ---------- |
| **3306**  | MySQL      |
| **5432**  | PostgreSQL |
| **27017** | MongoDB    |
| **1521**  | Oracle     |
| **1433**  | SQL Server |
| **6379**  | Redis      |

---

### ☸️ **Kubernetes / DevOps Ports**

| Port          | Component      |
| ------------- | -------------- |
| **6443**      | K8s API server |
| **10250**     | Kubelet API    |
| **2379–2380** | etcd           |
| **9090**      | Prometheus     |
| **9093**      | Alertmanager   |

---

### 🌐 **UDP-based Services**

| Port    | Service |
| ------- | ------- |
| **53**  | DNS     |
| **123** | NTP     |

---

