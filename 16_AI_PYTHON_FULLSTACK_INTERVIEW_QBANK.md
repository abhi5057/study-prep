# Python + AI Full-Stack Interview Question Bank (with Model Answer Direction)

Use this as mock interview drill material.

## 1) Python Backend Questions

1. Explain GIL and its impact on web services.
- Direction: GIL limits CPU-bound thread parallelism in CPython, but I/O concurrency is still effective via async/threads.

2. When would you choose asyncio over threads?
- Direction: high-concurrency I/O, many network waits, lower thread overhead.

3. How do you prevent memory blowups while serving large files?
- Direction: stream responses, chunk processing, avoid full materialization.

4. What is mutable default argument bug?
- Direction: default evaluated once; use `None` sentinel pattern.

5. How do you design error handling in FastAPI?
- Direction: domain errors -> HTTP mapping, consistent error schema, tracing IDs.

6. How do you validate payloads safely?
- Direction: strict schemas, field constraints, explicit coercion policy.

7. How do you design idempotent create endpoints?
- Direction: idempotency key persistence + deterministic response replay.

8. How do you test async endpoints?
- Direction: async test clients, dependency overrides, isolated resources.

9. How do you profile Python API latency?
- Direction: tracing + app metrics + sampling profiler.

10. How do you scale Python services?
- Direction: process workers, async I/O, caching, queue offloading.

## 2) LLM Integration Questions

11. How do you choose an LLM for a product feature?
- Direction: quality, latency, cost, safety, context size, tool support.

12. Why use structured outputs?
- Direction: deterministic downstream integration and schema validation.

13. How do you handle model timeouts and retries?
- Direction: deadline budget, retry transient errors, fallback model path.

14. What are top prompt design principles?
- Direction: clear role, constraints, output format, grounded context.

15. How do you reduce hallucination risk?
- Direction: retrieval grounding, source citation, constrained output, eval loop.

16. How do you implement provider abstraction?
- Direction: adapter interface for multi-provider portability and routing.

17. What telemetry do you collect per LLM request?
- Direction: latency, tokens, cost, failure class, safety flags.

18. How do you protect against prompt injection?
- Direction: tool allowlists, untrusted-context handling, policy checks.

19. How do you safely call tools from model outputs?
- Direction: strict function schema + validation + approval for risky actions.

20. When do you use small vs large model tiers?
- Direction: route simple tasks to cheap models, reserve premium models for complex reasoning.

## 3) RAG Questions

21. Walk through full RAG architecture.
- Direction: ingest -> chunk -> embed -> index -> retrieve -> rerank -> augment -> generate.

22. How do you choose chunk size and overlap?
- Direction: tune by doc structure, query style, context window, eval metrics.

23. Dense vs sparse retrieval?
- Direction: semantic similarity vs lexical precision; hybrid often best in enterprise data.

24. Why reranking?
- Direction: improves top-k relevance before generation.

25. How do you measure RAG quality?
- Direction: recall@k, citation correctness, factuality score, task success.

26. How do you handle document freshness?
- Direction: incremental sync pipeline, metadata versioning, cache invalidation.

27. What causes poor retrieval?
- Direction: bad chunking, wrong embeddings, noisy corpus, missing filters.

28. How do you enforce tenant isolation in retrieval?
- Direction: metadata filters and authorization at query layer.

29. When is fine-tuning better than RAG?
- Direction: stable task behavior adaptation, not frequently changing knowledge.

30. How do you design citation UX?
- Direction: source IDs, snippets, timestamps, clickable references.

## 4) Agent Questions

31. What is an AI agent in production terms?
- Direction: LLM + tools + state + policy + execution loop.

32. Single-agent vs multi-agent trade-offs?
- Direction: simplicity vs specialization and complexity overhead.

33. How do you prevent runaway agent loops?
- Direction: max steps, budget limits, termination conditions.

34. How do you design approval workflows?
- Direction: human-in-the-loop for high-risk actions.

35. How do you implement memory for agents?
- Direction: short-term session state + curated long-term memory.

36. How do you evaluate agents?
- Direction: workflow success, tool correctness, latency/cost, safety incidents.

37. How do you sandbox tools?
- Direction: execution boundaries, least privilege, resource quotas.

38. How do you debug agent failures?
- Direction: step traces, tool logs, state snapshots, replay.

