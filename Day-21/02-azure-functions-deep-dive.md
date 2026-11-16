# Azure Functions Deep Dive

## Core Concepts: Function App vs Function

- **Function App:** The execution context and deployment container in Azure. All functions inside a Function App share the same pricing plan, runtime version, operating system, and environment variables.
- **Function:** The individual code block executed in response to a specific trigger.

---

## Triggers and Bindings

One of the most powerful features of Azure Functions is the declarative **Triggers and Bindings** system. Instead of writing boilerplate code to authenticate and connect to SDKs, you define bindings in configuration:

```
[ Trigger (What starts it) ] ---> [ Function Code ] ---> [ Output Binding (Where it writes) ]
```

### 1. Triggers
Define *how* a function is invoked. A function must have **exactly one** trigger:
- **HTTP Trigger:** Executed when an HTTP request (REST API or webhook) arrives.
- **Timer Trigger:** Executed on a CRON schedule (e.g., every night at midnight).
- **Blob Storage Trigger:** Executed when a new blob is created or updated in a container (Project 5 / Day 22).
- **Queue / Service Bus Trigger:** Executed when a message is added to an Azure Queue or Service Bus.
- **Event Grid Trigger:** Executed when a reactive event is published across Azure.

### 2. Input Bindings
Declaratively fetch external data into your function before your code runs without calling SDK methods.

### 3. Output Bindings
Declaratively write return values or payloads to external services (e.g., automatically save output to a Cosmos DB document or write to a storage blob).

---

## Azure Functions Hosting Plans

Choosing the right hosting plan determines scaling behavior, performance, and cost:

| Plan | Scaling Behavior | Cold Start | Max Execution Timeout | VNet Integration | Best For |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Consumption Plan** | Dynamic auto-scale from 0 to 200 instances. | Yes (typically 1-3 seconds on first request). | Default 5 mins (Max 10 mins). | No (cannot access private subnets). | Unpredictable or low-traffic workloads; cost optimization. |
| **Premium Plan** | Pre-warmed instances + auto-scale. | **Zero cold start**. | Default 30 mins (Max unbounded). | **Yes** (connects to private Azure VNets). | Enterprise production APIs requiring guaranteed latency and VNet connectivity. |
| **Dedicated (App Service Plan)** | Runs on dedicated VMs. | None. | Unbounded. | Yes. | Predictable continuous workloads or utilizing existing unused App Service capacity. |

---

## Local Development & Azure Functions Core Tools

DevOps engineers develop and test functions locally before pushing to Azure using the **Azure Functions Core Tools**:

```bash
# 1. Install Azure Functions Core Tools (v4)
npm install -g azure-functions-core-tools@4 --unsafe-perm true

# 2. Initialize a new Function App project in Python
func init MyServerlessProject --python

# 3. Create a new HTTP-triggered function
cd MyServerlessProject
func new --name HttpHealthCheck --template "HTTP trigger" --authlevel "anonymous"

# 4. Start the function locally
func start
```
