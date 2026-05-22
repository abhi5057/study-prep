# RabbitMQ Production Guide for Senior Backend Interviews

## 1) RabbitMQ Core Concepts

- Broker implementing AMQP-style messaging.
- Producer publishes to exchange, not directly to queue.
- Exchange routes to queues based on binding rules.

Exchange types:
- direct.
- topic.
- fanout.
- headers.

## 2) Routing Design

Direct exchange:
- exact routing key match.

Topic exchange:
- wildcard patterns for flexible routing.

Fanout:
- broadcast to all bound queues.

Interview emphasis:
- routing topology design determines coupling and extensibility.

## 3) Delivery Guarantees and Acks

- Manual acks for reliable processing.
- Ack after successful processing.
- Nack/reject for retry or dead-letter handling.

Durability controls:
- durable queues.
- persistent messages.
- quorum queues for stronger reliability.

## 4) Prefetch and Consumer Throughput

- prefetch limits unacked messages per consumer.
- tune prefetch to balance throughput and fairness.

Too high prefetch:
- uneven work distribution and memory pressure.

## 5) Retry and Dead-Letter Patterns

Common pattern:
- main queue -> retry queue with TTL -> dead-letter back to main.
- after max attempts, route to DLQ.

Benefits:
- controlled retries without busy-looping.

## 6) Ordering and Idempotency

- Ordering is queue-specific and affected by redelivery/retries.
- Consumer idempotency is essential under at-least-once delivery.

## 7) RabbitMQ in Go Services

Consumer loop outline:

```go
for d := range deliveries {
    if err := handle(d.Body); err != nil {
        d.Nack(false, false) // route to DLQ policy
        continue
    }
    d.Ack(false)
}
```

Hardening:
- bounded worker pools.
- context cancellation.
- graceful shutdown waits for in-flight acks.

## 8) Operational Topics

Monitor:
- queue depth and ingress/egress rates.
- unacked messages.
- consumer utilization.
- node memory/disk alarms.

Operational controls:
- quorum queue sizing.
- lazy queues for large backlog patterns.
- policy-driven DLX and TTL rules.

## 9) Common RabbitMQ Failures

- queue backlog growth due to slow consumers.
- poison message loops.
- network partitions and mirrored queue behavior surprises.
- disk watermark flow control blocking publishers.

## 10) RabbitMQ vs Kafka (Interview Comparison)

RabbitMQ strengths:
- rich routing semantics.
- request/reply and task queue workflows.
- lower latency for many queue use cases.

Kafka strengths:
- high-throughput log streaming.
- durable replay over longer retention windows.
- partitioned scalability.

## 11) Security and Multi-Tenancy

- vhosts for isolation.
- scoped users/permissions.
- TLS for transport security.

## 12) Interview Questions

1. Why use exchanges instead of direct queue publish?
- decouples producers from queue topology and enables flexible routing.

2. How do you handle poison messages?
- retry with backoff and DLQ after max attempts.

3. How do you avoid consumer overload?
- prefetch tuning, worker pool bounds, and autoscaling.

4. What is quorum queue benefit?
- stronger data safety under node failures compared to classic mirrored setups.

5. How do you ensure processing correctness?
- idempotent consumers + manual ack strategy + observability.

## 13) Story Prompts

- Migrated from direct queues to topic routing for feature expansion.
- Stabilized backlog incident via prefetch and consumer concurrency tuning.
- Implemented DLQ/retry architecture reducing production failures.
- Improved reliability by moving to quorum queues.

## 14) RabbitMQ Snippets and Interview Explanations

### 14.1 Topic Exchange and Binding (Node.js amqplib)

```js
const amqp = require("amqplib");

async function setup() {
    const conn = await amqp.connect("amqp://localhost");
    const ch = await conn.createChannel();

    await ch.assertExchange("orders.events", "topic", { durable: true });
    await ch.assertQueue("orders.created.q", { durable: true });
    await ch.bindQueue("orders.created.q", "orders.events", "order.created");
}
```

Interview explanation:
- Exchange indirection decouples producers from queue topology.

### 14.2 Manual Ack with Retry/DLQ Strategy

```js
ch.consume("orders.created.q", async msg => {
    try {
        const payload = JSON.parse(msg.content.toString());
        await processOrder(payload);
        ch.ack(msg);
    } catch (err) {
        // reject and do not requeue so DLX policy can route to DLQ
        ch.nack(msg, false, false);
    }
}, { noAck: false });
```

Interview explanation:
- Manual ack gives explicit success boundary.
- DLQ prevents poison message loops.

### 14.3 Prefetch Tuning

```js
ch.prefetch(20);
```

Interview explanation:
- Controls in-flight unacked messages per consumer.
- Too high prefetch can cause unfair distribution and memory pressure.

## 15) L4 RabbitMQ Deep Drill

### 15.1 Queue Type Choices

- Classic queues: simpler and lighter for many use cases.
- Quorum queues: stronger data safety and leader-replica model.
- Lazy queues for memory-sensitive backlogs.

### 15.2 Delivery Guarantees in Real Systems

- At-least-once is common with manual ack.
- Exactly-once is application-level via idempotency keys.
- Poison messages need DLX routing, retry budget, and alerting.

### 15.3 Publisher Confirms and Backpressure

```js
ch.publish(exchange, key, Buffer.from(payload), { persistent: true });
// use confirm channel and handle nack/timeout
```

Interview explanation:
- Mature answers include producer-side reliability, not only consumer ack logic.

### 15.4 Senior Grilling Questions

1. When do you choose RabbitMQ over Kafka for workflow orchestration?
2. How do you prevent infinite retry loops?
3. How do you bound queue growth during downstream outage?
