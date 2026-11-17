# 06 - Azure Static Web Apps

> **Video Reference:** [Microsoft AZURE in ONE VIDEO (MPrashant)](https://youtu.be/M1erI5a2wxQ)  
> **Source Timeframe:** `03:36:13` to `03:47:55`

---

## 1. What is a Static Web Application?

In modern web development, websites fall into two broad architectures:

```
+------------------------------------+    +------------------------------------+
|    TRADITIONAL DYNAMIC WEB APP     |    |      MODERN STATIC WEB APP         |
|  (e.g., PHP, Django, WordPress)    |    |  (e.g., React, Vue, Angular, HTML) |
|                                    |    |                                    |
| [ User ] ---> [ Server executes ]  |    | [ User ] ---> [ Global CDN cache ] |
|               [ code for every  ]  |    |               [ serves pre-built ] |
|               [ single request  ]  |    |               [ HTML/CSS/JS files] |
|                                    |    |                                    |
| Requires heavy VM / web server     |    | Serverless, lightning fast & cheap |
+------------------------------------+    +------------------------------------+
```

A **Static Web Application** consists of pre-built static assets (HTML, CSS, JavaScript, images, and client-side single-page applications built with React, Angular, Vue, Next.js, or Svelte). 

When a user visits the website, there is no need to spin up a Python or PHP server to generate HTML on the fly. The browser simply downloads the pre-built files and executes the JavaScript client-side.

---

## 2. What is Azure Static Web Apps?

**Azure Static Web Apps (SWA)** is Microsoft's purpose-built, serverless hosting service for modern web developers.

Instead of paying for a Virtual Machine and configuring web servers, you simply connect Azure Static Web Apps to your **GitHub** or **Azure DevOps** code repository. 

Every time you `git push` new code or merge a Pull Request, Azure automatically builds your project and deploys it to a **globally distributed Content Delivery Network (CDN)**!

---

## 3. Superpowers of Azure Static Web Apps

### 1. Automated GitHub Actions CI/CD Built-In
You don't need to write deployment scripts. When you create an Azure Static Web App, Azure automatically writes a **GitHub Actions workflow file** (`.github/workflows/azure-static-web-apps.yml`) into your repository. Every commit to `main` deploys automatically!

### 2. Global Edge Distribution (Ultra-Fast)
Your HTML and JavaScript files are cached across hundreds of Microsoft edge datacenters worldwide. A user in Tokyo gets files from Japan; a user in London gets files from the UK. The site loads in under a second everywhere.

### 3. Free SSL/TLS Certificates & Custom Domains
You get a free `azurestaticapps.net` domain immediately, and you can add your custom domain (e.g., `portfolio.studentlab.com`) with **free, auto-renewing SSL certificates**.

### 4. Seamless Serverless APIs (Azure Functions)
Need a backend for your static website (e.g., a contact form or Stripe payment processing)? You can add an `api` folder with Python, Node.js, or C# code. Azure automatically turns it into serverless **Azure Functions** endpoints without managing any backend servers!

### 5. Pull Request Preview Environments
When a team member opens a Pull Request (PR) with changes, Azure Static Web Apps automatically builds a **temporary preview URL** (e.g., `https://salmon-sea-0a1b2c3d-preview.azurestaticapps.net`). You can test the changes live in your browser before merging to production. When the PR is merged, the preview environment is automatically destroyed!

### 6. Free Tier
The Free Tier of Azure Static Web Apps is 100% free with generous bandwidth (100 GB/month) and custom domains—ideal for student portfolios, blogs, and SaaS frontends.

---

## 4. Hands-on Walkthrough: Deploying a Web App from GitHub

In the video, M Prashant demonstrates the complete workflow from GitHub to live URL in under 3 minutes:

### Step 1: Prepare Code on GitHub
Ensure your frontend code (e.g. an `index.html` file or React app) is pushed to a GitHub repository (e.g., `https://github.com/my-username/my-portfolio`).

### Step 2: Create Azure Static Web App Resource
1. Search for **Static Web Apps** in the Azure Portal search bar -> Click **Create**.
2. **Project Details:**
   - **Subscription:** Select your subscription.
   - **Resource Group:** `rg-static-apps`.
   - **Name:** `my-student-portfolio`.
   - **Plan type:** Select **Free** (for personal/hobby apps).
   - **Region:** Choose a region close to your target audience (e.g., *East US 2* or *Central India*).

### Step 3: Connect to GitHub
1. Under **Deployment details**, choose **GitHub**.
2. Click **Sign in with GitHub** and authorize Microsoft Azure.
3. Select:
   - **Organization:** Your GitHub username.
   - **Repository:** `my-portfolio`.
   - **Branch:** `main`.

### Step 4: Build Presets
Under **Build Details**, choose your framework:
- If using plain HTML/CSS: Select **Custom**, App location: `/`.
- If using React: Select **React**, App location: `/`, Output location: `build`.
- If using Vue: Select **Vue.js**, Output location: `dist`.

### Step 5: Review & Automatic Deployment
1. Click **Review + Create**, then click **Create**.
2. Azure automatically commits a workflow file into your GitHub repo and triggers a GitHub Actions build!
3. Click **Go to resource** in the Azure Portal.
4. Copy the generated **URL**:
   ```
   https://agreeable-beach-05b1c940f.azurestaticapps.net
   ```
5. Paste it in your browser. Your website is live worldwide with full HTTPS encryption!

---

## 5. Summary Checklist
- [x] Static Web Apps host pre-built frontend files (React, Vue, HTML/CSS) over a global CDN.
- [x] Integrated seamlessly with GitHub Actions for automated Git-driven deployments.
- [x] Built-in free SSL certificates, custom domains, and staging preview links for Pull Requests.
- [x] Best suited for single-page applications (SPAs), developer portfolios, and documentation sites.
