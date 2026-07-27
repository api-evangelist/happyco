---
name: Subscribe to HappyCo inspection & report events
description: Consume HappyCo's gRPC server-streaming event services with the ack protocol.
api: grpc.happyco.com (happyco.inspect.inspection.v1.InspectionEventService + report.v1.ReportEventService)
operations: [OnInspectionStatusChanges, OnInspectionStatusChangesAck, OnReportsCreated, OnReportsCreatedAck]
---

# Subscribe to HappyCo events

HappyCo pushes changes through gRPC server-streaming rather than HTTP webhooks.

## Auth
HTTP Basic over gRPC to `grpc.happyco.com` (see `authentication/happyco-authentication.yml`).

## Steps
1. **Open the stream** — call `OnInspectionStatusChanges` (or `OnReportsCreated`)
   with an event request; HappyCo streams events as they occur. Each event is
   delivered to only one connected client at a time (round-robin), so you can run
   many workers.
2. **Process** — handle the `InspectionStatusChangesEvent` (or `ReportsCreatedEvent`).
3. **Acknowledge** — call `OnInspectionStatusChangesAck` (or `OnReportsCreatedAck`)
   with the `EventAck` (`event_ids`, `ack: true`). If you need more time, set
   `extend_timeout_seconds` instead of acking.

## Rules
- If an event is not acked before `ack_timeout_seconds`
  (`EventHandlerOptions`), it is redelivered — de-duplicate on the event `id`.
- Prefer large timeouts (10 min+) to absorb network travel time.
- See `asyncapi/happyco-events.yml` for the full event surface.
