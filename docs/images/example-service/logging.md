# Logging

This service uses [pino](https://github.com/pinojs/pino) for fast JSON structured logging and `pino-http` for HTTP request/response logging. Logs are emitted as single-line JSON for efficient ingestion by tooling (e.g. OpenTelemetry Collector tailing stdout).

## Objectives

- Machine‑parseable JSON only (no free‑form string concatenation)
- Consistent keys so downstream processors (search, alerting) remain stable
- Redaction of secrets/tokens
- Low overhead in production; readable in local dev
- Separation of concerns: one central logger module + lightweight middleware

## Source Files

- `src/logger.ts` – creates the base logger instance
- `src/example-service/example-http-server.ts` – mounts `pino-http` and uses child logger for lifecycle events

## Base Logger

- Pretty printing (color + human time) is enabled automatically when `NODE_ENV !== 'production'`.
- Log level defaults to `info` but can be overridden via `LOG_LEVEL`.
- `formatters.level` preserves the string level (e.g. `info`, `error`) instead of numeric codes, aiding readability and filtering.
- Redaction removes sensitive authorization data if accidentally logged.

## HTTP Request Logging

Mounted in `ExampleServer` after the health endpoint so `/v1/health` is excluded (ordering achieves suppression):

Behavior:

- Each request after mount gets a `req.log` logger instance.
- Responses are logged automatically at completion with latency & status (pino-http default fields).

## Child Loggers

- A module/component creates a contextual child logger:
- All emitted lines from this child will include `{ "module": "ExampleServer" }`. Prefer adding stable context keys once at creation rather than on every log event.

## Recommended Conventions

Key patterns to keep consistent:

- `module`: component/module name (already in place for server lifecycle)
- `msg`: human readable summary (pino’s first parameter when using object + message)
- `err`: error object
- `route`: application route handler identifier (add manually within handlers if useful)

## Log Levels

Use semantic levels to aid filtering:

- `trace`: Highly granular; normally disabled.
- `debug`: Developer diagnostics (branch decisions, payload sizes). Avoid in production unless needed.
- `info`: Normal lifecycle & successful operations (server start, connected to DB, request served).
- `warn`: Recoverable issues, slow operations, transient dependency hiccups.
- `error`: Failed operation impacting current request or background task.
- `fatal`: Process terminating state before exit.

Examples from the code:

```ts
this.log.info({ msg: 'Starting server' });
this.log.error({ err }, 'Failed to connect to MongoDB');
```

Prefer the pattern `logger.<level>(object, message)` so structured keys remain separate from the human message.

## Error Logging Guidelines

- Always pass the Error object as a field (`{ err }`) so pino captures `type`, `message`, `stack`.
- Use `warn` for expected validation or not-found scenarios (avoid polluting error rates).
- Attach classification when helpful: `{ err, errorType: 'MongoConnection', transient: true }`.

## Health Endpoint Strategy

Health checks are intentionally excluded from logging to reduce noise and cost. Keep the health route mounted before logging middleware. If additional noisy endpoints emerge (e.g. `/metrics`), mount them before `pino-http` or introduce `autoLogging: { ignore: ... }` (future enhancement).

## OpenTelemetry Interplay

- Tracing is initialized separately in `src/instrumentation.ts` (auto instrumentation). Logging code remains free of tracing concerns.
- Future enhancement: inject `traceId` / `spanId` if context is active via the OpenTelemetry API (`trace.getSpan(context.active())`). Add them as fields in child or request logger for correlation.

## Environment Variables

- `NODE_ENV`: controls pretty vs JSON-only output.
- `LOG_LEVEL`: overrides default info level.

## Style & Consistency Checklist

Before merging changes involving logging, verify:

- [ ] Sensitive headers/body segments excluded or redacted.
- [ ] Health endpoint remains unlogged.
- [ ] Errors include `err` object.
- [ ] Log level matches event severity.
- [ ] Message string is brief, descriptive; detailed data in structured fields.

## Sample Production Line (JSON)

```json
{
    "level": "info",
    "time": 1730720000000,
    "module": "ExampleServer",
    "msg": "Server running on port 3000"
}
```

## Testing Guidance

- Set `LOG_LEVEL=warn` during noisy integration tests to reduce output volume.
- For behavior verification, either parse stdout or (preferably) inject a custom pino transport in tests to capture emitted objects.
- Avoid snapshotting entire log lines with timestamps; focus on presence of required keys (`module`, `msg`, `level`).

```

```
