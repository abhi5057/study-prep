# Go Backend Senior Deep Dive (7 YOE Interview)

This guide focuses on what a senior backend interview expects beyond syntax.

## 1) Language Internals You Must Explain Clearly

### 1.1 Goroutine Scheduling

- Go runtime uses M:N scheduling.
- G = goroutine, M = OS thread, P = logical processor.
- Work stealing balances runnable goroutines across P.

Interview answer shape:
- Why goroutines are cheap: small initial stack + runtime scheduling.
- Why not free: context switching, blocked syscalls, memory growth, synchronization overhead.

### 1.2 Memory Model and Happens-Before

- Reads/writes across goroutines must be synchronized.
- Synchronization tools: channel send/receive, mutex lock/unlock, atomic operations.
- Without happens-before edges, behavior is a data race and logically invalid for production code.

### 1.3 Escape Analysis

- Compiler decides stack vs heap allocation.
- Variables that outlive scope or are captured often escape to heap.
- Heap allocations increase GC pressure.

Practice command:

```bash
go build -gcflags="-m" ./...
```

### 1.4 Interface Internals

- Interface value stores dynamic type + value pointer.
- Nil pitfall: typed nil pointer inside interface is non-nil interface.

Example:

```go
type customErr struct{}
func (c *customErr) Error() string { return "x" }

func bad() error {
    var e *customErr = nil
    return e // non-nil interface at call site
}
```

Fix:
- Return nil explicitly when no error.

## 2) Production-Grade Concurrency Patterns

### 2.1 Worker Pool with Backpressure

Use bounded channels and cancellation:

```go
type Job struct { ID string; Payload []byte }
type Result struct { ID string; Err error }

func StartWorkers(ctx context.Context, n int, jobs <-chan Job) <-chan Result {
    out := make(chan Result, n)
    var wg sync.WaitGroup
    wg.Add(n)

    worker := func() {
        defer wg.Done()
        for {
            select {
            case <-ctx.Done():
                return
            case j, ok := <-jobs:
                if !ok {
                    return
                }
                err := process(ctx, j)
                select {
                case out <- Result{ID: j.ID, Err: err}:
                case <-ctx.Done():
                    return
                }
            }
        }
    }

    for i := 0; i < n; i++ {
        go worker()
    }

    go func() {
        wg.Wait()
        close(out)
    }()

    return out
}
```

Why interviewers like this:
- Handles graceful shutdown.
- Avoids goroutine leaks.
- Explicit backpressure via bounded queue.

### 2.2 Fan-Out/Fan-In with Error Group

Use errgroup for cancellation on first failure:

```go
g, ctx := errgroup.WithContext(parent)
for _, task := range tasks {
    task := task
    g.Go(func() error {
        return doTask(ctx, task)
    })
}
if err := g.Wait(); err != nil {
    return fmt.Errorf("parallel tasks failed: %w", err)
}
```

## 3) API and Service Design in Go

### 3.1 Package Structure

Prefer:
- `internal` for private app code.
- `pkg` only for reusable libraries.
- clear boundaries: transport, service, repository, domain.

### 3.2 Interface Design

Guidelines:
- Keep interfaces near consumption site.
- Small interfaces improve testability.
- Avoid giant god interfaces.

```go
type UserReader interface {
    GetByID(ctx context.Context, id string) (User, error)
}

type UserWriter interface {
    Save(ctx context.Context, u User) error
}
```

### 3.3 Error Taxonomy

Use categories:
- Validation errors.
- Not found.
- Conflict.
- Upstream transient.
- Internal unknown.

Map to HTTP:
- Validation -> 400
- Not found -> 404
- Conflict -> 409
- Upstream transient -> 503
- Internal -> 500

## 4) Reliability Patterns in Go Services

### 4.1 Timeouts Everywhere

- Client timeout.
- Per request timeout.
- DB timeout.
- Message processing timeout.

```go
ctx, cancel := context.WithTimeout(ctx, 2*time.Second)
defer cancel()
resp, err := httpClient.Do(req.WithContext(ctx))
```

### 4.2 Retry with Exponential Backoff + Jitter

Retry only transient failures.
Never retry unsafe operations without idempotency.

### 4.3 Circuit Breaker

Use for unstable dependencies to avoid cascading failure.
Talk about states: closed, open, half-open.

### 4.4 Graceful Shutdown

Checklist:
- stop accepting new traffic.
- drain in-flight requests.
- cancel background workers.
- flush metrics/logs if needed.
- close DB and message clients.

## 5) Data Layer Patterns

### 5.1 Transaction Boundaries

- Keep transactions short.
- Avoid network calls inside transaction.
- Use explicit isolation where needed.

### 5.2 Idempotency

For payment/order style operations:
- idempotency key table with status.
- unique constraints for de-dup.
- safe retries.

### 5.3 Outbox Pattern

Need atomic DB write + event publish:
- write domain state + outbox row in one transaction.
- background publisher sends outbox rows.
- mark published with retry handling.

## 6) Performance and Profiling

### 6.1 Golden Rule

- Do not optimize blind.
- Profile first, optimize hot paths, then validate.

### 6.2 Tooling

- `go test -bench=. -benchmem`
- `go test -race ./...`
- pprof (CPU, heap, goroutine, mutex)
- trace for scheduler and latency analysis

### 6.3 Typical Wins

- Preallocate slices/maps.
- Use pooling for large temporary buffers.
- Reduce JSON encode/decode overhead where safe.
- Batch DB writes and message commits.
- Remove lock contention hot spots.

## 7) Security Topics in Go Backend

- Input validation and canonicalization.
- Context-aware authorization checks.
- Secret management (no secrets in code).
- mTLS/TLS settings.
- SQL injection prevention via parameterized queries.
- Safe deserialization patterns.

## 8) Testing Strategy at Senior Level

### 8.1 Test Pyramid

- Unit tests for business logic.
- Integration tests for DB/message interactions.
- Contract tests for APIs.
- End-to-end tests for critical user journeys.

