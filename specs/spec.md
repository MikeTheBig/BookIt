# Spec Template

Use this template for all feature specs. Add one file per feature under `/specs` and link the spec in the related issue/PR. Specs must be reviewed and approved before implementation.

---

- Title: <!-- Short, descriptive title -->
- Author: <!-- Name and contact -->
- Related issue(s): <!-- Issue numbers/links -->
- Date: <!-- YYYY-MM-DD -->

## Summary

One-paragraph summary of the feature and what it delivers.

## Motivation / Why

Why this feature is needed and what problem it solves. Include any metrics or user stories that justify it.

## Goals (Acceptance Criteria)

List clear, testable acceptance criteria. Example:

- [ ] Provider can create availability slots with start/end and optional timezone
- [ ] Customer can search available slots for a provider and book a slot
- [ ] Booking confirmation email is sent within 30s
- [ ] Cancellation flow updates availability and sends notification

## API Contract (Backend - required)

Define the endpoints, request and response shapes, status codes, and examples. This is the canonical contract the frontend will use.

Examples:

- POST /api/providers/{providerId}/availability
  - Request:
    ```json
    { "start": "2026-01-05T09:00:00Z", "end": "2026-01-05T10:00:00Z", "slotDurationMinutes": 30 }
    ```
  - Response 201 Created:
    ```json
    { "id": "uuid", "providerId": "uuid", "start": "...", "end": "..." }
    ```

- GET /api/providers/{providerId}/availability?from=2026-01-01&to=2026-01-31
  - Response 200 OK: array of available slots

- POST /api/bookings
  - Request: booking details (providerId, slotId, customer info, optional notes)
  - Response 201 Created: booking resource with status, confirmation token

Error responses:

- 400 Bad Request — validation errors (include field level messages)
- 401 Unauthorized — missing/invalid JWT
- 404 Not Found — provider or slot not found
- 409 Conflict — double-booking attempt

Security considerations:

- Required scopes/roles for each endpoint (e.g., provider-only, public)
- Rate limits if applicable

## Data Model Changes

List model changes and example entity definitions.

Example:

- ProviderAvailability
  - id: uuid
  - providerId: uuid
  - startUtc: datetime
  - endUtc: datetime
  - slotDurationMinutes: int

- Booking
  - id: uuid
  - providerId: uuid
  - slotStartUtc: datetime
  - slotEndUtc: datetime
  - customerName: string
  - customerEmail: string
  - status: enum (Booked, Cancelled, Completed)

## Database / Migrations

Describe required EF Core migrations and any non-trivial data migration steps. Include backwards-compatibility notes and a rollback strategy.

Example:

- Add `ProviderAvailability` table with indexes on `providerId` and `startUtc`.
- Add `Booking` table with unique constraint on (`providerId`, `slotStartUtc`) to help prevent double-booking.

Migration checklist:

- [ ] Add EF Core migration file and commit
- [ ] Run local migration and verify schema
- [ ] Add integration test(s) that run migrations against a disposable SQLite DB

## UI / UX Notes

Screens, flows, and acceptance criteria for the frontend. Include wireframes or links to designs if available.

Key behaviors:

- How slots are shown (time zone handling)
- Booking confirmation and error states
- Reschedule and cancellation flows

## Tests Required

Unit tests:

- Service layer: availability calculations, conflict detection, timezone handling.
- Repository layer: CRUD + constraints using in-memory/SQLite DB mocks.

Integration tests:

- End-to-end booking flow against a disposable SQLite instance.
- Migration test to ensure new migration applies cleanly.

Test acceptance:

- Business logic coverage ≥ 80% for modules changed.

## Rollout and Backward Compatibility

If the change is breaking, include rollout steps, feature flags, and communication plan.

Example:

- Deploy backend to staging
- Run migration job (no downtime expected)
- Enable feature flag for 5% of users, monitor metrics, then ramp

## Security & Privacy

Note any PII collected and retention policy. Include GDPR/consent considerations and encryption requirements.

## Observability & Metrics

List the key metrics and logs to emit (e.g., bookings_created_count, booking_latency_ms, booking_failures_total).

## Open Questions

List any unresolved questions and possible alternatives.

## Alternatives Considered

Short list of alternatives and why they were not chosen.

## Approval

- Reviewers:
- Approval date:

---

## Implementation Checklist (to include in PR)

- [ ] Link to this spec in the PR description
- [ ] Unit tests added and passing
- [ ] Integration tests added and passing
- [ ] EF Core migrations included (if applicable)
- [ ] Swagger/OpenAPI updated for endpoints
- [ ] No hard-coded secrets
- [ ] Self-review checklist completed
