# Azure Function App with Managed Identity - Sample POC

This repository contains a sample Azure Function app that demonstrates the use of managed identity for authentication with Azure services like Key Vault and Storage Account.

## Project Structure

```
├── HttpTriggerFunction/       # HTTP trigger function
│   ├── __init__.py
│   └── function.json
├── TimerTriggerFunction/      # Timer trigger function  
│   ├── __init__.py
│   └── function.json
├── infra/                     # Infrastructure as Code (Bicep)
│   ├── main.bicep
│   └── main.parameters.json
├── .github/workflows/         # GitHub Actions workflows
│   ├── deploy.yml
│   └── deploy-cli.yml
├── requirements.txt           # Python dependencies
├── host.json                 # Function app configuration
└── azure.yaml               # Azure Developer CLI configuration
```

## Features

- **HTTP Trigger Function**: Demonstrates managed identity access to Key Vault and Storage
- **Timer Trigger Function**: Background processing with managed identity
- **Infrastructure as Code**: Bicep templates for reproducible deployments
- **CI/CD Pipeline**: GitHub Actions for automated deployment
- **Managed Identity**: User-assigned managed identity for secure authentication

## Functions

### HttpTriggerFunction
- **Trigger**: HTTP (GET/POST)
- **Authentication**: Anonymous (for demo purposes)
- **Features**: 
  - Returns JSON response with function metadata
  - Demonstrates Key Vault access using managed identity
  - Shows Storage Account connectivity
  - Hardcoded parameters for POC

### TimerTriggerFunction  
- **Trigger**: Timer (runs every 5 minutes)
- **Features**:
  - Background processing simulation
  - Managed identity credential demonstration
  - Logging for monitoring

## Prerequisites

1. **Azure Subscription** with appropriate permissions
2. **GitHub Repository** for CI/CD
3. **Azure CLI** (for local development)
4. **Python 3.11** (for local testing)

## Quick Start

### Option 1: Using Azure Developer CLI (Recommended)

```bash
# Install Azure Developer CLI
# https://docs.microsoft.com/azure/developer/azure-developer-cli/install-azd

# Clone and initialize
azd init
azd up
```

### Option 2: Manual Deployment

1. **Deploy Infrastructure**:
```bash
# Create resource group
az group create --name rg-sample-function-app --location eastus

# Deploy Bicep template
az deployment group create \
  --resource-group rg-sample-function-app \
  --template-file infra/main.bicep \
  --parameters @infra/main.parameters.json
```

2. **Deploy Function Code**:
```bash
# Install dependencies
pip install -r requirements.txt

# Create deployment package
func azure functionapp publish <your-function-app-name>
```

## GitHub Actions Setup

### Required Secrets

Add these secrets to your GitHub repository:

1. **AZURE_CREDENTIALS**: Service principal credentials
```json
{
  "clientId": "<client-id>",
  "clientSecret": "<client-secret>",
  "subscriptionId": "<subscription-id>",
  "tenantId": "<tenant-id>"
}
```

2. **AZURE_SUBSCRIPTION_ID**: Your Azure subscription ID
3. **AZURE_RG**: Resource group name
4. **AZURE_FUNCTIONAPP_PUBLISH_PROFILE**: Function app publish profile (optional)

### Creating Service Principal

```bash
# Create service principal for GitHub Actions
az ad sp create-for-rbac --name "github-actions-sp" \
  --role contributor \
  --scopes /subscriptions/{subscription-id} \
  --sdk-auth
```

## Testing the Functions

### HTTP Trigger Function
```bash
# Test locally
curl http://localhost:7071/api/HttpTriggerFunction

# Test deployed function
curl https://your-function-app.azurewebsites.net/api/HttpTriggerFunction
```

### Expected Response
```json
{
  "message": "Hello from Azure Function with Managed Identity!",
  "status": "success",
  "timestamp": "2025-09-01T12:00:00.000Z",
  "managed_identity": "enabled",
  "environment": "production",
  "function_app_name": "your-function-app-name",
  "key_vault_status": "configured but not tested",
  "storage_status": "configured but not tested"
}
```

## Configuration

### Hardcoded Parameters (for POC)

The following parameters are hardcoded in the function code for demonstration:

- **Key Vault URL**: `https://your-keyvault.vault.azure.net/`
- **Storage Account URL**: `https://yourstorageaccount.blob.core.windows.net`
- **Timer Schedule**: Every 5 minutes (`0 */5 * * * *`)

### Environment Variables

The function automatically receives these environment variables:

- `AZURE_CLIENT_ID`: Managed identity client ID
- `WEBSITE_SITE_NAME`: Function app name
- `AZURE_FUNCTIONS_ENVIRONMENT`: Environment name

## Managed Identity Setup

The Bicep template automatically creates:

1. **User-assigned Managed Identity**
2. **Key Vault** with access policy for the managed identity
3. **Storage Account** with appropriate permissions
4. **Function App** configured with the managed identity

## Monitoring and Logging

- **Application Insights**: Automatically configured for monitoring
- **Function Logs**: Available in Azure Portal and Application Insights
- **Managed Identity**: Audit logs available in Azure AD

## Security Features

- **HTTPS Only**: Function app enforces HTTPS
- **Minimum TLS 1.2**: Enhanced security
- **Managed Identity**: No stored credentials
- **Key Vault Integration**: Secure secret management
- **Private Storage**: Public blob access disabled

## Troubleshooting

### Common Issues

1. **Function not starting**: Check Application Insights logs
2. **Managed Identity errors**: Verify permissions in Key Vault/Storage
3. **Deployment failures**: Check resource group permissions
4. **GitHub Actions failures**: Verify service principal permissions

### Useful Commands

```bash
# Check function logs
az functionapp logs tail --name <function-app-name> --resource-group <rg-name>

# Test managed identity
az rest --method get --url "https://management.azure.com/subscriptions" --headers "Authorization=Bearer $(az account get-access-token --query accessToken -o tsv)"

# Validate Bicep template
az deployment group validate --resource-group <rg-name> --template-file infra/main.bicep
```

## Next Steps

1. **Replace hardcoded parameters** with actual resource URLs
2. **Implement real business logic** in the functions
3. **Add more Azure service integrations** (Cosmos DB, Service Bus, etc.)
4. **Set up monitoring and alerting**
5. **Implement proper error handling and retry logic**
6. **Add unit tests and integration tests**

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Run tests locally
5. Submit a pull request

## License

This project is licensed under the MIT License - see the LICENSE file for details.