### 8.2 Deterministic Tests

- Fake clocks.
- Isolated DB schema/test containers.
- Controlled randomness seeds.

### 8.3 Chaos and Failure Injection

- Simulate timeouts and partial failures.
- Verify retries, breaker behavior, and idempotency.

## 9) Rapid-Fire Go Interview Questions (Senior)

1. Channel vs mutex: when and why?
- Mutex for shared mutable state with low overhead.
- Channels for coordination and ownership transfer.

2. What causes goroutine leaks in real services?
- Orphan workers, blocked sends, unbounded queue + no consumers, missing cancellation.

3. How do you reduce GC pauses?
- Lower allocation rate, reuse buffers, avoid large temporary objects, tune object lifetimes.

4. Why use context in every boundary?
- Deadline and cancellation propagation, trace correlation, request-scoped behavior.

5. How do you structure error handling for APIs?
- typed categories, wrapped context, central mapping to transport layer.

## 10) Senior Story Prompts (Speak with Numbers)

Prepare responses for:
- Biggest outage you handled in Go service.
- Throughput scaling project with before/after metrics.
- Migration from monolith to services or queue-driven processing.
- Performance optimization with pprof evidence.
- Reliability improvements and incident reduction.

Strong ending line for each story:
- "The change reduced X and improved Y, and we institutionalized it via runbooks/tests/dashboards."

## 11) Tricky Go Questions with Code and Explanations

### 11.1 nil Map Write Panic

```go
func main() {
    var m map[string]int
    // m["x"] = 1 // panic: assignment to entry in nil map

    m = make(map[string]int)
    m["x"] = 1
    fmt.Println(m["x"])
}
```

Interview explanation:
- Reading from nil map is allowed and returns zero value.
- Writing to nil map panics because map storage was never allocated.

### 11.2 Goroutine Leak Pattern

```go
func leaky(ch <-chan int) {
    go func() {
        v := <-ch
        fmt.Println(v)
    }()
}

func fixed(ctx context.Context, ch <-chan int) {
    go func() {
        select {
        case v := <-ch:
            fmt.Println(v)
        case <-ctx.Done():
            return
        }
    }()
}
```

### 11.3 Loop Variable Capture Trap

```go
vals := []int{10, 20, 30}
for _, v := range vals {
    v := v
    go func() {
        fmt.Println(v)
    }()
}
```

### 11.4 Context Timeout Around Dependency Call

```go
func callInventory(ctx context.Context, client *http.Client, req *http.Request) error {
    ctx, cancel := context.WithTimeout(ctx, 800*time.Millisecond)
    defer cancel()

    resp, err := client.Do(req.WithContext(ctx))
    if err != nil {
        return fmt.Errorf("inventory call failed: %w", err)
    }
    defer resp.Body.Close()

    if resp.StatusCode >= 500 {
        return fmt.Errorf("inventory temporary failure: %d", resp.StatusCode)
    }
    return nil
}
```

### 11.5 Table-Driven Test Example

```go
func TestClamp(t *testing.T) {
    tests := []struct {
        name string
        in   int
        lo   int
        hi   int
        want int
    }{
        {"below", -1, 0, 10, 0},
        {"inside", 5, 0, 10, 5},
        {"above", 50, 0, 10, 10},
    }

    for _, tt := range tests {
        tt := tt
        t.Run(tt.name, func(t *testing.T) {
            got := Clamp(tt.in, tt.lo, tt.hi)
            if got != tt.want {
                t.Fatalf("got=%d want=%d", got, tt.want)
            }
        })
    }
}
```

## 12) Go Production Architecture Interview Themes

- Latency budgets across sync and async hops.
- Retry storms and coordinated amplification.
- Idempotency and dedup boundaries.
- Contract evolution for APIs/events.
- Blast radius reduction and failure isolation.

## 13) Common Failure Modes and How To Explain Them

- Queue backlog growth due to hidden downstream slowdown.
- CPU saturation from unbounded goroutine fan-out.
- Memory pressure from retained slices and caches.
- Lock convoy under hot-key access pattern.
- Cascading timeouts from missing per-hop budgets.

## 14) Senior Design Drills (Go-Centric)

- Build notification fanout service with bounded worker pools.
- Design payment state machine with outbox and retries.
- Build audit-log pipeline with at-least-once delivery and idempotent consumer.
- Design search API with cache + stale-while-revalidate.

## 15) Go Tooling Commands You Should Mention

```bash
go test ./...
go test -race ./...
go test -bench=. -benchmem ./...
go tool pprof -http=:8080 cpu.prof
go tool trace trace.out
go vet ./...
```

## 16) System Design Tie-In (How Go Answers Differ)

- Prefer simple, explicit concurrency over clever abstractions.
- Design for cancellation and bounded resource usage first.
- Treat queue sizes and worker counts as SLO controls.
- Show observability from day one: metrics, tracing, structured logs.

## 17) Last-Minute Go Revision (30 Minutes)

0-10 min:
- memory model, context rules, channel close ownership.

10-20 min:
- worker pool, retries/idempotency, graceful shutdown sequence.

20-30 min:
- one DSA problem + one production incident narrative.

## 18) L4/L5 Go Deep Drill (Runtime, Memory, Production)

### 18.1 Go Memory Model You Must Explain Clearly

- A write in one goroutine is not guaranteed visible to another goroutine unless a synchronization event creates a happens-before edge.
- Synchronization edges: channel send/receive, mutex unlock/lock, atomic ops, `WaitGroup` completion.
- `context.Context` cancels work; it does not synchronize arbitrary data access.

Interview trap:
- "It worked locally" race conditions are undefined behavior, not sometimes stale data only.

### 18.2 Scheduler and Latency Under Load

- Go uses M:N scheduling (goroutines on OS threads).
- Blocking syscalls, cgo calls, and large GC pauses can increase tail latency.
- `GOMAXPROCS` should usually match available CPU quota in containers.

