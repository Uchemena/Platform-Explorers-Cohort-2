# Use Case 3: NovaTech's AI-Powered Customer Portal

> **Platform Explorers Cohort 2 — Azure Administration (AZ-104) Track**

| Detail | Value |
|--------|-------|
| **Duration** | Weeks 8–9 (23rd – 30th May 2026) |
| **Type** | Individual Project |
| **Difficulty** | Intermediate–Advanced |
| **Estimated Cost** | ~$5–10 for the period |
| **Prerequisites** | Completed Use Cases 1 & 2, Azure CLI, VS Code, GitHub account |
| **Builds On** | Use Cases 1 & 2 — NovaTech's multi-region infrastructure |

---

## Scenario

### Company Overview

NovaTech Solutions now has a robust multi-region Azure infrastructure spanning the US and Europe (Use Cases 1 & 2). The company wants to differentiate itself by launching an AI-powered customer support portal.

### Problem Statement

NovaTech's customer support team handles 500+ tickets per week, with 60% being repetitive questions about products, pricing, and troubleshooting. The company wants to deploy an AI assistant using Azure AI Foundry to handle these common queries, reducing response times and freeing the support team for complex issues.

### Objective

As an Azure administrator, deploy an AI-powered customer support assistant using Azure AI Foundry, host it on Azure App Service, add a serverless webhook via Azure Functions, export infrastructure as code with Bicep, and configure monitoring for production readiness.

---

## Architecture Diagram (Target State)

```
┌─────────────────────────────────────────────────────────────────┐
│            Resource Group: rg-novatech-{id}-ai-eastus           │
│                                                                  │
│  ┌─────────────────┐     ┌──────────────────────────────────┐   │
│  │  AI Foundry Hub │     │  App Service                     │   │
│  │  aih-{id}-prod  │     │  app-novatech-{id}-support       │   │
│  │                 │     │                                  │   │
│  │  ┌────────────┐ │     │  Chat UI (HTML/JS)               │   │
│  │  │  Project   │ │ ◄───│  Env vars:                       │   │
│  │  │  aip-{id}  │ │     │   AZURE_OPENAI_ENDPOINT          │   │
│  │  │  -support  │ │     │   AZURE_OPENAI_KEY                │   │
│  │  │            │ │     │   AZURE_OPENAI_DEPLOYMENT         │   │
│  │  │ GPT-4o-mini│ │     └──────────────────────────────────┘   │
│  │  └────────────┘ │                                             │
│  └─────────────────┘     ┌──────────────────────────────────┐   │
│                          │  Azure Function                   │   │
│                          │  func-novatech-{id}-webhook       │   │
│                          │                                   │   │
│                          │  HTTP Trigger → Storage Queue     │   │
│                          │  (logs support requests)          │   │
│                          └──────────────────────────────────┘   │
│                                                                  │
│  ┌─────────────────┐     ┌──────────────────────────────────┐   │
│  │  Monitor        │     │  (Optional)                      │   │
│  │  Dashboard      │     │  Private Endpoint                │   │
│  │  API usage      │     │  for AI Foundry                  │   │
│  │  Error rate     │     │                                  │   │
│  │  Latency        │     │                                  │   │
│  └─────────────────┘     └──────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

---

## Naming Convention

| Resource | Your Name |
|----------|-----------|
| Resource Group | `rg-novatech-{id}-ai-eastus` |
| AI Foundry Hub | `aih-novatech-{id}-prod` |
| AI Foundry Project | `aip-novatech-{id}-support` |
| App Service | `app-novatech-{id}-support` |
| App Service Plan | `asp-novatech-{id}-support` |
| Function App | `func-novatech-{id}-webhook` |

---

## Lab Tasks

### Task 1: Create AI Foundry Hub and Project

**What to do:**

1. Create a new resource group `rg-novatech-{id}-ai-eastus` in East US
2. Navigate to [Azure AI Foundry portal](https://ai.azure.com)
3. Create a new Hub: `aih-novatech-{id}-prod`
4. Inside the Hub, create a Project: `aip-novatech-{id}-support`

**Acceptance Criteria:**
- AI Foundry Hub exists in your resource group
- Project `aip-novatech-{id}-support` is created inside the hub
- You can access the project in the AI Foundry portal

**Hints:**
- [MS Learn: Create an AI Foundry hub](https://learn.microsoft.com/en-us/azure/ai-studio/how-to/create-azure-ai-resource)
- The Hub is the management container; Projects are where you do the work
- Creating a Hub also creates: a Storage Account, Key Vault, and Application Insights resource automatically

---

### Task 2: Deploy GPT-4o-mini Model

**What to do:**

1. In your AI Foundry project, go to **Deployments** → **+ Deploy model**
2. Select **gpt-4o-mini** (cost-efficient for this use case)
3. Deployment name: `gpt-4o-mini`
4. Configure a **system prompt** with the following NovaTech FAQ content:

```
You are NovaTech's AI Customer Support Assistant. You help customers with questions
about NovaTech's products and services. Be friendly, professional, and concise.

