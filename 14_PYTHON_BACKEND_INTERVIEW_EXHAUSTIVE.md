# Python Backend Interview Guide (Senior Full-Stack AI Focus)

This guide is for a JS/full-stack engineer preparing Python-heavy interviews.
It focuses on practical backend depth, tricky behavior, and production examples.

## 1) Python Runtime and Execution Model

### 1.1 Interpreter, Bytecode, and Runtime

- Python source is compiled to bytecode.
- Bytecode runs on Python VM.
- Dynamic typing + late binding increase flexibility and runtime checks.

Interview explanation:
- Python optimizes for developer productivity and readability.
- Performance tuning often means algorithm/data-structure improvements first.

### 1.2 GIL (Global Interpreter Lock)

- In CPython, only one thread executes Python bytecode at a time.
- Threads are still useful for I/O-bound tasks.
- CPU-bound parallelism usually needs multiprocessing, native extensions, or external services.

### 1.3 Tricky Interview Example: Mutable Default Argument

```python
def append_item(x, arr=[]):
    arr.append(x)
    return arr

print(append_item(1))  # [1]
print(append_item(2))  # [1, 2] (surprising for many)
```

Fix:

```python
def append_item(x, arr=None):
    if arr is None:
        arr = []
    arr.append(x)
    return arr
```

Interview explanation:
- Default argument is evaluated once at function definition time.

## 2) Async, Concurrency, and Parallelism

### 2.1 async/await and Event Loop

- `async` functions return coroutine objects.
- `await` yields control to loop while waiting.
- Great for high-concurrency I/O workloads.

### 2.2 asyncio.gather vs sequential awaits

```python
import asyncio

async def fetch_user():
    await asyncio.sleep(0.1)
    return {"id": 1}

async def fetch_orders():
    await asyncio.sleep(0.1)
    return ["o1", "o2"]

async def main():
    user, orders = await asyncio.gather(fetch_user(), fetch_orders())
    return user, orders
```

Interview explanation:
- `gather` runs awaitables concurrently, reducing total wait time.

### 2.3 Threading vs Multiprocessing (Quick Rule)

- I/O-bound: `threading` or `asyncio`.
- CPU-bound: `multiprocessing` or offload to compiled paths.

## 3) Python Data Modeling and Validation (Pydantic)

### 3.1 Pydantic Model Example

```python
from datetime import datetime
from pydantic import BaseModel, Field

class OrderCreate(BaseModel):
    user_id: str
    amount_cents: int = Field(gt=0)
    created_at: datetime
```

Interview explanation:
- Type-driven validation simplifies API reliability.
- Works especially well with FastAPI request/response schemas.

### 3.2 Strict vs Coercion Modes

- Lax/coercive mode can auto-convert many values.
- Strict mode is safer for boundary validation in critical systems.

## 4) FastAPI Production Patterns

### 4.1 Minimal API with Validation

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    price: float

@app.post("/items")
def create_item(item: Item):
    if item.price <= 0:
        raise HTTPException(status_code=400, detail="invalid price")
    return {"ok": True, "item": item.model_dump()}
```

### 4.2 Dependency Injection Pattern

```python
from fastapi import Depends

class Service:
    def ping(self):
        return "pong"

def get_service() -> Service:
    return Service()

@app.get("/health")
def health(svc: Service = Depends(get_service)):
    return {"status": svc.ping()}
```

Interview explanation:
- Dependency injection improves testability and boundary control.

### 4.3 Streaming Response for Large Output

```python
from fastapi.responses import StreamingResponse

def iter_lines():
    for i in range(1_000_000):
        yield f"line {i}\n"

@app.get("/dump")
def dump():
    return StreamingResponse(iter_lines(), media_type="text/plain")
```

Interview explanation:
- Streams avoid loading full payload in memory.

## 5) Python Backend Architecture and Reliability

### 5.1 Layered Design

- Router/controller layer.
- Service/business logic layer.
- Repository/data access layer.

### 5.2 Error Taxonomy

- Validation (4xx).
- Not found (404).
- Conflict (409).
- Transient upstream (503).
- Internal unknown (500).

### 5.3 Retry with Backoff and Jitter

```python
import random
import time

def retry(fn, attempts=3, base=0.1):
    for i in range(attempts):
        try:
            return fn()
        except TemporaryError:
            if i == attempts - 1:
                raise
            sleep_for = base * (2 ** i) + random.uniform(0, 0.05)
            time.sleep(sleep_for)
```

Interview explanation:
- Jitter reduces thundering-herd retries.

## 6) Database Patterns in Python Services

### 6.1 SQLAlchemy Session Scope Discipline

- Keep transaction scope small.
- No external network calls inside DB transaction.

### 6.2 Idempotency Key Pattern

```sql
CREATE TABLE idempotency_keys (
  key text PRIMARY KEY,
  response_json jsonb NOT NULL,
  created_at timestamptz NOT NULL DEFAULT now()
);
```

Interview explanation:
- Essential for safe retries in payment/order style APIs.

## 7) Testing Strategy (Senior Expectation)

### 7.1 Unit + Integration + Contract

- Unit: fast logic checks.
- Integration: DB/queue/cache behavior.
- Contract: API schema and compatibility.

### 7.2 pytest Example

```python
def clamp(x, lo, hi):
    return max(lo, min(x, hi))


