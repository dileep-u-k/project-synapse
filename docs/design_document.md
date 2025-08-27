# Project Synapse: Design Document & Objectives

**Version:** 1.0
**Status:** Inception
**Date:** August 27, 2025

## 1. Abstract

Project Synapse is a high-performance, cost-optimized serverless platform for LLM inference. It aims to solve the critical challenges of high latency (cold starts) and low GPU utilization inherent in naive serverless deployments. By introducing a stateful workload manager, it enables dynamic request batching and intelligent GPU worker pooling, drastically reducing costs and latency to production-grade levels.

## 2. Problem Statement

Deploying Large Language Models (LLMs) on standard serverless platforms (like AWS Lambda) is economically and technically inefficient for the following reasons:

* **Pervasive Cold Starts:** GPU-enabled containers can take 5-10 seconds to initialize, leading to unacceptable P99 latencies (estimated **~10,000ms**) for user-facing applications.
* **No Native Batching:** Each incoming request triggers a separate invocation. LLM inference is most efficient when multiple requests are batched together, but this is not possible in a standard serverless model.
* **GPU Underutilization:** A single inference request might only use the GPU for a few hundred milliseconds. For the remaining 9 seconds of a cold start, a multi-thousand-dollar GPU sits idle, leading to an estimated **<5% utilization rate** and wasted expenditure.

## 3. Proposed Solution

The architecture consists of a central **Go-based Workload Manager** acting as the "brain" and a fleet of **AWS Lambda Workers** as the "muscle."

1.  **Client Interaction:** Clients send inference requests to an asynchronous API endpoint (`/async-infer`).
2.  **Orchestration:** The Go Workload Manager, running on a highly-available service like ECS Fargate, ingests these requests.
3.  **Batching & Dispatch:** It intelligently batches requests based on size and time windows. It then dispatches these batches to available, pre-warmed Lambda workers from a managed pool.
4.  **Execution:** Each Lambda worker, equipped with an optimized NVIDIA Triton Inference Server container, executes the inference on the batch and returns the result.
5.  **Scaling:** The manager dynamically scales the pool of Lambda workers up and down—even to zero—based on real-time load.



## 4. Service Level Objectives (SLOs)

These are the non-negotiable, quantifiable goals that will define the success of this project.

* **P99 Latency:** End-to-end inference latency must be **< 250ms** under load.
* **Cost Reduction:** Achieve a **>90% reduction** in cost-per-inference compared to the baseline naive Lambda deployment.
* **GPU Utilization:** Maintain an average GPU utilization of **>70%** on active workers during sustained load.
* **Scalability:** The system must automatically scale from **0 to 1,000 concurrent requests** without manual intervention.

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
