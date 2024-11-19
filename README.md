# Deployment Workflow Repository

This repository contains the GitHub Actions workflow used to streamline the release and deployment processes for the following services:

- **CKAN**
- **Discovery Service**
- **Solr**

The workflow automates deployments to both `acceptance` (ACC) and `production` (PROD) environments using Azure Web Apps.

---

## Workflow Overview

The workflow is triggered manually via the GitHub Actions interface. Users can specify the versions of CKAN, Discovery Service, and Solr at the time of triggering the workflow.

### Services Deployed

1. **CKAN**: A data management solution.
2. **Discovery Service**: Enables dataset discovery.
3. **Solr**: A search platform for indexing and querying datasets.

### Deployment Environments

- **Acceptance (ACC)**: Used for testing and validation.
- **Production (PROD)**: The live environment.

The `PROD` deployment depends on the successful completion of the `ACC` deployment. Which happes automatically after a weak

---

## How It Works

1. **Trigger the Workflow**:
   - Navigate to the **Actions** tab in the GitHub repository.
   - Select the **Publish Release** workflow.
   - Click **Run Workflow**.
   - Input the following versions:
     - `version_ckan`: The version of CKAN to deploy (e.g., `1.0.0`).
     - `version_discovery`: The version of the Discovery Service to deploy (e.g., `1.0.0`).
     - `version_solr`: The version of Solr to deploy (e.g., `1.0.0`).
     - `version_solr_with_lower_dashes`: The Solr version in a lower-dashed format (e.g., `1_0_0`).

2. **Environment Variable Injection**:
   - The workflow uses the specified input versions as environment variables.

3. **Deploy to Acceptance**:
   - The workflow first deploys the specified versions of CKAN, Discovery Service, and Solr to the ACC environment.

4. **Deploy to Production**:
   - After the ACC deployment completes successfully, the workflow deploys the same versions to the PROD environment.

5. **Monitor Progress**:
   - Follow the workflow's logs in the GitHub Actions interface for status updates and any potential issues.

6. **Verify**:
   - Validate the deployment in the ACC environment.
   - Confirm successful deployment to PROD after one weak.


---

## Environment Variables

The workflow uses both predefined and user-provided environment variables:

| Variable                     | Description                             |
|------------------------------|-----------------------------------------|
| `CKAN_NAME`                  | Container image name for CKAN          |
| `DISCOVERY_IMAGE`            | Container image name for Discovery      |
| `SOLR_IMAGE`                 | Container image name for Solr           |
| `version_ckan`               | User-specified version for CKAN         |
| `version_discovery`          | User-specified version for Discovery    |
| `version_solr`               | User-specified version for Solr         |
| `version_solr_with_lower_dashes` | User-specified Solr version in lower-dashed format |

---

## Prerequisites

### Secrets

The workflow uses the following GitHub secrets:

- `AZURE_CREDENTIALS`: Azure authentication credentials for deploying the services.


