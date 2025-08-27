# Project Synapse 🧠

**Project Synapse is a high-performance, cost-optimized serverless platform for LLM inference on AWS.**

This project tackles the critical challenges of high latency and low GPU utilization in traditional serverless deployments. By implementing a custom Go-based workload manager, it enables intelligent request batching and worker pooling to drastically reduce costs and achieve production-grade P99 latencies.

## Core Architecture

The system uses a decoupled, asynchronous architecture to ensure scalability and resilience. For a detailed view, see the full diagram and design documents linked below.

```mermaid
graph TD
    subgraph "AWS VPC (Private Network)"
        A[Client] -- HTTPS --> B(API Gateway);
        B --> C{Go Workload Manager};
        C -- Writes Job/Reads Batch --> D[(Redis)];
        C -- Invokes --> E((Lambda Workers));

        subgraph Monitoring
            C -- /metrics --> P[Prometheus];
            F[Redis Exporter] -- Scrapes --> P;
            G[Prometheus Pushgateway] -- Scrapes --> P;
            H[Grafana] -- Queries --> P;
        end

        E -- Pushes Metrics --> G;
        E -- Executes Inference --> I[Triton Inference Engine];
        E -- Writes Result --> D;
    end

📜 Design & Architecture Documents
This project's design is formally documented to ensure clarity and capture key decisions.

📄 Main Design Document: The complete overview, problem statement, proposed solution, and Service Level Objectives (SLOs).

Architecture Decision Records (ADRs)

ADR-001: Why Go was chosen for the core Workload Manager.

ADR-002: Analysis of our custom worker pooling strategy versus AWS Provisioned Concurrency.

ADR-003: The decision to use an asynchronous API for system resilience and scalability.

🚀 Getting Started
(This section will be filled in during later project phases).

Tech Stack: Go | NVIDIA Triton Server | AWS Lambda | AWS ECS | Redis | Docker | Terraform | Prometheus | Grafana

---
## ## Step 2: Commit and Push the Changes

Now, save the updated file to your Git history.

1.  In your VS Code terminal, add the changes:
    ```bash
    git add README.md
    ```
2.  Commit the file with a clear message:
    ```bash
    git commit -m "docs: update README with links to design documents"
    ```
3.  Push the changes to GitHub:
    ```bash
    git push origin main
    ```