39. How do you version prompts and policies?
- Direction: source control + experiment tracking + rollout labels.

40. How do you enforce compliance constraints?
- Direction: policy engine, audit logs, redaction, region-aware controls.

## 5) AI Full-Stack Architecture Questions

41. Design enterprise Q&A assistant over private docs.
- Direction: RAG with auth-aware retrieval, citations, eval pipeline, feedback loop.

42. Design customer support copilot with action tools.
- Direction: retrieval + CRM/ticket tools + approval gates + audit trail.

43. Design real-time voice agent.
- Direction: streaming ASR/TTS, interruption handling, state sync, fallback prompts.

44. Design multi-tenant AI SaaS backend.
- Direction: tenant isolation, per-tenant indexes, quota and billing controls.

45. How do you combine frontend and backend for conversational apps?
- Direction: streaming responses, optimistic UI, robust retry, conversation state API.

46. What does an AI-ready CI/CD pipeline include?
- Direction: code tests + eval tests + canary + quality regression gate.

47. How do you set SLOs for AI services?
- Direction: latency, availability, answer quality thresholds, safety compliance rates.

48. How do you roll back AI regressions quickly?
- Direction: model/prompt version pinning and traffic shift rollback.

49. How do you handle user feedback loops?
- Direction: capture rating + correction + offline eval update.

50. How do you estimate cost per conversation?
- Direction: token accounting + retrieval infra + tool invocation + overhead.

## 6) Practical Coding Tasks Often Asked

51. Implement chunking with overlap.
52. Build top-k retrieval API endpoint.
53. Validate LLM JSON output with Pydantic.
54. Add retry with backoff around model calls.
55. Implement stream response endpoint in FastAPI.
56. Build prompt template with source citations.
57. Add tenant filter to vector search query.
58. Add tool-call guard for dangerous commands.
59. Create evaluation harness for QA pairs.
60. Build fallback path when provider is degraded.

## 7) Rapid Mock Round (10-Minute Drill)

- Explain GIL in 45 seconds.
- Explain hybrid retrieval in 45 seconds.
- Explain agent safety guardrails in 45 seconds.
- Explain idempotent tool execution in 45 seconds.
- Explain RAG evaluation metrics in 45 seconds.
- Explain AI production rollback plan in 45 seconds.

## 8) Interview-Ready Closing Statement

"I build AI features as production systems, not only prompt demos: schema-validated outputs, retrieval grounded on private data, agent safety controls, measurable evals, and deployment guardrails with rollback."

## 9) Additional L4/L5 Grilling Questions (61-100)

61. Explain GIL impact on API throughput.
62. Asyncio vs multiprocessing: selection criteria?
63. Why does p99 grow before CPU saturation?
64. Timeout budget split across service hops?
65. Idempotency key design for payment-like workflows?
66. Outbox pattern and why dual-write fails?
67. Kafka ordering guarantees and limitations?
68. Exactly-once myths in distributed systems?
69. How do you handle schema evolution safely?
70. Postgres MVCC and autovacuum impact?
71. Why can too many indexes hurt?
72. Redis cache stampede mitigation methods?
73. RabbitMQ DLQ and retry backoff topology?
74. CAP vs PACELC in one practical scenario.
75. Strong vs eventual consistency for user profile.
76. Read-your-writes implementation options?
77. What causes retry storms?
78. Circuit breaker states and tuning?
79. How to define SLO for AI assistant?
80. Error budget policy and burn-rate alerting?
81. How do you evaluate RAG faithfulness?
82. Hybrid retrieval vs dense-only trade-off?
83. Prompt injection defense strategy?
84. Tool-call safety checks for agents?
85. Tenant isolation in vector DB?
86. Cost per successful answer: how measured?
87. Canary for model/prompt rollout?
88. Rollback playbook for AI regression?
89. Hotstar-like streaming key bottlenecks?
90. CDN hit-ratio drop: immediate actions?
91. Kubernetes Pending pod triage steps?
92. Why HPA can fail during sudden spike?
93. IAM least-privilege pitfalls in fast-moving teams?
94. Multi-region DR: active-passive vs active-active?
95. Why does GC tuning affect tail latency?
96. How to prevent N+1 query regressions?
97. Design an audit trail for regulated workflows.
98. How to run effective game-day drills?
99. What metrics prove architecture success?
100. What trade-off did you intentionally choose and why?
