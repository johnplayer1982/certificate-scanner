# Certificate Scanner

The aim of this repo is to scan Azure Key Vaults for certificates and return all expiry dates:  

- Runs daily
- Scans all Key Vaults in the subscription
- Extracts certificate names + expiry dates
- Outputs a CSV artifact you can download or feed into monitoring

This is built for DevOps teams wanting reproducible infra‑safe workflows.

## How it works

1. Login: Uses federated identity — no secrets, no passwords, no ACR creds. Perfect for governance‑minded setup, the action leverages OIDC (OpenID Connect), which allows GitHub Actions to request a short-lived token from Azure AD using the workflow's identity.
2. Enumerates all Key Vaults

```
  az keyvault list --query "[].name" -o tsv
```

3. Enumerates certificates in each vault

```
  az keyvault certificate list --vault-name "$vault"
```

4. Extracts expiry dates: Using JMESPath → .attributes.expires
5. Writes a CSV in the following format:

```
  vault,certificate,expires
  myvault1,api-cert,2026-01-12T00:00:00Z
  myvault2,internal-ca,2025-09-01T00:00:00Z
```

6. Uploads as an artifact for download

## Action Secrets:

```
AZURE_CLIENT_ID
AZURE_TENANT_ID
AZURE_SUBSCRIPTION_ID
```

### Getting those values (A refresher)

1. AZURE_CLIENT_ID

    Go to “Azure Active Directory” → “App registrations” → select your app → “Application (client) ID”.

    ```
    az account show --query id -o tsv
    ```

2. AZURE_TENANT_ID

    Go to “Azure Active Directory” → “Overview” → “Tenant ID”.

    ```
    az account show --query tenantId -o tsv
    ```

3. AZURE_SUBSCRIPTION_ID

    Go to “Subscriptions” → select your subscription → “Subscription ID”.

    ```
    az ad sp list --display-name <app-name> --query "[0].appId" -o tsv
    ```

## Azure Setup

### Set up Azure App Registration with federated credentials

1. Register an App in Azure AD
Go to Azure Portal → Azure Active Directory → App registrations → New registration.  
Name your app (e.g., "GitHub OIDC App").
Set the supported account type (usually "Single tenant").
Click "Register".

2. Assign API Permissions (if needed)
In your app registration, go to "API permissions".  
Add permissions required for your workflow (e.g., Azure Key Vault access).

3. Assign a Role to the App
Go to your subscription/resource group/Key Vault.  
Access "Access control (IAM)" → "Add role assignment".  
Assign a role (e.g., "Key Vault Reader") to your app’s Service Principal.

4. Add a Federated Credential for GitHub Actions
In your app registration, go to "Certificates & secrets" → "Federated credentials" → "Add credential" → Select "GitHub Actions deploying Azure Resources".  
Fill in:
Issuer: https://token.actions.githubusercontent.com (pre-populated)
Subject identifier:
    - Oranisation (johnplayer1982)
    - Repository (certificate-scanner)
    - Entity type (branch)
    - GitHub branch name (main or * for all)
Name: (e.g., "GitHub Actions OID")
Audience: api://AzureADTokenExchange (pre-populated)
Save.

5. Use the App Registration in GitHub Actions
Set the AZURE_CLIENT_ID, AZURE_TENANT_ID, and AZURE_SUBSCRIPTION_ID as repository secrets in GitHub.  
The workflow can now use OIDC to authenticate without secrets.

# Troubleshooting

## CSV only contains vault,certificate,expires titles

Ensure your App Registration has at least the Reader role at the subscription level to list Key Vaults.  

For listing certificates, it needs Key Vault Reader or Key Vault Certificates Officer on each Key Vault.


