# Computer Science Foundations

**Purpose:** refresh only what your diagnostic shows is weak. Your Spring experience means you can focus on system internals and AI workload consequences rather than repeat introductory web development.

**Prerequisites:** practical programming experience.  
**Priority:** working knowledge, with concurrency, networking, complexity and data structures especially important.

## Study sequence

1. Complexity and data structures: arrays/lists, maps/sets, trees, heaps, graphs, sorting, searching, amortized analysis.
2. Operating systems and processes: address spaces, virtual memory, file descriptors, scheduling, I/O.
3. Networks: DNS, TCP, TLS, HTTP, sockets, connection pools, timeouts and retries.
4. Concurrency: visibility, race conditions, locks, executors, queues, parallelism, backpressure.
5. Memory and Linux shell: allocation, cache locality, process signals, logs, pipes and permissions.
6. Git/GitHub workflow: branches, commits, review, merge/rebase, resolving conflicts and safe secrets.

**AI backend connection:** vector retrieval is often approximate nearest-neighbor search; asynchronous inference relies on concurrency and backpressure; token streams cross HTTP and proxy boundaries; model workloads can saturate CPU, memory, sockets, database pools or provider quotas.

Use [CS Fundamentals](https://github.com/kavinash0310/cs-fundamentals) for its OS, network, database, algorithm and discrete-math notes. Use [Java Backend Interview Mastery](https://github.com/kavinash0310/java-backend-interview-mastery) for concurrency/JVM interview refreshers. Do not mark a topic complete until you can apply it to a traced service request or capacity problem.


## AI foundations chapter

- [Classical AI: search, logic, planning and decision-making](./01-classical-ai-search-and-reasoning.md) — state spaces, BFS/UCS/A*, constraints, logic, decision theory and RL preview.
