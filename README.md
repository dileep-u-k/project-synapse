# Project Synapse 🧠

![Build Status](https://img.shields.io/badge/build-passing-brightgreen)
![License](https://img.shields.io/badge/license-MIT-blue)
![Go Version](https://img.shields.io/badge/go-1.22-cyan.svg)

**Project Synapse is a high-performance, cost-optimized serverless platform for LLM inference on AWS.**

This project tackles the critical challenges of high latency and low GPU utilization in traditional serverless deployments. By implementing a custom Go-based workload manager, it enables intelligent request batching and worker pooling to drastically reduce costs and achieve production-grade P99 latencies.

---
## ✨ Key Features

* **Dynamic Request Batching:** Intelligently groups incoming requests to maximize GPU throughput.
* **Intelligent GPU Worker Pooling:** Maintains a pool of pre-warmed Lambda workers, eliminating cold starts.
* **Scale-to-Zero Cost Model:** The entire system scales down to zero resources during idle periods, eliminating all costs.
* **High-Performance Go Orchestrator:** A lightweight, concurrent manager capable of handling thousands of requests.
* **Full Observability:** Pre-configured with Prometheus and Grafana for deep insights into performance and SLOs.
* **Infrastructure as Code:** The entire stack is defined and deployed using Terraform for repeatable, automated setups.

---
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