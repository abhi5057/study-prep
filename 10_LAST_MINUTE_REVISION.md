# Last-Minute Revision Sprint (90 to 180 Minutes)

Use this right before interview day.

## 1) 90-Minute High-Impact Plan

0-15 min:
- Review Go concurrency pitfalls and context cancellation.
- Rehearse channel vs mutex, pointer vs value receiver, nil interface pitfall.

15-30 min:
- Review AWS architecture choices: ECS/EKS/Lambda, RDS/Aurora/DynamoDB, SQS/SNS/EventBridge/MSK.
- Rehearse one migration and one incident story.

30-45 min:
- Review Kubernetes probes, autoscaling, rollout safety, and common debugging flow.

45-60 min:
- Review Postgres indexes, explain analyze, transactions, locking, replication lag discussion.

60-75 min:
- Review Kafka reliability semantics, lag handling, idempotency, and schema compatibility.

75-90 min:
- Review Redis cache patterns and RabbitMQ retry/DLQ routing.
- End with 5 rapid-fire design questions.

## 2) Extended 180-Minute Plan

- Spend extra 30 min on Java backend talking points.
- Spend extra 30 min on observability and SLO-based incident response.
- Spend extra 30 min rehearsing 8 STAR stories with numbers.

## 3) 20 Must-Know Go Q and A

1. Why use context everywhere?
- cancellation, deadlines, and request-scoped propagation.

2. Channel vs mutex?
- channels for coordination/ownership transfer, mutex for shared mutable state.

3. Why can map crash in concurrent writes?
- native map is not write-safe concurrently.

4. What causes goroutine leaks?
- blocked channel ops and missing cancellation paths.

5. How to profile CPU and memory quickly?
- pprof with representative traffic.

6. Why preallocate slices?
- fewer reallocations and copy overhead.

7. How to avoid nil interface bug?
- return nil explicitly and use typed errors carefully.

8. How do you propagate timeouts to DB and HTTP?
- context.WithTimeout and context-aware client APIs.

9. How do you do safe retries?
- transient failures only + jitter + idempotency.

10. How do you do graceful shutdown?
- stop intake, drain requests, cancel workers, close resources.

11. What is outbox pattern?
- atomic DB write + deferred reliable publish.

12. How to ensure idempotency for create APIs?
- idempotency key with unique constraints.

13. Why race detector is useful?
- catches data races not obvious in tests.

14. Why can default case in select be risky?
- can cause busy loop and high CPU.

15. Why can time.After in loop be problematic?
- timer allocation/leak style overhead; prefer timer reuse.

16. How do you structure packages in backend services?
- cohesive boundaries and explicit dependencies.

17. What is a good error taxonomy?
- validation/not found/conflict/transient/internal.

18. How do you reduce lock contention?
- shorten critical section, sharding, lock-free paths where possible.

19. How do you discuss optimization in interviews?
- baseline, bottleneck evidence, change, measured result.

20. What makes a senior answer strong?
- trade-offs + operational safety + measurable impact.

## 4) 20 Must-Know AWS Q and A

1. SQS vs Kafka?
- queue simplicity vs log-streaming replay and partitioned scale.

2. RDS vs Aurora?
- compatibility/control vs cloud-optimized managed performance and failover.

3. DynamoDB design first principle?
- access patterns define keys and indexes.

4. ALB vs NLB?
- L7 HTTP routing vs L4 high-performance TCP/UDP.

5. How to design secure service access?
- IAM roles + private networking + least privilege.

6. How to reduce cloud cost quickly?
- right-size compute, storage lifecycle, reserved/savings plans.

7. How to run safe deployment?
- canary/blue-green with rollback signals.

8. How to handle multi-region DR?
- explicit RTO/RPO + tested failover runbooks.

9. CloudWatch key alerts?
- latency errors saturation and business KPI thresholds.

10. Why use DLQ?
- isolate poison messages and preserve system flow.

11. How to absorb traffic spikes?
- queue buffering and autoscaling.

12. How to avoid Lambda cold-start pain?
- provisioned concurrency and optimized init.

13. Why private subnets for app tiers?
- reduce attack surface and controlled egress.

14. How to secure secrets?
- Secrets Manager/SSM + KMS + rotation.

15. Why use RDS Proxy?
- connection management under burst traffic.

16. How to choose ECS vs EKS?
- simplicity and ops overhead vs Kubernetes flexibility/ecosystem.

17. What causes queue backlog incidents?
- producer-consumer mismatch, downstream slowness, retry storms.

18. How to reduce MTTR in AWS?
- runbooks, dashboards, tracing, clear ownership.

19. How to present migration story?
- risk plan, phased rollout, rollback, measurable outcome.

20. What do interviewers test with AWS questions?
- architecture judgment and operational maturity.

## 5) Final Interview Delivery Tips

- Speak in compact structure: problem, approach, trade-off, outcome.
- Always include one metric when describing project impact.
- If unknown, state assumptions and proceed methodically.
- Keep answers practical, not purely theoretical.

## 6) Night-Before Checklist

- Prepare 8 stories with numbers.
- Revise top 30 rapid-fire questions.
- Sleep and hydration matter more than one extra hour of random reading.

## 7) L4 Rapid-Fire Addendum (Cross-Domain)

21. Difference between idempotency and deduplication?
- Idempotency is operation safety on retries; dedup avoids repeated event processing.

22. Why can retry policies cause outages?
- Coordinated retries amplify load and create retry storms.

23. What is the first thing to show in incident update?
- Customer impact, scope, mitigation ETA, and owner.

24. When to choose eventual consistency?
- User experience tolerant of short staleness and high write scale is needed.

25. How to discuss database migration risk?
- Backfill strategy, dual-write/dual-read policy, cutover guardrails, rollback plan.

26. What is a mature timeout strategy?
- Per-hop budget with total request deadline and cancellation propagation.

27. Why can autoscaling fail during spikes?
- Bad metrics, cold starts, provisioning delay, downstream bottlenecks.

28. What is a strong architecture answer format?
- Requirements -> constraints -> design -> trade-offs -> failure handling -> metrics.