Quick command set:

```bash
go test -race ./...
go test -run TestCriticalPath -count=1 -v
go test -bench=. -benchmem ./...
go tool pprof -http=:8080 cpu.prof
```

### 18.3 Context, Timeouts, and Partial Failure

```go
ctx, cancel := context.WithTimeout(parent, 250*time.Millisecond)
defer cancel()

resp, err := client.Do(req.WithContext(ctx))
if err != nil {
    if errors.Is(err, context.DeadlineExceeded) {
        return fmt.Errorf("downstream timeout: %w", err)
    }
    return fmt.Errorf("downstream failure: %w", err)
}
defer resp.Body.Close()
```

### 18.4 Worker Pools and Backpressure

- Unbounded goroutine spawning under burst traffic causes memory blowups and scheduler thrash.
- Bound queue + bounded workers + rejection policy is safer than infinite buffering.

### 18.5 Senior-Level Go Grilling Questions

1. Why can a channel-only design still deadlock under partial downstream failure?
2. When do you choose `sync.Map` over map+mutex?
3. How do you prove idempotency for retrying handlers?
4. How do you set timeout budgets across API -> DB -> broker calls?
5. What pprof signal tells you allocation churn vs CPU hotspot?

---
## 19) L5 Runtime and Compiler Theory (What Architects Are Asked)

### 19.1 Memory Model in Interview Language

Use this framing:
- DRF-SC rule: data-race-free Go programs behave as if sequentially consistent.
- Happens-before edges come from sync events, not from wishful ordering.
- `go` statement synchronizes goroutine start, but goroutine completion must be synchronized explicitly (channel/lock/waitgroup).

High-value statement in interviews:
- "`context` propagates cancellation and deadline metadata; it does not provide memory synchronization for shared mutable state."

### 19.2 GC and Allocation Pressure

Architect-level points:
- Most latency regressions in Go services are not from raw CPU alone, but from allocation churn + GC assist pressure under burst load.
- Reducing allocations per request often improves both throughput and p95/p99.
- Large transient objects and avoidable copies (JSON, []byte transformations) are frequent offenders.

Operational checklist:
- Compare `allocs/op` before/after (`go test -bench . -benchmem`).
- Inspect heap and allocation flamegraphs (`pprof` heap/profile).
- Verify that optimization actually moved SLOs, not just microbenchmarks.

### 19.3 Scheduler Reality Under Production Load

- Goroutines are cheap, not free.
- Too many runnable goroutines can increase scheduling overhead and tail latency.
- cgo or long blocking syscalls can reduce effective parallelism.
- In containers, ensure CPU limits and `GOMAXPROCS` assumptions are aligned.

### 19.4 Escape Analysis in Practical Terms

Command:

```bash
go build -gcflags="-m" ./...
```

How to discuss it:
- "Escapes are not bugs; they are cost signals."
- "I optimize escapes only in hot paths validated by profiling."

---

## 20) Senior/Architect Backend Design Through Go

### 20.1 Correct Timeout Budgeting Across Hops

Rule:
- Inbound SLO budget is split across downstream hops.
- Never give every hop the full parent timeout.

Example:

```go
func handle(ctx context.Context) error {
    // Parent budget: 300ms
    dbCtx, dbCancel := context.WithTimeout(ctx, 80*time.Millisecond)
    defer dbCancel()

    if err := queryUser(dbCtx); err != nil {
        return fmt.Errorf("db query failed: %w", err)
    }

    invCtx, invCancel := context.WithTimeout(ctx, 120*time.Millisecond)
    defer invCancel()

    if err := callInventory(invCtx); err != nil {
        return fmt.Errorf("inventory call failed: %w", err)
    }

    return nil
}
```

Interview explanation:
- Budgeting prevents one dependency from consuming the entire request SLO.

### 20.2 Idempotency and Exactly-Once Myths

Architect answer pattern:
- Transport gives at-least-once most of the time.
- Exactly-once is an application-level property via idempotency keys + durable state transitions.

Example schema idea:
- `idempotency_key` UNIQUE
- status: `processing`, `done`, `failed`
- response snapshot stored for replay-safe retries

### 20.3 Outbox and Reliable Event Publication

Use when you need:
- DB state change + event publish without 2PC.

Pattern:
- In one DB transaction: write business row + outbox row.
- Separate publisher sends outbox to broker and marks sent.
- Consumer remains idempotent.

---

## 21) Go DSA/Algo Interview Section (Solved, Senior-Focused)

Interviewers often test:
- Problem-solving fundamentals (arrays/maps/sliding window/heap/graph).
- Go fluency (slices, maps, pointer/value semantics, custom heap).
- Complexity communication and trade-off reasoning.

### 21.1 Two Sum (Hash Map)

Problem:
- Given array `nums` and `target`, return indices of two numbers adding to target.

```go
func TwoSum(nums []int, target int) []int {
    seen := make(map[int]int, len(nums)) // value -> index
    for i, x := range nums {
        if j, ok := seen[target-x]; ok {
            return []int{j, i}
        }
        seen[x] = i
    }
    return nil
}
```

Complexity:
- Time: O(n)
- Space: O(n)

Interview explanation:
- Trade memory for linear-time lookup.
- Mention duplicate behavior (store latest or earliest index intentionally).

### 21.2 Longest Substring Without Repeating Characters (Sliding Window)

```go
func LengthOfLongestSubstring(s string) int {
    last := make(map[byte]int)
    left, best := 0, 0

    for right := 0; right < len(s); right++ {
        c := s[right]
        if p, ok := last[c]; ok && p >= left {
            left = p + 1
        }
        if right-left+1 > best {
            best = right - left + 1
        }
        last[c] = right
    }
    return best
}
```

Complexity:
- Time: O(n)
- Space: O(1) for fixed charset, else O(k)

Follow-up you should handle:
- Unicode-safe version using `rune` and index mapping.

### 21.3 Merge Intervals

