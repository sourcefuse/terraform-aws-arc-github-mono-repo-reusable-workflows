## Introduction
This document is designed to assist you in getting started with workflows and the project's Terraform mono repo.

## Prerequisites
- A GitHub account with access to the SourceFuse Organization
- Access to the projects mono repo repository
- Basic knowledge of Terraform and GitHub Actions

## Example Workflows
You can retrieve example workflows for use in your mono repo by doing the following:

- Clone the Repository:
   ```bash
   git clone https://github.com/sourcefuse/terraform-aws-arc-github-mono-repo-reusable-workflows.git
   ```

- Copy the `.github/` folder under `example/`
   ```bash
  cp -r example/.github/ /path/to/mono/repo/root/
  ```

- Copy the `scripts/` folder under `example/`
   ```bash
  cp -r example/scripts/ /path/to/mono/repo/root/
  ```

Once you've copied the `.github/` and `scripts/` folder to the root of your mono repo, you will need to update `sqs-*.yaml` and `network-*.yaml` files to include your relevant project information.

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

### Example Workflow Files
In the `.github/workflows` directory, you will find example workflow files, including `sqs-plan.yaml`, `sqs-apply.yaml`, `network-plan.yaml`, and `network-apply.yaml`.
These files are provided as examples to illustrate the configuration of Terraform workflows for a mono repo. You should modify these to suite your specific use case(s).

While the following files are a helpful starting point, it may not fulfill all the specific requirements of your use case. Consider customizing these files based on your project's needs, environment variables, and specific Terraform configurations.

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
            - `terraform/sqs/*`
            - `terraform/sqs/**`
            - `.github/workflows/apply-sqs.yaml`
            - `.github/workflows/reusable-cd-workflow.yaml`

- `snyk.yaml`
    - **Purpose:** This example workflow is designed to perform a security scan using Snyk.
    - **Usage:** Configured to run on every push to branches, excluding `main`.

### Customization and Considerations
When utilizing these example workflows, keep in mind the following:

- **Environment Variables:** Review, update, or add the environment variables in the workflows to match your project's requirements.
- **Workflow Triggers:** Adjust the trigger conditions to align with your preferred workflow events.
- **Terraform Configurations:** These examples assume a generic Terraform mono repo setup. Adapt the workflows to accommodate any specific configurations used in your project.

The provided example workflows are meant to be starting points that you can build upon to suit the needs of your Terraform configuration for the mono repo. Thoroughly test and validate the workflows in a lower environment before relying on them in production scenarios.
