# Project Synapse 🧠

![Build Status](https://img.shields.io/badge/build-passing-brightgreen)
![License](https://img.shields.io/badge/license-MIT-blue)
![Go Version](https://img.shields.io/badge/go-1.22-cyan.svg)

**Project Synapse is a high-performance, cost-optimized serverless platform for LLM inference on AWS.**

This project tackles the critical challenges of high latency and low GPU utilization in traditional serverless deployments. By implementing a custom Go-based workload manager, it enables intelligent request batching and worker pooling to drastically reduce costs and achieve production-grade P99 latencies.


## ✨ Key Features

* **Dynamic Request Batching:** Intelligently groups incoming requests to maximize GPU throughput.
* **Intelligent GPU Worker Pooling:** Maintains a pool of pre-warmed Lambda workers, eliminating cold starts.
* **Scale-to-Zero Cost Model:** The entire system scales down to zero resources during idle periods, eliminating all costs.
* **High-Performance Go Orchestrator:** A lightweight, concurrent manager capable of handling thousands of requests.
* **Full Observability:** Pre-configured with Prometheus and Grafana for deep insights into performance and SLOs.
* **Infrastructure as Code:** The entire stack is defined and deployed using Terraform for repeatable, automated setups.
  
## 🏛️ Core Architecture

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

🛠️ Prerequisites
Before you begin, ensure you have the following tools installed:

Go (version 1.21+)

Docker & Docker Compose

Terraform (version 1.5+)

AWS CLI (configured with appropriate credentials)

⚙️ API Usage Example
The service uses an asynchronous API. You first submit a job and then poll for the result using the returned jobId.

1. Submit an Inference Job

    # Replace YOUR_API_GATEWAY_URL with the actual deployed URL
    curl -X POST YOUR_API_GATEWAY_URL/async-infer \
    -H "Content-Type: application/json" \
    -d '{
    "model": "distilgpt2",
    "prompt": "The future of AI is"
    }'

    # You will receive an immediate response with a job ID
    # {
    #   "jobId": "a1b2c3d4-e5f6-7890-1234-567890abcdef"
    # }

2. Retrieve the Result

    # Use the jobId from the previous step
    curl YOUR_API_GATEWAY_URL/results/a1b2c3d4-e5f6-7890-1234-567890abcdef

    # Poll this endpoint until the status is "COMPLETED"
    # {
    #   "jobId": "a1b2c3d4-e5f6-7890-1234-567890abcdef",
    #   "status": "COMPLETED",
    #   "result": "The future of AI is decentralized and collaborative."
    # }

🚀 Deployment
The entire infrastructure is managed by Terraform.

1. Navigate to the infrastructure directory:

    Bash
    cd infra/

2. Initialize Terraform:

    Bash
    terraform init

3. Apply the configuration:

    Bash
    terraform apply

Tech Stack: Go | NVIDIA Triton Server | AWS Lambda | AWS ECS | Redis | Docker | Terraform | Prometheus | Grafana