```go
import "sort"

func MergeIntervals(intervals [][]int) [][]int {
    if len(intervals) == 0 {
        return nil
    }

    sort.Slice(intervals, func(i, j int) bool {
        return intervals[i][0] < intervals[j][0]
    })

    out := make([][]int, 0, len(intervals))
    cur := []int{intervals[0][0], intervals[0][1]}

    for i := 1; i < len(intervals); i++ {
        nxt := intervals[i]
        if nxt[0] <= cur[1] {
            if nxt[1] > cur[1] {
                cur[1] = nxt[1]
            }
        } else {
            out = append(out, cur)
            cur = []int{nxt[0], nxt[1]}
        }
    }
    out = append(out, cur)
    return out
}
```

Complexity:
- Time: O(n log n) due to sort
- Space: O(n)

Interview explanation:
- Sorting creates local adjacency so linear merge becomes possible.

### 21.4 Top K Frequent Elements (Heap)

```go
import "container/heap"

type pair struct {
    num  int
    freq int
}

type minHeap []pair

func (h minHeap) Len() int            { return len(h) }
func (h minHeap) Less(i, j int) bool  { return h[i].freq < h[j].freq }
func (h minHeap) Swap(i, j int)       { h[i], h[j] = h[j], h[i] }
func (h *minHeap) Push(x any)         { *h = append(*h, x.(pair)) }
func (h *minHeap) Pop() any {
    old := *h
    n := len(old)
    x := old[n-1]
    *h = old[:n-1]
    return x
}

func TopKFrequent(nums []int, k int) []int {
    freq := make(map[int]int, len(nums))
    for _, x := range nums {
        freq[x]++
    }

    h := &minHeap{}
    heap.Init(h)

    for n, f := range freq {
        heap.Push(h, pair{num: n, freq: f})
        if h.Len() > k {
            heap.Pop(h)
        }
    }

    out := make([]int, 0, k)
    for h.Len() > 0 {
        out = append(out, heap.Pop(h).(pair).num)
    }
    return out
}
```

Complexity:
- Time: O(n log k)
- Space: O(n)

Interview explanation:
- Min-heap of size `k` beats full sort when `k << n`.

### 21.5 LRU Cache (Map + Doubly Linked List)

```go
type node struct {
    key, val int
    prev, next *node
}

type LRU struct {
    cap int
    m   map[int]*node
    head, tail *node // dummy sentinels
}

func NewLRU(capacity int) *LRU {
    h, t := &node{}, &node{}
    h.next = t
    t.prev = h
    return &LRU{
        cap: capacity,
        m:   make(map[int]*node, capacity),
        head: h,
        tail: t,
    }
}

func (l *LRU) remove(n *node) {
    n.prev.next = n.next
    n.next.prev = n.prev
}

func (l *LRU) pushFront(n *node) {
    n.next = l.head.next
    n.prev = l.head
    l.head.next.prev = n
    l.head.next = n
}

func (l *LRU) Get(key int) (int, bool) {
    if n, ok := l.m[key]; ok {
        l.remove(n)
        l.pushFront(n)
        return n.val, true
    }
    return 0, false
}

func (l *LRU) Put(key, val int) {
    if n, ok := l.m[key]; ok {
        n.val = val
        l.remove(n)
        l.pushFront(n)
        return
    }

    n := &node{key: key, val: val}
    l.m[key] = n
    l.pushFront(n)

    if len(l.m) > l.cap {
        evict := l.tail.prev
        l.remove(evict)
        delete(l.m, evict.key)
    }
}
```

Complexity:
- `Get`: O(1)
- `Put`: O(1)

Follow-up:
- Add thread safety using `sync.Mutex` and measure contention before adding sharding.

### 21.6 BFS Shortest Path in Grid

```go
type point struct{ r, c, d int }

func ShortestPathBinaryMatrix(grid [][]int) int {
    n := len(grid)
    if n == 0 || grid[0][0] == 1 || grid[n-1][n-1] == 1 {
        return -1
    }

    dirs := [][2]int{{1,0},{-1,0},{0,1},{0,-1},{1,1},{1,-1},{-1,1},{-1,-1}}
    q := []point{{0, 0, 1}}
    grid[0][0] = 1 // visited mark

    for len(q) > 0 {
        cur := q[0]
        q = q[1:]

        if cur.r == n-1 && cur.c == n-1 {
            return cur.d
        }

        for _, d := range dirs {
            nr, nc := cur.r+d[0], cur.c+d[1]
            if nr >= 0 && nr < n && nc >= 0 && nc < n && grid[nr][nc] == 0 {
                grid[nr][nc] = 1
                q = append(q, point{nr, nc, cur.d + 1})
            }
        }
    }
    return -1
}
```

Complexity:
- Time: O(n^2)
- Space: O(n^2)

Interview explanation:
- BFS gives shortest path in unweighted graph by level-order expansion.

---

## 22) Go-Specific Coding Rounds (Beyond Pure DSA)

### 22.1 Bounded Worker Pool with Cancellation (Common Senior Round)

```go
func RunPool(ctx context.Context, workers int, in <-chan int, fn func(context.Context, int) error) error {
    g, ctx := errgroup.WithContext(ctx)

    for w := 0; w < workers; w++ {
        g.Go(func() error {
            for {
                select {
                case <-ctx.Done():
                    return ctx.Err()
                case x, ok := <-in:
                    if !ok {
                        return nil
                    }
                    if err := fn(ctx, x); err != nil {
                        return err
                    }
                }
            }
        })
    }

    return g.Wait()
}
```

What interviewer is testing:
- Leak-free shutdown, cancellation propagation, bounded concurrency, first-error behavior.

### 22.2 Token Bucket Rate Limiter (No External Library)

