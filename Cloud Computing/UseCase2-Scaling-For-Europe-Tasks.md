# Use Case 2: Scaling NovaTech for Europe

> **Platform Explorers Cohort 2 — Azure Administration (AZ-104) Track**

| Detail | Value |
|--------|-------|
| **Duration** | Weeks 5–6 (2nd – 9th May 2026) |
| **Type** | Individual Project |
| **Difficulty** | Intermediate |
| **Estimated Cost** | ~$15–18 for the period |
| **Prerequisites** | Completed Use Case 1, Azure CLI, VS Code, GitHub account |
| **Builds On** | Use Case 1 — NovaTech's US-based cloud foundation |

---

## Scenario

### Company Overview

NovaTech Solutions has successfully established its Azure foundation in the US (Use Case 1). The company is now expanding operations to Western Europe with a new team of 50 employees who need access to the same applications and data as the US team.

### Problem Statement

NovaTech's European team needs reliable connectivity to the US infrastructure, local compute capacity in Europe, and the company requires load balancing for the web tier, containerized microservices, centralized secrets management, and comprehensive monitoring to track infrastructure health and costs across both regions.

### Objective

As an Azure administrator, extend NovaTech's infrastructure to Europe, implement cross-region networking, load balancing, container deployment, secrets management with Key Vault, and set up monitoring and cost alerts.

---

## Architecture Diagram (Target State)

```
                    ┌─────────────────────────────┐
                    │   Private DNS Zone           │
                    │   novatech-{id}.internal     │
                    │   Linked to both VNets       │
                    └──────────┬──────────────────┘
                               │
          ┌────────────────────┼────────────────────┐
          │                    │                     │
          ▼                    │                     ▼
┌─────────────────────┐       │       ┌─────────────────────────┐
│  East US             │  VNet Peering │  West Europe             │
│  rg-novatech-{id}-  │◄────────────►│  rg-novatech-{id}-       │
│  prod-eastus         │              │  prod-westeurope          │
│                      │              │                           │
│ vnet: 10.0.0.0/16   │              │ vnet: 172.16.0.0/16      │
│                      │              │                           │
│ ┌──────────────────┐ │              │ ┌───────────────────────┐ │
│ │ WebSubnet        │ │              │ │ WebSubnet             │ │
│ │ 10.0.1.0/24      │ │              │ │ 172.16.1.0/24         │ │
│ │                  │ │              │ │                       │ │
│ │ ┌──────────────┐ │ │              │ │ ┌───────────────────┐ │ │
│ │ │ Load Balancer│ │ │              │ │ │ ACI: Nginx        │ │ │
│ │ │ lb-{id}-web  │ │ │              │ │ │ aci-{id}-api      │ │ │
│ │ │  ┌───┐ ┌───┐│ │ │              │ │ └───────────────────┘ │ │
│ │ │  │VM1│ │VM2││ │ │              │ └───────────────────────┘ │
│ │ │  └───┘ └───┘│ │ │              │                           │
│ │ └──────────────┘ │ │              │ ┌───────────────────────┐ │
│ └──────────────────┘ │              │ │ AppSubnet             │ │
│                      │              │ │ 172.16.2.0/24         │ │
│ ┌──────────────────┐ │              │ └───────────────────────┘ │
│ │ Key Vault        │ │              │                           │
│ │ kv-{id}-prod     │ │              │ ┌───────────────────────┐ │
│ └──────────────────┘ │              │ │ DataSubnet            │ │
│                      │              │ │ 172.16.3.0/24         │ │
│ ┌──────────────────┐ │              │ └───────────────────────┘ │
│ │ Log Analytics    │ │              └─────────────────────────┘
│ │ + Monitor Alerts │ │
│ │ + Budget ($30)   │ │
│ └──────────────────┘ │
└─────────────────────┘
```

---

## Naming Convention

Replace `{id}` with your unique student identifier in ALL resource names.

