# Project 2: Multi-Container Voting App Architecture

## Overview

The voting application is a multi-tier, multi-container microservices demo application designed to demonstrate real-world Continuous Integration (CI) and Continuous Deployment (CD) workflows using Azure DevOps, Azure Container Registry (ACR), and Kubernetes (AKS).

---

## Architectural Components

The application consists of five independent microservices built with different technology stacks:

```
                  +--------------------------------+
                  |         End Users              |
                  +---------------+----------------+
                                  |
                                  v
                      +-----------------------+
                      |      Vote App         | (Port 80/5000)
                      |   (Python / Flask)    |
                      +-----------+-----------+
                                  |
                           (Writes votes)
                                  v
                      +-----------------------+
                      |      Redis Queue      | (Port 6379)
                      |   (In-Memory Cache)   |
                      +-----------+-----------+
                                  |
                           (Pulls votes)
                                  v
                      +-----------------------+
                      |      Worker App       |
                      |     (.NET Core)       |
                      +-----------+-----------+
                                  |
                          (Persists votes)
                                  v
                      +-----------------------+
                      |      Postgres DB      | (Port 5432)
                      |  (Relational Storage) |
                      +-----------+-----------+
                                  ^
                           (Reads results)
                                  |
                      +-----------------------+
                      |      Result App       | (Port 80/4000)
                      |  (Node.js / Express)  |
                      +-----------+-----------+
                                  ^
                                  |
                  +---------------+----------------+
                  |         Audience / Admin       |
                  +--------------------------------+
```

### 1. Vote App (Frontend - User Voting)
- **Technology:** Python (Flask web application).
- **Functionality:** Presents an interactive web interface offering users a choice between two options (e.g., *Cats vs Dogs*).
- **Communication:** Sends the captured vote to the in-memory **Redis** queue.

### 2. Redis (Message Broker / Cache)
- **Technology:** Redis (official Docker image).
- **Functionality:** Acts as an in-memory buffer to absorb high-concurrency voting spikes without overwhelming the database.
- **Communication:** Receives votes from `vote` and serves them to the background `worker`.

### 3. Worker App (Background Processor)
- **Technology:** .NET Core / C#.
- **Functionality:** Continuously polls and pops votes from Redis, processes the data, and writes the persistent vote counts into the PostgreSQL database.
- **Communication:** Connects to Redis as a consumer and PostgreSQL as a writer.

### 4. Database (Persistent Storage)
- **Technology:** PostgreSQL (official Docker image).
- **Functionality:** Relational database storing the permanent record of all processed votes.
- **Storage:** Backed by persistent storage volume so data survives container restarts.

### 5. Result App (Frontend - Results Dashboard)
- **Technology:** Node.js (Express, Angular/Socket.io).
- **Functionality:** Queries PostgreSQL and displays real-time voting percentages and updates to users via WebSockets.

---

## Why Microservices for DevOps Demonstrations?

1. **Polyglot Stacks:** Real-world enterprise systems rarely use a single programming language. This project tests CI pipelines across Python, .NET, and Node.js.
2. **Decoupled Lifecycle:** Each service has its own `Dockerfile` and can be built, tested, and updated independently without redeploying the whole system.
3. **Selective CI Triggers:** In Azure Pipelines, path filters (`paths: include: [vote/*]`) ensure only the modified microservice is built and pushed to the container registry, saving build minutes and computational resources.
