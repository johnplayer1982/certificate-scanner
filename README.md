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

## Getting those values (A refresher)

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