```go
type Limiter struct {
    tokens chan struct{}
}

func NewLimiter(ratePerSec, burst int) *Limiter {
    l := &Limiter{tokens: make(chan struct{}, burst)}

    // Fill burst initially.
    for i := 0; i < burst; i++ {
        l.tokens <- struct{}{}
    }

    ticker := time.NewTicker(time.Second / time.Duration(ratePerSec))
    go func() {
        for range ticker.C {
            select {
            case l.tokens <- struct{}{}:
            default:
            }
        }
    }()

    return l
}

func (l *Limiter) Allow() bool {
    select {
    case <-l.tokens:
        return true
    default:
        return false
    }
}
```

Follow-up:
- Add `AllowN`, context-aware blocking acquire, and per-tenant limiters.

---

## 23) Senior Architect Go Interview Grilling (L5)

1. How do you prove no goroutine leaks in a request pipeline?
2. Why can buffered channels hide deadlocks until traffic shape changes?
3. When do you use channels vs mutexes vs atomics?
4. How do you map p99 regressions to allocation churn vs lock contention?
5. What is your rollback strategy when a Go release changes runtime behavior?
6. How do you design idempotent handlers for retries across HTTP and Kafka?
7. How would you run canary analysis for a Go service under heterogeneous traffic?
8. How do you guard against retry storms across layers (client, gateway, service)?
9. Where do you enforce authz in a Go microservice topology and why?
10. How do you prevent noisy-neighbor effects in multi-tenant Go APIs?

---

## 24) DSA Practice Plan for Go Interviews (10 Days)

Day 1:
- Arrays, hash map, two pointers
- Solve 6 easy + 2 medium

Day 2:
- Sliding window, prefix sum
- Solve 5 medium

Day 3:
- Stack, monotonic stack, deque
- Solve 5 medium

Day 4:
- Binary search on answer, intervals
- Solve 5 medium

Day 5:
- Linked list patterns, LRU, fast/slow pointers
- Solve 4 medium + 1 hard

Day 6:
- Trees (DFS/BFS, LCA)
- Solve 5 medium

Day 7:
- Heap, top-k, merge-k
- Solve 5 medium

Day 8:
- Graphs (BFS/DFS/topo/union-find)
- Solve 5 medium

Day 9:
- Dynamic programming fundamentals
- Solve 5 medium

Day 10:
- Mock round in Go (45 min DSA + 45 min concurrency/system)

Execution rule:
- For each problem, speak: brute force -> better -> optimal -> edge cases -> tests -> complexity.

---

## 25) 30 Rapid-Fire Go Interview Questions (Senior)

1. Why is `map` concurrent read/write unsafe?
2. When do you choose pointer receiver vs value receiver?
3. Why can a typed nil error be non-nil?
4. Difference between `new` and `make`?
5. Why can slice re-slicing retain large backing arrays?
6. How do you avoid accidental memory retention after parsing large buffers?
7. What does `go test -race` miss, and how do you compensate?
8. Why should `context` be first argument and not stored in struct fields?
9. Why avoid swallowing `ctx.Err()`?
10. How do you bound fan-out under burst load?
11. Why is retry without idempotency dangerous?
12. How do you avoid duplicate side effects in handlers?
13. Difference between backpressure and rate limiting?
14. How do you inspect lock contention in Go?
15. When is `sync.Map` appropriate?
16. Why can more goroutines reduce throughput?
17. How do you reason about channel close ownership?
18. What are safe close patterns for multi-producer systems?
19. How do you build graceful shutdown for HTTP + workers + consumers?
20. What does `pprof` heap profile tell you that CPU profile does not?
21. How do you identify allocation hot paths quickly?
22. Why can `append` reallocate and break aliasing assumptions?
23. How do you design package boundaries in large Go repos?
24. How do you enforce dependency direction in Go modules?
25. What is your API error contract policy?
26. How do you version protobuf/JSON contracts safely?
27. How do you implement circuit breaker in Go service clients?
28. How do you test timeout behavior deterministically?
29. How do you test cancellation propagation?
30. What production incident changed how you write Go code today?

---

## 26) Interview Story Bank (Go-Specific, Quantified)

Prepare STAR stories with metrics:
- Reduced p99 from 420ms to 170ms by cutting allocs/op and fixing lock contention.
- Eliminated goroutine leaks in ingestion pipeline, reducing steady-state memory by 35%.
- Migrated unbounded fan-out to bounded worker pool, stabilizing CPU during traffic spikes.
- Added idempotency + outbox for payment events, reducing duplicate side effects to near zero.
- Built timeout-budget policy across service graph, improving error isolation under downstream slowness.

Strong closing line:
- "We improved reliability and made the fix durable via tests, dashboards, and runbooks."

---

## 27) Source Anchors for This Go Expansion

This section aligns with:
- Go Memory Model (`go.dev/ref/mem`) for happens-before and DRF-SC.
- Go blog pipeline/context patterns for cancellation-safe concurrency.
- Effective Go for idioms (`new` vs `make`, interfaces, slice/map behavior).
- Go 1.22 loop-variable semantics changes from official Go blog.

Interview signal:
- "I can solve coding rounds in idiomatic Go and I can reason about production trade-offs under failure and scale."

---

## 28) Hard Go Coding Rounds (15 Solved Problems + Follow-ups)

This section mirrors the depth expected in senior/architect coding interviews: correctness first, then complexity, then production-minded follow-ups.

### 28.1 Kth Largest Element in Stream (Min-Heap)

```go
import "container/heap"

type IntMinHeap []int

func (h IntMinHeap) Len() int           { return len(h) }
func (h IntMinHeap) Less(i, j int) bool { return h[i] < h[j] }
func (h IntMinHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }
func (h *IntMinHeap) Push(x any)        { *h = append(*h, x.(int)) }
func (h *IntMinHeap) Pop() any {
    old := *h
    x := old[len(old)-1]
    *h = old[:len(old)-1]
    return x
}

type KthLargest struct {
    k int
    h IntMinHeap
}

func Constructor(k int, nums []int) KthLargest {
    kl := KthLargest{k: k, h: IntMinHeap{}}
    heap.Init(&kl.h)
    for _, n := range nums {
        kl.Add(n)
    }
    return kl
}

func (kl *KthLargest) Add(val int) int {
    heap.Push(&kl.h, val)
    if kl.h.Len() > kl.k {
        heap.Pop(&kl.h)
    }
    return kl.h[0]
}
```

