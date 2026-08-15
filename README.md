# observer

Observer is a Go-based TUI for debugging latency in distributed HTTP requests.

The core question Observer answers is:

> Where did the latency go, and why?

Observer should consume OpenTelemetry traces and turn a single slow request into a concise, actionable diagnosis.

The phased roadmap and product spec live in [`docs/roadmap.md`](docs/roadmap.md).
