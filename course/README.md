# AI backend engineering course

This repository combines a **Java and Spring Boot-first backend course** with a connected [AI foundations track](ai-foundations/README.md). Use the foundations track and [AI From Scratch](https://github.com/kavinash0310/ai-from-scratch) for model and AI/ML theory; use the Java/Spring route for services, data, identity, workflows, evaluation, security, deployment, and operations.

## Main learning path

Follow the numbered [Java/Spring course](java-spring/README.md). The route assumes you know basic Java and Spring Boot.

1. **AI literacy:** follow the [AI foundations course](ai-foundations/README.md) or [AI From Scratch textbook](https://github.com/kavinash0310/ai-from-scratch/blob/main/course/TEXTBOOK.md), then continue with ML and LLM introductions.
2. **Modern Java:** [patterns for model I/O](java-spring/01-modern-java-for-ai-backends.md).
3. **Spring architecture:** [service design and APIs](java-spring/02-spring-boot-service-architecture.md).
4. **Data and security:** [PostgreSQL](data/01-postgresql-and-data-modeling.md), [Spring Security and AI APIs](java-spring/03-spring-security-for-ai-apis.md).
5. **Models and retrieval:** [Spring AI integration](java-spring/04-spring-ai-model-integration.md), [RAG backends](java-spring/05-spring-ai-rag-backends.md).
6. **Durable processing:** [jobs, virtual threads, and messaging](java-spring/06-java-jobs-virtual-threads-and-messaging.md).
7. **Quality:** [testing and AI evaluation](java-spring/07-testing-spring-ai-applications.md).
8. **Production:** [observability/JVM](java-spring/08-spring-observability-and-jvm-performance.md), [cloud/Kubernetes](java-spring/09-cloud-kubernetes-spring-services.md).
9. **Senior design:** [system design and job readiness](java-spring/10-senior-system-design-and-job-readiness.md).

## Subject folders

| Folder | What it covers | Java/Spring route |
| --- | --- | --- |
| [AI foundations](ai-foundations/README.md) | From programming/math through AI, ML, NLP, LLMs, RAG, agents, evaluation, and operations | Theory companion |
| [Java + Spring](java-spring/README.md) | Java, Spring Boot/Security/AI, data, RAG, tests, operations | Primary, required |
| [Backend](backend/README.md) | Linux, HTTP, APIs, async; includes a FastAPI implementation | Concepts useful; FastAPI optional |
| [Data](data/README.md) | PostgreSQL, transactions, caching, queues, vector storage | Required concepts |
| [Security](security/README.md) | Identity, API risks, LLM threats | Required concepts, paired with Spring lessons |
| [AI systems](ai-systems/README.md) | Provider integration, RAG, tools, serving | Cross-stack concepts; use Java chapters for implementation |
| [Testing](testing/README.md) | Software, API, AI evaluation, load/failure testing | Required ideas; Java examples in Java chapters |
| [Operations](operations/README.md) | Docker, deployment, cloud, observability, reliability | Required, with Java-specific chapters |
| [Projects](../projects/README.md) | Inference API, RAG, production capstone | Implement in Java/Spring |
| [Glossary](GLOSSARY.md) | AI backend vocabulary | Use while studying |

For each chapter, explain the idea from memory, reproduce its exercise, test a failure case, and state a trade-off. A finished project needs a diagram, setup instructions, tests, AI quality evidence, security boundaries, operational measurements, and known limitations. Reading alone does not complete a stage.