Complexity:
- Add: O(log k)
- Space: O(k)

Follow-up:
- How would you shard this by tenant and periodically compact old states?

### 28.2 Median From Data Stream (Two Heaps)

```go
import "container/heap"

type MinH []int
func (h MinH) Len() int           { return len(h) }
func (h MinH) Less(i, j int) bool { return h[i] < h[j] }
func (h MinH) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }
func (h *MinH) Push(x any)        { *h = append(*h, x.(int)) }
func (h *MinH) Pop() any { old := *h; x := old[len(old)-1]; *h = old[:len(old)-1]; return x }

type MaxH []int
func (h MaxH) Len() int           { return len(h) }
func (h MaxH) Less(i, j int) bool { return h[i] > h[j] }
func (h MaxH) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }
func (h *MaxH) Push(x any)        { *h = append(*h, x.(int)) }
func (h *MaxH) Pop() any { old := *h; x := old[len(old)-1]; *h = old[:len(old)-1]; return x }

type MedianFinder struct {
    low  MaxH // max-heap
    high MinH // min-heap
}

func NewMedianFinder() *MedianFinder {
    m := &MedianFinder{}
    heap.Init(&m.low)
    heap.Init(&m.high)
    return m
}

func (m *MedianFinder) AddNum(x int) {
    if m.low.Len() == 0 || x <= m.low[0] {
        heap.Push(&m.low, x)
    } else {
        heap.Push(&m.high, x)
    }

    if m.low.Len() > m.high.Len()+1 {
        heap.Push(&m.high, heap.Pop(&m.low))
    }
    if m.high.Len() > m.low.Len() {
        heap.Push(&m.low, heap.Pop(&m.high))
    }
}

func (m *MedianFinder) FindMedian() float64 {
    if m.low.Len() > m.high.Len() {
        return float64(m.low[0])
    }
    return float64(m.low[0]+m.high[0]) / 2.0
}
```

Complexity:
- Insert O(log n), median O(1)

### 28.3 Sliding Window Maximum (Monotonic Deque)

```go
func MaxSlidingWindow(nums []int, k int) []int {
    if len(nums) == 0 || k == 0 {
        return nil
    }
    dq := make([]int, 0) // indices, decreasing by nums[idx]
    out := make([]int, 0, len(nums)-k+1)

    for i := 0; i < len(nums); i++ {
        if len(dq) > 0 && dq[0] <= i-k {
            dq = dq[1:]
        }
        for len(dq) > 0 && nums[dq[len(dq)-1]] <= nums[i] {
            dq = dq[:len(dq)-1]
        }
        dq = append(dq, i)

        if i >= k-1 {
            out = append(out, nums[dq[0]])
        }
    }
    return out
}
```

Complexity: O(n) time, O(k) space.

### 28.4 Minimum Window Substring

```go
func MinWindow(s, t string) string {
    if len(t) == 0 || len(s) < len(t) {
        return ""
    }

    need := make(map[byte]int)
    for i := 0; i < len(t); i++ {
        need[t[i]]++
    }

    have := make(map[byte]int)
    formed, required := 0, len(need)
    left := 0
    bestLen, bestL := 1<<30, 0

    for right := 0; right < len(s); right++ {
        c := s[right]
        have[c]++
        if need[c] > 0 && have[c] == need[c] {
            formed++
        }

        for formed == required {
            if right-left+1 < bestLen {
                bestLen = right - left + 1
                bestL = left
            }
            d := s[left]
            have[d]--
            if need[d] > 0 && have[d] < need[d] {
                formed--
            }
            left++
        }
    }

    if bestLen == 1<<30 {
        return ""
    }
    return s[bestL : bestL+bestLen]
}
```

Complexity: O(|s| + |t|).

### 28.5 Subarray Sum Equals K (Prefix Sum + Hash)

```go
func SubarraySum(nums []int, k int) int {
    count := 0
    prefix := 0
    freq := map[int]int{0: 1}

    for _, x := range nums {
        prefix += x
        count += freq[prefix-k]
        freq[prefix]++
    }
    return count
}
```

Complexity: O(n) time, O(n) space.

### 28.6 Number of Islands (DFS)

```go
func NumIslands(grid [][]byte) int {
    if len(grid) == 0 {
        return 0
    }
    rows, cols := len(grid), len(grid[0])

    var dfs func(int, int)
    dfs = func(r, c int) {
        if r < 0 || r >= rows || c < 0 || c >= cols || grid[r][c] != '1' {
            return
        }
        grid[r][c] = '0'
        dfs(r+1, c)
        dfs(r-1, c)
        dfs(r, c+1)
        dfs(r, c-1)
    }

    islands := 0
    for r := 0; r < rows; r++ {
        for c := 0; c < cols; c++ {
            if grid[r][c] == '1' {
                islands++
                dfs(r, c)
            }
        }
    }
    return islands
}
```

Complexity: O(m*n).

### 28.7 Course Schedule (Topological Sort)

```go
func CanFinish(numCourses int, prerequisites [][]int) bool {
    graph := make([][]int, numCourses)
    indeg := make([]int, numCourses)

    for _, p := range prerequisites {
        a, b := p[0], p[1]
        graph[b] = append(graph[b], a)
        indeg[a]++
    }

    q := make([]int, 0)
    for i := 0; i < numCourses; i++ {
        if indeg[i] == 0 {
            q = append(q, i)
        }
    }

    taken := 0
    for len(q) > 0 {
        v := q[0]
        q = q[1:]
        taken++

        for _, nei := range graph[v] {
            indeg[nei]--
            if indeg[nei] == 0 {
                q = append(q, nei)
            }
        }
    }

    return taken == numCourses
}
```

Complexity: O(V+E).

