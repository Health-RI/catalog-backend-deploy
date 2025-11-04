# Deployment Workflow Repository

This repository contains the GitHub Actions workflow used to streamline the release and deployment processes for the following services:

- **CKAN**
- **Discovery Service**
- **Solr**

The workflow automates deployments to both `acceptance` (ACC) and `production` (PROD) environments using Azure Web Apps. Deployments now use Azure Site Containers, so each web app is updated by running `az webapp sitecontainers update` during the workflow.

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
     - `version_solr`: The version of Solr to deploy (e.g., `1.0.0`). CHECK FIRST [SOLR Reindexing Instructions](#️important-solr-reindex-required)!!!
     - `version_solr_with_lower_dashes`: The Solr version in a lower-dashed format (e.g., `1_0_0`). CHECK FIRST [SOLR Reindexing Instructions](#️important-solr-reindex-required)!!!

2. **Environment Variable Injection**:
   - The workflow uses the specified input versions as environment variables.

### Azure Site Containers

- After authenticating with Azure, each job lists the configured site containers (`az webapp sitecontainers list`) and picks the one flagged as `isMain`, so the workflow does not need hard-coded container names.
- Image swaps are executed with `az webapp sitecontainers update --image IMAGE:TAG`, preserving every other container setting (ports, commands, volumes, etc.).
- Resource groups are tracked per environment (e.g., `healthri-uac` for ACC, `healthri` for PROD) via dedicated environment variables to ensure the CLI targets the correct App Service instance.

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


## ⚠️IMPORTANT: SOLR Reindex Required
> When updating the SOLR version, a reindex is required. Follow the steps below to ensure proper reindexing:
>
> 1. **Clone the corresponding version of CKAN GDI**.
> 2. **Change the SOLR URL in environment variables**:
>    - **Acceptance**:
>      ```
>      CKAN_SOLR_URL=https://healthri-ckan-solr-uac.azurewebsites.net/solr/ckan
>      ```
>    - **Production**:
>      ```
>      CKAN_SOLR_URL=https://healthri-ckan-solr.azurewebsites.net/solr/ckan
>      ```
> 3. **Copy `CKAN_SQLALCHEMY_URL`**:
>    - **Acceptance**: Retrieve the `CKAN_SQLALCHEMY_URL` from the [Azure Portal](https://portal.azure.com/#@health-ri.nl/resource/subscriptions/0969fd57-dd8b-4543-ad41-d889e068285e/resourceGroups/healthri-uac/providers/Microsoft.Web/sites/healthri-ckan-uac/environmentVariablesAppSettings) and set it in your local environment.
>    - **Production**: Retrieve the `CKAN_SQLALCHEMY_URL` from the [Azure Portal](https://portal.azure.com/#@health-ri.nl/resource/subscriptions/0969fd57-dd8b-4543-ad41-d889e068285e/resourceGroups/healthri/providers/Microsoft.Web/sites/healthri-ckan/environmentVariablesAppSettings) and set it in your local environment.
> 4. **Run Docker Compose build**:
>    ```
>    docker compose build
>    ```
> 5. **Run Docker Compose up**:
>    ```
>    docker compose up
>    ```
> 6. **Run the reindex command within the container**:
>    ```
>    ckan -c ckan.ini search-index rebuild
>    ```

The `docker-compose.yml` file in this repository now serves only the manual reindex procedure described above. Release deployments update Solr directly via Azure Site Containers, so the workflow no longer edits the compose file.
