# Observer — AGENTS.md

## Project

Observer is a Go-based TUI for debugging latency in distributed HTTP requests.

The core question Observer answers is:

> Where did the latency go, and why?

Observer should consume OpenTelemetry traces and turn a single slow request into a concise, actionable diagnosis.

## Product principles

- Optimize for debugging speed, not observability-dashboard completeness.
- Prefer evidence over guesses.
- Show the latency contribution of each operation.
- Identify the critical path.
- Distinguish root latency from cascading/downstream latency.
- Keep the first version deterministic; AI is optional and comes later.
- Keep the core provider-independent. OpenTelemetry/OTLP is the first integration.
- Local development is the first target before production-scale deployments.
- Every feature should make answering "why is this request slow?" faster.

## Engineering principles

- Go.
- Keep packages small and focused.
- Prefer standard library APIs where practical.
- Use interfaces at boundaries, not everywhere.
- Keep the domain model independent of OTLP and TUI implementation details.
- Unit test analysis logic heavily.
- Use integration tests for OTLP/collector interactions.
- Avoid premature abstractions.
- Do not add an external dependency without a concrete reason.
- Preserve deterministic behavior in diagnosis.
- Never expose secrets or full sensitive request payloads by default.

---

# Phase 0 — Prerequisites / learning

Before implementing Observer, understand these topics.

## Go

- [ ] Go modules
- [ ] packages and package boundaries
- [ ] interfaces
- [ ] structs and composition
- [ ] pointers and value semantics
- [ ] errors and error wrapping
- [ ] context.Context
- [ ] goroutines
- [ ] channels
- [ ] select
- [ ] mutexes / sync primitives
- [ ] HTTP clients and servers
- [ ] JSON
- [ ] testing and benchmarks
- [ ] profiling with pprof

## Networking

- [ ] DNS
- [ ] TCP connection establishment
- [ ] TLS handshake
- [ ] HTTP/1.1
- [ ] HTTP/2 basics
- [ ] HTTP request lifecycle
- [ ] connection reuse / keep-alive
- [ ] connection pools
- [ ] latency vs throughput
- [ ] timeouts
- [ ] retries
- [ ] backoff
- [ ] serialization/deserialization

## Distributed systems

- [ ] synchronous vs asynchronous calls
- [ ] service-to-service communication
- [ ] cascading failures
- [ ] retries and retry amplification
- [ ] queues
- [ ] connection pool exhaustion
- [ ] locks / contention
- [ ] critical path
- [ ] fan-out / fan-in
- [ ] partial failure

## Databases

- [ ] SQL basics
- [ ] indexes
- [ ] query execution basics
- [ ] transactions
- [ ] locks
- [ ] connection pools
- [ ] query latency vs pool-wait latency

## Observability

- [ ] logs vs metrics vs traces
- [ ] tracing concepts
- [ ] spans
- [ ] traces
- [trace IDs / span IDs]
- [ ] parent-child relationships
- [ ] span attributes
- [ ] events
- [ ] status / error information
- [ ] context propagation
- [ ] W3C Trace Context
- [ ] OpenTelemetry architecture
- [ ] OTLP
- [ ] OpenTelemetry Collector

## TUI

- [ ] terminal input/output
- [ ] event loops
- [ ] keyboard navigation
- [ ] rendering
- [ ] viewport / scrolling
- [ ] state management
- [ ] asynchronous data updates

Suggested Go TUI stack:

- Bubble Tea
- Lip Gloss
- Bubbles

Do not commit to these libraries until the basic TUI architecture is understood.

---

# Phase 1 — Project skeleton

Goal: establish a clean Go project.

Tasks:

- [ ] Initialize Go module.
- [ ] Create CLI entry point.
- [ ] Create package structure.
- [ ] Add configuration model.
- [ ] Add structured logging.
- [ ] Add basic error handling.
- [ ] Add unit-test infrastructure.
- [ ] Add linting/formatting.
- [ ] Add CI.
- [ ] Add README with architecture and local setup.
- [ ] Add a minimal TUI that starts and exits cleanly.

Suggested initial structure:

```text
observer/
├── cmd/
│   └── observer/
├── internal/
│   ├── domain/
│   ├── trace/
│   ├── analysis/
│   ├── transport/
│   ├── tui/
│   └── config/
├── testdata/
├── go.mod
├── go.sum
├── README.md
└── AGENTS.md
```

Do not over-engineer the package structure if the implementation does not require it.

---

# Phase 2 — Trace domain model

Goal: create Observer's provider-independent internal representation.

Define concepts such as:

```text
Trace
Span
SpanID
TraceID
SpanKind
SpanStatus
SpanAttributes
Duration
Timestamp
Service
Operation
ParentSpan
ChildSpans
```