### 28.8 Clone Graph (BFS)

```go
type Node struct {
    Val       int
    Neighbors []*Node
}

func CloneGraph(node *Node) *Node {
    if node == nil {
        return nil
    }

    mp := map[*Node]*Node{node: {Val: node.Val}}
    q := []*Node{node}

    for len(q) > 0 {
        cur := q[0]
        q = q[1:]

        for _, nei := range cur.Neighbors {
            if _, ok := mp[nei]; !ok {
                mp[nei] = &Node{Val: nei.Val}
                q = append(q, nei)
            }
            mp[cur].Neighbors = append(mp[cur].Neighbors, mp[nei])
        }
    }
    return mp[node]
}
```

### 28.9 Trie (Prefix Tree)

```go
type Trie struct {
    children [26]*Trie
    end      bool
}

func NewTrie() *Trie { return &Trie{} }

func (t *Trie) Insert(word string) {
    cur := t
    for i := 0; i < len(word); i++ {
        idx := word[i] - 'a'
        if cur.children[idx] == nil {
            cur.children[idx] = &Trie{}
        }
        cur = cur.children[idx]
    }
    cur.end = true
}

func (t *Trie) Search(word string) bool {
    cur := t
    for i := 0; i < len(word); i++ {
        idx := word[i] - 'a'
        if cur.children[idx] == nil {
            return false
        }
        cur = cur.children[idx]
    }
    return cur.end
}

func (t *Trie) StartsWith(prefix string) bool {
    cur := t
    for i := 0; i < len(prefix); i++ {
        idx := prefix[i] - 'a'
        if cur.children[idx] == nil {
            return false
        }
        cur = cur.children[idx]
    }
    return true
}
```

### 28.10 Word Break (DP)

```go
func WordBreak(s string, wordDict []string) bool {
    dict := make(map[string]struct{}, len(wordDict))
    for _, w := range wordDict {
        dict[w] = struct{}{}
    }

    dp := make([]bool, len(s)+1)
    dp[0] = true

    for i := 1; i <= len(s); i++ {
        for j := 0; j < i; j++ {
            if dp[j] {
                if _, ok := dict[s[j:i]]; ok {
                    dp[i] = true
                    break
                }
            }
        }
    }
    return dp[len(s)]
}
```

Complexity: O(n^2) substring checks (plus slicing/hash costs).

### 28.11 Coin Change (DP)

```go
func CoinChange(coins []int, amount int) int {
    const inf = int(1e9)
    dp := make([]int, amount+1)
    for i := 1; i <= amount; i++ {
        dp[i] = inf
    }

    for a := 1; a <= amount; a++ {
        for _, c := range coins {
            if a-c >= 0 && dp[a-c] != inf {
                if dp[a-c]+1 < dp[a] {
                    dp[a] = dp[a-c] + 1
                }
            }
        }
    }

    if dp[amount] == inf {
        return -1
    }
    return dp[amount]
}
```

### 28.12 Longest Increasing Subsequence (Binary Search)

```go
import "sort"

func LengthOfLIS(nums []int) int {
    tails := make([]int, 0, len(nums))
    for _, x := range nums {
        i := sort.Search(len(tails), func(i int) bool { return tails[i] >= x })
        if i == len(tails) {
            tails = append(tails, x)
        } else {
            tails[i] = x
        }
    }
    return len(tails)
}
```

Complexity: O(n log n).

### 28.13 Merge K Sorted Lists (Min-Heap)

```go
import "container/heap"

type ListNode struct {
    Val  int
    Next *ListNode
}

type NodeHeap []*ListNode

func (h NodeHeap) Len() int           { return len(h) }
func (h NodeHeap) Less(i, j int) bool { return h[i].Val < h[j].Val }
func (h NodeHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }
func (h *NodeHeap) Push(x any)        { *h = append(*h, x.(*ListNode)) }
func (h *NodeHeap) Pop() any {
    old := *h
    x := old[len(old)-1]
    *h = old[:len(old)-1]
    return x
}

func MergeKLists(lists []*ListNode) *ListNode {
    h := &NodeHeap{}
    heap.Init(h)

    for _, head := range lists {
        if head != nil {
            heap.Push(h, head)
        }
    }

    dummy := &ListNode{}
    tail := dummy

    for h.Len() > 0 {
        n := heap.Pop(h).(*ListNode)
        tail.Next = n
        tail = tail.Next
        if n.Next != nil {
            heap.Push(h, n.Next)
        }
    }
    return dummy.Next
}
```

Complexity: O(N log k).

### 28.14 LFU Cache (Design Discussion Prompt)

Expected senior answer:
- `key -> (value, freq, node*)`
- `freq -> doubly linked list`
- `minFreq` tracking
- O(1) get/put amortized with careful linked-list operations.

Interviewers often use LFU to test:
- data structure composition,
- invariant maintenance,
- correctness under edge cases.

### 28.15 LSM-Style Log Compaction Toy Problem

Prompt:
- Implement in-memory key-value store with append-only log and periodic compaction.

What to explain:
- write path O(1) append,
- read path with latest-offset index,
- compaction and tombstones,
- crash consistency considerations.

This is a common senior-architect style coding+design hybrid round.

---

## 29) How To Answer Hard Go Coding Rounds (Framework)

For each problem, say this out loud:
1. Clarify constraints and edge cases.
2. Start with brute force and state why it is too slow.
3. Move to optimal approach and state invariant.
4. Implement cleanly in Go (maps/slices/heap idioms).
5. Run sample + adversarial test case.
6. State complexity and production caveats.

High-signal finish line:
- "If this runs in production, I would add metrics, timeouts, and memory guards around worst-case inputs."

---

## 30) Go Concurrency Mock Rounds (8 Timed Drills + Ideal Rubrics)

Use this section for high-pressure practice that simulates senior and architect interview loops.

### How To Run Each Drill

