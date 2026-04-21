# 🚀 Scalable CI/CD with Kubernetes Self-Hosted Runners (ARC)

> Production-style implementation of autoscaling GitHub Actions runners on Kubernetes using Actions Runner Controller (ARC)

---

## 📌 Table of Contents
- [Overview](#-overview)
- [Problem Statement](#-problem-statement)
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

## 📸 Proof of Execution

Below are the key observations captured during testing, showing how the system behaves under load.

---

### ⚡ Parallel Job Execution

![Parallel Jobs](./screenshots/parallel-jobs.png)

Multiple CI jobs are triggered at the same time and start executing immediately without any queue delay.  
This confirms that the system supports parallel execution.

---

### 📈 Dynamic Runner Scaling

![Pods Scaling](./screenshots/pods-scaling.png)

Kubernetes dynamically creates multiple runner pods based on the number of jobs triggered.  
Each job is assigned to a separate runner pod.

---

### 🗑️ Ephemeral Runner Behavior

![Pods Terminated](./screenshots/pods-terminated.png)

Runner pods are automatically deleted after job completion.  
This ensures efficient resource usage and prevents idle infrastructure.

---

### 🧪 Job Execution Logs

![Job Logs](./screenshots/job-logs.png)

The logs confirm that jobs are executed successfully on self-hosted runners inside Kubernetes.  
Each job runs in an isolated environment.

---

### 🔗 Runner Registration

![Runner](./screenshots/runner-registered.png)

The self-hosted runner is successfully registered and connected to GitHub, allowing workflows to be executed on the cluster.

---

### 🚀 Impact on CI Performance

![Before vs After](./screenshots/before-after.png)

Before: Jobs were queued due to limited runners  
After: Jobs execute immediately with dynamic scaling  

This demonstrates how the system eliminates workflow delays.