| Resource | Your Name |
|----------|-----------|
| Europe Resource Group | `rg-novatech-{id}-prod-westeurope` |
| Europe VNet | `vnet-novatech-{id}-westeurope` |
| VNet Peering (US→EU) | `peer-eastus-to-westeurope` |
| VNet Peering (EU→US) | `peer-westeurope-to-eastus` |
| Private DNS Zone | `novatech-{id}.internal` |
| Load Balancer | `lb-novatech-{id}-web` |
| Web VM 1 | `vm-novatech-{id}-web01` |
| Web VM 2 | `vm-novatech-{id}-web02` |
| ACI | `aci-novatech-{id}-api` |
| Key Vault | `kv-novatech-{id}-prod` |
| Log Analytics | `log-novatech-{id}-prod` |
| Budget | `budget-novatech-{id}` |

---

## Lab Tasks

### Task 1: Create Europe Resource Group

**What to do:**

Create a resource group in the **West Europe** region with the same tagging strategy as Use Case 1:

| Tag | Value |
|-----|-------|
| Department | IT |
| Environment | Production |
| CostCenter | 1002 |
| Owner | *Your full name* |

**Acceptance Criteria:**
- Resource group exists in West Europe with all 4 tags

**Hints:**
- Same process as UC1 Task 1, just different region and CostCenter value

---

### Task 2: Create Europe Virtual Network

**What to do:**

Create a VNet in West Europe mirroring the US structure:

| Setting | Value |
|---------|-------|
| Address space | 172.16.0.0/16 |
| Subnet 1 | WebSubnet — 172.16.1.0/24 |
| Subnet 2 | AppSubnet — 172.16.2.0/24 |
| Subnet 3 | DataSubnet — 172.16.3.0/24 |

> ⚠️ **Critical**: The Europe VNet MUST use a different address space (172.16.0.0/16) from the US VNet (10.0.0.0/16). Overlapping address spaces prevent VNet Peering.

**Acceptance Criteria:**
- VNet created in West Europe with 172.16.0.0/16
- All 3 subnets exist with correct address ranges
- No overlap with the US VNet

