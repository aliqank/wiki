# GET /bookings/{id}/timeline

**Created:** 2026-06-05 05:15  
**Last updated:** 2026-06-05 05:15  
**Author:** Telman Nurzhanov (SA)

---

## Method Card

| Parameter | Value |
|---|---|
| Description | Return a unified audit timeline for a booking item |
| Authenticated users only | `+` |
| System module | `Booking / Common` |
| Endpoint URL | `/api/booking/v1/bookings/{id}/timeline` |
| Request method | `GET` |
| Publication target | `Prepared result in source/results; not published to wiki` |

---

## 1. Change Scope

New dev-ready version of the method.

Purpose of this revision:
- keep the timeline as a separate, richer projection than status history;
- combine lifecycle status changes, approval decisions, and selected booking execution milestones;
- make the timeline suitable for detail view, audit view, and expandable UI activity panels.

---

## 2. Functional Requirements

| Project | Requirement ID | Requirement text | Status | Source | Comment |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-NEW-37 | Audit trail and history on demand | Confirmed | BRD v13 | Direct coverage |
| TCO Booking Tool | FR-077 | System notifies FO when request submitted | Confirmed | BRD v13 | Timeline must reflect submission status change |
| TCO Booking Tool | FR-079 | System notifies Requestor/SWP of FO decision | Confirmed | BRD v13 | Timeline must reflect approval decisions |
| TCO Booking Tool | FR-NEW-75 | System notifies FleetOwners' Supervisor when Long-term rented booking reaches Confirmed by FO | Confirmed | BRD v13 | Timeline must reflect multi-step approval chain |
| TCO Booking Tool | FR-NEW-76 | System notifies Requestor and FO when Supervisor makes a decision | Confirmed | BRD v13 | Timeline must show supervisor decision event |

---

## 3. Method Logic

1. Validate that booking `id` is a valid UUID.
2. Load booking base record from `Bookings`.
3. Check access rights for the current user.
4. Read lifecycle history from `BookingStatuses`.
5. Read approval decision history from `BookingApprovals`.
6. Read actor data from `Users` for both lifecycle and approval records.
7. Read current and terminal semantic references from `ref_booking_status`, `ref_booking_closure_reason`, `ref_booking_approval_type`, `ref_booking_approval_status`.
8. Build a unified ordered event collection.
9. Add derived execution milestone events from `Bookings.actualStartDateTime` and `Bookings.actualEndDateTime` only when they add information not already represented by an identical lifecycle event timestamp.
10. Sort by `occurredAt ASC`, then by `eventOrder ASC`, then by `id ASC`.

Entities:
- read: `Bookings`
- read: `BookingStatuses`
- read: `BookingApprovals`
- read: `Users`
- read: `ref_booking_status`
- read: `ref_booking_closure_reason`
- read: `ref_booking_approval_type`
- read: `ref_booking_approval_status`

Event composition rules:
- `StatusChanged` events are built from `BookingStatuses` only.
- `ApprovalRecorded` events are built from `BookingApprovals` only.
- `ExecutionMilestone` events are derived from booking factual timestamps and must not replace lifecycle or approval events.
- One timeline event must have exactly one `eventType`.
- The timeline is a UI/audit projection, not a source-of-truth table.

Derived milestone rules:
- Add `ExecutionStarted` when `actualStartDateTime` is not null and no `StatusChanged` event with status `InProgress` already exists with the same timestamp.
- Add `ExecutionFinished` when `actualEndDateTime` is not null and no `StatusChanged` event with status `Closed` already exists with the same timestamp.

Recommended `eventOrder` for identical timestamps:
- `10` = `ApprovalRecorded`
- `20` = `StatusChanged`
- `30` = `ExecutionMilestone`

---

## 4. Access Permissions

| Permission | Description |
|---|---|
| `Requestor` | Can view timeline of own booking |
| `ServiceWorkProcessor` | Can view timeline of bookings belonging to accessible Service Work Requests |
| `FleetOwner` | Can view timeline of bookings from own fleets |
| `FleetOwnersSupervisor` | Can view timeline of long-term rented bookings available in supervisor flow |
| `Admin` | Full access |

---

## 5. System Settings Used

| Name | Code | Value type | Description | Default |
|---|---|---|---|---|
| — | — | — | Not used | — |

