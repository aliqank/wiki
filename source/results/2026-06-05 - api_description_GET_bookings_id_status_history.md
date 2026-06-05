# GET /bookings/{id}/status-history

**Created:** 2026-06-05 05:15  
**Last updated:** 2026-06-05 05:15  
**Author:** Telman Nurzhanov (SA)

---

## Method Card

| Parameter | Value |
|---|---|
| Description | Return dev-ready lifecycle status history for a booking item |
| Authenticated users only | `+` |
| System module | `Booking / Common` |
| Endpoint URL | `/api/booking/v1/bookings/{id}/status-history` |
| Request method | `GET` |
| Publication target | `Prepared result in source/results; not published to wiki` |

---

## 1. Change Scope

New dev-ready version of the method.

Purpose of this revision:
- keep the method focused strictly on lifecycle status history;
- add actor information (`who changed the status`);
- add closure reason fields for terminal `Closed` records;
- make the response directly usable for UI table rendering without frontend hardcoding.

---

## 2. Functional Requirements

| Project | Requirement ID | Requirement text | Status | Source | Comment |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-NEW-37 | Audit trail and history on demand | Confirmed | BRD v13 | Direct coverage |
| TCO Booking Tool | FR-025 | Requestor can view request details and status | Confirmed | BRD v13 | History supports details page transparency |
| TCO Booking Tool | FR-042 | Status tracking for bookings | Confirmed | BRD v13 | Method exposes booking lifecycle transitions |

---

## 3. Method Logic

1. Validate that booking `id` is a valid UUID.
2. Load booking base record from `Bookings`.
3. Check access rights for the current user.
4. Read all `BookingStatuses` rows for the booking.
5. Join status reference data from `ref_booking_status`.
6. Left join closure reason reference data from `ref_booking_closure_reason`.
7. Left join actor data from `Users` by `BookingStatuses.createdBy`.
8. Sort records by `BookingStatuses.createdAt ASC`, then by `BookingStatuses.id ASC` for deterministic ordering.
9. Return one response item per actual lifecycle transition record.

Entities:
- read: [`Bookings`](../../wiki/db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL,%20Equipments,%20Booking%29.md) or published equivalent in `wiki/db/`
- read: `BookingStatuses`
- read: `ref_booking_status`
- read: `ref_booking_closure_reason`
- read: `Users`

Business rules:
- This method returns only lifecycle status history.
- Approval decisions from `BookingApprovals` must not be mixed into this method.
- Repeated statuses are allowed and must be returned if they were written as separate history records.
- If `status != Closed`, then `closureReason` and `closureReasonLabel` must be `null`.
- `changedAt` is the business-facing alias of `BookingStatuses.createdAt`.
- `changedBy` must be resolved from `BookingStatuses.createdBy`.
- If actor user record is not resolved, backend must still return the audit identifier and a fallback caption.

Fallback rules for actor resolution:
- if user is found in `Users`, return `userId`, `displayName`, `email`, `actorType = User`;
- if audit actor belongs to a technical account or service principal known to backend, return `actorType = System` and backend-generated display name;
- otherwise return `actorType = Unknown`, preserve `userId`, and set `displayName = Unknown user`.

---

## 4. Access Permissions

| Permission | Description |
|---|---|
| `Requestor` | Can view status history of own booking |
| `ServiceWorkProcessor` | Can view status history of bookings belonging to accessible Service Work Requests |
| `FleetOwner` | Can view status history of bookings from own fleets |
| `FleetOwnersSupervisor` | Can view status history of long-term rented bookings available in supervisor flow |
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
| `FORBIDDEN` | User has no access to this booking history |
| `NOT_FOUND` | Booking not found |
| `ACTOR_RESOLUTION_FAILED` | Non-blocking internal resolution problem; should not break response, backend uses fallback actor representation |

HTTP codes:
- `401 Unauthorized`
- `403 Forbidden`
- `404 Not Found`

Note:
- `ACTOR_RESOLUTION_FAILED` should normally be logged internally rather than returned as a blocking API error.

---

## 7. Method Parameters

| # | Description | Model field | Backend type | Required | Validation | Default | Location | Comment |
|---|---|---|---|---|---|---|---|---|
| 1 | Booking identifier | `id` | `uuid` | `+` | Must be a valid UUID and existing booking ID | — | Path param | |

---

## 8. Request Example

