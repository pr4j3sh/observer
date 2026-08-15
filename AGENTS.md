# Observer — AGENTS.md

Observer is a Go TUI that consumes OpenTelemetry traces and turns a single slow HTTP request into a concise, evidence-based latency diagnosis.

> Where did the latency go, and why?

## Current state

- Greenfield: no Go code yet, no `go.mod`, no test/lint/CI infrastructure.
- Phase 1 (project skeleton) is the current phase and has not started.
- Full phased spec lives in `docs/roadmap.md`; README links to it.

## Phase map (detail: `docs/roadmap.md`)

- 0 Prerequisites / learning — knowledge only, no code.
- 1 Project skeleton — go.mod, CLI entry, package layout, config, logging, tests, lint, CI, minimal TUI.
- 2 Trace domain model — provider-independent structs, validation, deterministic ordering.
- 3 OTLP ingestion over HTTP — collector integration, decode into domain model.
- 4 Trace tree & critical path — inclusive/self duration, overlaps, service/operation contribution.
- 5 Latency diagnosis engine — deterministic findings with type/severity/confidence/evidence.
- 6 CLI request capture — `observer http <url>`, DNS/TCP/TLS timing, W3C propagation.
- 7 First TUI — overview, span tree, timeline, span details; vim keybindings.
- 8 `observer why <trace-id>` — root diagnosis; the core product experience.
- 9 Historical comparison — slow vs healthy trace, regression detection.
- 10 Flakiness & patterns — baselines, failure frequency.
- 11 Production integrations — Jaeger/Tempo/OTLP backends, after the local workflow is solid.
- 12 AI-assisted diagnosis — last and optional; summarizes deterministic findings only.
- 13 Performance & hardening — large traces, malformed input, clock skew.
- 14 Release — cross-compile, shell completion, docs, releases.

## Working rules

- Determine the current phase before implementing; do not build future-phase functionality unless explicitly requested.
- Prefer small, independently testable changes. Once a module exists, verify with `gofmt` + `go vet` + `go test ./...`. No CI is configured — do not assume one exists.
- Do not introduce an external dependency without explaining why. Prefer the standard library.
- Preserve layering: `internal/domain` must not import OTLP or TUI types; `internal/trace` (OTLP ingestion) and `internal/analysis` sit between them; no business logic inside TUI rendering code. Use interfaces at boundaries, not everywhere.
- No AI in the diagnosis engine. Diagnosis must stay deterministic.
- Never claim a root cause when the evidence only supports correlation. Correctness and trustworthy diagnosis beat feature count.
- Never expose secrets or full sensitive request payloads by default.

## Task workflow

For each task:

1. Identify the current phase.
2. Inspect the existing implementation.
3. State the intended change.
4. Implement the smallest complete change.
5. Add/update tests.
6. Run formatting and tests.
7. Report: files changed, behavior added, tests run, remaining concerns.
