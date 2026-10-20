# AI-Powered Kubernetes Self-Healing Platform

**Date:** October 13, 2026  
**Version:** v1.0  
**Domain:** AIOps / SRE / Agentic AI  

---


## 📋 Executive Summary

The **AI-Powered Kubernetes Self-Healing Platform** is an end-to-end AIOps solution designed to automate incident triage, root cause analysis (RCA), and remediation for Kubernetes workloads.

By leveraging a Large Language Model (LLM) operating over a vector-search operational context (Chroma DB), the platform replaces static, brittle alert scripts with an intelligent **Agentic AI** loop. Upon receiving an alert via Kafka, the agentic worker gathers real-time diagnostics, evaluates remediation strategies against vector-indexed runbooks, and automatically executes declarative Ansible playbooks to restore system stability.

---

## 🏗️ System Architecture & Workflow

```
+-------------------+      +---------------+      +-------------------+
|  Prometheus /     | ---> | Kafka Topic   | ---> | Agentic AI Worker |
|  Alertmanager     |      | 'test-alerts' |      | (Python Consumer) |
+-------------------+      +---------------+      +---------+---------+
                                                            |
                                        +-------------------+-------------------+
                                        |                                       |
                                        v                                       v
                             +--------------------+                  +--------------------+
                             |  Chroma DB & LLM   |                  | Diagnostic Checks  |
                             |  (Runbook Context) |                  | (kubectl, metrics) |
                             +---------+----------+                  +---------+----------+
                                       |                                       |
                                       +-------------------+-------------------+
                                                           |
                                                           v
                                             +---------------------------+
                                             | Automated Remediation     |
                                             | (Ansible Playbooks)       |
                                             +-------------+-------------+
                                                           |
                                                           v
                                             +---------------------------+
                                             | Kubernetes Cluster        |
                                             | (Self-Healing Rollout)    |
                                             +---------------------------+
```

### Incident Lifecycle Flow

1. **Monitoring & Alerting:** Prometheus continuously monitors cluster health. Critical conditions trigger alerts sent via Alertmanager to Kafka topics.
2. **Alert Ingestion:** The Python agent consumer listens to Kafka events and ingests structured alert payloads.
3. **Diagnostic Execution:** The agent executes live cluster diagnostics (e.g., `kubectl top`, `kubectl get events`) to gather real-time state.
4. **Agentic Reasoning (RAG):** LangChain queries Chroma DB to retrieve relevant operational runbooks. The Gemini model analyzes the alert alongside diagnostic telemetry to determine the precise root cause.
5. **Automated Remediation:** The AI agent triggers targeted Ansible playbooks with dynamically generated extra variables (e.g., dynamic resource limit increases, deployment restarts).
6. **Verification & Rollout:** Ansible manages rolling updates to Kubernetes deployments and confirms successful rollout completion.

---

## ⚙️ Installation & Setup

### Prerequisites

Ensure you have the following installed and configured on your development machine/control node:

* **Python:** `3.10+`
* **Kubernetes Cluster:** `kubectl` configured with cluster-admin access (Minikube, Kind, or managed k8s)
* **Kafka Cluster:** Apache Kafka broker running locally or accessible over network
* **Ansible:** `ansible-core` installed locally
* **API Key:** Gemini API key for LLM reasoning

---

### Step 1: Clone Repository & Prepare Environment

```bash
# Clone the repository
git clone https://github.com/your-username/sre-project.git
cd sre-project/AlertsAIResponse

# Create and activate virtual environment
python3 -m venv venv
source venv/bin/activate

# Install Python dependencies
pip install -r requirements.txt
```

---

### Step 2: Configure Environment Variables

Create a `.env` file or export your required API key in your current shell:

```bash
export GEMINI_API_KEY="your-gemini-api-key-here"
export KAFKA_BOOTSTRAP_SERVERS="localhost:9092"
export KAFKA_TOPIC="test-alerts"
```

---

### Step 3: Populate Chroma Vector Database

Ingest the operational runbooks into Chroma DB so the RAG module can query relevant remediation strategies:

```bash
python ingest_runbooks.py
```

---

### Step 4: Run Agent Worker

Start the main agentic consumer listening to Kafka:

```bash
python agent_consumer.py
```

---

### Step 5: Simulate Test Alert (Optional)

In a separate terminal, trigger a test alert message to verify end-to-end self-healing:

```bash
python produce_test_alert.py --service payment-processor --alert HighCPUUsage --severity critical
```

---

## 🧪 Validated Test Case: High CPU / Memory Exhaustion Remediation

The platform was subjected to an automated test case involving an out-of-memory-induced high CPU usage incident.

### Scenario Logs & Verification

1. **Alert Ingestion & Triage**
   * **Service:** `payment-processor`
   * **Alert:** `HighCPUUsage` (Critical)
   * **Action:** Event consumed from Kafka topic `test-alerts`.

2. **Diagnostic Evaluation**
   * **Command:** `kubectl top pods -n default`
   * **Observation:** `payment-processor` pods operational at 0m CPU / low memory baseline, indicating past memory pressure/OOM condition.

3. **AI Reasoning Output**
   * **Root Cause Identified:** `oom_high_cpu`
   * **Selected Action:** Update container memory limit to alleviate constraints.
   * **Selected Playbook:** `playbooks/remediate_high_cpu.yml`
   * **Dynamic Parameters:** `{'target_service': 'payment-processor', 'namespace': 'default', 'root_cause': 'oom_high_cpu', 'memory_limit': '2Gi'}`

4. **Ansible Execution & Kubernetes State Change**

```text
PLAY [High CPU Automated Remediation] ******************************************

TASK [Determine remediation action] ********************************************
ok: [localhost] => {"msg": "Root Cause=oom_high_cpu -> Action=update_memory"}

TASK [Increase memory limit] ***************************************************
changed: [localhost]

TASK [Show rollout status] *****************************************************
ok: [localhost] => {"rollout_status.stdout": "deployment \"payment-processor\" successfully rolled out"}
```

5. **Cluster Verification (Before vs. After)**

| Resource Spec | Before Remediation | After Automated Self-Healing |
|---|---|---|
| **Memory Request** | `128Mi` | `128Mi` |
| **Memory Limit** | `1Gi` | **`2Gi`** |
| **Rollout Status** | Degraded / Pending OOM | **`successfully rolled out`** |

---

## 🚀 Key SRE & AIOps Benefits

* **Reduced Mean Time to Resolution (MTTR):** Decreases incident handling time from minutes/hours to seconds without human intervention.
* **Contextual Reasoning:** Combines static knowledge (runbooks) with real-time cluster state to eliminate brittle hardcoded alert rules.
* **Declarative Safety:** Uses Ansible playbooks as idempotent safety guardrails rather than executing raw, unvalidated commands on production clusters.