```http
GET /api/booking/v1/bookings/8c4c8b6d-7bc0-41fb-9038-422cf55d1111/status-history
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
| 1 | Method result | `value` | `array<object>` | `object[]` | — | backend aggregation | Collection of booking status history records |
| 2 | Success flag | `isSuccess` | `bool` | `boolean` | — | backend | |
| 3 | Errors | `errors` | `array<object>` | `object[]` | `[]` | backend | |

### 9.2 Structure of `value[]`

| # | Description | Model field | Backend type | Format | Default | Data source | Comment |
|---|---|---|---|---|---|---|---|
| 1 | History record ID | `id` | `uuid` | `UUID v4` | — | `BookingStatuses.id` | |
| 2 | Lifecycle status code | `status` | `string` | `string` | — | `BookingStatuses.statusId -> ref_booking_status.code` | Stable machine-readable value |
| 3 | Lifecycle status label | `statusLabel` | `string` | `string` | — | `ref_booking_status` | UI-facing caption |
| 4 | Closure reason code | `closureReason` | `string \| null` | `string \| null` | `null` | `BookingStatuses.closureReasonId -> ref_booking_closure_reason.code` | Returned only for `Closed` records |
| 5 | Closure reason label | `closureReasonLabel` | `string \| null` | `string \| null` | `null` | `ref_booking_closure_reason` | Returned only for `Closed` records |
| 6 | Status change timestamp | `changedAt` | `datetime` | `ISO 8601` | — | `BookingStatuses.createdAt` | Business-facing alias of `createdAt` |
| 7 | Status change comment | `comment` | `string \| null` | `string \| null` | `null` | `BookingStatuses.comment` | Optional comment stored on lifecycle change |
| 8 | Actor block | `changedBy` | `object` | `object` | — | backend composition from `BookingStatuses.createdBy` + `Users` | Never omitted |
| 8.1 | Actor identifier | `changedBy.userId` | `uuid \| null` | `UUID v4 \| null` | `null` | `BookingStatuses.createdBy` | Audit actor ID |
| 8.2 | Actor display name | `changedBy.displayName` | `string` | `string` | — | `Users.fullName` or backend fallback | Example: `John Smith`, `System`, `Unknown user` |
| 8.3 | Actor email | `changedBy.email` | `string \| null` | `email \| null` | `null` | `Users.email` | Optional if actor is resolved as user |
| 8.4 | Actor type | `changedBy.actorType` | `string` | `string` | — | backend composition | Allowed values: `User`, `System`, `Unknown` |

### 9.3 Response Example

```json
{
  "value": [
    {
      "id": "11111111-2222-3333-4444-555555550001",
      "status": "Draft",
      "statusLabel": "Draft",
      "closureReason": null,
      "closureReasonLabel": null,
      "changedAt": "2026-05-14T09:15:00Z",
      "comment": null,
      "changedBy": {
        "userId": "8f83f79c-3d25-4f07-a17b-9dc4b9f25001",
        "displayName": "John Smith",
        "email": "john.smith@tco.example",
        "actorType": "User"
      }
    },
    {
      "id": "11111111-2222-3333-4444-555555550002",
      "status": "Submitted",
      "statusLabel": "Submitted",
      "closureReason": null,
      "closureReasonLabel": null,
      "changedAt": "2026-05-14T10:00:00Z",
      "comment": "Submitted by Requestor",
      "changedBy": {
        "userId": "8f83f79c-3d25-4f07-a17b-9dc4b9f25001",
        "displayName": "John Smith",
        "email": "john.smith@tco.example",
        "actorType": "User"
      }
    },
    {
      "id": "11111111-2222-3333-4444-555555550003",
      "status": "Closed",
      "statusLabel": "Closed",
      "closureReason": "Completed",
      "closureReasonLabel": "Completed",
      "changedAt": "2026-05-20T17:45:00Z",
      "comment": "Booking closed manually by Fleet Owner",
      "changedBy": {
        "userId": "d61f5b36-c1c1-40de-a37a-11d6d1898002",
        "displayName": "Aigerim Sarsenova",
        "email": "aigerim.sarsenova@tco.example",
        "actorType": "User"
      }
    }
  ],
  "isSuccess": true,
  "errors": []
}
```

---

## 10. Notes for Backend and Frontend

Backend:
- do not collapse repeated status records;
- do not replace lifecycle history with approval history;
- preserve raw audit ordering from `BookingStatuses`.

Frontend:
- render `changedAt`, `statusLabel`, `changedBy.displayName` as the primary table columns;
- render `closureReasonLabel` only when `status = Closed`;
- render `comment` as optional secondary information or tooltip.
