# 06 - Serverless Hosting: Azure Static Web Apps (SWA)

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `03:36:13` to `03:47:55`

---

## 1. Static vs. Dynamic Web Architectures

Before deploying web applications, modern software engineers must understand the architectural distinction between static and server-rendered dynamic web hosting:

```
STATIC WEB ARCHITECTURE (Pre-built assets - JAMstack)
Browser ---> Global CDN / Edge Storage ---> Returns index.html, styles.css, bundle.js
             (Ultra-low latency, globally cached, no running servers required!)

DYNAMIC WEB ARCHITECTURE (Traditional Server-side rendering)
Browser ---> Cloud Web Server (Node.js/Python) ---> Database Query ---> Compiles HTML
             (Requires 24/7 active VM or container, higher operational cost and latency)
```

| Metric | Static Web Apps (JAMstack) | Traditional Server Apps |
| :--- | :--- | :--- |
| **Frontend Frameworks** | React, Vue.js, Angular, Svelte, Next.js (Static Export), Vanilla HTML/CSS/JS | Express (Node.js), Django (Python), Spring Boot (Java), ASP.NET Core |
| **Compute Requirement** | None! Pre-rendered static files are served directly from global edge storage. | Requires continuous CPU and RAM running 24/7/365 to execute code per request. |
| **Global Performance** | **Sub-second load times** globally due to edge CDN point-of-presence caching. | Variable; dependent on server load, database response times, and geographic distance. |
| **Hosting Cost** | **Free or fractions of a cent** per month. | Continuous hourly billing for VMs or App Service plans ($15–$150+/month). |
| **Maintenance** | Zero OS or web server patching. | Regular OS updates, security patches, runtime version upgrades. |

---

## 2. What is Azure Static Web Apps?

**Azure Static Web Apps (SWA)** is a modern cloud hosting service purpose-built for frontend developers. It automatically builds and deploys full-stack web applications to Microsoft Azure directly from a source code repository (GitHub or Azure DevOps).

### Key Features of Azure Static Web Apps:
1. **Global CDN Distribution:** Content is geographically replicated across Microsoft’s edge locations worldwide nearest to end users.
2. **Automated CI/CD via GitHub Actions:** Merging a Git commit automatically triggers a build pipeline and deploys changes within 60 seconds without manual FTP or portal clicking.
3. **Free Automatic SSL Certificates:** Out-of-the-box HTTPS with automated background renewals.
4. **Staging / Preview Environments:** Opening a Pull Request in GitHub automatically spins up a dedicated, isolated preview URL to review changes before merging!
5. **Integrated Serverless APIs:** Add seamless backend API endpoints powered by Azure Functions in the same Git repo.
6. **Free Tier:** The Free tier supports production-ready personal websites, student portfolios, and custom domains at $0 cost.

---

## 3. Hands-on Lab: Deploying a Web App from GitHub to Azure SWA

In this lab, we build a simple web page, push it to GitHub, and link it to Azure Static Web Apps to observe automated CI/CD:

### Step 1: Create a GitHub Repository
1. Log into your [GitHub](https://github.com) account.
2. Create a new public repository named `my-azure-portfolio`.
3. Create an `index.html` file in the root directory:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Azure Static Web App Masterclass</title>
  <style>
    body {
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
      text-align: center;
      background: linear-gradient(135deg, #0078d4, #002050);
      color: #ffffff;
      padding: 100px 20px;
      margin: 0;
    }
    .card {
      background: rgba(255, 255, 255, 0.12);
      backdrop-filter: blur(10px);
      padding: 40px;
      border-radius: 16px;
      display: inline-block;
      max-width: 600px;
      box-shadow: 0 8px 32px rgba(0, 0, 0, 0.35);
      border: 1px solid rgba(255, 255, 255, 0.2);
    }
    h1 { font-size: 2.2rem; margin-bottom: 15px; }
    p { font-size: 1.1rem; line-height: 1.6; color: #e1e1e1; }
    .badge {
      display: inline-block;
      padding: 8px 16px;
      background: #00bcf2;
      color: #002050;
      font-weight: bold;
      border-radius: 20px;
      margin-top: 15px;
    }
  </style>
</head>
<body>
  <div class="card">
    <h1>🚀 Live on Azure Static Web Apps!</h1>
    <p>This web application was automatically built and deployed via continuous integration using GitHub Actions and Microsoft Azure.</p>
    <div class="badge">Zero Infrastructure Management</div>
  </div>
</body>
</html>
```

Commit and push `index.html` to the `main` branch of your GitHub repository.

---

### Step 2: Create the Static Web App in the Azure Portal
1. Open the [Azure Portal](https://portal.azure.com).
2. In the top search bar, search for **Static Web Apps** and click **+ Create**.

#### Tab 1: Basics
- **Subscription:** Select your subscription.
- **Resource group:** Create new -> `rg-staticweb-demo`.
- **Name:** `swa-portfolio-demo`.
- **Plan type:** **Free** (Supports custom domains, global CDN, and free SSL).
- **Region:** Select your nearest region (e.g., `East US 2`).

#### Deployment Details:
- **Source:** Select **GitHub**.
- Click **Sign in with GitHub** and authorize Microsoft Azure.
- **Organization:** Select your GitHub username.
- **Repository:** Select `my-azure-portfolio`.
- **Branch:** Select `main`.

#### Build Details:
- **Build Presets:** Select **Custom** (or choose *React*, *Vue*, *Angular* if using a frontend framework).
- **App location:** `/` (The folder where `index.html` or `package.json` resides).
- **Api location:** Leave blank (for now).
- **Output location:** Leave blank for raw HTML (or set to `build` / `dist` for React/Vue).

3. Click **Review + create** -> **Create**.

---

### Step 3: Understanding the Automated GitHub Actions Workflow
The moment you click **Create**, Azure automatically commits a deployment workflow file into your GitHub repository:
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

#### What this workflow does:
1. **Trigger:** Fires whenever code is pushed to `main` OR when a Pull Request is opened against `main`.
2. **API Token:** Uses an encrypted GitHub repository secret (`AZURE_STATIC_WEB_APPS_API_TOKEN`) securely injected by Azure during creation.
3. **Action:** Builds the application, uploads static assets to Azure's edge CDN, and invalidates global caches.

---

### Step 4: Verify the Live Deployment
1. In the Azure Portal, open your Static Web App resource: `swa-portfolio-demo`.
2. On the **Overview** page, locate the **URL** (e.g., `https://proud-sea-039401.azurestaticapps.net`).
3. Click the link: Your web application opens immediately over secure HTTPS with an official Microsoft SSL certificate!

---

### Step 5: Automatic Updates via Git Commits
1. Locally or on GitHub, edit `index.html`.
2. Change the header text to:
   ```html
   <h1>🚀 Updated Live via Git Commit!</h1>
   ```
3. Commit and push:
   ```bash
   git commit -am "Update header text"
   git push origin main
   ```
4. Open the **Actions** tab in your GitHub repository.
5. You will see the workflow automatically triggered. Within 45 seconds, the job turns green.
6. Refresh your browser: the website updates globally with zero manual effort!

---

## 4. Staging Environments with Pull Requests

One of Azure Static Web Apps' most powerful features is **Automated Staging / Preview Environments**:

```
Developer creates feature branch: 'redesign-navbar'
                        |
                        v
              Open Pull Request on GitHub
                        |
                        v
GitHub Action detects PR -> Deploys to temporary URL:
https://proud-sea-039401-1.azurestaticapps.net
(Allows QA testers, designers, and team leads to test changes in live cloud!)
                        |
                        v
Pull Request Merged into 'main'
                        |
                        v
GitHub Action closes staging environment and automatically deletes the temporary URL!
```

---

## 5. Troubleshooting Static Web Apps

| Problem | Cause | Solution |
| :--- | :--- | :--- |
| GitHub Action fails with `App location does not exist`. | Incorrect `app_location` in YAML. | If `package.json` or `index.html` is in repo root, ensure `app_location: "/"`. If inside a subfolder, set `app_location: "/frontend"`. |
| Website displays blank page after React/Vue build. | Incorrect `output_location`. | For React (CRA), set `output_location: "build"`. For Vite/Vue, set `output_location: "dist"`. |
| Custom domain SSL certificate fails validation. | CNAME record not pointing to the generated Azure hostname. | Add a CNAME record at your registrar pointing your subdomain (e.g., `www`) to `<app-name>.azurestaticapps.net`. |

---

## 6. Summary Checklist
- [x] Static Web Apps are optimized for modern frontend frameworks (React, Vue, HTML/JS).
- [x] Pre-rendered static assets are served from global CDN edge locations with zero VM overhead.
- [x] Fully automated CI/CD is integrated natively with GitHub Actions.
- [x] Out-of-the-box HTTPS with free, auto-renewing Microsoft SSL certificates.
- [x] Pull Requests automatically generate temporary staging environments for testing.
