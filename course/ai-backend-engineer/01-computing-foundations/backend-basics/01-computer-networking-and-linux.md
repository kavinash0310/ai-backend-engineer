# Computer, process, Linux, and network foundations

## Why a backend engineer needs this

A backend is a program that waits for requests, performs work, and sends responses. It runs as a process on an operating system and communicates over networks. When an endpoint fails, knowing where the process, port, DNS lookup, connection, and application fit makes debugging much easier.

## Program, process, port

A program is a file containing instructions. A process is a running instance with memory, open files, environment variables, and a process identifier. A server process binds to an IP address and port, then waits for incoming connections. Two processes cannot normally bind the same address and port at once.

Localhost, often written 127.0.0.1, means this machine. A port is a numbered endpoint on a machine; it is not the same as an application or domain name. If the server listens only on localhost inside a container, an external client may not reach it even when the container port is published.

Environment variables configure a process at launch. They can hold non-secret configuration such as environment name or port. Secrets require a controlled secret manager or protected runtime injection; committing a .env file with real credentials to a public repository is unsafe.

## A request crosses multiple layers

A browser or client resolves a domain name using DNS to find an address. It establishes a transport connection, commonly TCP, and for HTTPS negotiates TLS encryption and server identity. It sends an HTTP request containing a method, path, headers, and sometimes a body. A proxy or load balancer may route it to an application process. The application parses it, performs work, and returns an HTTP response.

~~~text
client -> DNS -> TCP/TLS connection -> proxy/load balancer -> app process
       -> database or model provider -> app process -> HTTP response -> client
~~~

Each arrow may fail differently: name lookup, connection refusal, TLS configuration, timeout, bad input, database error, provider rate limit, or application bug. Good logs and request IDs help identify the layer.

## Synchronous and asynchronous waiting

A synchronous function waits for a result before doing other work in that execution path. An asynchronous function can yield control while waiting for network I/O, allowing one process to serve other requests. Async is useful when work mostly waits on sockets or databases. It does not automatically make CPU-heavy Python code faster; image processing or local model inference may need worker processes, native libraries, or specialized compute.

Do not call blocking operations inside an async handler without understanding the effect. A slow blocking function can stall the event loop and delay unrelated users. Use an asynchronous client for async work or move long/CPU-heavy jobs to a worker.

## Basic command-line debugging

Learn to inspect the current directory, list files, read logs, check environment configuration, and see which process listens on a port. Use Git to save checkpoints. A virtual environment isolates Python packages for a project. A dependency lock file helps reproduce package versions.

When a local endpoint does not answer, check in this order: is the process running; which host and port is it bound to; can the same machine reach it; is a container or firewall blocking access; does the route exist; what do the logs say?

## Questions and practice

**Is a port a server?** No. A port is a numbered communication endpoint used by a process.

**Does async speed up all code?** No. It helps overlap waiting on I/O; CPU-bound work needs a different strategy.

**Why use HTTPS?** TLS encrypts data in transit and helps authenticate the server. It does not prove the application itself is secure.

Practice: run a local web service, identify its process and port, send a request from a command line, stop the process, and explain how the error changes. Draw the request path from client to database and back.

Further reading: [Python asyncio overview](https://docs.python.org/3/howto/a-conceptual-overview-of-asyncio.html) and [HTTP Semantics, RFC 9110](https://www.rfc-editor.org/rfc/rfc9110).

## Trace a failed request layer by layer

A client resolves api.example.com through DNS, opens a TCP connection to port 443, negotiates TLS, then sends HTTP. A proxy may route the request to an application container, which queries a database and a model provider before responding. Failures at these layers look different: DNS failure means the name did not resolve; connection refusal means no process accepted the connection; timeout means no reply arrived before the deadline, even though a remote operation may have completed; a 401 or 403 is an application identity/permission result.

Use a request ID to connect client, proxy, API, worker, and provider records. Do not use raw prompts or access tokens as correlation identifiers. When debugging, check process health, bind address, port mapping, network route, application logs, and downstream dependencies in that order.

## Process lifecycle

A process can own open database connections and in-flight jobs. Graceful shutdown stops new work, completes or cancels active requests within a bounded time, closes pools, then exits. Abrupt termination can leave a job in an uncertain state. API and worker processes have separate capacity: adding API replicas does not automatically add job-processing throughput.

**Practice answer:** If a service works on the host but not from another container, inspect the listening address, published port, and container DNS/network. If a provider call times out, record that its outcome may be unknown and use an idempotency/status mechanism before repeating a side effect.
