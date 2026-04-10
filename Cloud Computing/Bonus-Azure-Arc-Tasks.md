# Bonus Lab: Azure Arc — Hybrid Server Management

> **Platform Explorers Cohort 2 — Azure Administration (AZ-104) Track**

| Detail | Value |
|--------|-------|
| **Duration** | 1–2 hours (Optional, anytime) |
| **Type** | Individual, Optional |
| **Difficulty** | Intermediate |
| **Estimated Cost** | FREE (Azure Arc management is free) |
| **Prerequisites** | A running VM from Use Case 1 or 2 |

---

## Scenario

NovaTech has legacy servers that can't be migrated to Azure immediately. The IT team wants to manage these servers alongside Azure resources using a single pane of glass. You'll simulate this by onboarding one of your existing Azure VMs as an Arc-enabled server (pretending it's an on-premises machine).

---

## Tasks

### Task 1: Install Azure Arc Agent on a VM

**What to do:**

1. SSH into one of your existing VMs (from UC1 or UC2)
2. Download and run the Azure Arc agent installation script
3. Register the server with your Azure subscription

> **Note**: In a real scenario, this VM would be an on-premises server. We're using an Azure VM to simulate the experience.

**Acceptance Criteria:**
- Arc agent is installed and running on the VM
- The connected machine agent shows "Connected" status

**Hints:**
- [MS Learn: Connect hybrid machines with Azure Arc](https://learn.microsoft.com/en-us/azure/azure-arc/servers/learn/quick-enable-hybrid-vm)
- You'll need to generate an onboarding script from the Azure Portal

---

### Task 2: View Arc-Enabled Server in Azure Portal

**What to do:**

1. Search for **"Azure Arc"** in the portal
2. Go to **Servers** → verify your VM appears as an Arc-enabled server
3. Click on the server and explore the available options (Extensions, Policies, Update management)

**Acceptance Criteria:**
- Server appears in Azure Arc → Servers
- Status shows "Connected"
- You can see the server's properties (OS, agent version, last heartbeat)

---

### Task 3: Apply Tags to Arc-Enabled Server

**What to do:**

1. Go to the Arc-enabled server → **Tags**
2. Add tags: `ManagedBy=AzureArc`, `Location=OnPremises-Simulated`, `Department=IT`

**Acceptance Criteria:**
- All 3 tags visible on the Arc-enabled server

---

### Task 4: Run Azure Resource Graph Query

**What to do:**

1. Search for **"Resource Graph Explorer"** in the portal
2. Run this query to find all Arc-enabled servers:

```kusto
resources
| where type == "microsoft.hybridcompute/machines"
| project name, location, properties.status, tags
```

**Acceptance Criteria:**
- Query returns at least 1 result (your Arc-enabled server)
- You understand the KQL query syntax

---

### Task 5: (Optional) Enable VM Extensions via Arc

**What to do:**

1. Go to Arc-enabled server → **Extensions**
2. Install the **Azure Monitor Agent** extension
3. Verify in Log Analytics that the Arc server is sending heartbeat data

**Acceptance Criteria:**
- Extension shows "Provisioning succeeded"
- (Bonus) Heartbeat data appears in Log Analytics

---

## 📸 Screenshot Submission Checklist

| # | Screenshot Required | Filename |
|---|-------------------|----------|
| 1 | Arc agent installation success output | `01-arc-agent-install.png` |
| 2 | Arc-enabled server visible in Azure Portal | `02-arc-server-portal.png` |
| 3 | Tags applied to the Arc-enabled server | `03-arc-tags.png` |
| 4 | Resource Graph query results | `04-resource-graph.png` |

---

## AZ-104 Concepts Covered

- Azure Arc (hybrid and multi-cloud resource management)
- Azure Resource Graph (querying resources at scale with KQL)
- VM Extensions deployed via Arc
- Unified management pane for on-premises + cloud resources

---

*Azure Arc is free to use — you only pay for the extensions and services you enable on Arc-managed servers (like Azure Monitor, Defender for Cloud, etc.).*
