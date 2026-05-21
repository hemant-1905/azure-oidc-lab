# 🧪 Azure OIDC GitHub Actions Lab (CI + Deploy)

This repository demonstrates passwordless authentication from GitHub Actions to Azure using OIDC federation.

It covers:
- CI login validation workflow
- Real deployment workflow
- Azure Entra ID federated identity trust
- RBAC-based authorization (no secrets used)

---

# 🧠 Architecture Overview

```text
GitHub Actions
   ↓ (OIDC token)
Microsoft Entra ID (Federated Trust)
   ↓
Service Principal (App Registration)
   ↓
Azure RBAC
   ↓
Resource Group / Resources
```

---

# 🔐 Identity Model

This setup uses:
- Service Principal created via App Registration
- Federated credentials (OIDC trust with GitHub)
- No client secrets or certificates
- Azure RBAC for authorization

---

# 📦 Workflows

## 1️⃣ CI Workflow — Identity Validation (`oidc.yaml`)

### Purpose
Validates Azure login and confirms identity context.

### Workflow

```yaml
name: Azure OIDC Login

on:
  workflow_dispatch:

permissions:
  id-token: write
  contents: read

jobs:
  login-test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Azure Login via OIDC
        uses: azure/login@v1
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

      - name: Verify Azure Login
        run: az account show
```

---

## 2️⃣ Deployment Workflow (`oidc2.yaml`)

### Purpose
Simulates real CI/CD deployment into Azure.

### Workflow

```yaml
name: Azure OIDC Deploy

on:
  workflow_dispatch:

permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Azure Login via OIDC
        uses: azure/login@v1
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

      - name: Show Current Account
        run: az account show

      - name: List Resource Groups
        run: az group list -o table

      - name: Create Storage Account
        run: |
          az storage account create \
            --name <unique-storage-name> \
            --resource-group github-oidc-lab \
            --location centralindia \
            --sku Standard_LRS
```

---

# 🔑 Required GitHub Secrets

Configure these in repository settings:

| Secret | Description |
|---|---|
| `AZURE_CLIENT_ID` | App Registration (Service Principal client ID) |
| `AZURE_TENANT_ID` | Entra tenant ID |
| `AZURE_SUBSCRIPTION_ID` | Azure subscription ID |

---

# 🧠 One-Time Azure Setup

## 1. Create App Registration

Name:
`github-oidc-lab`

## 2. Add Federated Credential

Issuer:
`https://token.actions.githubusercontent.com`

Subject:
`repo:<org>/<repo>:ref:refs/heads/main`

## 3. Assign RBAC Role

Role:
`Contributor`

Scope:
- Resource Group OR Subscription

---

# 🧪 How to Run

1. Push workflows to repo
2. Go to GitHub Actions tab
3. Manually trigger:
   - Azure OIDC Login
   - Azure OIDC Deploy

---

# 🔥 Expected Output

## CI Workflow
- Azure login succeeds
- Subscription details printed

## Deployment Workflow
- Resource groups listed
- Storage account created

---

# 🚨 Common Issues

## AADSTS700016
- App not created in correct tenant

## Forbidden / RBAC error
- Contributor role missing

## No subscription found
- Wrong subscription ID

---

# 🧠 Key Learnings

- Passwordless authentication using OIDC
- Entra ID federated identity trust
- Azure RBAC authorization model
- Separation of identity and secrets
- Production-grade CI/CD pattern

---

# 🚀 Next Improvements

- Separate dev/prod identities
- Branch-based deployment control
- Key Vault integration with Managed Identity
- Terraform-based identity provisioning