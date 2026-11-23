# Master Roadmap — Backend Engineering Mastery

> **Authority Notice:** This document is owned by **ChatGPT (Curriculum Owner & Instructor)**.
> The approved 29-phase roadmap, phase order, scope, schedule, allocated hours, and progression criteria are authoritative.
> Antigravity must not independently invent, redesign, optimize, rename, reorder, merge, split, expand, narrow, or revise curriculum content.

---

## Approved 29-Phase Roadmap

| Phase | Title | Hours |
|---|---|---:|
| 1 | Engineering Mindset & AI-Augmented Engineering | 26 |
| 2 | Complexity and Performance Reasoning | 22 |
| 3 | Operating Systems | 36 |
| 4 | Computer Networking | 32 |
| 5 | JavaScript Language Internals | 32 |
| 6 | Concurrency and Asynchronous Systems | 34 |
| 7 | Requirements Engineering | 22 |
| 8 | Domain Modeling | 26 |
| 9 | HTTP and API Engineering | 26 |
| 10 | Database Engineering | 48 |
| 11 | Node.js Runtime Engineering | 30 |
| 12 | OOP, SOLID and Design Patterns | 26 |
| 13 | Backend Application Engineering | 40 |
| 14 | Database Integration and Data Access | 30 |
| 15 | Software Testing Engineering | 30 |
| 16 | Caching Engineering | 18 |
| 17 | Queues and Asynchronous Processing | 22 |
| 18 | Authentication and Authorization | 22 |
| 19 | Backend Security | 28 |
| 20 | Observability Engineering | 18 |
| 21 | Reliability Engineering | 22 |
| 22 | Software Architecture | 32 |
| 23 | Linux and Backend Infrastructure | 22 |
| 24 | Docker, Cloud, CI/CD and Deployment | 32 |
| 25 | Performance Engineering | 28 |
| 26 | Distributed Systems Fundamentals | 28 |
| 27 | Advanced Backend Architecture and System Design | 32 |
| 28 | Production-Grade Backend Projects | 104 |
| 29 | Engineering Consolidation and Career Preparation | 26 |

**Total planned learning effort:** **894 hours**.

---

## Study Schedule and Timeline

- **Sunday–Thursday:** 2 focused hours per day.
- **Friday and Saturday:** 4 focused hours per day.
- **Weekly total:** 18 focused hours.
- **Nominal annual capacity:** 936 hours.
- **Planned curriculum:** 894 hours.
- **Contingency reserve:** 42 hours.
- **Approximate target:** 12 months, subject to actual mastery and progress.

### Study-Hour Accounting Rule

- AI-generated material, research performed by agents, and time spent generating artifacts **do not** count as the learner's study hours.
- Only the learner's actual time studying, reasoning, experimenting, implementing, revising, and demonstrating understanding counts.

### Weekly Allocation Guideline (Flexible)

These are guidelines, not rigid requirements for every week:

- Approximately **6 hours:** reading and structured learning.
- Approximately **6 hours:** practice and experiments.
- Approximately **3 hours:** discussion and problem-solving.
- Approximately **2 hours:** revision and assessment.
- Approximately **1 hour:** progress records and planning.

---

## Knowledge-Depth Framework

The curriculum uses **exactly three official knowledge levels**. Use these three levels to determine the required depth for each concept; do not force every technology to Practical Competence.

### Awareness
Know what a concept or technology is, what problem it solves, its role in the ecosystem, and when it may or may not be appropriate.

### Working Understanding
Understand its key mechanisms, invariants, alternatives, trade-offs, and failure modes well enough to reason about designs, debug problems, and make informed engineering decisions.

### Practical Competence
Be able to design, implement, verify, and operate it effectively in practice, especially within our primary Node.js and PostgreSQL specialization.

### Important Clarification
- Mechanism-level depth is a dimension of learning.
- Production-level understanding is a dimension of learning.
- Neither is an additional official knowledge level.

---

## Breadth of Backend Knowledge

The curriculum develops broad ecosystem awareness while maintaining practical specialization in **Node.js** and **PostgreSQL**. The following areas must be represented across the roadmap and `BACKEND_KNOWLEDGE_INDEX.md` with appropriate depth:

- **Architecture:** Monolith, modular monolith, microservices, SOA, event-driven architecture, CQRS, event sourcing.
- **Databases:** PostgreSQL, MySQL, MongoDB, DynamoDB, ACID, MVCC, indexing, replication, sharding.
- **Networking:** TCP, UDP, DNS, TLS, HTTP/2, HTTP/3, WebSocket, gRPC, reverse proxies.
- **Concurrency:** Threads, processes, event loop, mutexes, semaphores, race conditions, deadlocks.
- **Messaging:** Kafka, RabbitMQ, NATS, SQS, Pub/Sub, delivery guarantees, idempotency.
- **Caching:** Redis, CDN, cache-aside, write-through, invalidation, eviction.
- **Security:** OAuth 2.0, OIDC, JWT, RBAC, ABAC, CSRF, XSS, SSRF, SQL injection.
- **Infrastructure:** Linux, Docker, Kubernetes, Terraform, CI/CD, load balancers.
- **Reliability:** SLI, SLO, SLA, circuit breakers, retries, backpressure, graceful degradation.
- **Observability:** Logs, metrics, traces, OpenTelemetry, Prometheus, Grafana.
- **Performance:** Profiling, benchmarking, latency, throughput, connection pooling, query optimization.
- **Cloud:** AWS, GCP, Azure, serverless, containers, object storage, managed databases.

> **Guideline:** Do not turn this into a disconnected glossary. Learning materials must explain relationships, distinctions, mechanisms, trade-offs, and failure modes. Track concepts in `BACKEND_KNOWLEDGE_INDEX.md` as they are taught in each phase; never falsely mark concepts as mastered merely because they appear in the roadmap.

---

## AI / LLM Topic Mapping Across Phases

The following 16 phase-specific AI/LLM topics are integrated into the approved backend roadmap:

- **Phase 1:** AI-augmented engineering, delegation, verification, and engineering responsibility.
- **Phase 5:** Streaming and asynchronous interactions with model APIs.
- **Phase 7:** Requirements engineering for AI-powered features.
- **Phase 9:** LLM API integration, streaming, and structured outputs.
- **Phase 10:** Embeddings, pgvector, and vector indexing.
- **Phase 13:** Building AI-powered backend features.
- **Phase 15:** Testing and evaluating LLM-powered features.
- **Phase 16:** Semantic caching and model-response caching.
- **Phase 17:** Asynchronous AI jobs.
- **Phase 19:** Prompt injection and AI-specific security.
- **Phase 20:** LLM tracing, token usage, latency, and cost observability.
- **Phase 22:** RAG architectures and agentic workflows.
- **Phase 24:** Model API deployment and awareness of self-hosted inference.
- **Phase 25:** Batching, streaming, token latency, and inference optimization.
- **Phase 27:** Agent orchestration and AI architecture.
- **Phase 28:** A production-grade AI-powered backend feature.

### AI Curriculum Scope Boundary

- The current backend program covers the AI knowledge a modern backend engineer needs. It is **not** a complete AI Engineering curriculum.
- A separate, deeper **AI Engineering Mastery** program will follow Phase 29. Advanced model architecture, inference internals, advanced RAG, fine-tuning, model serving, multimodal systems, and advanced agentic systems belong to that future program, not a parallel track.
