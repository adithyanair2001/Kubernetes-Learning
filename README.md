# ☸️ Kubernetes (K8s): The Definitive Beginner's Guide

> **Concept:** Kubernetes is an open-source container orchestration engine for automating deployment, scaling, and management of containerized applications.

---

## 📚 Table of Contents
1. [Core Architecture](#1-core-architecture)
2. [Key Objects & Vocabulary](#2-key-objects--vocabulary)
3. [The Declarative Model (YAML)](#3-the-declarative-model-yaml)
4. [Networking & Services](#4-networking--services)
5. [Storage](#5-storage)
6. [Cheatsheet](#6-cheatsheet)

---

## 1. Core Architecture

Kubernetes is a **Distributed System** consisting of a Control Plane (Master) and Workers.

### 🧠 The Control Plane (The Brain)
The Control Plane makes global decisions (scheduling) and detects/responds to cluster events.

| Component | Role | Analogy |
| :--- | :--- | :--- |
| **API Server** | The frontend for the K8s Control Plane. All commands go here. | The Receptionist |
| **etcd** | Consistent and highly-available key-value store for all cluster data. | The Brain/Memory |
| **Scheduler** | Watches for new Pods and assigns them to Nodes based on resources. | Logistics Manager |
| **Controller Manager** | Runs controller processes (Node controller, Job controller, etc.). | The Factory Foreman |

### 👷 The Worker Node (The Muscle)
The machines that actually run the applications.

| Component | Role | Analogy |
| :--- | :--- | :--- |
| **Kubelet** | An agent ensuring containers are running in a Pod. | The Supervisor |
| **Kube-proxy** | Maintains network rules on nodes; handles traffic. | Traffic Cop |
| **Container Runtime** | The software running the container (e.g., Docker, containerd). | The Engine |

---

## 2. Key Objects & Vocabulary

### 🧬 The Pod
*   The smallest deployable unit in K8s.
*   Wraps one or more containers (usually just one).
*   **Note:** Pods are *ephemeral* (mortal). They are not designed to live forever.

### 📦 The Deployment
*   A higher-level abstraction that manages Pods.
*   You describe a *desired state* (e.g., "I want 3 replicas of Nginx"), and the Deployment Controller changes the actual state to the desired state.
*   Handles **Scaling** and **Rolling Updates**.

### 📑 Namespaces
*   Virtual clusters backed by the same physical cluster.
*   Used to separate environments (e.g., `dev`, `staging`, `production`) or teams.

---

## 3. The Declarative Model (YAML)

K8s uses YAML files to define infrastructure as code.

**Example `deployment.yaml`:**

```yaml
apiVersion: apps/v1
kind: Deployment         # What object are we creating?
metadata:
  name: nginx-deployment
spec:
  replicas: 3            # DESIRED STATE: We want 3 copies
  selector:
    matchLabels:
      app: nginx
  template:              # The blueprint for the Pods
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80

## 4. The "Desired State" Model
Kubernetes functions on **Declarative Configuration**.
1.  **User:** Submits a YAML file defining the "Desired State" (e.g., "Run 3 copies of App X").
2.  **Kubernetes:** continuously monitors the "Current State."
3.  **Reconciliation:** If Current State != Desired State, K8s acts automatically to fix it.

## 5. Basic Glossary for Beginners
*   **Container:** The packaging of software (e.g., Docker).
*   **Orchestration:** The management of many containers.
*   **Replica:** A copy of a Pod. If you want high availability, you run multiple replicas.
*   **Service:** An abstraction which defines a logical set of Pods and a policy by which to access them (often acts as a stable IP address).
*   **Namespace:** Virtual clusters backed by the same physical cluster (used to separate teams or environments like Dev/Prod).


---

## 7. Storage (Persistence)

By default, containers are **stateless**. If a container crashes, any file inside it is deleted. This is bad for databases. Kubernetes solves this by decoupling "Storage Hardware" from "Storage Requests."

### The Concepts
1.  **Persistent Volume (PV):** The actual piece of hardware (e.g., an AWS EBS drive, a Google Disk, or a physical Hard Drive). It is managed by the Admin.
2.  **Persistent Volume Claim (PVC):** A "ticket" created by the Developer. It says "I need 10GB of storage."

**The Analogy:**
*   **PV** is a wall socket (Electricity).
*   **PVC** is the plug you hold. You don't care if the electricity comes from wind or coal; you just need the plug to fit the socket.

### Storage YAML Example (The Claim)
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-db-storage
spec:
  accessModes:
    - ReadWriteOnce # Can be mounted by one node only
  resources:
    requests:
      storage: 5Gi  # Requesting 5 Gigabytes