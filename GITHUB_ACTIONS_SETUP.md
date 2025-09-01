# GitHub Actions Setup Guide

This guide will help you set up GitHub Actions for automated deployment of your Azure Function App.

## Prerequisites

1. **Azure Subscription** with appropriate permissions
2. **GitHub Repository** with your function app code
3. **Azure CLI** installed locally

## Step 1: Create Azure Service Principal

Create a service principal for GitHub Actions to authenticate with Azure:

```bash
# Replace with your actual subscription ID
SUBSCRIPTION_ID="your-subscription-id"

# Create service principal
az ad sp create-for-rbac \
  --name "github-actions-function-app" \
  --role contributor \
  --scopes /subscriptions/$SUBSCRIPTION_ID \
  --sdk-auth
```

This will output JSON similar to:
```json
{
  "clientId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "clientSecret": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "subscriptionId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}
```

## Step 2: Add GitHub Secrets

In your GitHub repository, go to **Settings > Secrets and variables > Actions** and add these secrets:

### Required Secrets

1. **AZURE_CREDENTIALS**
   - Value: The entire JSON output from the service principal creation
   - This allows GitHub Actions to authenticate with Azure

2. **AZURE_SUBSCRIPTION_ID**
   - Value: Your Azure subscription ID
   - Used for resource deployment

### Example:
```
AZURE_CREDENTIALS = {
  "clientId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "clientSecret": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "subscriptionId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}

AZURE_SUBSCRIPTION_ID = xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

## Step 3: Customize Workflow Variables

Edit `.github/workflows/deploy.yml` and update these environment variables:

```yaml
env:
  AZURE_FUNCTIONAPP_NAME: 'your-function-app-name'  # Change this
  RESOURCE_GROUP: 'your-resource-group-name'        # Change this
  LOCATION: 'eastus'                                 # Change if needed
```

## Step 4: Update Bicep Parameters

Edit `infra/main.parameters.json` to customize your deployment:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "functionAppName": {
      "value": "your-function-app-name"
    },
    "location": {
      "value": "East US"
    },
    "environmentName": {
      "value": "production"
    }
  }
}
```

## Step 5: Push Code to Trigger Deployment

Once everything is configured, push your code to the `main` branch:

```bash
git add .
git commit -m "Initial function app setup"
git push origin main
```

The GitHub Actions workflow will automatically:
1. Build the Python function app
2. Deploy the Azure infrastructure using Bicep
3. Deploy the function code
4. Test the deployment

## Workflow Overview

The GitHub Actions workflow consists of three jobs:

### 1. Build Job
- Sets up Python environment
- Installs dependencies
- Creates deployment artifact

### 2. Deploy Infrastructure Job
- Deploys Bicep template
- Creates all Azure resources
- Outputs function app details

### 3. Deploy Function Job
- Downloads build artifact
- Deploys function code to Azure
- Tests the deployment

## Monitoring Deployment

You can monitor the deployment progress in:

1. **GitHub Actions tab** in your repository
2. **Azure Portal** > Resource Groups > Your Resource Group
3. **Function App logs** in Azure Portal

## Troubleshooting

### Common Issues

1. **Authentication Failures**
   - Verify `AZURE_CREDENTIALS` secret is correct
   - Check service principal has sufficient permissions

2. **Resource Naming Conflicts**
   - Change `AZURE_FUNCTIONAPP_NAME` to a unique value
   - Resource names must be globally unique

3. **Deployment Timeouts**
   - Function apps may take 2-3 minutes to start
   - The workflow includes wait times for this

4. **Permission Errors**
   - Ensure service principal has `Contributor` role
   - Check subscription permissions

### Useful Commands

```bash
# Check service principal permissions
az role assignment list --assignee "your-client-id"

# View deployment logs
az deployment group list --resource-group "your-rg"

# Check function app status
az functionapp show --name "your-function-app" --resource-group "your-rg"
```

## Manual Deployment Alternative

If you prefer manual deployment, you can use the provided scripts:

- **Linux/Mac**: `./deploy.sh`
- **Windows**: `./deploy.ps1`

## Security Best Practices

1. **Use least privilege** for service principal
2. **Rotate secrets regularly**
3. **Monitor deployment logs** for security issues
4. **Use managed identity** in production (already configured)
5. **Enable diagnostic logging** in Azure

## Next Steps

After successful deployment:

1. **Test your functions** in Azure Portal
2. **Set up monitoring** and alerts
3. **Configure custom domains** if needed
4. **Implement CI/CD for multiple environments**
5. **Add integration tests** to the workflow
