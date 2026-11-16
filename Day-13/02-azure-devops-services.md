# The Five Core Services of Azure DevOps

Azure DevOps consists of five primary independent services that can be used together or individually:

---

## 1. Azure Boards (Plan & Track)

Azure Boards provides interactive tools to track ideas, requirements, bugs, and tasks using Agile, Scrum, or Kanban methodologies.

### Key Capabilities:
- **Work Items:** Fundamental building blocks used to track information (Epic, Feature, User Story, Product Backlog Item, Task, Bug, Issue).
- **Kanban Boards:** Visual drag-and-drop boards to monitor work progression across stages (e.g., *To Do*, *In Progress*, *Code Review*, *Done*).
- **Backlogs & Sprints:** Organize priorities, plan upcoming sprint cycles, and allocate work across team members.
- **Dashboards & Queries:** Create custom queries (e.g., all high-severity bugs assigned to my team) and build visual charts.
- **GitHub Integration:** Link commits, branches, and pull requests directly to Azure Boards work items using `#<ID>`.

---

## 2. Azure Repos (Version Control)

Azure Repos provides unlimited, cloud-hosted private Git repositories or Team Foundation Version Control (TFVC).

### Key Capabilities:
- **Standard Git Support:** Compatible with any standard Git client (`git clone`, `git push`, `git checkout`).
- **Pull Requests (PR):** Collaborative code reviews with threaded discussions, inline comments, and required reviewers.
- **Branch Policies:** Protect sensitive branches like `main`:
  - Require a minimum number of reviewers before merging.
  - Require successful build validation (CI pipeline must pass).
  - Require linked work items.
  - Check for resolved comments.
- **Forks:** Isolate experimental or open-source contributions.
- **Semantic Code Search:** Fast, indexed searching across all repositories.

---

## 3. Azure Pipelines (Build & Release Automation)

Azure Pipelines is a cloud-hosted Continuous Integration (CI) and Continuous Delivery (CD) engine that automatically builds, tests, and deploys code to any destination.

### Key Capabilities:
- **YAML Pipelines (Pipeline as Code):** Define build and deployment workflows directly in version-controlled `azure-pipelines.yml` files.
- **Classic Editor (UI-based):** Visual drag-and-drop interface for creating build and release pipelines (primarily for legacy projects).
- **Cross-Platform Agents:**
  - **Microsoft-hosted Agents:** Managed VMs provisioned on demand with pre-installed developer tools (Ubuntu, Windows Server, macOS).
  - **Self-hosted Agents:** Agents installed on your own VMs or on-prem hardware for private network access or custom software dependencies.
- **Multi-Stage Pipelines:** Orchestrate end-to-end flows: `Build` -> `Test` -> `Deploy to Dev` -> `Approval Gate` -> `Deploy to Prod`.
- **Integration with Container Registries:** Seamlessly build and push Docker images to Azure Container Registry (ACR), Docker Hub, or private registries.

---

## 4. Azure Artifacts (Package Management)

Azure Artifacts enables developers to create, host, and share packages within an organization or publish them publicly.

### Key Capabilities:
- **Supported Package Types:**
  - **Maven / Gradle** (Java)
  - **npm** (Node.js / JavaScript)
  - **NuGet** (.NET / C#)
  - **pip / twine** (Python)
  - **Universal Packages** (Raw binaries, compiled datasets)
- **Upstream Sources:** Cache packages from public registries (npmjs.com, nuget.org, Maven Central) to ensure builds remain resilient even if external sources face downtime.
- **Pipeline Integration:** Feed dependencies into build pipelines and publish build outputs as versioned artifacts.

---

## 5. Azure Test Plans (Quality Assurance)

Azure Test Plans provides browser-based tools for manual, exploratory, and acceptance testing.

### Key Capabilities:
- **Manual Test Execution:** Create and execute scripted test cases with step-by-step instructions.
- **Exploratory Testing:** Browser extension to capture screenshots, record user actions, and file bugs directly with rich contextual diagnostics.
- **Traceability:** Full end-to-end traceability connecting requirements, test cases, execution results, and associated bugs.
- **User Acceptance Testing (UAT):** Enable stakeholders and business users to test and approve features before general release.