Tasks:

- [ ] Define domain structs.
- [ ] Define parent-child relationships.
- [ ] Define span duration calculation.
- [ ] Define trace validation.
- [ ] Define deterministic ordering.
- [ ] Write unit tests.
- [ ] Create representative test traces.

The domain model must not depend on the TUI.

The domain model must not require OTLP types.

---

# Phase 3 — OTLP ingestion

Goal: consume OpenTelemetry traces.

Start with one source:

> OTLP over HTTP.

Tasks:

- [ ] Understand OTLP.
- [ ] Connect to an OpenTelemetry Collector.
- [ ] Receive/query traces.
- [ ] Decode trace data.
- [ ] Convert OTLP structures into Observer's domain model.
- [ ] Preserve timestamps.
- [ ] Preserve parent-child relationships.
- [ ] Preserve useful attributes.
- [ ] Handle malformed traces.
- [ ] Add integration tests.
- [ ] Add local Collector configuration for development.

Acceptance criterion:

Observer can receive a real trace and convert it into the internal model without the TUI knowing anything about OTLP.

---

# Phase 4 — Trace tree and critical path

Goal: answer the first useful question:

> What happened during this request?

Tasks:

- [ ] Build the span tree.
- [ ] Calculate inclusive duration.
- [ ] Calculate self duration.
- [ ] Identify leaf spans.
- [ ] Identify overlapping spans.
- [ ] Calculate the request critical path.
- [ ] Detect sequential operations.
- [ ] Detect parallel operations.
- [ ] Calculate service-level contribution.
- [ ] Calculate operation-level contribution.

Example:

```text
HTTP request       1200ms
├── Redis              5ms
├── PostgreSQL       700ms
└── Recommendation    400ms
```

Observer should be able to say:

```text
PostgreSQL: 58.3%
Recommendation: 33.3%
Redis: 0.4%
```

Be careful with overlapping spans. Do not simply sum child durations and call that total latency.

---

# Phase 5 — Latency diagnosis engine

Goal: turn trace data into useful findings.

Create a deterministic analyzer.

Possible findings:

- [ ] Dominant span
- [ ] Slow database operation
- [ ] Slow downstream service
- [ ] Retry detected
- [ ] Timeout detected
- [ ] Connection-pool wait
- [ ] Queue wait
- [ ] Lock/contention wait
- [ ] Excessive fan-out
- [ ] Sequential independent calls
- [ ] Error before latency spike
- [ ] Large serialization/deserialization cost
- [ ] Cascading downstream latency

Each finding should contain:

```text
type
severity
confidence
evidence
affected spans
human-readable explanation
```

Example:

```text
Finding:
DATABASE_LATENCY

Severity:
HIGH

Evidence:
PostgreSQL span consumed 921ms of a 1277ms request.

Conclusion:
PostgreSQL is the dominant latency contributor.
```

Do not generate explanations without evidence.

---

# Phase 6 — CLI request capture

Goal:

```bash
observer http https://api.example.com/users/123
```

Tasks:

- [ ] Implement HTTP request execution.
- [ ] Add request headers.
- [ ] Add method selection.
- [ ] Add request body support.
- [ ] Add timeout configuration.
- [ ] Capture DNS timing where possible.
- [ ] Capture connection timing where possible.
- [ ] Capture TLS timing where possible.
- [ ] Capture response timing.
- [ ] Propagate W3C trace context where appropriate.
- [ ] Correlate the outgoing request with the resulting trace.

The CLI should eventually be able to produce:

```text
DNS       4ms
TCP      12ms
TLS      31ms
Server  842ms
Total   898ms
```

---

# Phase 7 — First TUI

Goal: make the core debugging workflow usable.

Screens:

## Overview

```text
Request
Total latency
Critical path
Dominant contributor
Diagnosis
```

## Trace tree

```text
HTTP
├── Auth
├── Redis
├── PostgreSQL
└── Recommendation API
```

## Timeline

Display spans horizontally against time.

## Span details

Show:

- service
- operation
- duration
- start time
- attributes
- status
- errors
- parent
- children

Keyboard navigation:

VIM like keybindings are preferred.

```text
hjkl       navigate
ENTER/SPACE     inspect
s         logs/details
t         timeline
/         search
ESC/Backspace       back
q         quit
```

Do not optimize visual polish before the interaction model works.

---

# Phase 8 — "Why?" command

Goal:

```bash
observer why <trace-id>
```

Output should prioritize diagnosis.

Example:

```text
Latency: 1.42s

Root finding:
PostgreSQL consumed 73% of request latency.

Evidence:
- PostgreSQL: 1032ms
- Total request: 1412ms
- Same endpoint baseline: 180ms

Next:
Press ENTER to inspect PostgreSQL.
```

