ADR 002: Custom Worker Pooling vs. Provisioned Concurrency
Status: Accepted
Date: 2025-08-27

1. Context
A primary objective of Project Synapse is to eliminate the P99 latency impact of AWS Lambda cold starts for GPU-enabled containers. A naive on-demand invocation model is unacceptable. Two potential solutions exist: using AWS's native Provisioned Concurrency feature, or implementing a custom worker pooling and pre-warming strategy within our Go Workload Manager.

2. Decision
We will implement a custom worker pooling and pre-warming strategy. The Go Workload Manager will be solely responsible for maintaining a pool of "hot" Lambda workers, managing their lifecycle, and dispatching work to them.

3. Consequences
Pros:

Enables Dynamic Batching: This is the most critical advantage. By managing the queue, the Workload Manager can hold incoming requests and intelligently group them into optimal batches before dispatching them. This is the primary driver of GPU utilization and cost-efficiency, a capability that Provisioned Concurrency does not provide in any way.

Total Control and Intelligent Scaling: A custom solution allows for sophisticated, application-aware scaling logic. We can scale the pool based on queue depth, predicted traffic patterns (e.g., time of day), or average batch size, not just the raw invocation count. This also allows for a true scale-to-zero during idle periods, eliminating all costs.

Superior Cost-Efficiency for Variable Loads: Provisioned Concurrency incurs a fixed, high cost for every hour it is enabled, regardless of utilization. Our custom strategy only invokes "warming" functions as needed, providing a more granular, pay-for-what-you-use model that is far cheaper for sporadic or unpredictable traffic patterns.

Explicit State Management: The manager maintains a precise state for each worker (WARMING, IDLE, BUSY, ERROR). This enables smarter dispatching (e.g., avoiding workers that recently errored) and adds a layer of resilience not possible with the native solution.

Cons:

Increased System Complexity: We are now responsible for building and maintaining the entire worker lifecycle and scaling logic. This is a non-trivial engineering task that requires careful implementation to avoid issues like race conditions or inefficient scaling.

Rejected Alternative:

AWS Lambda Provisioned Concurrency:

Fundamentally Incompatible with Batching: This is the deal-breaker. Provisioned Concurrency prepares a fixed number of execution environments to receive one invocation each. It does nothing to help group multiple separate API requests into a single Lambda invocation. It solves the cold start problem but completely fails to address the far more important GPU underutilization problem.

Prohibitively Expensive: You pay a significant premium for every configured instance, for every hour, whether it receives an invocation or not. For a system designed to scale to zero, this is economically unviable.

Inflexible "Black Box": It is a blunt instrument. You set a target number of concurrent executions, and AWS manages the rest. It offers no control over which specific instance receives a request and no visibility into the state of the pool.