- 2 minutes: restate problem + constraints.
- 8 minutes: propose design and invariants.
- 12 minutes: code core implementation.
- 5 minutes: tests, edge cases, complexity, failure modes.
- 3 minutes: production hardening discussion.

Target: complete each drill in ~30 minutes.

### 30.1 Drill A: Leak-Free Fan-Out/Fan-In Pipeline

Prompt:
- Build a 3-stage integer pipeline: source -> transform workers -> sink.
- Add cancellation so early sink return does not leak goroutines.

What interviewer scores:
- Proper `done` or `context` broadcast cancellation.
- Correct channel close ownership.
- No send-on-closed-channel panic.

Ideal answer checklist:
- Each stage closes only its own outbound channel.
- Multi-producer close guarded via `WaitGroup` + closer goroutine.
- Sink exits cleanly on context cancellation.

### 30.2 Drill B: Bounded Worker Pool with Backpressure

Prompt:
- Process jobs with exactly `N` workers and bounded input queue.
- Reject or block when queue is full (choose and justify).

What interviewer scores:
- Backpressure strategy and trade-off clarity.
- Safe shutdown: stop intake, drain in-flight, return aggregated errors.

Ideal answer checklist:
- `select` with `ctx.Done()` around send/receive paths.
- Explicit policy: drop, block, or fail-fast under overload.
- Metrics hooks for queue depth and processing latency.

### 30.3 Drill C: Per-Key Serialization (Single Flight Per Key)

Prompt:
- For same key, ensure only one expensive function call executes; others wait and share result.

What interviewer scores:
- Correctness under race.
- Avoid global lock bottleneck.

Ideal answer checklist:
- `map[key]*call` guarded by mutex.
- waiter count or done channel per key call.
- cleanup path on success and failure.

### 30.4 Drill D: Rate-Limited HTTP Client Wrapper

Prompt:
- Wrap HTTP client with token-bucket and timeout budget propagation.

What interviewer scores:
- Correct context usage (`req.WithContext`).
- No goroutine leak in refill goroutine during shutdown.

Ideal answer checklist:
- Refill ticker stopped on `ctx.Done()`.
- Request timeout split from parent budget.
- Distinguish retryable vs non-retryable responses.

### 30.5 Drill E: Concurrent Map Access Strategy

Prompt:
- Implement read-heavy cache with periodic refresh.
- Choose between map+RWMutex, sharded map, or `sync.Map` and justify.

What interviewer scores:
- Data-structure choice by access pattern.
- Correctness of refresh path and staleness bounds.

Ideal answer checklist:
- Explain why `sync.Map` fits mostly-read, write-once/read-many patterns.
- For mixed writes, prefer sharded map + mutexes.
- TTL and stale-read policy explicitly documented.

### 30.6 Drill F: Ordered Processing with Parallel Workers

Prompt:
- Process messages in parallel but emit results in original order.

What interviewer scores:
- Correct order restoration logic.
- Memory bound for reordering buffer.

Ideal answer checklist:
- Tag each item with sequence id.
- Maintain `nextExpected` pointer and pending result map.
- Bound pending map to prevent unbounded growth.

### 30.7 Drill G: Graceful Shutdown of HTTP + Background Consumers

Prompt:
- Service has HTTP server and Kafka/Rabbit consumer loop.
- On SIGTERM: stop intake, finish in-flight work, flush telemetry, exit before deadline.

What interviewer scores:
- Correct shutdown ordering.
- Deadline-aware termination behavior.

Ideal answer checklist:
- `signal.NotifyContext` root cancellation.
- `http.Server.Shutdown(ctx)` with bounded timeout.
- consumer stop signal + waitgroup join.
- final flush for logs/metrics with timeout.

### 30.8 Drill H: Deadlock and Race Debugging Round

Prompt:
- Given buggy code with occasional deadlock and race, explain triage plan.

What interviewer scores:
- Debugging methodology, not guesswork.

Ideal answer checklist:
- Reproduce with minimal deterministic harness.
- Run `go test -race ./...`.
- Capture goroutine dump and block profile.
- Use `pprof`/trace for scheduler and contention analysis.
- Fix one class of bug at a time and verify with targeted tests.

---

## 31) Rubric: What “Senior-Level” Looks Like In Go Concurrency Answers

A strong answer includes all of these:

1. Correctness invariants:
- who owns close,
- who can cancel,
- who drains,
- what happens on partial failure.

2. Resource safety:
- no leaked goroutines,
- no unbounded queue growth,
- no runaway retry loops.

3. SLO thinking:
- timeout budgets per hop,
- p95/p99 impact,
- overload behavior.

4. Observability:
- queue depth,
- active workers,
- reject count,
- timeout/error cardinality,
- trace span coverage.

5. Operational playbook:
- canary and rollback plan,
- runbook for saturation,
- failure injection test cases.

Interview signal line:
- "I design concurrency as a correctness problem first, then latency/cost optimization, then operational resilience."

---

## 32) Go Concurrency Anti-Patterns (Call Out In Interviews)

- Launching goroutines without cancellation path.
- Closing channels from receiver side (without ownership).
- Using buffered channels as a hidden fix for deadlocks.
- Retrying non-idempotent operations blindly.
- Ignoring context errors and losing root cause.
- Global mutex around hot path causing convoy effects.
- Unbounded fan-out from request handler.
- Swallowing backpressure and OOMing under burst traffic.

Quick remediation language:
- "I would first bound concurrency and queue size, then add cancellation and timeout budgeting, then verify with race/block profiles."

---

## 33) 60-Minute Concurrency Revision Sprint (Pre-Interview)

0-15 min:
- Memory model basics, happens-before edges, map race rules.

15-30 min:
- Worker pools, fan-in/fan-out, cancellation-safe pipeline patterns.

30-45 min:
- Graceful shutdown sequence and timeout budgeting.

45-60 min:
- One mock implementation + one debugging narrative (`-race`, goroutine dump, pprof).

Final reminder:
- In senior rounds, the interviewer is testing reliability thinking as much as code syntax.