This is the core product experience.

---

# Phase 9 — Historical comparison

Goal:

Compare a slow request with a healthy request.

Tasks:

- [ ] Store or retrieve previous traces.
- [ ] Identify comparable requests.
- [ ] Compare span structure.
- [ ] Compare durations.
- [ ] Detect new spans.
- [ ] Detect removed spans.
- [ ] Detect latency regressions.
- [ ] Show before/after timeline.

Example:

```text
PostgreSQL
Healthy: 18ms
Slow:   921ms

Regression: +903ms
```

---

# Phase 10 — Flakiness and patterns

Goal: identify recurring behavior.

Tasks:

- [ ] Track span duration history.
- [ ] Calculate failure frequency.
- [ ] Detect flaky operations.
- [ ] Detect intermittent timeouts.
- [ ] Detect recurring downstream latency.
- [ ] Establish simple baselines.
- [ ] Avoid pretending statistical correlation is causation.

Example:

```text
TestPaymentTimeout

Failures:
4 / 30

Failure rate:
13.3%

Pattern:
Failures correlate with requests > 5s.
```

---

# Phase 11 — Production integrations

Only after the local workflow is solid.

Potential backends:

- [ ] OpenTelemetry Collector
- [ ] Jaeger
- [ ] Grafana Tempo
- [ ] Other OTLP-compatible backends

Potential cloud integrations can come later.

Keep the analysis engine independent of the backend.

---

# Phase 12 — AI-assisted diagnosis

AI is optional and should be built last.

Input:

```text
trace
findings
historical comparison
relevant logs
metadata
```

Output:

```text
Likely cause
Evidence
Confidence
Suggested investigation
```

The LLM must not receive arbitrary secrets by default.

AI must not replace deterministic analysis.

It should summarize evidence and connect observations.

---

# Phase 13 — Performance and hardening

Tasks:

- [ ] Benchmark large traces.
- [ ] Test traces with 10k+ spans.
- [ ] Profile memory.
- [ ] Profile CPU.
- [ ] Handle malformed traces.
- [ ] Handle disconnected collectors.
- [ ] Handle slow collectors.
- [ ] Handle missing parent spans.
- [ ] Handle incomplete traces.
- [ ] Handle clock skew.
- [ ] Handle concurrent traces.
- [ ] Add cancellation with context.Context.
- [ ] Add graceful shutdown.
- [ ] Ensure sensitive data is not accidentally printed.

---

# Phase 14 — Release

Tasks:

- [ ] Cross compile.
- [ ] Linux binaries.
- [ ] macOS binaries.
- [ ] Windows support if practical.
- [ ] Shell completion.
- [ ] Man page.
- [ ] Configuration documentation.
- [ ] Example OpenTelemetry setup.
- [ ] Example Docker Compose environment.
- [ ] Demo recording.
- [ ] GitHub release.
- [ ] Versioning policy.

---

# Definition of Done for v1

Observer v1 should let a developer:

```bash
observer http http://localhost:8080/users/123
```

and get:

1. The request latency.
2. DNS/TCP/TLS timing where available.
3. The distributed trace.
4. A span tree.
5. A timeline.
6. Critical-path analysis.
7. Latency contribution by service.
8. A deterministic diagnosis.
9. Drill-down into the suspicious span.
10. Useful evidence for why that span is suspicious.

The primary success metric is:

> Can a developer go from "this endpoint is slow" to "this is where the latency went" in under a minute?

---

# Rules for OpenCode

When working on Observer:

1. Read `AGENTS.md` before modifying code.
2. Determine the current phase before implementing a feature.
3. Do not implement future-phase functionality unless explicitly requested.
4. Prefer small, independently testable changes.
5. Run tests after meaningful changes.
6. Do not introduce dependencies without explaining why.
7. Preserve the domain/transport/TUI separation.
8. Do not put business logic inside TUI rendering code.
9. Do not put OTLP-specific types into the domain layer.
10. Do not add AI to the diagnosis engine.
11. When requirements are ambiguous, inspect the existing architecture before making assumptions.
12. When a design decision has significant architectural consequences, explain the trade-off before implementing it.

## OpenCode task format

For each task:

```text
1. Identify the current phase.
2. Inspect the existing implementation.
3. State the intended change.
4. Implement the smallest complete change.
5. Add/update tests.
6. Run formatting and tests.
7. Report:
   - files changed
   - behavior added
   - tests run
   - remaining concerns
```

## Important

Observer is a debugging tool.

Correctness and trustworthy diagnosis are more important than feature count.

Never claim a root cause when the available evidence only supports correlation.
