# Implementing Continuous Integration (CI) in Azure Pipelines

Continuous Integration (CI) automates the process of validating code changes, executing test suites, building container images, and pushing artifacts to a secure registry.

---

## Key Stages of the Voting App CI Pipeline

1. **Trigger Configuration (Path Filters):**
   - Automatically trigger pipeline runs only when code in specific directories (e.g., `vote/*`) changes on the `main` branch.
   - Prevents unnecessary builds when unrelated services or documentation are modified.

2. **Agent Pool Selection:**
   - **Microsoft-Hosted (`vmImage: 'ubuntu-latest'`):** Fresh, pre-configured VM managed by Microsoft.
   - **Self-Hosted (`name: 'azureagent'`):** Dedicated virtual machine or container running inside your private VNet with cached Docker layers for faster builds.

3. **Multi-Stage Structure:**
   - **Stage 1: Build:** Builds the Docker container image tagged with `$(Build.BuildId)` to guarantee immutable versioning.
   - **Stage 2: Push:** Authenticates against Azure Container Registry (ACR) via the Service Connection and pushes the tagged image.
   - **Stage 3: Scan / Validate (Optional):** Runs container vulnerability scans (e.g., Trivy or Microsoft Defender for Cloud).
   - **Stage 4: Update Manifests:** Runs a script to update the Kubernetes deployment YAML file with the new container image tag so the CD pipeline (Day 15 / ArgoCD) can deploy it.

---

## Step-by-Step Pipeline Creation in Azure DevOps

1. Navigate to **Pipelines** -> **Pipelines** in your Azure DevOps project.
2. Click **New Pipeline**.
3. Select **Azure Repos Git** and select your repository.
4. Select **Existing Azure Pipelines YAML file**.
5. Path: `/Day-14/voting-app-ci-pipeline.yaml` (or paste the pipeline YAML directly).
6. Click **Run**.
7. Monitor the build logs in real time as Docker builds each layer and pushes the final image to your Azure Container Registry.
8. Verify the pushed image in Azure CLI:

```bash
az acr repository list --name <your-acr-name> --output table
az acr repository show-tags --name <your-acr-name> --repository votingapp --output table
```
