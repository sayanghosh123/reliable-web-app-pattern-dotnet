# Skill: Validate Deployment

## Purpose

Run the deployment health-check script to verify that the Azure environment was provisioned and deployed correctly. The script checks resource group existence, validates App Service settings (including that the required App Configuration URIs are set), and provides troubleshooting recommendations for known issues.

## Prerequisites

- **Azure CLI** must be installed and available on the PATH.
- The user must be logged into Azure with sufficient permissions to read the target resource group.
- The environment must have been deployed using `azd up` or `azd provision` before running validation.

## Steps

### 1. Verify Azure Login

Run the following command to confirm the user is authenticated:

```bash
az account show
```

- If the command **succeeds**, the user is logged in. Proceed to Step 2.
- If the command **fails**, instruct the user to log in:

```bash
az login
```

### 2. Run the Validation Script

Determine the resource group name for the deployed environment, then run the script:

**PowerShell:**

```powershell
./testscripts/validate-deployment.ps1 -ResourceGroupName <resource-group-name>
```

**Bash:**

```bash
./testscripts/validate-deployment.sh --resource-group <resource-group-name>
```

The `-ResourceGroupName` parameter (aliased as `-g`) is required. It is the name of the resource group created by the `azd` command.

### 3. Analyze the Output

- **Success:** The script completes without errors. All App Service settings, App Configuration URIs, and resource tags are validated.
- **Failure — Resource group not found:** The resource group does not exist. Run `azd provision` to create the infrastructure.
- **Failure — Missing App Service settings:** The App Configuration URI or other required settings are not set on the App Service. Run `azd provision` to overlay the missing configuration.

### 4. Troubleshooting Common Failures

If the validation script reports failures, check the following:

- **Firewall rules:** Ensure the Azure Firewall in the hub network allows outbound traffic to required Azure service FQDNs. Review `infra/modules/hub-network.bicep` for firewall configuration.
- **Key Vault access (RBAC role assignments):** Verify that the application's Managed Identity has the appropriate Key Vault Secrets User (or equivalent) role assignment scoped to the required secrets or Key Vault. Review `infra/modules/grant-secret-user.bicep`.
- **Private endpoint DNS:** Confirm that private DNS zones are correctly linked to the spoke VNet. Review `infra/modules/private-dns-zones.bicep`.
- **Re-provision:** Many transient deployment issues can be resolved by re-running `azd provision`.