**Hints:**
- [MS Learn: Virtual network peering prerequisites](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-network-peering-overview#requirements-and-constraints)
- Think about: why can't two peered VNets share the same address space?

---

### Task 3: Configure VNet Peering

**What to do:**

Create **bidirectional** VNet Peering between the US and Europe VNets:
- US VNet → Europe VNet (name: `peer-eastus-to-westeurope`)
- Europe VNet → US VNet (name: `peer-westeurope-to-eastus`)

**Acceptance Criteria:**
- Both peering connections show status **"Connected"**
- Allow forwarded traffic is enabled on both sides
- Allow gateway transit is disabled (no VPN gateway in this lab)

**Hints:**
- [MS Learn: Create VNet peering](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-network-manage-peering)
- Peering is NOT transitive — if VNet A peers with B, and B peers with C, A cannot reach C through B
- You must create peering from BOTH sides

---

### Task 4: Create Private DNS Zone

**What to do:**

1. Create a Private DNS Zone: `novatech-{id}.internal`
2. Link it to BOTH the US and Europe VNets (enable auto-registration on both links)
3. After linking, VMs should automatically get DNS records

**Acceptance Criteria:**
- DNS Zone exists with the name `novatech-{id}.internal`
- Two VNet links are visible (one for each VNet)
- Auto-registration is enabled on both links
- A records appear for any existing VMs

**Hints:**
- [MS Learn: Private DNS zones](https://learn.microsoft.com/en-us/azure/dns/private-dns-privatednszone)
- Auto-registration automatically creates DNS records for VMs in the linked VNets
- This means VMs can reach each other by name (e.g., `vm-novatech-sa-web01.novatech-sa.internal`) instead of IP

---

### Task 5: Deploy Load Balancer with Two VMs

**What to do:**

1. Deploy **2 VMs** (B1s, Ubuntu 22.04) in the **US WebSubnet**:
   - `vm-novatech-{id}-web01`
   - `vm-novatech-{id}-web02`
   - Install Nginx on both (same as UC1)
   - **Do NOT assign public IPs** to these VMs — they'll be behind the LB
2. Create a **Standard Load Balancer** (public):
   - **Frontend IP**: New public IP
   - **Backend pool**: Both VMs
   - **Health probe**: HTTP on port 80, interval 5 seconds
   - **Load balancing rule**: HTTP (port 80) → backend pool port 80

**Acceptance Criteria:**
- Both VMs are running (no public IPs)
- Load Balancer has a public IP
- Backend pool shows both VMs as "Healthy"
- Browsing to the LB's public IP shows the Nginx welcome page
- Refreshing multiple times continues to work (traffic distributed)

**Hints:**
- [MS Learn: Create a public load balancer](https://learn.microsoft.com/en-us/azure/load-balancer/quickstart-load-balancer-standard-public-portal)
- The Load Balancer must be **Standard SKU** (Basic is being retired)
- VMs behind a Standard LB need to be in the same VNet and can't have individual public IPs (by design)
- If health probes fail, the LB won't send traffic to that VM

---

### Task 6: Deploy Azure Container Instance

**What to do:**

Deploy a container instance in the **Europe WebSubnet** running Nginx:

| Setting | Value |
|---------|-------|
| Name | `aci-novatech-{id}-api` |
| Image | `nginx:latest` (from Docker Hub) |
| OS type | Linux |
| CPU | 1 |
| Memory | 1.5 GB |
| Networking | Public IP (for testing) |
| Restart policy | Always |

**Acceptance Criteria:**
- ACI shows status "Running"
- Nginx is accessible via the ACI's public IP
- Container logs show Nginx started successfully

**Hints:**
- [MS Learn: Deploy a container instance](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-quickstart-portal)
- ACI is serverless containers — no VM to manage, pay per second
- Think about: when would you use ACI vs. a VM vs. AKS?

---

### Task 7: Create Key Vault and Store Secret

**What to do:**

1. Create an Azure Key Vault with **purge protection enabled**
2. Store a secret:
   - **Name**: `DbConnectionString`
   - **Value**: `Server=tcp:novatech-db.database.windows.net;Database=NovaTechDB;` (this is a fake connection string for practice)
3. From your VM (SSH in), retrieve the secret using Azure CLI:
   ```
   az keyvault secret show --vault-name kv-novatech-{id}-prod --name DbConnectionString --query value -o tsv
   ```
   *(You'll need to install Azure CLI on the VM or use Cloud Shell)*

**Acceptance Criteria:**
- Key Vault exists with purge protection enabled
- Secret `DbConnectionString` is stored (value hidden in Portal)
- You can retrieve the secret value via CLI (screenshot the command output, redact the value)

**Hints:**
- [MS Learn: Set and retrieve a secret](https://learn.microsoft.com/en-us/azure/key-vault/secrets/quick-create-portal)
- Purge protection prevents permanent deletion for a retention period (even by admins)
- In production, applications access Key Vault via **Managed Identity** — not CLI

---

### Task 8: Create Log Analytics Workspace & Diagnostic Settings

**What to do:**

1. Create a Log Analytics workspace: `log-novatech-{id}-prod` in East US
2. Enable diagnostic settings on:
   - Both VMs (send Guest OS metrics to Log Analytics)
   - The US VNet (send flow logs to Log Analytics)
3. Wait ~10 minutes for data to appear, then run a basic KQL query:
   ```kql
   Heartbeat
   | where TimeGenerated > ago(1h)
   | summarize count() by Computer
   ```

**Acceptance Criteria:**
- Log Analytics workspace exists
- At least one diagnostic setting is configured and sending data
- You can run a KQL query and see results

**Hints:**
- [MS Learn: Create a Log Analytics workspace](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/quick-create-workspace)
- Diagnostic settings tell Azure resources WHERE to send their logs (Log Analytics, Storage Account, Event Hub)
- KQL (Kusto Query Language) is the query language for Log Analytics — similar to SQL but different

---

### Task 9: Create Azure Monitor Alerts

**What to do:**

Create two alert rules:

| Alert | Condition | Threshold | Action |
|-------|-----------|-----------|--------|
| High CPU | VM CPU Percentage | > 80% for 5 minutes | Email action group |
| High Disk | VM OS Disk Used Percentage | > 90% for 5 minutes | Email action group |

Create an **Action Group** first:
- Name: `ag-novatech-{id}-email`
- Notification type: Email
- Email: Your email address

**Acceptance Criteria:**
- Action group exists with your email configured
- Both alert rules exist and are enabled
- Alert conditions reference the correct VMs and thresholds

**Hints:**
- [MS Learn: Create metric alerts](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-create-metric-alert-rule)
- Action groups define WHO gets notified and HOW (email, SMS, webhook, Azure Function)
- Alert rules define WHEN to trigger (which metric, what threshold, how long)

---

### Task 10: Create Azure Budget

**What to do:**

Create a budget to track your monthly spending:

| Setting | Value |
|---------|-------|
| Name | `budget-novatech-{id}` |
| Amount | $30 |
| Reset period | Monthly |
| Alert 1 | Actual cost at 50% ($15) — email notification |
| Alert 2 | Actual cost at 80% ($24) — email notification |
| Alert 3 | Actual cost at 100% ($30) — email notification |

**Acceptance Criteria:**
- Budget exists with $30 limit
- Three alert thresholds are configured (50%, 80%, 100%)
- Your email is set as the notification recipient

**Hints:**
- [MS Learn: Create and manage budgets](https://learn.microsoft.com/en-us/azure/cost-management-billing/costs/tutorial-acm-create-budgets)
- Budgets don't STOP spending — they only ALERT you. There's no automatic shutoff.
- Check your current spending in **Cost Management + Billing** → **Cost analysis**

---

### Task 11: 🔒 Security Review

**What to do:**

1. Navigate to **Azure Advisor** → review recommendations for BOTH regions
2. Navigate to **Microsoft Defender for Cloud** → review security posture (read-only)

**Answer the following questions in your `thought-process.md`:**

1. Compare Azure Advisor security scores between East US and West Europe resource groups. Why might they differ?
2. Are your Load Balancer health probes using HTTP or HTTPS? Which is more secure and why? What's the trade-off?
3. Is your Key Vault using **RBAC** or **access policies** for permission management? Which does Microsoft recommend for new deployments and why?
4. Are NSG flow logs enabled on your VNets? Why are flow logs important for security incident investigation?
5. Your ACI container uses the `nginx:latest` image from Docker Hub. Is this a trusted source? What risks does using public container images introduce? What would you do differently in production?
6. What Azure Advisor cost recommendations appear for your multi-region setup? Would you implement them?

---

## 📸 Screenshot Submission Checklist

Upload all screenshots to your GitHub repo: `UseCase2-Scaling-For-Europe/screenshots/`

| # | Screenshot Required | Filename |
|---|-------------------|----------|
| 1 | Europe resource group overview with tags | `01-europe-resource-group.png` |
| 2 | Europe VNet overview with address space and subnets | `02-europe-vnet.png` |
| 3 | VNet Peering status showing "Connected" on both sides | `03a-peering-eastus.png`, `03b-peering-westeurope.png` |
| 4 | Private DNS Zone showing record sets and VNet links | `04-dns-zone.png` |
| 5 | Load Balancer overview with backend pool (2 VMs), health probe, and LB rule | `05a-lb-overview.png`, `05b-lb-backend-pool.png`, `05c-lb-health-probe.png` |
| 6 | Browser showing Nginx via Load Balancer public IP | `05d-lb-browser-test.png` |
| 7 | ACI container instance overview showing running status | `06-aci-overview.png` |
| 8 | Key Vault overview showing the secret (name visible, value hidden) | `07-keyvault-secret.png` |
| 9 | Log Analytics workspace with at least one diagnostic setting connected | `08-log-analytics.png` |
| 10 | Azure Monitor alert rule showing CPU > 80% condition | `09-monitor-alert.png` |
| 11 | Azure Budget showing $30 limit with alert thresholds | `10-budget.png` |
| 12 | Azure Advisor recommendations page for both regions | `11a-advisor-eastus.png`, `11b-advisor-westeurope.png` |
| 13 | Empty resource groups or deleted confirmation after cleanup | `12-cleanup-confirmation.png` |

---

## Cleanup Checklist

⚠️ **You MUST delete all resources after completing this use case.**

Delete the following resources (you determine the correct order):

- [ ] Budget and Action Groups
- [ ] Azure Monitor alerts
- [ ] Log Analytics workspace
- [ ] Key Vault (soft-delete retained 90 days, purge if needed)
- [ ] ACI container
- [ ] Load Balancer, then backend VMs (NICs, disks, public IPs)
- [ ] DNS Zone and VNet links
- [ ] VNet Peering from both sides
- [ ] Europe VNet and NSGs
- [ ] US VNet and NSGs
- [ ] Resource groups (both)

---

## Challenge Questions

Answer these in your `challenge-answers.md` file.

1. **Scenario**: NovaTech's CTO says: "VNet Peering is too expensive for our use case. Can we just use public IPs on both VMs and communicate over the internet?" What are the security and performance risks? What alternative to peering exists for hybrid connectivity?

2. **Scenario**: Your Load Balancer distributes traffic to 2 VMs, but during peak hours the web application slows down. The CEO asks for automatic scaling. What Azure service would you recommend instead of manually adding VMs? How would autoscale rules work?

3. **Scenario**: A developer stores the database connection string directly in their application code and pushes it to GitHub. How does Key Vault prevent this? Describe the flow: application → Key Vault → secret.

4. **Scenario**: The security team asks you to prove that no unauthorized traffic reached the DataSubnet in the last 30 days. Which Azure feature provides this data? What tool would you use to analyze it?

5. **Scenario**: NovaTech wants to deploy a small API in Europe but doesn't want to manage VMs. You chose ACI. A colleague suggests Azure Kubernetes Service (AKS) instead. When would ACI be the wrong choice and AKS the right one?

---

## Documentation Requirements

### `thought-process.md`
For each task, explain:
- Why you chose that configuration
- What challenges you encountered
- How this builds on Use Case 1
- Your security review answers (Task 11)

### `challenge-answers.md`
Written answers to all 5 challenge questions.

---

## GitHub Repo Structure

```
UseCase2-Scaling-For-Europe/
├── screenshots/
│   ├── 01-europe-resource-group.png
│   ├── 02-europe-vnet.png
│   ├── 03a-peering-eastus.png
│   ├── 03b-peering-westeurope.png
│   ├── 04-dns-zone.png
│   ├── 05a-lb-overview.png
│   ├── 05b-lb-backend-pool.png
│   ├── 05c-lb-health-probe.png
│   ├── 05d-lb-browser-test.png
│   ├── 06-aci-overview.png
│   ├── 07-keyvault-secret.png
│   ├── 08-log-analytics.png
│   ├── 09-monitor-alert.png
│   ├── 10-budget.png
│   ├── 11a-advisor-eastus.png
│   ├── 11b-advisor-westeurope.png
│   └── 12-cleanup-confirmation.png
├── thought-process.md
└── challenge-answers.md
```

---

*This use case is more complex than UC1. Take your time — break it into two sessions (Week 5: Tasks 1–6, Week 6: Tasks 7–11). Don't forget to clean up!*
