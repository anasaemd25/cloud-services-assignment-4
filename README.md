# Assignment 4 - Containerized 3-Tier Application on CSC Rahti

## Application Description
This project is a 3-tier web application running on CSC Rahti (OpenShift / Kubernetes):
- **Frontend**: Nginx web server accessible over the internet using a Rahti Route.
- **Backend**: Python Flask REST API running internally inside the cluster.
- **Database**: MySQL database using a Persistent Volume Claim (PVC) to save data permanently.

**Live URL**: http://frontend-cloud-services-assignment-2.2.rahtiapp.fi

---

## Configuration: ConfigMaps vs. Secrets
- **ConfigMap (`app-config`)**: Stores non sensitive configuration values like `DB_HOST`, `DB_PORT`, `DB_NAME`, and `DB_USER`.
- **Secret (`generic-mysql-secret`)**: Stores sensitive data like `MYSQL_ROOT_PASSWORD` and `DB_PASSWORD`.
- **Why they are different**: ConfigMaps are for plain configuration values. Secrets keep the sensitive credentials encrypted and safe in Rahti so passwords don't end up on public GitHub repositories or inside Docker images.

---d

## Database Persistence
- **Rahti PVC vs. Local Docker Volume**: Local Docker Compose saves data directly to a folder on your local drive. Rahti uses a **PersistentVolumeClaim (PVC)** connected to cloud network storage. If a cloud server crashes or gets restarted, Rahti moves the database pod to a new node and reattaches the same PVC volume without losing any data.

---

## Orchestration Experiments

### 1. Pod Recovery (Self-Healing)
- **Action**: I manually deleted the running backend pod using `oc delete pod <backend-pod-name>`.
- **Result**: Rahti noticed the pod was missing and immediately started a replacement pod.
- **Explanation**: The **Deployment** and **ReplicaSet** constantly check the cluster to make sure the desired number of pods (`replicas: 1`) is always active and healthy.

![Pod Recovery Screenshot](screenshots/Experiment%201%3A%20Pod%20Recovery.png)

### 2. Scaling
- **Action**: Scaled up the backend deployment to 3 pods using `oc scale deployment/backend --replicas=3`.
- **Result**: 3 identical backend pods started running at the same time.
- **Explanation**: The frontend connects to the backend through the **Backend Service name** (`http://backend:8000`) using Kubernetes internal DNS. The Service acts as a load balancer, spreading traffic across all active pods without caring about specific pod IPs.

![Scaling Screenshot](screenshots/Experiment%202%3A%20Scaling.png)

### 3. Database Persistence
- **Action**: I deleted the MySQL database pod using `oc delete pod <database-pod-name>`.
- **Result**: A new database pod came up, reconnected to `mysql-pvc`, and kept all previous data without losing anything.
- **Explanation**: The **PersistentVolumeClaim (PVC)** is attached to the new pod, ensuring that the data is preserved even if the pod is deleted or moved to a different node.

![Database Persistence Screenshot](screenshots/Experiment%203%3A%20Persistence.png)

### 4. Application Update
- **Action**: I updated the backend code to version `1.0.1`, pushed the new image to Docker Hub, updated `rahti/backend-deployment.yaml`, and applied it with `oc apply`.
- **Result**: Rahti did a **Rolling Update**, starting the new container before turning off the old one so there was no downtime for users.

- **Explanation**: The update process is designed to ensure zero downtime by gradually replacing the old pods with new ones, maintaining service availability throughout the update.

**Update Initiated (`ContainerCreating`)**:
![Application Update Start](screenshots/experiment-4-update.png)

**Update Completed (`Terminating` / `Completed`)**:
![Application Update End](screenshots/experiment-4-update-End.png)

**Web App V2.0 Verification**:
![App Version 2 Verification](screenshots/V2.0.png)

**Live WebSite**:
![Live Website](screenshots/Assigment-4-website-live.png)

---

## Problems Encountered and Solutions
- **Problem**: I had plain text passwords sitting inside my config files.
- **Solution**: I removed the plain passwords, created a Rahti Secret to hold them securely, loaded them using environment variables, and created a `.gitignore` file so local secret files don't get uploaded to GitHub.

---

## Platform Comparison: Rahti (PaaS) vs. cPouta + Docker Compose (IaaS)
- **Deployment**: On IaaS, you have to spin up virtual machines manually, install Docker, and handle OS updates yourself. On PaaS (Rahti), the platform handles servers automatically and you only need to deploy your container YAML files.
- **Networking**: IaaS requires to do manual port mapping and add security group firewall rules. PaaS gives you internal service DNS and secure public Routes automatically.
- **Management**: On IaaS, if a VM or container dies, you have to jump in and fix it manually. PaaS includes a built-in self healing, simple pod scaling, and smooth rolling updates.