# 06 - Serverless Hosting: Azure Static Web Apps (SWA)

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `03:36:13` to `03:47:55`

---

## 1. Static vs. Dynamic Web Applications

Before choosing a cloud hosting service, it is critical to understand the distinction between static and dynamic web architectures:

```
STATIC WEB ARCHITECTURE (Pre-built assets)
Browser ---> CDN / Edge Storage ---> Returns index.html, styles.css, bundle.js
             (Extremely fast, global cache, no running server required!)

DYNAMIC WEB ARCHITECTURE (Server-side rendering)
Browser ---> Web Server (Node.js/Python) ---> Database Query ---> Compiles HTML
             (Requires 24/7 running VM or container, higher cost & latency)
```

| Characteristic | Static Web Apps | Dynamic Server Apps |
| :--- | :--- | :--- |
| **Technologies** | React, Vue.js, Angular, Svelte, Next.js (Static Export), Vanilla HTML/CSS/JS | Node.js Express, Django, Spring Boot, ASP.NET Core |
| **Compute Need** | Pre-rendered files served directly from edge locations (CDN). | Requires active CPU and RAM running 24/7 to process requests. |
| **Performance** | Sub-second load times globally due to edge caching. | Depends on server load, database query latency, and geography. |
| **Hosting Cost** | **Free or fractions of a cent** per month. | Billed continuously for VMs or App Service plans. |

---

## 2. What is Azure Static Web Apps?

**Azure Static Web Apps (SWA)** is a specialized cloud hosting service tailored for modern web frameworks and JAMstack architectures. It automatically builds and hosts full-stack web applications to Azure directly from a code repository (GitHub or Azure DevOps).

### Key Features of Azure Static Web Apps:
1. **Global CDN Distribution:** Content is replicated to Microsoft's global edge network nearest to the end user.
2. **Automated CI/CD via GitHub Actions:** Committing code to GitHub automatically builds and deploys your site without configuring deployment pipelines manually.
3. **Free Automatic SSL Certificates:** HTTPS is enabled out-of-the-box with automated renewals.
4. **Staging / Preview Environments:** Opening a Pull Request in GitHub automatically generates an isolated temporary preview URL so you can test changes before merging!
5. **Integrated Serverless APIs:** Seamlessly attach serverless API endpoints using Azure Functions or Azure App Services.
6. **Custom Domains:** Easily map your personal domain with free SSL.

---

## 3. Hands-on Lab: Deploying a Static Web App from GitHub

In this lab, we connect a GitHub repository to Azure Static Web Apps and observe automated CI/CD in action:

### Step 1: Prepare a Simple Web Project in GitHub
Create a new GitHub repository named `my-portfolio-app` with an `index.html` file:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Azure Static Web App Demo</title>
  <style>
    body {
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
      text-align: center;
      background: linear-gradient(135deg, #0078d4, #002050);
      color: white;
      padding: 100px 20px;
    }
    .card {
      background: rgba(255, 255, 255, 0.1);
      padding: 40px;
      border-radius: 12px;
      display: inline-block;
      box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
    }
  </style>
</head>
<body>
  <div class="card">
    <h1>🚀 Deployed with Azure Static Web Apps!</h1>
    <p>Automated continuous delivery powered by GitHub Actions and Microsoft Azure.</p>
  </div>
</body>
</html>
```

Push this to your GitHub repository on the `main` branch.

---

### Step 2: Create the Azure Static Web App in the Portal
1. Open the [Azure Portal](https://portal.azure.com).
2. Search for **Static Web Apps** and click **+ Create**.
3. **Basics Tab:**
   - **Subscription:** Select your subscription.
   - **Resource Group:** Create new -> `rg-staticweb-demo`.
   - **Name:** `swa-portfolio-demo`.
   - **Plan type:** **Free** (Supports personal projects, custom domains, and SSL).
   - **Region:** Select the region closest to you (e.g., `East US 2`).
4. **Deployment Details:**
   - **Source:** Select **GitHub**.
   - Click **Sign in with GitHub** and authorize Microsoft Azure.
   - **Organization:** Select your GitHub username.
   - **Repository:** Select `my-portfolio-app`.
   - **Branch:** Select `main`.
5. **Build Details:**
   - **Build Presets:** Select **Custom** (or choose *React*, *Vue*, *Angular* if using a framework).
   - **App location:** `/` (the root directory where `index.html` lives).
   - **Api location:** Leave blank.
   - **Output location:** Leave blank for raw HTML (or `build` / `dist` for React/Vue).
6. Click **Review + create** -> **Create**.

---

### Step 3: Understanding the Automated GitHub Actions Workflow
The moment you click **Create**, Azure commits a new GitHub Actions workflow file directly into your repository:
`.github/workflows/azure-static-web-apps-<random-string>.yml`

```yaml
name: Azure Static Web Apps CI/CD

on:
  push:
    branches:
      - main
  pull_request:
    types: [opened, synchronize, reopened, closed]
    branches:
      - main

jobs:
  build_and_deploy_job:
    if: github.event_name == 'push' || (github.event_name == 'pull_request' && github.event.action != 'closed')
    runs-on: ubuntu-latest
    name: Build and Deploy Job
    steps:
      - uses: actions/checkout@v3
        with:
          submodules: true
      - name: Build And Deploy
        id: builddeploy
        uses: Azure/static-web-apps-deploy@v1
        with:
          azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN }}
          repo_token: ${{ secrets.GITHUB_TOKEN }}
          action: "upload"
          app_location: "/"
          output_location: ""

  close_pull_request_job:
    if: github.event_name == 'pull_request' && github.event.action == 'closed'
    runs-on: ubuntu-latest
    name: Close Pull Request Job
    steps:
      - name: Close Pull Request
        id: closepullrequest
        uses: Azure/static-web-apps-deploy@v1
        with:
          azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN }}
          action: "close"
```

---

### Step 4: Verify the Live Deployment
1. Go back to your Azure Portal -> `swa-portfolio-demo`.
2. On the **Overview** page, look for the **URL** (e.g., `https://salmon-sea-039401.azurestaticapps.net`).
3. Click the link: Your custom web page loads globally over HTTPS with an official Microsoft SSL certificate!

---

### Step 5: Testing the Git Workflow (Automatic Updates)
1. In your local repository or on GitHub, edit `index.html`.
2. Change the header text to:
   ```html
   <h1>🚀 Updated Live via Git Commit!</h1>
   ```
3. Commit and push:
   ```bash
   git commit -am "Update header text"
   git push origin main
   ```
4. Open the **Actions** tab on your GitHub repo.
5. You will see the workflow automatically triggered. Within 45 seconds, the job turns green.
6. Refresh your live URL: the website updates instantly without touching the Azure Portal!

---

## 4. Staging Environments with Pull Requests

One of the most powerful features of Azure Static Web Apps is **Automatic Preview Environments**:

```
Feature Branch 'add-about-page'
               |
               v
      Create Pull Request
               |
               v
GitHub Action detects PR
               |
               v
Spins up temporary URL: https://salmon-sea-039401-1.azurestaticapps.net
(Allows QA, designers, and managers to review live changes before merging!)
               |
               v
PR Merged -> Temporary URL is automatically destroyed!
```

---

## 5. Summary Checklist
- [x] Static Web Apps are optimized for pre-rendered frameworks (React, Vue, HTML/JS).
- [x] No virtual machine or web server administration is required.
- [x] Seamless CI/CD is automatically established via GitHub Actions workflows.
- [x] Includes free global CDN edge distribution and auto-renewing SSL certificates.
- [x] Pull Requests automatically generate temporary staging URLs for pre-production testing.
