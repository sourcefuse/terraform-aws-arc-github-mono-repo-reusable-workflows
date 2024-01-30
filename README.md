## Introduction
This document and repo are designed to assist you in getting started with workflows and the project's Terraform mono repo.

## Prerequisites
- A GitHub account with access to your Organization
  - GitHub OIDC configured, see [terraform-aws-arc-github-iam](https://github.com/sourcefuse/terraform-aws-arc-github-iam) for implementation information for your account
- Access to the project's mono repo repository
- Basic knowledge of Terraform and GitHub Actions

## Example Workflows
You can retrieve example workflows for use in your mono repo by doing the following:

- Clone the Repository:
   ```bash
   git clone https://github.com/sourcefuse/terraform-aws-arc-github-mono-repo-reusable-workflows.git
   ```

- Copy the `.github/` folder under `examples/`
   ```bash
  cp -r examples/.github/ /path/to/mono/repo/root/
  ```

- Copy the `scripts/` folder under `examples/`
   ```bash
  cp -r examples/scripts/ /path/to/mono/repo/root/
  ```

Once you've copied the `.github/` and `scripts/` folder to the root of your mono repo, you will need to update `sqs-*.yaml` and `network-*.yaml` files
to include your relevant project information.

See [Workflow Details](#workflow-details) for more information on the different workflow files and their usage.

## Workflow Details
The `.github/workflows` directory contains YAML files that define the different workflows available. Review the files to understand the steps and configurations.

- **Customization:**
    - Not all projects will use the same workflows. The files are to assist you in getting started.
    - Customize workflow files as needed for your project, (e.g. bash to pwsh, needs to include tagging, etc.).
    - Adjust environment variables, workflow triggers, and steps according to your requirements.

- **Output:**
    - Workflow runs can be found in the "Actions" tab of your repository.
    - Check for any errors or issues in the workflow runs.

### Terraform plan on Pull Requests
- **Purpose:** This workflow is triggered on pull requests to `main` and performs a Terraform plan to preview changes.
- **Trigger:** Triggered on pull requests.
- **Steps:**
    - Check out the repository
    - Set up pre-requisites for job runner
    - Connect to AWS account via assumed role
    - Run Terraform plan to preview changes
    - Publish plan output to Pull Request

### Terraform plan and apply on merging to main
- **Purpose:** This workflow is triggered when changes are merged to the `main` branch. It performs a Terraform plan and applies the changes.
- **Trigger:** Triggered on push to `main`.
- **Steps:**
    - Check out the repository
    - Set up pre-requisites for job runner
    - Connect to AWS account via assumed role
    - Run Terraform plan and publish `tfplan` as artifact
    - Apply Terraform changes if plan is successful, using the published `tfplan`

### Snyk Scan
- **Purpose:** The `snyk.yaml` workflow is designed to perform a scan using Snyk.
- **Trigger:** Configured to run on every push to branches, excluding `main`.
- **Steps:**
    - Check out the repository.
    - Set up Snyk.
    - Run IaC test to identify vulnerabilities.
- **Environment Variables:**
    - **ARC_SNYK_TOKEN**:  This is an Organization secret that can be assigned to your repo upon request.

### Reusable Workflows
- `reusable-ci-workflow.yaml`
  - **Purpose:** To provide a reusable way of implementing Continuous Integration (CI) workflows for a Terraform mono-repo.
  - **Usage:**
    - **Event Trigger:**
      - The workflow is triggered by the `workflow_call` event, indicating that it is intended to be called by another workflow.
    - **Input Parameters:**
      - Defines several input parameters for the workflow, such as `working_directory`, `environment`, `assume_role_arn`, `aws_region`, and `publish_plan_artifact`. 
        These parameters are meant to be provided when calling this workflow.
    - **Output Parameter:**
      - Specifies an output parameter `plan_id`, which is derived from the `plan` job's output.
    - **Environment Variables:**
      - Defines environment variables `DIRECTORY` and `ENV` based on the provided input parameters.
        - These are used in the `action-init-plan-apply.sh` script
    - **Job Definition:**
      - Defines a job named `plan` that runs on a `self-hosted` / `arc` runner. This runner can be updated to accommodate your organization runner usage. 
    - **Job Outputs:**
      - The `plan` job produces an output parameter `plan_id`, which is then used in the workflow's output section.
    - **Permissions:**
      - Specifies permissions for the job, granting write access to certain tokens for subsequent steps.
    - **Job Steps:**
      - The job includes multiple steps that perform various actions:
        - **Checkout:**
          - Checks out the repository code.
        - **Install Dependencies:**
          - Installs `tfenv` and the AWS CLI using custom scripts.
        - **Set Outputs:**
          - Sets an output parameter `plan_id` by echoing the short commit SHA to a file (`$GITHUB_OUTPUT`).
        - **Configure AWS Credentials:**
          - Configures AWS credentials for the specified environment.
        - **Initialize Backend:**
          - Initializes the Terraform backend using a custom script (`action-init-plan-apply.sh`).
        - **Plan:**
          - Runs the Terraform plan using the same custom script.
        - **Publish Terraform Plan:**
          - Publishes the Terraform plan artifact if the `publish_plan_artifact` option is true and the plan was successful.
        - **Update Plan Output to PR:**
          - Adds a comment to the pull request with the Terraform plan output.

- `reusable-cd-workflow.yaml`
  - **Purpose:** To provide a reusable way of implementing Continuous Delivery (CD) workflows for a Terraform mono-repo.
  - **Usage:**
    - **Event Trigger:**
      - The workflow is triggered by the `workflow_call` event, indicating that it is intended to be called by another workflow.
    - **Input Parameters:**
      - Defines several input parameters for the workflow, such as `working_directory`, `environment`, `assume_role_arn`, `aws_region`, and `plan_id`. 
        These parameters are meant to be provided when calling this workflow.
    - **Environment Variables:**
      - Defines environment variables `DIRECTORY` and `ENV` based on the provided input parameters.
        - These are used in the `action-init-plan-apply.sh` script
    - **Job Definition:**
      - Defines a job named `plan` that runs on a `self-hosted` / `arc` runner. This runner can be updated to accommodate your organization runner usage.
    - **Permissions:**
      - Specifies permissions for the job, granting write access to certain tokens for subsequent steps.
    - **Job Steps:**
      - The job includes multiple steps that perform various actions:
        - **Checkout:**
          - Checks out the repository code.
        - **Install Dependencies:**
          - Installs `tfenv` and the AWS CLI using custom scripts.
        - **Download tfplan Artifact:**
          - Downloads the Terraform plan (`tfplan`) artifact from previous steps using the `actions/download-artifact` action.
        - **Configure AWS Credentials:**
          - Configures AWS credentials for the specified environment.
        - **Initialize Backend:**
          - Initializes the Terraform backend using a custom script (`action-init-plan-apply.sh`).
        - **Apply:**
          - Runs the Terraform apply using the same custom script.

### Example Workflow Files
In the `.github/workflows` directory, you will find example workflow files, including `sqs-plan.yaml`, `sqs-apply.yaml`, `network-plan.yaml`, and `network-apply.yaml`.
These files are provided as examples to illustrate the configuration of Terraform workflows for a mono repo. You should modify these to suite your specific use case(s).

While the following files are a helpful starting point, it may not fulfill all the specific requirements of your use case. Consider customizing these files based on your
project's needs, environment variables, and specific Terraform configurations.

- `plan-sqs.yaml`
    - **Purpose:** This example workflow is designed to perform a Terraform plan operation for changes introduced by pull requests.
    - **Usage:**
        - It is triggered on pull requests to preview the changes before applying them.
        - Only runs on changes to:
            - `terraform/sqs/*`
            - `terraform/sqs/**`
            - `.github/workflows/plan-sqs.yaml`
            - `.github/workflows/reusable-ci-workflow.yaml`

- `apply-sqs.yaml`
    - **Purpose:** This example workflow is configured to execute Terraform apply for changes merged into the `main` branch.
    - **Usage:**
        - It is triggered on push events to the `main` branch.
        - Only runs on changes to:
            - `terraform/sqs/*`
            - `terraform/sqs/**`
            - `.github/workflows/apply-sqs.yaml`
            - `.github/workflows/reusable-cd-workflow.yaml`

- `plan-network.yaml`
    - **Purpose:** This example workflow is designed to perform a Terraform plan operation for network related changes.
    - **Usage:**
        - It is triggered on pull requests to preview the changes before applying them.
        - Only runs on changes to:
            - `terraform/network/*`
            - `terraform/network/**`
            - `.github/workflows/plan-network.yaml`
            - `.github/workflows/reusable-ci-workflow.yaml`

- `network-apply.yaml`
    - **Purpose:** This example workflow is configured to execute Terraform apply for network related changes.
    - **Usage:**
        - It is triggered on push events to the `main` branch.
        - Only runs on changes to:
            - `terraform/network/*`
            - `terraform/network/**`
            - `.github/workflows/apply-network.yaml`
            - `.github/workflows/reusable-cd-workflow.yaml`

- `snyk.yaml`
    - **Purpose:** This example workflow is designed to perform a security scan using Snyk.
    - **Usage:** Configured to run on every push to branches, excluding `main`.

### Customization and Considerations
When utilizing these example workflows, keep in mind the following:

- **Environment Variables:** Review, update, and / or add the environment variables in the workflows to match your project's requirements.
- **Workflow Triggers:** Adjust the trigger conditions to align with your preferred workflow events.
- **Terraform Configurations:** These examples assume a generic Terraform mono repo setup. Adapt the workflows to accommodate any specific configurations used in your project.

The provided example workflows are meant to be starting points that you can build upon to suit the needs of your Terraform configuration for the mono repo.
Thoroughly test and validate the workflows in a lower environment before relying on them in production scenarios.
