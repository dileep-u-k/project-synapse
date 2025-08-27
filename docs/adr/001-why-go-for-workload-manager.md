ADR 001: Why Go for the Workload Manager?
Status: Accepted
Date: 2025-08-27

1. Context
The Workload Manager is the stateful, operational core of Project Synapse. It is primarily an I/O-bound application responsible for handling thousands of concurrent API requests, managing a request queue, orchestrating a pool of serverless workers over the network, and maintaining the state of each worker. The choice of language for this component directly impacts the system's performance, concurrency handling, maintainability, and operational overhead.

2. Decision
We will implement the Workload Manager in Go (Golang).

3. Consequences
Pros:

First-Class Concurrency: Go's model of goroutines and channels is a perfect fit for this problem domain. It allows us to manage tens of thousands of concurrent operations (API connections, Lambda invocations, state updates) with minimal boilerplate, high performance, and a greatly reduced risk of concurrency bugs like race conditions compared to traditional thread-and-lock models.

High Performance for I/O: As a compiled language with an efficient networking stack and scheduler, Go introduces negligible overhead in the orchestration path. This ensures that the manager itself is never the bottleneck, keeping end-to-end latency low.

Operational Excellence: Go compiles to a single, statically-linked binary with no external runtime dependencies. This makes it trivial to package into a minimal scratch or alpine Docker container, resulting in smaller image sizes, faster startup times on ECS Fargate, and a reduced security surface area.

Mature Ecosystem: The ecosystem for cloud-native applications in Go is unparalleled. This includes a mature AWS SDK, robust HTTP frameworks (like Gin), and first-party client libraries for essential tools like Prometheus and Redis.

Cons:

Verbose Error Handling: Go's explicit if err != nil pattern can be more verbose than exceptions in languages like Python, though it forces developers to handle errors more deliberately.

Less Suited for Rapid Prototyping: Compared to a dynamic language like Python, Go's static typing and compilation step can slightly slow down the initial exploratory phase of development.

Rejected Alternatives:

Python (with FastAPI/Flask): While excellent for rapid development, Python's Global Interpreter Lock (GIL) prevents true CPU parallelism. Although our application is I/O-bound, Go's explicit and more efficient concurrency primitives (goroutines) are better suited for orchestrating a high volume of concurrent tasks than Python's asyncio or threading models. Furthermore, Python dependency management and packaging for production can be more complex.

Rust: Rust offers superior memory safety and potentially higher performance. However, its steep learning curve and focus on zero-cost abstractions are overkill for an I/O-bound service where the primary bottleneck will always be network latency, not CPU. The development velocity in Go is significantly higher for this specific use case.

Java (with Spring Boot): While a mature and powerful option, the JVM's resource footprint (memory and CPU usage) and longer startup times are less ideal for a lean, containerized microservice compared to a compiled Go binary.