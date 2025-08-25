1. Agent Setup on Personal Computer / Azure VM (Self-hosted Agent)
  Step-by-step:

1. **Azure DevOps me Agent Pool Create karo:**

  * Azure DevOps > Project settings > Agent Pools > New Agent Pool
  * Naam de do jaise: `SelfHostedPool`

2. **Personal Access Token (PAT) Create karo:**

  * Azure DevOps > User Settings (top right) > Personal access tokens
  * Scope: Agent Pools (Read & Manage), Deployment Groups, etc.
  * Save token securely (will be used only once)

3. **Agent Download aur Configure karo:**

  * VM ya apne system par agent download karo:
    [https://aka.ms/downloadagent](https://aka.ms/downloadagent)
  * Extract the zip file in a folder, e.g., `C:\agent`

4. **Command Prompt se Agent Configure karo:**

  ```bash
  config.cmd --unattended ^
  --url https://dev.azure.com/YOUR_ORG ^
  --auth pat ^
  --token YOUR_PAT_TOKEN ^
  --pool SelfHostedPool ^
  --agent AGENT_NAME ^
  --acceptTeeEula
  ```

5. **Agent ko Run karo as a Service:**

  ```bash
  run.cmd
  ```

6. (Optional but recommended) Install as service:

  ```bash
  svc install
  svc start
  ```

---

## 🔹 **2. Service Connection Setup (App Registration ke through)**

### ✅ **2A. App Registration ke sath (Manual):**

1. **Azure Portal me App Register karo:**

  * Azure Active Directory > App Registrations > New Registration
  * Name: `DevOps-SP`
  * Redirect URI: (leave blank or add placeholder)

2. **Client ID, Tenant ID note karo**

3. **Secret Generate karo:**

  * Certificates & secrets > New client secret
  * Note this secret securely.

4. **Subscription me Role Assign karo:**

  * Azure > Subscriptions > IAM > Add Role Assignment
  * Role: Contributor (or required role)
  * Assign to App (SPN)

5. **Azure DevOps me Service Connection banao:**

  * Project Settings > Service Connections > New > Azure Resource Manager
  * Authentication method: Service principal (manual)
  * Enter:

    * Tenant ID
    * Client ID
    * Client Secret
    * Subscription ID
    * Subscription Name

---

### ✅ **2B. Without Secret - Workload Identity Federation (Secure, secretless)**

1. **Azure AD me App Register karo (jaise upar)**

2. **Federated Credential Add karo:**

  * App > Certificates & secrets > Federated credentials > Add credential
  * Issuer: `https://vstoken.dev.azure.com/{org}`
  * Subject identifier: `repo:{org}/{project}:{ref}`
  * Audience: `api://AzureADTokenExchange`

3. **Azure DevOps Service Connection banao:**

  * Project Settings > Service Connections > New > Azure Resource Manager
  * Select: Workload Identity Federation
  * Provide:

    * Tenant ID
    * Client ID
    * Subscription ID

4. **Role Assign karo SPN ko Subscription me (as before)**

---

## 🔹 **3. Pipeline Structure: Stages → Jobs → Steps**

### ✅ **Pipeline Structure Breakdown:**

#### 📁 **Stage 1: Initialization & Planning (e.g., Terraform)**

```yaml
stages:
- stage: InitPlan
 jobs:
 - job: Init
   steps:
   - checkout: self
   - script: terraform init
   - script: terraform plan -out=tfplan
```

#### 🛡️ **Stage 2: Scanning (e.g., Security or Policy Scans)**

```yaml
- stage: Scanning
 dependsOn: InitPlan
 jobs:
 - job: RunScans
   steps:
   - script: echo "Running security scans..."
   - script: ./run-scan.sh
```

#### 🚀 **Stage 3: Deploy with Manual Approval**

```yaml
- stage: Deploy
 dependsOn: Scanning
 approval:
   approvals:
     - approvers: ['user@domain.com']
 jobs:
 - job: Apply
   steps:
   - script: terraform apply tfplan
```

---