---

## 6. Errors Returned

| Code | Description |
|---|---|
| `UNAUTHORIZED` | User is not authenticated |
| `FORBIDDEN` | User has no access to this booking timeline |
| `NOT_FOUND` | Booking not found |

HTTP codes:
- `401 Unauthorized`
- `403 Forbidden`
- `404 Not Found`

---

## 7. Method Parameters

| # | Description | Model field | Backend type | Required | Validation | Default | Location | Comment |
|---|---|---|---|---|---|---|---|---|
| 1 | Booking identifier | `id` | `uuid` | `+` | Must be a valid UUID and existing booking ID | — | Path param | |

---

## 8. Request Example

```http
GET /api/booking/v1/bookings/8c4c8b6d-7bc0-41fb-9038-422cf55d1111/timeline
Authorization: Bearer <token>
Content-Type: application/json
```

---

## 9. Returned Data

The method returns a collection inside the common result wrapper.

Reference templates:
- [`2026-05-05 - Шаблон обертки результата API.md`](2026-05-05%20-%20%D0%A8%D0%B0%D0%B1%D0%BB%D0%BE%D0%BD%20%D0%BE%D0%B1%D0%B5%D1%80%D1%82%D0%BA%D0%B8%20%D1%80%D0%B5%D0%B7%D1%83%D0%BB%D1%8C%D1%82%D0%B0%D1%82%D0%B0%20API.md)

### 9.1 Result wrapper

| # | Description | Model field | Backend type | Format | Default | Data source | Comment |
|---|---|---|---|---|---|---|---|
| 1 | Method result | `value` | `array<object>` | `object[]` | — | backend timeline aggregation | Ordered timeline collection |
| 2 | Success flag | `isSuccess` | `bool` | `boolean` | — | backend | |
| 3 | Errors | `errors` | `array<object>` | `object[]` | `[]` | backend | |

### 9.2 Structure of `value[]`

| # | Description | Model field | Backend type | Format | Default | Data source | Comment |
|---|---|---|---|---|---|---|---|
| 1 | Timeline event ID | `id` | `uuid \| string` | `UUID v4 \| string` | — | source entity ID or backend-generated synthetic ID | Synthetic ID allowed for derived milestone events |
| 2 | Event type | `eventType` | `string` | `string` | — | backend composition | Allowed values: `StatusChanged`, `ApprovalRecorded`, `ExecutionMilestone` |
| 3 | Event code | `eventCode` | `string` | `string` | — | backend composition | Example: `BookingSubmitted`, `FoApprovalApproved`, `ExecutionStarted` |
| 4 | Event timestamp | `occurredAt` | `datetime` | `ISO 8601` | — | backend aggregation | Main timeline sort field |
| 5 | Event title | `title` | `string` | `string` | — | backend composition | Short UI-ready caption |
| 6 | Event description | `description` | `string \| null` | `string \| null` | `null` | backend composition | Expanded explanation for UI details drawer |
| 7 | Actor block | `actor` | `object` | `object` | — | backend composition from audit fields + `Users` | Never omitted |
| 7.1 | Actor identifier | `actor.userId` | `uuid \| null` | `UUID v4 \| null` | `null` | audit source field | |
| 7.2 | Actor display name | `actor.displayName` | `string` | `string` | — | `Users.fullName` or backend fallback | |
| 7.3 | Actor email | `actor.email` | `string \| null` | `email \| null` | `null` | `Users.email` | |
| 7.4 | Actor type | `actor.actorType` | `string` | `string` | — | backend composition | Allowed values: `User`, `System`, `Unknown` |
| 8 | Status payload | `status` | `object \| null` | `object \| null` | `null` | status-based events only | Used for lifecycle interpretation |
| 8.1 | Lifecycle status code | `status.code` | `string` | `string` | — | `ref_booking_status` | Present for `StatusChanged` events |
| 8.2 | Lifecycle status label | `status.label` | `string` | `string` | — | `ref_booking_status` | |
| 8.3 | Closure reason code | `status.closureReason` | `string \| null` | `string \| null` | `null` | `ref_booking_closure_reason` | Only for `Closed` status |
| 8.4 | Closure reason label | `status.closureReasonLabel` | `string \| null` | `string \| null` | `null` | `ref_booking_closure_reason` | Only for `Closed` status |
| 9 | Approval payload | `approval` | `object \| null` | `object \| null` | `null` | approval-based events only | Used for decision audit |
| 9.1 | Approval type code | `approval.type` | `string` | `string` | — | `ref_booking_approval_type` | Example: `FoApproval`, `SupervisorApproval` |
| 9.2 | Approval type label | `approval.typeLabel` | `string` | `string` | — | `ref_booking_approval_type` | |
| 9.3 | Approval result code | `approval.status` | `string` | `string` | — | `ref_booking_approval_status` | Example: `Approved`, `Declined` |
| 9.4 | Approval result label | `approval.statusLabel` | `string` | `string` | — | `ref_booking_approval_status` | |
| 9.5 | Approval order | `approval.order` | `int` | `int` | — | `BookingApprovals.approvalOrder` | Position in approval chain |
| 10 | Comment | `comment` | `string \| null` | `string \| null` | `null` | source record comment or backend-generated milestone text | Optional |

