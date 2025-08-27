ADR 003: Asynchronous API over Synchronous
Status: Accepted
Date: 2025-08-27

1. Context
The end-to-end process for an inference request involves API ingestion, queuing, batch formation, dispatch, and model execution. While our SLO is <250ms, edge cases or heavy load could push this duration. Crucially, AWS API Gateway has a hard integration timeout of 29 seconds. We must design an API contract that is resilient to these constraints and provides a reliable client experience.

2. Decision
We will implement an asynchronous, job-based API pattern.

The primary endpoint (POST /async-infer) will validate the request, place it in a queue, and immediately return a 202 Accepted response containing a unique jobId.

A secondary endpoint (GET /results/{jobId}) will allow the client to poll for the job's status and final result.

3. Consequences
Pros:

System Resilience and Decoupling: This pattern completely decouples the client's connection from the backend's processing lifecycle. The system is now immune to the 29-second API Gateway timeout. A job can take minutes to process if necessary, without any risk of the client-facing connection failing.

Enhanced Scalability and Throughput: The API Gateway and Go Workload Manager can accept and queue thousands of requests per second without being blocked by the actual inference speed. This allows the system to gracefully absorb massive traffic spikes, providing a stable and predictable front door.

Superior Client Experience: The client receives immediate (<50ms) acknowledgment that their request is being processed. This feels far more responsive than a long-lived connection that might hang for seconds before returning a result or, worse, a timeout error.

Foundation for Advanced Features: This pattern provides a natural foundation for future features like webhooks, where the server could actively notify a client's callback URL when a job is complete, eliminating the need for polling.

Cons:

Increased Client-Side Complexity: The client is now responsible for implementing a polling mechanism or another callback strategy to retrieve the result. This is a more complex integration than a simple, single synchronous request-response.

Rejected Alternative:

Synchronous API:

Brittle and Unreliable: This model is fundamentally fragile. Any delay in the system—a spike in queue depth, a momentary slowdown in batching, or a complex model inference—that pushes the total processing time past 29 seconds would result in a 504 Gateway Timeout error, even if the inference itself was ultimately successful.

Poor Scalability: Long-held connections consume resources on both the client and server. During a traffic spike, the Workload Manager could become saturated with waiting connections, preventing it from accepting new requests and leading to cascading failures.

Blocks Throughput: The maximum throughput of the system would be limited by the P99 latency of inference, rather than the speed at which the manager can ingest requests.