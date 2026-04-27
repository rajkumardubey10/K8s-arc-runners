# 🚀 Scalable CI/CD with Kubernetes Self-Hosted Runners (ARC)

> Production-style implementation of autoscaling GitHub Actions runners on Kubernetes using Actions Runner Controller (ARC)

---

## 📌 Table of Contents
- [Overview](#-overview)
- [Problem Statement](#-problem-statement)
- [Before vs After](#-before-vs-after)
- [Solution](#-solution)
- [Architecture](#-architecture)
- [Tech Stack](#-tech-stack)
- [Project Structure](#-project-structure)
- [Implementation](#-implementation)
- [CI Workflow Demo](#-ci-workflow-demo)
- [Validation & Proof](#-validation--proof)
- [Results](#-results)
- [Challenges & Fixes](#-challenges--fixes)
- [Key Learnings](#-key-learnings)
- [Future Improvements](#-future-improvements)
- [How to Run](#-how-to-run)
- [Summary](#-summary)

---

## 📖 Overview
This project demonstrates a **scalable CI/CD system** using Kubernetes-based self-hosted runners powered by Actions Runner Controller (ARC).

It simulates real-world CI workloads with **parallel execution and autoscaling**, similar to what modern DevOps teams implement in production.

---

## 🎯 Problem statement 

In many setups, self-hosted runners are deployed on static machines (like EC2), which leads to:

- ⏳ Workflows getting queued during peak time  
- 📉 Poor scalability when multiple jobs run together  
- 💸 Idle resources when no jobs are running  
- ⚠️ Shared environments causing inconsistent builds  

I wanted a setup that could handle multiple jobs at once without delays.

---

## 🔍 Before vs After

### 🐢 Before: Queued Workflows
Workflows were waiting due to limited runners.

### ⚡ After: Scaled Runners
Runners scale dynamically and jobs run in parallel.

<img width="1536" height="1024" alt="arc-before-after" src="https://github.com/user-attachments/assets/7656c7d3-ab12-4a4b-a30f-656cf9ed0eb3" />

---

## 💡 Solution

To solve this, I used Kubernetes with ARC.

The approach:

- ⚙️ Create runners only when jobs are triggered  
- 🧼 Run each job in a fresh, isolated environment  
- 🔁 Automatically remove runners after execution  

This makes the system scalable, efficient, and reliable.

---

## 🏗️ Architecture

<img width="1600" height="771" alt="arc-architecture" src="https://github.com/user-attachments/assets/baf5532a-8683-4aa8-a8f8-b80b3a166e20" />

--- 

## 🔄 Workflow Execution Flow

The architecture above shows the system components.  
This diagram shows how a CI job actually flows through the system.

<img width="1536" height="1024" alt="ARC-Flow_Diagram" src="https://github.com/user-attachments/assets/68ddae19-aa52-4493-a7da-01d104d95ded" />


📌 When a workflow is triggered:
- Job enters GitHub queue  
- ARC controller detects the job  
- Runner pod is created dynamically  
- Pod registers and executes the job  
- Pod is deleted after completion  

This ensures dynamic scaling and efficient resource usage.
---

## 🔧 Implementation

The setup was done in two main steps.

First, I installed the ARC controller inside the Kubernetes cluster. This component is responsible for watching GitHub job queues and managing runner lifecycle.

Then, I deployed a runner scale set connected to my GitHub repository. This allows the system to create runner pods dynamically whenever new jobs are triggered.

I used Helm for deployment and configuration instead of writing raw Kubernetes manifests, which made it easier to manage and update the setup.


---

## 🔍 Logging & Observability

### 🏗️ Logging Architecture

<img width="1536" height="1024" alt="logging-architecture" src="https://github.com/user-attachments/assets/0fa62f3d-4f89-4bc5-9c15-f593d1d99fab" />


---

### 🧠 Overview

To enable debugging of ephemeral runner pods, I implemented centralized logging using:

- Grafana Alloy (DaemonSet log collection)  
- Grafana Loki (log storage & indexing)  
- Grafana (log querying & visualization)  

---

### 🔄 Log Flow

```
Runner Pods → Alloy → Loki Gateway → Loki → Grafana
```

---

### 📸 Centralized Logs in Grafana

<img width="1366" height="768" alt="grafana-logs-loki" src="https://github.com/user-attachments/assets/7c1593be-738d-4507-87f9-53bf9fa003db" />


👉 Logs from all runner pods are aggregated and searchable.

---
### 🎯 Why this matters

- Logs persist even after pod deletion  
- Enables debugging of CI failures  
- Provides historical visibility  
- Makes system production-ready  

 ---
 ## 🧰 Tech Stack

| Category              | Tools / Technologies                          | Purpose |
|----------------------|-----------------------------------------------|--------|
| 🚀 CI/CD Platform     | GitHub Actions                                | Workflow orchestration and job execution |
| ☸️ Orchestration      | Kubernetes                                    | Container orchestration and runner lifecycle |
| 🔁 Autoscaling        | Actions Runner Controller (ARC)               | Dynamic provisioning of self-hosted runners |
| 📦 Package Manager    | Helm                                          | Deployment and management of Kubernetes resources |
| 🧾 Log Collection     | Grafana Alloy                                 | Collect logs from Kubernetes nodes (DaemonSet) |
| 📊 Log Storage        | Grafana Loki                                  | Centralized log aggregation and indexing |
| 📈 Visualization      | Grafana                                       | Log querying and visualization using LogQL |
| 🐳 Container Runtime  | Docker                                        | Container execution environment |
| ⚙️ CLI Tools          | kubectl                                       | Kubernetes cluster management |
| 📝 Configuration      | YAML                                          | Workflow and infrastructure configuration |
 
## 📸 Proof of Execution

Below are the key observations captured during testing, showing how the system behaves under load.

---

### 🗑️ Logs Persist After Pod Deletion (🔥 Key Capability)

<!-- 📸 Screenshot: pods-terminating.png -->
<img width="1366" height="768" alt="terminated-pod-log" src="https://github.com/user-attachments/assets/4a7cb3dd-7ed0-429c-a366-c936ae870f6b" />

---

<!-- 📸 Screenshot: deleted-pod-logs.png -->
<img width="1366" height="768" alt="deleted-pod-logs" src="https://github.com/user-attachments/assets/021fe23d-9474-4ba2-b141-f224516878a0" />


Even after the runner pod enters **Terminating state and gets deleted**, logs are still accessible in Grafana.

#### 🔍 What this proves:

- Runner pods are ephemeral and get deleted after job completion  
- Logs are **not lost** after pod termination  
- Loki stores logs independently of pod lifecycle  
- Enables debugging of completed or failed CI jobs  

#### 🧠 How it works:

- Logs are collected by Alloy before pod termination  
- Logs are pushed to Loki and stored persistently  
- Grafana queries logs from Loki, not from live pods  

👉 This is critical for real-world CI/CD systems where workloads are short-lived.

---

### ⚡ Parallel Job Execution

<img width="1366" height="768" alt="Screenshot 2026-04-27 193039" src="https://github.com/user-attachments/assets/186a2733-6086-4b27-9e34-206d791a266e" />


- Multiple CI jobs are triggered at the same time and start executing immediately without any queue delay.  
This confirms that the system supports parallel execution.

- Kubernetes dynamically creates multiple runner pods based on the number of jobs triggered.  
Each job is assigned to a separate runner pod.

---

### 🗑️ Ephemeral Runner Behavior
<img width="1366" height="768" alt="deleted-pod-logs" src="https://github.com/user-attachments/assets/60d571d2-1223-42b6-806f-2ade88cd1ee9" />


- Runner pods are automatically deleted after job completion.  
- This ensures efficient resource usage and prevents idle infrastructure.

---

### 🧪 All Pods Running inside the Monitoring Namespace

<img width="1366" height="768" alt="terminal-screenshot-daemonset" src="https://github.com/user-attachments/assets/b007fc82-d758-48a7-8d57-723080e7492a" />

- This screenshot shows the Kubernetes cluster with ARC controller, runner scale sets, and the monitoring stack (Grafana, Loki, Alloy) running successfully.

- It confirms that both the CI/CD infrastructure and centralized logging system are properly deployed and operational within the cluster.
---

### 🔗 Runner Registration

<img width="1366" height="768" alt="ARC_runner_screenshot " src="https://github.com/user-attachments/assets/13c0b079-8bff-4b4b-99d2-89c025fe81a1" />



- The self-hosted runner is successfully registered and connected to GitHub, allowing workflows to be executed on the cluster.

---

### ✅ Alloy Daemonset Running on Every Node
---
<img width="1366" height="768" alt="runner-scaling" src="https://github.com/user-attachments/assets/18546ecb-cc94-46cc-a74a-694c8d6d46a6" />

- One Alloy pod per node ensure complete log coverage.
- No matter which node a runner pod schedules on - Alloyis there to collect its logs.


### 🚀 Impact on CI Performance
## 📊 Results

| Metric                              | Before                     | After                          |
|-------------------------------------|----------------------------|--------------------------------|
| Workflow Queue Time                 | High during peak           | Zero — instant execution       |
| Concurrent Workflows                | 1–2 max                    | 8–12 simultaneously            |
| Infrastructure Cost                 | Always-on EC2              | On-demand pods only            |
| Cost Reduction                      | —                          | ~30–40%                        |
| Pipeline Execution Time             | Baseline                   | ~60% faster                    |
| Log Availability After Pod Deletion | ❌ Lost forever            | ✅ Retained for 30 days        |
| Debug Capability for Failed Jobs    | ❌ None                    | ✅ Full log history available  |

This demonstrates how the system eliminates workflow delays.
