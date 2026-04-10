# Use Case 1: Building NovaTech's Cloud Foundation

> **Platform Explorers Cohort 2 — Azure Administration (AZ-104) Track**

| Detail | Value |
|--------|-------|
| **Duration** | Weeks 2–3 (11th – 18th April 2026) |
| **Type** | Individual Project |
| **Difficulty** | Beginner |
| **Estimated Cost** | ~$10–12 for the period |
| **Prerequisites** | Azure subscription (Contributor access), Azure CLI installed, VS Code, GitHub account |
| **Builds On** | None — this is the first use case |

---

## Scenario

### Company Overview

NovaTech Solutions is a mid-sized tech company with 150 employees in the United States. The company is migrating its on-premises infrastructure to Microsoft Azure to improve scalability, reduce hardware costs, and enable remote work capabilities.

### Problem Statement

NovaTech currently manages all IT infrastructure on-premises, leading to high maintenance costs, limited scalability, and disaster recovery concerns. The IT team has been tasked with establishing a cloud foundation in Azure that will serve as the base for all future workloads.

### Objective

As a junior Azure administrator, your job is to set up foundational Azure infrastructure for NovaTech — including networking, compute, and storage resources — following best practices. You must document your decisions and prove that the infrastructure works.

---

## Architecture Diagram (Target State)

```
┌──────────────────────────────────────────────────────────┐
│               Resource Group                              │
│           rg-novatech-{id}-prod-eastus                    │
│     Tags: Department=IT, Environment=Production,          │
│           CostCenter=1001, Owner=YourName                 │
│                                                           │
│  ┌──────────────────────────────────────────────────────┐ │
│  │       VNet: vnet-novatech-{id}-eastus                │ │
│  │       Address Space: 10.0.0.0/16                     │ │
│  │                                                      │ │
│  │  ┌────────────────┐  ┌──────────────┐  ┌──────────┐ │ │
│  │  │  WebSubnet     │  │  AppSubnet   │  │DataSubnet│ │ │
│  │  │ 10.0.1.0/24   │  │ 10.0.2.0/24  │  │10.0.3.0/ │ │ │
│  │  │               │  │              │  │  24      │ │ │
│  │  │ ┌───────────┐ │  │              │  │          │ │ │
│  │  │ │  VM:      │ │  │              │  │ No public│ │ │
│  │  │ │  web01    │ │  │              │  │ access   │ │ │
│  │  │ │  (B1s)    │ │  │              │  │          │ │ │
│  │  │ │  Nginx    │ │  │              │  │          │ │ │
│  │  │ └───────────┘ │  │              │  │          │ │ │
│  │  │  NSG: allow   │  │ NSG: allow   │  │NSG: deny │ │ │
│  │  │  80, 443      │  │ 8080 from    │  │internet  │ │ │
│  │  │               │  │ WebSubnet    │  │          │ │ │
│  │  └────────────────┘  └──────────────┘  └──────────┘ │ │
│  └──────────────────────────────────────────────────────┘ │
│                                                           │
│  ┌──────────────────┐  ┌──────────────────────────────┐   │
│  │  Storage Account │  │  Recovery Services Vault     │   │
│  │  stnovatech{id}  │  │  rsv-novatech-{id}-prod     │   │
│  │  prod001         │  │  Daily backup of VM          │   │
│  │  - Blob: assets  │  │                              │   │
│  │  - File: IT dept │  │                              │   │
│  └──────────────────┘  └──────────────────────────────┘   │
└──────────────────────────────────────────────────────────┘
```

---

## Naming Convention

Replace `{id}` with your unique student identifier (your initials, e.g., `sa`, `jd`, `mk`) in ALL resource names.

| Resource | Your Name |
|----------|-----------|
| Resource Group | `rg-novatech-{id}-prod-eastus` |
| VNet | `vnet-novatech-{id}-eastus` |
| WebSubnet NSG | `nsg-novatech-{id}-web` |
| AppSubnet NSG | `nsg-novatech-{id}-app` |
| DataSubnet NSG | `nsg-novatech-{id}-data` |
| Virtual Machine | `vm-novatech-{id}-web01` |
| Storage Account | `stnovatech{id}prod001` |
| Recovery Services Vault | `rsv-novatech-{id}-prod` |

> **Important**: Storage Account names must be globally unique, lowercase, 3–24 characters, no hyphens.

---

## Lab Tasks

### Task 1: Create Resource Group with Tags

**What to do:**

Create a resource group in the **East US** region with the following tags:

| Tag | Value |
|-----|-------|
| Department | IT |
| Environment | Production |
| CostCenter | 1001 |
| Owner | *Your full name* |

**Acceptance Criteria:**
- Resource group exists in East US
- All 4 tags are visible in the resource group overview

**Hints:**
- [MS Learn: Manage Azure resource groups](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/manage-resource-groups-portal)
- Tags are key-value pairs used for cost tracking and organization

