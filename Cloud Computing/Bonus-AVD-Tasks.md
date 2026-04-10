# Bonus Lab: Azure Virtual Desktop (AVD)

> **Platform Explorers Cohort 2 — Azure Administration (AZ-104) Track**

| Detail | Value |
|--------|-------|
| **Duration** | 1–2 hours (deploy, test, DELETE immediately) |
| **Type** | Individual, Optional |
| **Difficulty** | Advanced |
| **Estimated Cost** | ~$15–20 for a few hours |
| **Prerequisites** | Azure subscription (Contributor), a VNet from UC1 or UC2 |

> ⚠️ **COST WARNING**: AVD requires session host VMs (B2s+) that cost ~$0.05/hour. Deploy, screenshot, and **DELETE ALL RESOURCES WITHIN THE SAME SESSION**. Do NOT leave running overnight. A forgotten B2s VM costs ~$1.20/day.



---

## Scenario

NovaTech wants to provide remote desktops for contractors who shouldn't have VPN access to the corporate network. Azure Virtual Desktop allows them to provide full Windows desktops accessible from any device, with data staying in Azure.

---

## Tasks

### Task 1: Create a Host Pool

**What to do:**

1. Search **"Azure Virtual Desktop"** → **Host pools** → **+ Create**
2. Configure:
   - Name: `hp-novatech-{id}`
   - Host pool type: **Pooled**
   - Load balancing: **Breadth-first**
   - Max session limit: 2
   - Region: East US
3. On the **Virtual Machines** tab: configure in Task 2

**Acceptance Criteria:**
- Host pool exists with type "Pooled" and breadth-first load balancing

**Hints:**
- [MS Learn: Create a host pool](https://learn.microsoft.com/en-us/azure/virtual-desktop/create-host-pool)
- **Pooled** = multiple users share VMs (cheaper). **Personal** = each user gets their own VM.
- **Breadth-first** = spread users across all VMs evenly. **Depth-first** = fill one VM before using the next.

---

### Task 2: Deploy a Session Host

**What to do:**

During host pool creation (or after, via **Session hosts** → **+ Add**):
1. Add 1 session host
2. VM size: **Standard_B2s** (2 vCPUs, 4 GB RAM — minimum for Windows)
3. Image: **Windows 11 Enterprise multi-session** (with Microsoft 365 apps if available)
4. VNet/Subnet: Use your existing WebSubnet
5. Domain join method: **Microsoft Entra ID** (simplest for this lab)

**Acceptance Criteria:**
- Session host VM is running
- Session host shows "Available" status in the host pool

---

### Task 3: Create an Application Group

**What to do:**

1. Go to **Azure Virtual Desktop** → **Application groups**
2. The host pool automatically created a **Desktop** application group
3. Go to it → **Assignments** → add your own user account (or a test user)

**Acceptance Criteria:**
- Desktop application group exists and is linked to the host pool
- Your user account is assigned to the application group

---

### Task 4: Create a Workspace

**What to do:**

1. **Azure Virtual Desktop** → **Workspaces** → **+ Create**
2. Name: `ws-novatech-{id}`
3. Register the Desktop application group to this workspace

**Acceptance Criteria:**
- Workspace exists with the application group registered
- Workspace is visible to assigned users

---

### Task 5: Connect via Remote Desktop

**What to do:**

1. Go to [https://client.wvd.microsoft.com/arm/webclient/index.html](https://client.wvd.microsoft.com/arm/webclient/index.html)
2. Sign in with your assigned user account
3. You should see the "Desktop" icon — click to connect
4. A full Windows desktop should load in your browser
5. Take a screenshot!

**Acceptance Criteria:**
- You can sign into the web client
- A Windows desktop loads successfully
- You can open apps (Edge, File Explorer) within the session

---

### Task 6: DELETE ALL AVD RESOURCES IMMEDIATELY

**What to do:**

⚠️ **Do this immediately after taking your screenshots!**

1. Delete the session host VM (and its disk, NIC, public IP)
2. Delete the host pool (remove session hosts first)
3. Delete the application group
4. Delete the workspace
5. Verify the VM is deallocated/deleted in the Portal

---

## 📸 Screenshot Submission Checklist

| # | Screenshot Required | Filename |
|---|-------------------|----------|
| 1 | Host Pool overview | `01-host-pool.png` |
| 2 | Session Host showing running status | `02-session-host.png` |
| 3 | Application Group with user assigned | `03-app-group.png` |
| 4 | Workspace overview | `04-workspace.png` |
| 5 | Remote Desktop session connected (showing Windows desktop) | `05-rdp-session.png` |
| 6 | All AVD resources deleted confirmation | `06-cleanup-confirmation.png` |

---

## Concepts Covered 

- AVD architecture: Host Pools → Session Hosts → Application Groups → Workspaces
- Pooled vs Personal host pools
- Breadth-first vs Depth-first load balancing
- Session host sizing and Windows multi-session licensing
- Cost implications of always-on VMs for virtual desktops

---

*Remember: DELETE EVERYTHING when done. This lab is for awareness only — don't let forgetting cost your team's monthly budget!*
