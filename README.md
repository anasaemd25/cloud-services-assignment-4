# Assignment 4 - Containerized 3-Tier Application on CSC Rahti

## Application Description
This application is a 3 tier web app deployed on CSC Rahti (OpenShift/Kubernetes):
- **Frontend**: Nginx web server exposed to the public internet via a Rahti Route.
- **Backend**: Python Flask REST API running inside the internal cluster network.
- **Database**: MySQL 8.0 database with a Persistent Volume Claim (PVC) for data storage.

**Live URL**: http://frontend-cloud-services-assignment-2.2.rahtiapp.fi

---

## Configuration: ConfigMaps vs. Secrets
- **ConfigMap (`app-config`)**: Used for non-sensitive settings like `DB_HOST`, `DB_PORT`, `DB_NAME`, and `DB_USER`. These settings can safely be stored in Git.
- **Secret (`generic-mysql-secret`)**: Used for sensitive credentials like `MYSQL_ROOT_PASSWORD` and `DB_PASSWORD`. Secrets keep sensitive data hidden so passwords are never committed to GitHub.

---

## Orchestration Experiments

### 1. Pod Recovery (Self-Healing)
- **Action**: Manually deleted the backend pod using `oc delete pod <backend-pod-name>`.
- **Result**: Rahti noticed the pod was missing and automatically launched a new backend pod to keep the required replica count active.

![Pod Recovery Screenshot](screenshots/experiment-1-pod-recovery.png)

### 2. Scaling
- **Action**: Scaled the backend deployment to 3 replicas using `oc scale deployment/backend --replicas=3`.
- **Result**: Rahti started 3 running backend pods simultaneously. The frontend service automatically load-balances requests across all available backend pods without needing individual IP addresses.

![Scaling Screenshot](screenshots/experiment-2-scaling.png)

### 3. Database Persistence
- **Action**: Deleted the MySQL database pod using `oc delete pod <database-pod-name>`.
- **Result**: A new database pod started and connected to the existing Persistent Volume Claim (`mysql-pvc`), keeping all database data intact.

![Database Persistence Screenshot](screenshots/experiment-3-persistence.png)

---

## Comparison: Rahti (PaaS) vs. cPouta + Docker Compose (IaaS)
- **cPouta + Docker Compose (IaaS)**: Required manually renting and configuring a virtual machine, installing Docker, and managing containers on a single host. If the VM went down, everything stopped.
- **Rahti (PaaS / Kubernetes)**: Handles the underlying virtual machines automatically. It provides built-in container orchestration, self-healing, scaling, rolling updates, and internal networking across cluster nodes.