NovaTech Products:
- NovaTech Pro Suite ($49/month): Project management, time tracking, invoicing
- NovaTech Team Hub ($29/month): Team chat, file sharing, video calls
- NovaTech Analytics ($79/month): Business intelligence, dashboards, reporting

Common Questions:
- Free trial: All products offer a 14-day free trial. No credit card required.
- Cancellation: Cancel anytime from Account Settings → Subscription → Cancel.
- Data export: Go to Settings → Data → Export. Formats: CSV, JSON, PDF.
- Uptime SLA: 99.9% for all paid plans.
- Support hours: Mon-Fri 9AM-6PM EST. Premium support: 24/7.
- Integrations: Slack, Microsoft Teams, Google Workspace, Salesforce, Jira.
- SSO: Available on Pro Suite and Analytics plans. SAML 2.0 and OpenID Connect.

If you don't know the answer, say: "I don't have that information. Let me connect
you with a human agent. Please describe your issue and our team will respond within
4 business hours."
```

**Acceptance Criteria:**
- GPT-4o-mini model is deployed and shows "Succeeded" status
- System prompt is configured with the NovaTech FAQ content
- Deployment name is visible

**Hints:**
- [MS Learn: Deploy models in AI Foundry](https://learn.microsoft.com/en-us/azure/ai-studio/how-to/deploy-models-openai)
- GPT-4o-mini costs ~$0.15/1M input tokens — very economical for testing

---

### Task 3: Test in AI Foundry Playground

**What to do:**

Test the assistant with these 5 customer scenarios in the Playground:

| # | Customer Query |
|---|---------------|
| 1 | "How much does NovaTech Pro Suite cost? Is there a free trial?" |
| 2 | "I want to cancel my subscription. How do I do that?" |
| 3 | "Does NovaTech integrate with Slack and Microsoft Teams?" |
| 4 | "I'm having trouble logging in. My password reset email never arrived." |
| 5 | "Can I export my data if I decide to leave NovaTech?" |

**Acceptance Criteria:**
- All 5 queries receive relevant, accurate responses
- Query 4 should trigger the "connect you with a human agent" response (it's not in the FAQ)
- Responses are professional and concise

**Hints:**
- The Playground allows you to test the model without writing code
- Try adjusting the **temperature** (0 = deterministic, 1 = creative) and see how responses change
- System prompt quality directly impacts response quality — if answers are wrong, improve the prompt

---

### Task 4: Deploy Azure App Service

**What to do:**

1. Create an App Service Plan: `asp-novatech-{id}-support` (Free F1 or Basic B1 tier)
2. Create a Web App: `app-novatech-{id}-support`
   - Runtime: Node.js 20 LTS (or Python 3.11)
3. Deploy a simple chat UI. Use this minimal HTML file as `index.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NovaTech Support</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { font-family: -apple-system, BlinkMacSystemFont, sans-serif; background: #f5f5f5; }
        .container { max-width: 800px; margin: 40px auto; background: white; border-radius: 12px; box-shadow: 0 2px 12px rgba(0,0,0,0.1); overflow: hidden; }
        .header { background: #0078d4; color: white; padding: 20px; text-align: center; }
        .header h1 { font-size: 1.5rem; }
        .chat-box { height: 400px; overflow-y: auto; padding: 20px; }
        .message { margin: 10px 0; padding: 12px 16px; border-radius: 12px; max-width: 80%; }
        .user { background: #0078d4; color: white; margin-left: auto; text-align: right; }
        .bot { background: #e9e9e9; }
        .input-area { display: flex; padding: 20px; border-top: 1px solid #ddd; }
        .input-area input { flex: 1; padding: 12px; border: 1px solid #ddd; border-radius: 8px; font-size: 1rem; }
        .input-area button { margin-left: 10px; padding: 12px 24px; background: #0078d4; color: white; border: none; border-radius: 8px; cursor: pointer; font-size: 1rem; }
        .input-area button:hover { background: #106ebe; }
    </style>
</head>
<body>
    <div class="container">
        <div class="header"><h1>NovaTech Customer Support</h1><p>Powered by Azure AI</p></div>
        <div class="chat-box" id="chatBox">
            <div class="message bot">Hello! I'm NovaTech's AI assistant. How can I help you today?</div>
        </div>
        <div class="input-area">
            <input type="text" id="userInput" placeholder="Type your question..." onkeypress="if(event.key==='Enter')sendMessage()">
            <button onclick="sendMessage()">Send</button>
        </div>
    </div>
    <script>
        async function sendMessage() {
            const input = document.getElementById('userInput');
            const chatBox = document.getElementById('chatBox');
            const message = input.value.trim();
            if (!message) return;

            chatBox.innerHTML += '<div class="message user">' + message + '</div>';
            input.value = '';
            chatBox.scrollTop = chatBox.scrollHeight;

            try {
                const response = await fetch('/api/chat', {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ message: message })
                });
                const data = await response.json();
                chatBox.innerHTML += '<div class="message bot">' + data.reply + '</div>';
            } catch (error) {
                chatBox.innerHTML += '<div class="message bot">Sorry, I encountered an error. Please try again.</div>';
            }
            chatBox.scrollTop = chatBox.scrollHeight;
        }
    </script>
</body>
</html>
```

> **Note**: The `/api/chat` endpoint needs a backend (Node.js/Python) that calls the AI Foundry API. For this lab, you can test the chat UI by deploying just the static HTML and verifying it loads, then connect the backend in Task 5.

#### Portal-Only Deployment (No CLI needed)

1. In the Azure Portal, search **"App Services"** → **+ Create**
2. **Basics** tab:
   - Resource Group: `rg-novatech-{id}-ai-eastus`
   - Name: `app-novatech-{id}-support`
   - Publish: **Code**
   - Runtime stack: **Node 20 LTS**
   - Operating System: **Linux**
   - Region: **East US**
   - Pricing plan: **Free F1**
3. Click **Review + create** → **Create**
4. Once created, go to your App Service → **Advanced Tools** (Kudu) → **Go →**
5. In Kudu, click **Debug console** → **SSH**
6. Navigate to `/home/site/wwwroot/`
7. Create `public/index.html` — paste the HTML code above
8. Create `server.js` — paste the server code from Task 5 below
9. Create `package.json` with:
   ```json
   { "name": "novatech-support", "version": "1.0.0", "main": "server.js", "scripts": { "start": "node server.js" }, "dependencies": { "express": "^4.21.0" } }
   ```
10. Run `npm install` in the SSH console
11. Restart the App Service from the **Overview** page

**Alternative — App Service Editor (even simpler):**
1. Go to your App Service → **App Service Editor (Preview)** in the left menu
2. This opens a browser-based VS Code editor
3. Create the files directly and paste the code
4. The app restarts automatically when you save

**Acceptance Criteria:**
- App Service is running and accessible via its URL (`https://app-novatech-{id}-support.azurewebsites.net`)
- The chat UI loads in the browser
- App Service shows "Running" status in the Portal

**Hints:**
- [MS Learn: Deploy a web app](https://learn.microsoft.com/en-us/azure/app-service/quickstart-html)
- Free F1 tier has limitations: no custom domain, no TLS, 60 CPU minutes/day
- You can deploy via: VS Code Azure extension, `az webapp up`, GitHub Actions, or ZIP deploy

---

### Task 5: Connect App Service to AI Foundry

**What to do:**

1. Get your AI Foundry endpoint URL and API key from the deployment in Task 2 (go to Deployments → your model → find the endpoint and key)
2. Add these as **Application Settings** (environment variables) in the App Service:

| Setting Name | Value |
|-------------|-------|
| `AZURE_OPENAI_ENDPOINT` | Your AI Foundry endpoint URL |
| `AZURE_OPENAI_KEY` | Your API key |
| `AZURE_OPENAI_DEPLOYMENT` | `gpt-4o-mini` |

> ⚠️ **Security**: NEVER hardcode the API key in source code. Use App Settings or Key Vault references.

3. (Optional) Create a minimal backend API. For Node.js:

```javascript
// server.js — add an /api/chat endpoint
const express = require('express');
const app = express();
app.use(express.json());
app.use(express.static('public')); // serves index.html

app.post('/api/chat', async (req, res) => {
    const endpoint = process.env.AZURE_OPENAI_ENDPOINT;
    const apiKey = process.env.AZURE_OPENAI_KEY;
    const deployment = process.env.AZURE_OPENAI_DEPLOYMENT;

    const response = await fetch(
        `${endpoint}/openai/deployments/${deployment}/chat/completions?api-version=2024-08-01-preview`,
        {
            method: 'POST',
            headers: { 'Content-Type': 'application/json', 'api-key': apiKey },
            body: JSON.stringify({
                messages: [
                    { role: 'system', content: 'You are NovaTech AI support assistant...' },
                    { role: 'user', content: req.body.message }
                ],
                max_tokens: 500, temperature: 0.3
            })
        }
    );
    const data = await response.json();
    res.json({ reply: data.choices[0].message.content });
});

app.listen(process.env.PORT || 8080);
```

**Acceptance Criteria:**
- App Settings contain the 3 environment variables (key value hidden in Portal)
- API key is NOT visible in any source code file
- (If backend deployed) Chat UI successfully queries the AI model and returns responses

**Hints:**
- [MS Learn: Configure App Service settings](https://learn.microsoft.com/en-us/azure/app-service/configure-common)
- App Settings are injected as environment variables at runtime — your code reads them with `process.env.VARIABLE_NAME`
- For even better security, use **Key Vault references**: `@Microsoft.KeyVault(VaultName=kv-novatech-{id}-prod;SecretName=OpenAIKey)`

---

### Task 6: Deploy Azure Function (Webhook)

**What to do:**

1. Create a Function App:
   - Name: `func-novatech-{id}-webhook`
   - Runtime: Node.js 20 or Python 3.11
   - Hosting plan: **Consumption (Serverless)** — free tier
   - Storage Account: Create new (auto-generated name is fine)
2. Create an HTTP-triggered function that:
   - Accepts a POST request with `{ "customerId": "...", "message": "..." }`
   - Logs the request to a **Storage Queue** named `support-requests`
   - Returns a confirmation response

**Acceptance Criteria:**
- Function App exists and shows "Running"
- HTTP trigger function is accessible at its URL
- Sending a test POST request returns a success response
- (Bonus) Messages appear in the Storage Queue

**Hints:**
- [MS Learn: Create your first function](https://learn.microsoft.com/en-us/azure/azure-functions/functions-create-function-app-portal)
- Consumption plan = pay only when the function executes, first 1 million executions/month free
- Functions are ideal for event-driven, short-lived tasks (webhook receiving, queue processing, scheduled jobs)

#### Portal-Only Deployment (No CLI needed)

1. In the Azure Portal, search **"Function App"** → **+ Create**
2. **Basics** tab:
   - Resource Group: `rg-novatech-{id}-ai-eastus`
   - Function App name: `func-novatech-{id}-webhook`
   - Runtime stack: **Node.js**, Version: **20 LTS**
   - Region: **East US**
   - Operating System: **Linux**
   - Hosting plan: **Consumption (Serverless)**
3. **Storage** tab: Create new or use existing Storage Account
4. Click **Review + create** → **Create**
5. Once created, go to your Function App → **Functions** → **+ Create**
6. Select **HTTP trigger**, name it `webhook`, Authorization level: **Function**
7. Click **Create** → then click on the function → **Code + Test**
8. Replace the default code with:

```javascript
module.exports = async function (context, req) {
    const { customerId, message } = req.body || {};

    if (!customerId || !message) {
        context.res = { status: 400, body: { error: 'Both customerId and message are required.' } };
        return;
    }

    // Log to queue (configure Storage Queue output binding in Integration tab)
    context.bindings.supportQueue = JSON.stringify({
        customerId,
        message,
        timestamp: new Date().toISOString()
    });

    context.res = {
        status: 200,
        headers: { 'Content-Type': 'application/json' },
        body: { success: true, message: 'Support request logged.', requestId: `req-${Date.now()}` }
    };
};
```

9. Click **Save**
10. To add the Storage Queue output:
    - Go to **Integration** tab → **+ Add output**
    - Binding type: **Azure Queue Storage**
    - Queue name: `support-requests`
    - Parameter name: `supportQueue`
    - Storage account connection: Select your storage account
    - Click **OK**
11. Click **Test/Run** → set Method: POST, Body: `{"customerId": "CUST-001", "message": "Test"}`
12. Click **Run** — you should get a 200 response

---

### Task 7: Export ARM Template and Create Bicep

**What to do:**

1. Go to your resource group → **Export template** (in the left menu)
2. Download the ARM template JSON
3. Open it in VS Code — observe the structure (parameters, variables, resources, outputs)
4. Create a simplified **Bicep file** for just the App Service resource:

```bicep
// app-service.bicep — fill in the values
param location string = resourceGroup().location
param appName string = 'app-novatech-{id}-support'

resource appServicePlan 'Microsoft.Web/serverfarms@2022-09-01' = {
  name: 'asp-novatech-{id}-support'
  location: location
  sku: {
    name: 'F1'
    tier: 'Free'
  }
}

resource webApp 'Microsoft.Web/sites@2022-09-01' = {
  name: appName
  location: location
  properties: {
    serverFarmId: appServicePlan.id
    httpsOnly: true
  }
}

output appUrl string = 'https://${webApp.properties.defaultHostName}'
```

5. Deploy the Bicep file:
   ```
   az deployment group create --resource-group rg-novatech-{id}-ai-eastus --template-file app-service.bicep
   ```

**Acceptance Criteria:**
- ARM template JSON is downloaded and saved in your GitHub repo
- Bicep file is created and saved in your GitHub repo
- You can explain the difference between ARM JSON and Bicep syntax
- (Bonus) Bicep deployment succeeds

**Hints:**
- [MS Learn: Deploy Bicep templates](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/deploy-cli)
- Install the Bicep VS Code extension for syntax highlighting and IntelliSense
- Bicep is a simpler, more readable alternative to ARM JSON — it compiles to ARM
- Note `httpsOnly: true` — this is a security best practice (enforces HTTPS)

---

### Task 8: Azure Monitor Dashboard

**What to do:**

1. Enable diagnostic settings on the AI Foundry / OpenAI resource:
   - Send to Log Analytics workspace (create one if needed)
   - Category: RequestResponse, Audit
2. Create an Azure Monitor Dashboard:
   - Add tiles for: API request count, average latency, error rate (HTTP 4xx/5xx)
   - (If App Service deployed) Add: App Service response time, HTTP server errors

**Acceptance Criteria:**
- Diagnostic settings are enabled on at least one resource
- Dashboard exists with at least 2 metric tiles
- Metrics show data (even if minimal — a few test requests count)

**Hints:**
- [MS Learn: Create a dashboard](https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/tutorial-metrics)
- Dashboards can be shared with your team — useful for the Week 10 presentation
- Pin metrics directly from the resource's Metrics blade to a dashboard

---

### Task 9: (Optional Advanced) Private Endpoint

**What to do:**

1. Create a Private Endpoint for the AI Foundry / OpenAI resource
2. Link it to a subnet in your VNet (e.g., AppSubnet)
3. Disable public network access on the AI resource

> ⚠️ After disabling public access, the AI Foundry Playground will only work from within the VNet.

**Acceptance Criteria:**
- Private Endpoint exists and shows "Approved" status
- The AI resource shows "Public access: Disabled"
- (Verify) Access from outside the VNet returns 403

**Hints:**
- [MS Learn: Use private endpoints](https://learn.microsoft.com/en-us/azure/ai-services/cognitive-services-virtual-networks)
- Private Endpoints give your resource a private IP from your VNet — traffic stays on Microsoft backbone
- This is a critical security control for production AI workloads

---

### Task 10: 🔒 Security Review

**What to do:**

1. Run **Azure Advisor** on your AI resource group
2. Review **Microsoft Defender for Cloud** recommendations (read-only)

**Answer the following in your `thought-process.md`:**

1. Is your App Service enforcing **HTTPS-only**? How do you verify this? Why does it matter?
2. Are the AI Foundry API keys stored securely? Where are they stored? What would be the ideal approach using managed identity?
3. Does your AI Foundry resource have **network restrictions**? What's the risk of leaving it publicly accessible?
4. Is your Function App using **managed identity** or **key-based authentication**? Which is more secure and why?
5. Review Microsoft Defender for Cloud — what is the **Secure Score** for your subscription? What are the top 3 recommendations?
6. If a malicious user sent 1 million API requests to your AI endpoint, what would happen? What Azure service could prevent this?

---

## 📸 Screenshot Submission Checklist

Upload all screenshots to your GitHub repo: `UseCase3-AI-Customer-Portal/screenshots/`

| # | Screenshot Required | Filename |
|---|-------------------|----------|
| 1 | AI Foundry hub and project overview | `01-ai-foundry-hub.png` |
| 2 | Deployed GPT-4o-mini model showing deployment name and status | `02-model-deployment.png` |
| 3 | AI Foundry Playground conversation showing at least 2 test queries with responses | `03-playground-test.png` |
| 4 | App Service overview showing name, URL, and running status | `04-app-service-overview.png` |
| 5 | Chat UI running in browser at the App Service URL | `05-chat-ui-browser.png` |
| 6 | App Service Configuration page showing environment variables (endpoint visible, key hidden) | `06-app-settings.png` |
| 7 | Function App overview showing the HTTP trigger function | `07-function-app.png` |
| 8 | ARM template export page (or JSON file open in VS Code) | `08a-arm-export.png` |
| 9 | Bicep file open in VS Code showing the App Service resource | `08b-bicep-file.png` |
| 10 | Azure Monitor dashboard showing at least 2 metrics | `09-monitor-dashboard.png` |
| 11 | (If completed) Private Endpoint overview | `10-private-endpoint.png` |
| 12 | Azure Advisor + Defender for Cloud Secure Score | `11a-advisor.png`, `11b-defender-score.png` |
| 13 | Empty resource group or deleted confirmation after cleanup | `12-cleanup-confirmation.png` |

---

## Cleanup Checklist

⚠️ **Delete all resources after completing this use case.**

- [ ] Azure Function and its auto-created Storage Account
- [ ] App Service and App Service Plan
- [ ] AI Foundry project, then hub
- [ ] AI Services / OpenAI resource (created automatically with the hub)
- [ ] Private Endpoint (if created)
- [ ] Monitor dashboard
- [ ] Log Analytics workspace (if created)
- [ ] Resource group

---

## Challenge Questions

1. **Scenario**: NovaTech's CEO asks: "How much will this AI assistant cost us per month if we handle 10,000 customer queries?" Calculate the estimated cost using GPT-4o-mini pricing ($0.15/1M input tokens, $0.60/1M output tokens). Assume average query = 100 input tokens, 200 output tokens.

2. **Scenario**: A developer accidentally pushes the AI API key to a public GitHub repo. What immediate steps would you take? How could this have been prevented?

3. **Scenario**: The AI assistant gives a customer incorrect pricing information and the customer demands a refund. Who is responsible? What guardrails should NovaTech implement?

4. **Scenario**: NovaTech wants to add the AI assistant to their mobile app as well. The current setup uses App Service. What architecture changes would you recommend to support both web and mobile clients?

5. **Scenario**: Your ARM template export is 500 lines of JSON. Your Bicep file for the same resources is 50 lines. Why is Bicep preferred for authoring? When would you still need ARM JSON?

---

## Documentation & Presentation Prep

### `thought-process.md`
For each task, explain your decisions, challenges, and security review answers.

### `challenge-answers.md`
Written answers to all 5 challenge questions.

### Week 10 Presentation
In Week 10, you'll form teams to present. Prepare:
- Architecture diagram (use the one in this lab as a starting point)
- Live demo (if resources are still available) or recorded demo
- Cost analysis (actual Azure cost from Cost Management)
- Security posture summary (Advisor score, Defender findings, mitigations)
- Lessons learned

---

## GitHub Repo Structure

```
UseCase3-AI-Customer-Portal/
├── screenshots/
│   ├── 01-ai-foundry-hub.png
│   ├── 02-model-deployment.png
│   ├── 03-playground-test.png
│   ├── 04-app-service-overview.png
│   ├── 05-chat-ui-browser.png
│   ├── 06-app-settings.png
│   ├── 07-function-app.png
│   ├── 08a-arm-export.png
│   ├── 08b-bicep-file.png
│   ├── 09-monitor-dashboard.png
│   ├── 10-private-endpoint.png (optional)
│   ├── 11a-advisor.png
│   ├── 11b-defender-score.png
│   └── 12-cleanup-confirmation.png
├── code/
│   ├── index.html
│   ├── server.js (or app.py)
│   └── app-service.bicep
├── arm-export/
│   └── template.json
├── thought-process.md
└── challenge-answers.md
```

---

*This is the most complex use case. Break it into two sessions (Week 8: Tasks 1–5, Week 9: Tasks 6–10). Focus on understanding how services connect together — that's the real skill.*
