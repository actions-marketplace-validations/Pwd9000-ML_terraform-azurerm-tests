# Terraform Tests - GitHub Action (terraform-azurerm-tests)

## Overview

Do you need a quick way to test if your terraform code changes will work?  
This Action can be used to run **Terraform tests** using an **AZURE** backend in a variety of scenarios.  

Say for example you are scanning and checking your terraform code with **dependabot** and **dependabot** raises a PR showing that you have dependencies that needs their versions upgraded. You can use this action to create a **GitHub Workflow** to run when **dependabot** creates the PR and run a series of tests before merging the PR.  

This action can also be used to run tests on any sort of changes in your terraform code and is not limited to usage with **dependabot**.  

**NOTE:** It is best to write a separate terraform configuration specifically for testing. See [Marketplace_Example_Tests.yml](https://github.com/Pwd9000-ML/Azure-Terraform-Deployments/blob/master/.github/workflows/Marketplace_Example.yml) for examples.

## Test Types

What sort of tests can be run using this Action?  

This action has a special input called `test_type:` which can be used to run different types of tests:  

* `test_type: "plan"`
    * This test type will only perform a terraform `plan` ONLY against a terraform configuration.
* `test_type: "plan-apply"`
    * This test type will perform a terraform `plan` AND a terraform `apply` in sequence against a terraform configuration.
* `test_type: "plan-apply-destroy"`
    * This test type will perform a terraform `plan`, a terraform `apply` AND a terraform `destroy` in sequence against a terraform configuration.

See my [detailed tutorial]() for more usage details.  

The following terraform tasks are run depending on the test chosen: `fmt`, `validate`, `init`, `plan`, `apply`, `destroy`.  
Any issues detected in these terraform tasks are commented on the PR and a valid `github_token` is needed.  
`plan` output is always commented onto the PR.  

![image.png]()

Additionally tests will include `plan` artifacts of the relevant `apply` and `destroy` operations added to the workflow.

![image.png]()

**WARNING:** Apply tests will create resources in your environment. Please be aware of cost and also please be aware of the environment used. When applying new resources ensure you are using a test subscription or resource group inside of your configuration file being targeted by the `path:` input.  
See [Marketplace_Example_Tests.yml](https://github.com/Pwd9000-ML/Azure-Terraform-Deployments/blob/master/.github/workflows/Marketplace_Example.yml) for examples.

## Installation

```yaml
steps:
  - name: Run-Tests
    uses: Pwd9000-ML/terraform-azurerm-tests@v1.0.0
    with:
      test_type: plan                          ## (Required) Valid options are "plan", "plan-apply", "plan-apply-destroy". Default="plan"
      path: "path-to-test-module"              ## (Optional) Specify path TF module relevant to repo root. Default="."
      tf_version: "latest"                     ## (Optional) Specifies version of Terraform to use. e.g: 1.0.0 Default=latest.
      tf_vars_file: "test-tfvars-file-name"    ## (Required) Specifies Terraform TFVARS file name inside module path (Testing vars)
      tf_key: "test-state-file-name"           ## (Required) AZ backend - Specifies name that will be given to terraform state file and plan artifact (testing state)
      az_resource_group: "resource-group-name" ## (Required) AZ backend - AZURE Resource Group hosting terraform backend storage account
      az_storage_acc: "storage-account-name"   ## (Required) AZ backend - AZURE terraform backend storage account
      az_container_name: "container-name"      ## (Required) AZ backend - AZURE storage container hosting state files 
      arm_client_id: ${{ secrets.ARM_CLIENT_ID }}             ## (Required - Dependabot Secrets) ARM Client ID 
      arm_client_secret: ${{ secrets.ARM_CLIENT_SECRET }}     ## (Required - Dependabot Secrets) ARM Client Secret
      arm_subscription_id: ${{ secrets.ARM_SUBSCRIPTION_ID }} ## (Required - Dependabot Secrets) ARM Subscription ID
      arm_tenant_id: ${{ secrets.ARM_TENANT_ID }}             ## (Required - Dependabot Secrets) ARM Tenant ID
      github_token: ${{ secrets.GITHUB_TOKEN }} ## (Required) Needed to comment output on PR's. ${{ secrets.GITHUB_TOKEN }} already has permissions.
```

## Examples

Check out the following [GitHub repository](https://github.com/Pwd9000-ML/Azure-Terraform-Deployments) for a full working demo and usage examples of this action under a workflow called [Marketplace_Example_Tests.yml](https://github.com/Pwd9000-ML/Azure-Terraform-Deployments/blob/master/.github/workflows/Marketplace_Example.yml).

## Usage Example 1 - Run Test Plan Only

Only perform a terraform plan against a terraform **test** module.

```yaml
#This workflow will automatically run tests when Dependabot opens a PR.
#Tests are only partial and is performed by creating a (PLAN ONLY) and outputting any issues detected to the PR.

name: "Marketplace-Example-Test-Plan-Only"
on:
  workflow_dispatch:
  pull_request:
    branches:
      - master

jobs:
# Dependabot will open a PR on terraform version changes, this 'dependabot' job is only used to test TF version changes by running a plan.
  Run_Tests:
    dependabot-plan-only: ubuntu-latest
    permissions:
      pull-requests: write
      issues: write
      repository-projects: write
    if: ${{ github.actor == 'dependabot[bot]' }}
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Plan-Only
        uses: Pwd9000-ML/terraform-azurerm-tests@v1.0.0
        with:
          test_type: plan                          ## (Required) Valid options are "plan", "plan-apply", "plan-apply-destroy". Default="plan"
          path: "path-to-test-module"              ## (Optional) Specify path TF module relevant to repo root. Default="."
          tf_version: "latest"                     ## (Optional) Specifies version of Terraform to use. e.g: 1.0.0 Default=latest.
          tf_vars_file: "test-tfvars-file-name"    ## (Required) Specifies Terraform TFVARS file name inside module path (Testing vars)
          tf_key: "test-state-file-name"           ## (Required) AZ backend - Specifies name that will be given to terraform state file and plan artifact (testing state)
          az_resource_group: "resource-group-name" ## (Required) AZ backend - AZURE Resource Group hosting terraform backend storage account
          az_storage_acc: "storage-account-name"   ## (Required) AZ backend - AZURE terraform backend storage account
          az_container_name: "container-name"      ## (Required) AZ backend - AZURE storage container hosting state files 
          arm_client_id: ${{ secrets.ARM_CLIENT_ID }}             ## (Required - Dependabot Secrets) ARM Client ID 
          arm_client_secret: ${{ secrets.ARM_CLIENT_SECRET }}     ## (Required - Dependabot Secrets) ARM Client Secret
          arm_subscription_id: ${{ secrets.ARM_SUBSCRIPTION_ID }} ## (Required - Dependabot Secrets) ARM Subscription ID
          arm_tenant_id: ${{ secrets.ARM_TENANT_ID }}             ## (Required - Dependabot Secrets) ARM Tenant ID
          github_token: ${{ secrets.GITHUB_TOKEN }} ## (Required) Needed to comment output on PR's. ${{ secrets.GITHUB_TOKEN }} already has permissions.
```

![image.png]()

## Usage Example 2 - Run Test Plan and Apply Plan

**WARNING:** This test will create resources in your environment. Please be aware of cost and also please be aware of the environment used. When applying new resources ensure you are using a test subscription or resource group inside of your configuration file being targeted by the `path:` input.  
See [Marketplace_Example_Tests.yml](https://github.com/Pwd9000-ML/Azure-Terraform-Deployments/blob/master/.github/workflows/Marketplace_Example.yml) for examples.

```yaml
#This workflow will automatically run tests when Dependabot opens a PR.
#Tests are only partial and is performed by creating a (PLAN AND APPLY) and outputting any issues detected to the PR.
#WARNING!!! This TEST will create resources!!

name: "Marketplace-Example-Test-Apply"
on:
  workflow_dispatch:
  pull_request:
    branches:
      - master

jobs:
# Dependabot will open a PR on terraform version changes, this 'dependabot' job is used to test TF version changes by running a plan and an apply.
  Run_Tests:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
      issues: write
      repository-projects: write
    if: ${{ github.actor == 'dependabot[bot]' }}
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Plan-Apply
        uses: Pwd9000-ML/terraform-azurerm-tests@v1.0.0
        with:
          test_type: plan-apply                    ## (Required) Valid options are "plan", "plan-apply", "plan-apply-destroy". Default="plan"
          path: "path-to-test-module"              ## (Optional) Specify path TF module relevant to repo root. Default="."
          tf_version: "latest"                     ## (Optional) Specifies version of Terraform to use. e.g: 1.0.0 Default=latest.
          tf_vars_file: "test-tfvars-file-name"    ## (Required) Specifies Terraform TFVARS file name inside module path (Testing vars)
          tf_key: "test-state-file-name"           ## (Required) AZ backend - Specifies name that will be given to terraform state file and plan artifact (testing state)
          az_resource_group: "resource-group-name" ## (Required) AZ backend - AZURE Resource Group hosting terraform backend storage account
          az_storage_acc: "storage-account-name"   ## (Required) AZ backend - AZURE terraform backend storage account
          az_container_name: "container-name"      ## (Required) AZ backend - AZURE storage container hosting state files 
          arm_client_id: ${{ secrets.ARM_CLIENT_ID }}             ## (Required - Dependabot Secrets) ARM Client ID 
          arm_client_secret: ${{ secrets.ARM_CLIENT_SECRET }}     ## (Required - Dependabot Secrets) ARM Client Secret
          arm_subscription_id: ${{ secrets.ARM_SUBSCRIPTION_ID }} ## (Required - Dependabot Secrets) ARM Subscription ID
          arm_tenant_id: ${{ secrets.ARM_TENANT_ID }}             ## (Required - Dependabot Secrets) ARM Tenant ID
          github_token: ${{ secrets.GITHUB_TOKEN }} ## (Required) Needed to comment output on PR's. ${{ secrets.GITHUB_TOKEN }} already has permissions.
```

![image.png]()

## Usage Example 3 - Run Test Plan -> Apply -> Destroy

**WARNING:** This test will create resources in your environment. Although the resources are immediately destroyed after being built. Please be aware of cost or errors that may cause the workflow to fail and leave resources behind, also please be aware of the environment used. When applying new resources ensure you are using a test subscription or resource group inside of your configuration file being targeted by the `path:` input.  
See [Marketplace_Example_Tests.yml](https://github.com/Pwd9000-ML/Azure-Terraform-Deployments/blob/master/.github/workflows/Marketplace_Example.yml) for examples.

```yaml
#This workflow will automatically run tests when Dependabot opens a PR.
#Tests are full end-to-end and is performed by creating a (PLAN AND APPLY AND DESTROY) and outputting any issues detected to the PR.
#WARNING!!! This TEST will create resources!!

name: "Marketplace-Example-Test-Apply-Destroy"
on:
  workflow_dispatch:
  pull_request:
    branches:
      - master

jobs:
# Dependabot will open a PR on terraform version changes, this 'dependabot' job is used to test TF version changes by running a plan, apply and destroy.
  Run_Tests:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
      issues: write
      repository-projects: write
    if: ${{ github.actor == 'dependabot[bot]' }}
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Plan-Apply-Destroy
        uses: Pwd9000-ML/terraform-azurerm-tests@v1.0.0
        with:
          test_type: plan-apply-destroy                    ## (Required) Valid options are "plan", "plan-apply", "plan-apply-destroy". Default="plan"
          path: "path-to-test-module"              ## (Optional) Specify path TF module relevant to repo root. Default="."
          tf_version: "latest"                     ## (Optional) Specifies version of Terraform to use. e.g: 1.0.0 Default=latest.
          tf_vars_file: "test-tfvars-file-name"    ## (Required) Specifies Terraform TFVARS file name inside module path (Testing vars)
          tf_key: "test-state-file-name"           ## (Required) AZ backend - Specifies name that will be given to terraform state file and plan artifact (testing state)
          az_resource_group: "resource-group-name" ## (Required) AZ backend - AZURE Resource Group hosting terraform backend storage account
          az_storage_acc: "storage-account-name"   ## (Required) AZ backend - AZURE terraform backend storage account
          az_container_name: "container-name"      ## (Required) AZ backend - AZURE storage container hosting state files 
          arm_client_id: ${{ secrets.ARM_CLIENT_ID }}             ## (Required - Dependabot Secrets) ARM Client ID 
          arm_client_secret: ${{ secrets.ARM_CLIENT_SECRET }}     ## (Required - Dependabot Secrets) ARM Client Secret
          arm_subscription_id: ${{ secrets.ARM_SUBSCRIPTION_ID }} ## (Required - Dependabot Secrets) ARM Subscription ID
          arm_tenant_id: ${{ secrets.ARM_TENANT_ID }}             ## (Required - Dependabot Secrets) ARM Tenant ID
          github_token: ${{ secrets.GITHUB_TOKEN }} ## (Required) Needed to comment output on PR's. ${{ secrets.GITHUB_TOKEN }} already has permissions.
```

![image.png]()

## Inputs

| Input | Required |Description |Default |
| ----- | -------- | ---------- | ------ |
| `path` | FALSE | Specify path to Terraform module relevant to repo root. | "." |
| `plan_mode` | FALSE | Specify plan mode. Valid options are `deploy` or `destroy`. | "deploy" |
| `tf_version` | FALSE | Specifies the Terraform version to use. | "latest" |
| `tf_vars_file` | TRUE | Specifies Terraform TFVARS file name inside module path. | N/A |
| `tf_key` | TRUE | AZ backend - Specifies name that will be given to terraform state file and plan artifact| N/A |
| `enable_TFSEC` | FALSE | Enable IaC TFSEC scan, results are posted to GitHub Project Security Tab. (Private repos require GitHub enterprise). | FALSE |
| `az_resource_group` | TRUE | AZ backend - AZURE Resource Group name hosting terraform backend storage account | N/A |
| `az_storage_acc` | TRUE | AZ backend - AZURE terraform backend storage account name | N/A |
| `az_container_name` | TRUE | AZ backend - AZURE storage container hosting state files  | N/A |
| `arm_client_id` | TRUE | The Azure Service Principal Client ID | N/A |
| `arm_client_secret` | TRUE | The Azure Service Principal Secret | N/A |
| `arm_subscription_id` | TRUE | The Azure Subscription ID | N/A |
| `arm_tenant_id` | TRUE | The Azure Service Principal Tenant ID | N/A |
| `github_token` | TRUE | Specify GITHUB TOKEN, only used in PRs to comment outputs such as `plan`, `fmt`, `init` and `validate`. `${{ secrets.GITHUB_TOKEN }}` already has permissions, but if using own token, ensure repo scope. | N/A |

## Outputs

None.  

* Plan is uploaded to workflow as an artifact. (Can be deployed using `Pwd9000-ML/terraform-azurerm-apply` action.)
* In a Pull Request, `plan` output will be added as a comment on the PR. Additionally failures on `fmt`, `init` and `validate` will also be added to the PR.

## Plan output to Pull Request

![image.png](https://raw.githubusercontent.com/Pwd9000-ML/terraform-azurerm-plan/master/assets/pr.png)  

The `github_token` input must be set for the PR comment to be added. `${{ secrets.GITHUB_TOKEN }}` already has permissions, but if using own token, ensure repo scope.  
Plan failures on `fmt`, `init` and `validate` will also be added to the PR.

## Versions of runner that can be used

At the moment only linux runners are supported. Support for all runner types will be released soon.

| Environment | YAML Label |
| --------------------|---------------------|
| Ubuntu 20.04 | `ubuntu-latest` or `ubuntu-20-04` |
| Ubuntu 18.04 | `ubuntu-18.04` |

## License

The associated scripts and documentation in this project are released under the [MIT License](LICENSE).