### 9.3 Response Example

```json
{
  "value": [
    {
      "id": "11111111-2222-3333-4444-555555550001",
      "eventType": "StatusChanged",
      "eventCode": "BookingSubmitted",
      "occurredAt": "2026-05-14T10:00:00Z",
      "title": "Booking submitted",
      "description": "Booking entered the approval flow.",
      "actor": {
        "userId": "8f83f79c-3d25-4f07-a17b-9dc4b9f25001",
        "displayName": "John Smith",
        "email": "john.smith@tco.example",
        "actorType": "User"
      },
      "status": {
        "code": "Submitted",
        "label": "Submitted",
        "closureReason": null,
        "closureReasonLabel": null
      },
      "approval": null,
      "comment": null
    },
    {
      "id": "22222222-3333-4444-5555-666666660001",
      "eventType": "ApprovalRecorded",
      "eventCode": "FoApprovalApproved",
      "occurredAt": "2026-05-14T11:20:00Z",
      "title": "Fleet Owner approved booking",
      "description": "Approval step 1 was completed with positive decision.",
      "actor": {
        "userId": "d61f5b36-c1c1-40de-a37a-11d6d1898002",
        "displayName": "Aigerim Sarsenova",
        "email": "aigerim.sarsenova@tco.example",
        "actorType": "User"
      },
      "status": null,
      "approval": {
        "type": "FoApproval",
        "typeLabel": "Fleet Owner approval",
        "status": "Approved",
        "statusLabel": "Approved",
        "order": 1
      },
      "comment": "Approved for planned work window."
    },
    {
      "id": "11111111-2222-3333-4444-555555550002",
      "eventType": "StatusChanged",
      "eventCode": "BookingConfirmed",
      "occurredAt": "2026-05-14T11:20:00Z",
      "title": "Booking confirmed",
      "description": "All mandatory approvals were completed and booking became ready for execution.",
      "actor": {
        "userId": "d61f5b36-c1c1-40de-a37a-11d6d1898002",
        "displayName": "Aigerim Sarsenova",
        "email": "aigerim.sarsenova@tco.example",
        "actorType": "User"
      },
      "status": {
        "code": "Confirmed",
        "label": "Confirmed",
        "closureReason": null,
        "closureReasonLabel": null
      },
      "approval": null,
      "comment": null
    },
    {
      "id": "timeline-execution-start-8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
      "eventType": "ExecutionMilestone",
      "eventCode": "ExecutionStarted",
      "occurredAt": "2026-05-20T08:05:00Z",
      "title": "Execution started",
      "description": "Actual booking execution start was recorded.",
      "actor": {
        "userId": null,
        "displayName": "System",
        "email": null,
        "actorType": "System"
      },
      "status": null,
      "approval": null,
      "comment": null
    }
  ],
  "isSuccess": true,
  "errors": []
}
```

---

## 10. Notes for Backend and Frontend

Backend:
- status history remains the source of truth for lifecycle transitions;
- approval history remains the source of truth for approval decisions;
- timeline is an aggregated read model and may use synthetic IDs for derived milestone events.

Frontend:
- use timeline for vertical activity feed, expandable audit panel, or booking details event stream;
- use `GET /bookings/{id}/status-history` for a strict status table;
- do not infer lifecycle transitions from approval events alone.