---

### Task 2: Create Virtual Network with Subnets

**What to do:**

Create a Virtual Network with the following configuration:

| Setting | Value |
|---------|-------|
| Address space | 10.0.0.0/16 |
| Subnet 1 | WebSubnet — 10.0.1.0/24 |
| Subnet 2 | AppSubnet — 10.0.2.0/24 |
| Subnet 3 | DataSubnet — 10.0.3.0/24 |

**Acceptance Criteria:**
- VNet shows address space 10.0.0.0/16
- All 3 subnets are visible with correct address ranges
- No overlapping address ranges

**Hints:**
- [MS Learn: Create a virtual network](https://learn.microsoft.com/en-us/azure/virtual-network/quick-create-portal)
- Think about why you'd separate web, app, and data tiers into different subnets

---

### Task 3: Configure Network Security Groups (NSGs)

**What to do:**

Create three NSGs and associate them with their respective subnets:

| NSG | Rules | Associate With |
|-----|-------|----------------|
| `nsg-novatech-{id}-web` | Allow inbound HTTP (80) and HTTPS (443) from Any | WebSubnet |
| `nsg-novatech-{id}-app` | Allow inbound port 8080 from **WebSubnet only** (10.0.1.0/24) | AppSubnet |
| `nsg-novatech-{id}-data` | Deny all inbound from Internet. Allow inbound from **AppSubnet only** (10.0.2.0/24) | DataSubnet |

**Acceptance Criteria:**
- Each NSG exists and is associated with the correct subnet
- WebNSG allows 80 and 443 from Any source
- AppNSG allows 8080 only from 10.0.1.0/24
- DataNSG has an explicit deny for Internet and allow from 10.0.2.0/24

**Hints:**
- [MS Learn: Network security groups](https://learn.microsoft.com/en-us/azure/virtual-network/network-security-groups-overview)
- NSG rules are evaluated by priority number (lower = higher priority)
- Think about the principle of least privilege — why not just allow everything?

---

### Task 4: Deploy a Linux Virtual Machine

**What to do:**

Deploy a VM with the following configuration:

| Setting | Value |
|---------|-------|
| Image | Ubuntu Server 22.04 LTS |
| Size | Standard_B1s |
| Subnet | WebSubnet |
| Public IP | Yes (for testing — in production you'd use a load balancer) |
| Authentication | SSH public key (recommended) or password |
| Inbound ports | SSH (22) — for initial setup only |

After the VM is deployed:
1. SSH into the VM
2. Install Nginx: `sudo apt update && sudo apt install nginx -y`
3. Verify Nginx is running: `sudo systemctl status nginx`
4. Open a browser and navigate to the VM's public IP — you should see the Nginx welcome page

**Acceptance Criteria:**
- VM is running with status "Running" in the Portal
- VM size is Standard_B1s
- VM is in WebSubnet
- Nginx welcome page loads in browser at the VM's public IP

**Hints:**
- [MS Learn: Create a Linux VM](https://learn.microsoft.com/en-us/azure/virtual-machines/linux/quick-create-portal)
- B1s is the smallest (cheapest) general-purpose VM — perfect for dev/test
- If Nginx page doesn't load, check: Is the NSG allowing port 80? Is the VM running?

---

### Task 5: Create Storage Account with Blob and File Share

**What to do:**

Create a Storage Account and configure:

| Setting | Value |
|---------|-------|
| Name | `stnovatech{id}prod001` |
| Performance | Standard |
| Redundancy | LRS (Locally Redundant Storage) |
| Blob container | `company-assets` (Private access level) |
| File share | `it-department` (Quota: 5 GB) |

Upload a test file (any small file — a text file, image, etc.) to the blob container.

**Acceptance Criteria:**
- Storage Account exists with LRS redundancy
- Blob container `company-assets` exists with Private access level
- File share `it-department` exists with 5 GB quota
- At least one test file is uploaded to the blob container

**Hints:**
- [MS Learn: Create a storage account](https://learn.microsoft.com/en-us/azure/storage/common/storage-account-create)
- Why Private access instead of Public? Think about what would happen if anyone on the internet could read your blobs
- LRS keeps 3 copies in one datacenter. When would you choose GRS instead?

---

### Task 6: Configure Azure Backup

**What to do:**

1. Create a Recovery Services vault
2. Create a backup policy with daily backups at 11:00 PM, retained for 7 days
3. Enable backup for your VM using this policy

**Acceptance Criteria:**
- Recovery Services vault exists
- Backup policy shows daily backup at 11:00 PM, 7-day retention
- VM appears as a protected item in the vault

**Hints:**
- [MS Learn: Back up a VM](https://learn.microsoft.com/en-us/azure/backup/quick-backup-vm-portal)
- A backup without a tested restore is not a backup — think about what RPO and RTO mean
- The vault must be in the same region as the VM

---

### Task 7: 🔒 Security Review

**What to do:**

1. Navigate to **Azure Advisor** in the portal (search "Advisor" in the top search bar)
2. Review recommendations across all categories: Security, Reliability, Cost, Performance, Operational Excellence
3. Take a screenshot of the recommendations page

**Answer the following questions in your `thought-process.md` file:**

1. Why is the DataSubnet NSG configured to deny all internet traffic? What types of resources would go in this subnet and why shouldn't they be internet-accessible?
2. What would happen if the Storage Account allowed public blob access instead of private? Describe a real-world scenario where this could cause a data breach.
3. What is your Azure Advisor score? What specific improvements does it suggest? Would you implement all of them — why or why not?
4. Your VM currently has a public IP and SSH (port 22) open. In a production environment, what would you do differently?
5. You chose LRS for the Storage Account. A manager asks: "What happens if the entire datacenter goes down?" How do you respond?

---

## 📸 Screenshot Submission Checklist

Upload all screenshots to your GitHub repo: `UseCase1-Cloud-Foundation/screenshots/`

| # | Screenshot Required | Filename |
|---|-------------------|----------|
| 1 | Resource group overview showing name, region, and all 4 tags | `01-resource-group-tags.png` |
| 2 | VNet overview showing address space (10.0.0.0/16) and all 3 subnets | `02-vnet-subnets.png` |
| 3 | Each NSG's inbound security rules (3 screenshots — one per NSG) | `03a-nsg-web-rules.png`, `03b-nsg-app-rules.png`, `03c-nsg-data-rules.png` |
| 4 | VM overview page showing name, size (B1s), and running status | `04a-vm-overview.png` |
| 5 | Nginx welcome page in browser with public IP visible in URL bar | `04b-nginx-browser.png` |
| 6 | Storage Account overview showing blob container and file share | `05-storage-overview.png` |
| 7 | Recovery Services vault showing backup policy and protected VM | `06-backup-vault.png` |
| 8 | Azure Advisor recommendations page (all categories) | `07-azure-advisor.png` |
| 9 | Empty resource group or deleted confirmation after cleanup | `08-cleanup-confirmation.png` |

---

## Cleanup Checklist

⚠️ **You MUST delete all resources after completing this use case to stay within your $30/month budget.**

Delete the following resources (you determine the correct order — hint: some resources depend on others):

- [ ] Recovery Services vault (and backup items inside it)
- [ ] Virtual Machine and associated resources (disk, NIC, public IP)
- [ ] Storage Account
- [ ] Network Security Groups
- [ ] Virtual Network
- [ ] Resource Group

> **Tip**: If you try to delete a resource and get an error about dependencies, that's a clue about the correct deletion order.

---

## Challenge Questions

Answer these in your `challenge-answers.md` file. These are AZ-104 and AZ-305 exam-style scenario questions.

1. **Scenario**: NovaTech's CTO asks you to ensure the web server is accessible from the internet but the database server (in DataSubnet) should never be directly reachable. A new developer accidentally creates a VM in DataSubnet with a public IP. What Azure feature would **prevent** this from happening again? *(Hint: Think beyond NSGs)*

2. **Scenario**: The CFO is concerned about the cost of running all infrastructure in a single region. They ask: "What happens to our web server if the East US region goes completely offline?" How would you design for this? What Azure services would you recommend?

3. **Scenario**: An auditor asks you: "How do you know who created or modified resources in your Azure environment?" Which Azure service provides this audit trail, and where would you find it?

4. **Scenario**: NovaTech's storage account currently uses LRS. The company handles customer financial data. The compliance team requires data to survive a complete datacenter failure. What would you change, and what are the cost implications?

5. **Scenario**: You need to give a contractor temporary access to upload files to the blob container without giving them full Storage Account keys. What Azure feature allows this?

---

## Documentation Requirements

In addition to screenshots, create the following files in your GitHub repo:

### `thought-process.md`
For each task, write 2–3 sentences explaining:
- **What** you did
- **Why** you chose that specific configuration (e.g., why B1s? why LRS? why those NSG rules?)
- **What you learned** or found surprising

### `challenge-answers.md`
Written answers to all 5 challenge questions above.

---

## GitHub Repo Structure

```
UseCase1-Cloud-Foundation/
├── screenshots/
│   ├── 01-resource-group-tags.png
│   ├── 02-vnet-subnets.png
│   ├── 03a-nsg-web-rules.png
│   ├── 03b-nsg-app-rules.png
│   ├── 03c-nsg-data-rules.png
│   ├── 04a-vm-overview.png
│   ├── 04b-nginx-browser.png
│   ├── 05-storage-overview.png
│   ├── 06-backup-vault.png
│   ├── 07-azure-advisor.png
│   └── 08-cleanup-confirmation.png
├── thought-process.md
└── challenge-answers.md
```

---

*Good luck! Remember: try every task yourself first before looking at any external walkthroughs. The learning happens when you get stuck and figure it out.*