def test_clamp():
    assert clamp(-1, 0, 10) == 0
    assert clamp(5, 0, 10) == 5
    assert clamp(50, 0, 10) == 10
```

## 8) Performance and Profiling

### 8.1 Common Tools

- `cProfile` for call-level profiling.
- `py-spy` for low-overhead sampling.
- `pytest-benchmark` for microbenchmarks.

### 8.2 Common Wins

- Use streaming for large payloads.
- Avoid repeated heavy object creation in hot path.
- Batch I/O operations.
- Use async for high fan-out network calls.

## 9) Tricky Python Interview Questions (Output Style)

### Q1

```python
x = [1, 2, 3]
y = x
z = x[:]
y.append(4)
print(x, z)
```

Answer:
- `[1, 2, 3, 4] [1, 2, 3]`

### Q2

```python
a = "hello"
print(a[0], a[-1])
```

Answer:
- `h o`

### Q3

```python
def f(v, acc=[]):
    acc.append(v)
    return acc

print(f(1), f(2))
```

Answer:
- `[1] [1, 2]`

### Q4

```python
print({i: i*i for i in range(3)})
```

Answer:
- `{0: 0, 1: 1, 2: 4}`

### Q5

```python
import asyncio

async def work(x):
    await asyncio.sleep(0)
    return x * 2

async def main():
    res = await asyncio.gather(work(1), work(2))
    print(res)

asyncio.run(main())
```

Answer:
- `[2, 4]`

## 10) Senior Interview Story Prompts (Python)

- Migrated sync bottleneck service to async FastAPI and improved p95.
- Reduced incident rate by introducing idempotency + retries + timeouts.
- Stabilized memory usage with streaming and chunked processing.
- Improved API contract safety with strict Pydantic schemas and tests.

## 11) 60-Minute Revision Path for Python

1. Review sections 1, 2, 4, 5 first.
2. Rehearse 5 output questions and explain why.
3. Rehearse one async scaling story and one reliability story.
4. Revisit DB idempotency and transaction boundary patterns.

## 12) L4 Python Backend Deep Drill

### 12.1 Runtime and Performance Reality

- Understand GIL implications: CPU-bound threads do not scale linearly.
- Use multiprocessing or native extensions for CPU-heavy workloads.
- Use asyncio for high-concurrency IO paths.

### 12.2 Async Failure Modes

- Missing timeout/cancellation propagation causes hung requests.
- Unbounded task spawning causes memory pressure.
- Blocking code inside async handlers destroys event-loop latency.

Pattern:

```python
async with asyncio.timeout(0.25):
    result = await downstream_call()
```

### 12.3 Type Safety and Contract Discipline

- Enforce strict schemas with Pydantic/dataclasses + mypy/pyright.
- Validate external payloads at boundaries.
- Keep domain types explicit to avoid silent contract drift.

### 12.4 Senior Grilling Questions

1. When to choose FastAPI async vs sync endpoints?
2. How do you prevent duplicate side effects under retries?
3. How do you tune Python service memory under burst load?
4. How do you structure observability for async call chains?

---

## 13) L5 Python Platform Engineering

### 13.1 Runtime Strategy by Workload

- CPU-bound: multiprocessing or native extension offload.
- IO-bound: asyncio with strict timeout/cancellation discipline.
- Hybrid: isolate slow/CPU paths behind worker queues.

### 13.2 Contract and Type Governance

- Strict request/response contracts at boundaries.
- Runtime validation for external inputs.
- Type checking in CI to reduce production ambiguity.

### 13.3 Performance Guardrails

- Bound coroutine fan-out.
- Avoid hidden synchronous calls in async paths.
- Control payload sizes and serialization costs.

## 14) Python Failure Mode Drills

### 14.1 Event Loop Starvation

Triage:
- blocking calls in async path,
- large synchronous CPU in handlers,
- missing executor offload.

### 14.2 Memory Bloat in Long-Lived Workers

Triage:
- unbounded caches,
- object retention/reference cycles,
- large response buffering.

### 14.3 Duplicate Side Effects Under Retries

Triage:
- missing idempotency keys,
- non-atomic state transitions,
- retry policy not aligned with operation safety.

## 15) Python Coding and Design Mock Rounds

1. Build cancellation-safe async worker pool with bounded queue.
2. Implement idempotent API endpoint with dedup store.
3. Optimize slow endpoint by profiling and targeted fixes.
4. Design resilient integration with flaky upstream service.

## 16) L5 Grilling Questions (Python)

1. How do you decide between asyncio and multiprocessing for a given path?
2. What are common async anti-patterns that pass tests but fail in prod?
3. How do you enforce API contract discipline across many teams?
4. How do you prevent event-loop collapse under partial downstream outages?
5. How do you prove your optimizations improved real user latency?

## 17) 60-Minute Python Revision Sprint

0-20 min:
- GIL, asyncio, process model trade-offs.

20-40 min:
- boundary validation, retries/idempotency, and reliability patterns.

40-60 min:
- one incident narrative: async failure, memory bloat, or duplicate side effects.

## 18) Source Anchors for Python Expansion

Grounding references:
- Python official docs for runtime/library behavior.
- FastAPI and pydantic production practices.
- async failure-mode patterns from real backend operations.

---
