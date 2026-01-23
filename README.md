# Terraform Multi-Cloud Destroy Workflow

![Release](https://github.com/subhamay-bhattacharyya-gha/terraform-destroy-wf/actions/workflows/release.yaml/badge.svg)&nbsp;![Built with Kiro.dev](https://img.shields.io/badge/Built_with-Kiro.dev-brightgreen?logo=data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMjQiIGhlaWdodD0iMjQiIHZpZXdCb3g9IjAgMCAyNCAyNCIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj4KPHBhdGggZD0iTTEyIDJMMTMuMDkgOC4yNkwyMCA5TDEzLjA5IDE1Ljc0TDEyIDIyTDEwLjkxIDE1Ljc0TDQgOUwxMC45MSA4LjI2TDEyIDJaIiBmaWxsPSJ3aGl0ZSIvPgo8L3N2Zz4K)&nbsp;![Terraform](https://img.shields.io/badge/Terraform-7B42BC?logo=terraform&logoColor=white)![Commit Activity](https://img.shields.io/github/commit-activity/t/subhamay-bhattacharyya-gha/terraform-destroy-wf)&nbsp;![Last Commit](https://img.shields.io/github/last-commit/subhamay-bhattacharyya-gha/terraform-destroy-wf)&nbsp;![Release Date](https://img.shields.io/github/release-date/subhamay-bhattacharyya-gha/terraform-destroy-wf)&nbsp;![Repo Size](https://img.shields.io/github/repo-size/subhamay-bhattacharyya-gha/terraform-destroy-wf)&nbsp;![File Count](https://img.shields.io/github/directory-file-count/subhamay-bhattacharyya-gha/terraform-destroy-wf)&nbsp;![Issues](https://img.shields.io/github/issues/subhamay-bhattacharyya-gha/terraform-destroy-wf)&nbsp;![Top Language](https://img.shields.io/github/languages/top/subhamay-bhattacharyya-gha/terraform-destroy-wf)&nbsp;![Custom Endpoint](https://img.shields.io/endpoint?url=https://gist.githubusercontent.com/bsubhamay/c94e325256e7e02c59bb5ab40c563924/raw/terraform-destroy-wf.json?)

A reusable GitHub Actions workflow for safely destroying Terraform infrastructure across multiple cloud providers (AWS, GCP, Azure, Snowflake, Databricks) with support for different backend configurations.

## Overview

This workflow provides a standardized approach to destroying Terraform infrastructure with built-in safety measures, multi-cloud authentication, and flexible backend support. It's designed to be called from other workflows as a reusable component.

### Key Features

- **Multi-Cloud Support**: AWS, Google Cloud Platform, Azure, Snowflake, and Databricks
- **Flexible Backend Configuration**: S3 and Terraform Cloud backends
- **Secure Authentication**: OIDC/Workload Identity Federation for keyless authentication
- **Input Validation**: Comprehensive validation of cloud provider and required parameters
- **Debug Output**: Detailed logging of inputs and configuration for troubleshooting

---

## Workflow Steps

The workflow performs the following steps:

1. **Debug Output**: Prints all input parameters and secrets (with sensitive values redacted) for troubleshooting
2. **Cloud Provider Validation**: Validates that the cloud provider is one of: `aws`, `gcp`, or `azure` (Note: Snowflake and Databricks validation is handled by the underlying action)
3. **Cloud Authentication**: Configures credentials based on the selected cloud provider:
   - **AWS**: Uses OIDC to assume an IAM role
   - **GCP**: Configures Workload Identity Federation
   - **Azure**: Authenticates using service principal with OIDC
   - **Snowflake**: Authentication is handled by the Terraform Destroy action
   - **Databricks**: Authentication is handled by the Terraform Destroy action
4. **Terraform Destroy**: Executes the destroy operation using the `tf-destroy-action`

---

## Supported Cloud Providers

| Provider | Authentication Method | Required Secrets |
|----------|----------------------|------------------|
| **AWS** | IAM Role Assumption | `aws-role-to-assume` |
| **GCP** | Workload Identity Federation | `gcp-wif-provider`, `gcp-service-account` |
| **Azure** | Service Principal with OIDC | `azure-client-id`, `azure-tenant-id`, `azure-subscription-id` |
| **Snowflake** | Private Key Authentication (via action) | `snowflake-private-key` |
| **Databricks** | Personal Access Token (via action) | `databricks-host`, `databricks-token` |

**Note**: AWS, GCP, and Azure authentication is configured directly in the workflow. Snowflake and Databricks authentication is handled by the underlying `tf-destroy-action`.

---

## Inputs

| Name | Description | Required | Default | Type |
|------|-------------|----------|---------|------|
| `cloud-provider` | Target cloud provider (`aws`, `gcp`, `azure`, `snowflake`, `databricks`) | Yes | — | string |
| `terraform-dir` | Directory containing Terraform files | No | `tf` | string |
| `backend-type` | Backend type (`s3` for AWS S3 or `remote` for Terraform Cloud) | No | `s3` | string |
| `aws-region` | AWS region for authentication (required for AWS) | No | — | string |
| `s3-bucket` | S3 bucket name for Terraform state (required for S3 backend) | No | — | string |
| `s3-region` | S3 bucket region for Terraform state (required for S3 backend) | No | — | string |
| `s3-key-prefix` | Optional prefix for the S3 state key (AWS only) | No | `""` | string |
| `snowflake-account` | Snowflake account identifier (required for Snowflake) | No | — | string |
| `snowflake-user` | Snowflake user name (required for Snowflake) | No | — | string |
| `snowflake-role` | Snowflake role name (required for Snowflake) | No | — | string |

## Secrets

| Name | Description | Required When |
|------|-------------|---------------|
| `tfc-token` | Terraform Cloud API token | `backend-type` is `remote` |
| `aws-role-to-assume` | AWS IAM role ARN to assume | `cloud-provider` is `aws` |
| `gcp-wif-provider` | GCP Workload Identity Federation provider | `cloud-provider` is `gcp` |
| `gcp-service-account` | GCP service account email | `cloud-provider` is `gcp` |
| `azure-client-id` | Azure client ID for service principal authentication | `cloud-provider` is `azure` |
| `azure-tenant-id` | Azure tenant ID for service principal authentication | `cloud-provider` is `azure` |
| `azure-subscription-id` | Azure subscription ID for service principal authentication | `cloud-provider` is `azure` |
| `snowflake-private-key` | Snowflake private key for authentication | `cloud-provider` is `snowflake` |
| `databricks-host` | Databricks workspace URL | `cloud-provider` is `databricks` |
| `databricks-token` | Databricks personal access token | `cloud-provider` is `databricks` |

---

## Usage Examples

### AWS with S3 Backend

```yaml
name: Destroy AWS Infrastructure

on:
  workflow_dispatch:
    inputs:
      confirm-destroy:
        description: 'Type "destroy" to confirm'
        required: true

jobs:
  destroy-aws:
    if: github.event.inputs.confirm-destroy == 'destroy'
    uses: subhamay-bhattacharyya-gha/terraform-destroy-wf/.github/workflows/terraform-destroy.yaml@main
    with:
      cloud-provider: aws
      terraform-dir: infra/aws/tf
      backend-type: s3
      aws-region: us-east-1
      s3-bucket: my-terraform-state-bucket
      s3-region: us-east-1
    secrets:
      aws-role-to-assume: ${{ secrets.AWS_DESTROY_ROLE_ARN }}
```

### GCP with Terraform Cloud Backend

```yaml
name: Destroy GCP Infrastructure

on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to destroy'
        required: true
        type: choice
        options:
          - dev
          - staging

jobs:
  destroy-gcp:
    uses: subhamay-bhattacharyya-gha/terraform-destroy-wf/.github/workflows/terraform-destroy.yaml@main
    with:
      cloud-provider: gcp
      terraform-dir: infra/gcp/tf
      backend-type: remote
    secrets:
      tfc-token: ${{ secrets.TF_API_TOKEN }}
      gcp-wif-provider: ${{ secrets.GCP_WIF_PROVIDER }}
      gcp-service-account: ${{ secrets.GCP_SERVICE_ACCOUNT }}
```

### Azure with S3 Backend

```yaml
name: Destroy Azure Infrastructure

on:
  workflow_dispatch:
    inputs:
      resource-group:
        description: 'Resource group to destroy'
        required: true

jobs:
  destroy-azure:
    uses: subhamay-bhattacharyya-gha/terraform-destroy-wf/.github/workflows/terraform-destroy.yaml@main
    with:
      cloud-provider: azure
      terraform-dir: infra/azure/tf
      backend-type: s3
      s3-bucket: terraform-state-azure
      s3-region: us-east-1
    secrets:
      azure-client-id: ${{ secrets.AZURE_CLIENT_ID }}
      azure-tenant-id: ${{ secrets.AZURE_TENANT_ID }}
      azure-subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
```

### Snowflake with S3 Backend

```yaml
name: Destroy Snowflake Infrastructure

on:
  workflow_dispatch:
    inputs:
      confirm-destroy:
        description: 'Type "destroy" to confirm'
        required: true

jobs:
  destroy-snowflake:
    if: github.event.inputs.confirm-destroy == 'destroy'
    uses: subhamay-bhattacharyya-gha/terraform-destroy-wf/.github/workflows/terraform-destroy.yaml@main
    with:
      cloud-provider: snowflake
      terraform-dir: infra/snowflake/tf
      backend-type: s3
      s3-bucket: my-terraform-state-bucket
      s3-region: us-east-1
      snowflake-account: my-account
      snowflake-user: terraform-user
      snowflake-role: TERRAFORM_ROLE
    secrets:
      snowflake-private-key: ${{ secrets.SNOWFLAKE_PRIVATE_KEY }}
```

### Databricks with Terraform Cloud Backend

```yaml
name: Destroy Databricks Infrastructure

on:
  workflow_dispatch:
    inputs:
      workspace:
        description: 'Databricks workspace to destroy'
        required: true

jobs:
  destroy-databricks:
    uses: subhamay-bhattacharyya-gha/terraform-destroy-wf/.github/workflows/terraform-destroy.yaml@main
    with:
      cloud-provider: databricks
      terraform-dir: infra/databricks/tf
      backend-type: remote
    secrets:
      tfc-token: ${{ secrets.TF_API_TOKEN }}
      databricks-host: ${{ secrets.DATABRICKS_HOST }}
      databricks-token: ${{ secrets.DATABRICKS_TOKEN }}
```

### Multi-Environment Destroy

```yaml
name: Destroy All Environments

on:
  workflow_dispatch:
    inputs:
      environments:
        description: 'Comma-separated list of environments'
        required: true
        default: 'dev,staging'

jobs:
  destroy-environments:
    strategy:
      matrix:
        environment: ${{ fromJson(format('["{0}"]', join(split(github.event.inputs.environments, ','), '","'))) }}
    uses: subhamay-bhattacharyya-gha/terraform-destroy-wf/.github/workflows/terraform-destroy.yaml@main
    with:
      cloud-provider: aws
      terraform-dir: infra/aws/tf
      backend-type: s3
      aws-region: us-west-2
      s3-bucket: terraform-state-${{ matrix.environment }}
      s3-region: us-west-2
    secrets:
      aws-role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
```

---

## Prerequisites

### AWS Setup
1. Create an IAM role with appropriate permissions for Terraform destroy operations
2. Configure OIDC identity provider in AWS for GitHub Actions
3. Set up S3 bucket for state storage (if using S3 backend)

### GCP Setup
1. Enable Workload Identity Federation
2. Create a service account with necessary permissions
3. Configure the WIF provider to trust your GitHub repository

### Azure Setup
1. Create a service principal with appropriate permissions for Terraform destroy operations
2. Configure Workload Identity Federation in Azure for GitHub Actions
3. Register your GitHub repository as a federated credential
4. Ensure the service principal has access to the subscription and resource groups

### Terraform Cloud Setup
1. Create a Terraform Cloud organization and workspace
2. Generate an API token with appropriate permissions
3. Configure your Terraform backend to use Terraform Cloud

### Snowflake Setup
1. Create a Snowflake user for Terraform operations
2. Generate a private key for key-pair authentication
3. Assign appropriate role with necessary permissions for destroy operations
4. Store the private key securely in GitHub Secrets
5. Ensure the `tf-destroy-action` supports Snowflake authentication

### Databricks Setup
1. Create a Databricks workspace
2. Generate a personal access token with appropriate permissions
3. Ensure the token has access to the resources you want to destroy
4. Store the workspace URL and token securely in GitHub Secrets
5. Ensure the `tf-destroy-action` supports Databricks authentication

---

## Security Considerations

- **Principle of Least Privilege**: Ensure IAM roles and service accounts have minimal required permissions
- **Environment Protection**: Use environment protection rules for production destroys
- **Manual Approval**: Consider requiring manual approval for destroy operations
- **State Backup**: Always backup Terraform state before destroy operations
- **Audit Logging**: Enable audit logging for all destroy operations

---

## Troubleshooting

### Common Issues

**Empty terraform-dir input**
- Ensure the calling workflow passes the `terraform-dir` input correctly
- Check that you're using the correct version/branch of this workflow

**Authentication failures**
- Verify that all required secrets are set in the repository
- Check that OIDC/WIF is properly configured for your cloud provider (AWS, GCP, Azure)
- Ensure the service account/role has sufficient permissions
- For Azure: Verify that the service principal has the correct permissions and the federated credential is properly configured
- For Snowflake/Databricks: Ensure the `tf-destroy-action` is properly configured to handle authentication

**Backend configuration errors**
- Verify S3 bucket exists and is accessible (for S3 backend)
- Check Terraform Cloud workspace configuration (for remote backend)
- Ensure backend configuration matches the specified backend-type

### Debug Output

The workflow provides detailed debug output including:
- All input parameters (with secrets masked)
- Backend configuration (S3 or Terraform Cloud)
- Authentication inputs for all cloud providers
- Cloud provider validation (for AWS, GCP, Azure)
- Authentication status for AWS, GCP, and Azure

**Note**: The workflow validates only AWS, GCP, and Azure in the validation step. Snowflake and Databricks validation is delegated to the `tf-destroy-action`.

---

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## License

MIT
