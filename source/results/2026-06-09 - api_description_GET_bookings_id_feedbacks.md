# GET /bookings/{id}/feedbacks

**Created:** 2026-06-09 07:31  
**Last updated:** 2026-06-09 07:31  
**Author:** Telman Nurzhanov (SA)

---

## Method Card

| Parameter | Value |
|---|---|
| Description | Return feedback list for equipment in the context of a specific booking |
| Authenticated users only | `+` |
| System module | `Booking / Common` |
| Endpoint URL | `/api/booking/v1/bookings/{id}/feedbacks` |
| Request method | `GET` |
| Publication target | `Prepared result in source/results; not published to wiki` |

---

## 1. Change Scope

New dev-ready method.

Purpose of this method:
- provide read access to `EquipmentFeedbacks` for one конкретная бронь;
- support `Requestor` / `ServiceWorkProcessor` booking details UI;
- support `FleetOwner` approval / detail view UI;
- return author data, timestamp, and feedback text in a frontend-ready shape.

---

## 2. Functional Requirements

| Project | Requirement ID | Requirement text | Status | Source | Comment |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-022 | Requestor / SWP can submit feedback on equipment with confirmed booking | Confirmed | BRD v13 | Existing write-side requirement; this method is the read-side continuation |
| TCO Booking Tool | FR-025 | Requestor can view request details and status | Confirmed | BRD v13 | Feedback list is part of booking details transparency |
| TCO Booking Tool | FR-092 | Fleet Owner can work with request / booking details in approvals context | Confirmed | BRD v13 | FO needs read access to feedback for own fleet bookings |

---

## 3. Method Logic

1. Validate that booking `id` is a valid UUID.
2. Load booking base record from `Bookings`.
3. Check access rights for the current user.
4. Read all `EquipmentFeedbacks` rows for the booking.
5. Join actor data from `Users` by `EquipmentFeedbacks.createdBy`.
6. Join optional organizational context from `Departments` if frontend needs author department caption.
7. Sort feedback records in deterministic order.
8. Return one response item per stored feedback record.

Entities:
- read: `Bookings`
- read: `EquipmentFeedbacks`
- read: `Users`
- read: `Departments` (optional for author organization block)

Business rules:
- this method is booking-scoped, not equipment-scoped;
- method must return only feedback records where `EquipmentFeedbacks.bookingId = {id}`;
- feedback from other bookings of the same equipment must not be mixed into response;
- method is read-only and must not modify booking lifecycle or feedback data;
- backend should return author block even if some user details are partially unavailable;
- if no feedback exists for the booking, backend returns an empty collection, not an error.

Recommended sort order:
- default: `createdAt DESC`, then `id DESC` for UI lists where newest feedback is shown first.

Fallback rules for author resolution:
- if user is found in `Users`, return `userId`, `displayName`, `email`, and optional `department`;
- if user record is not resolved, backend must still return `userId` if available and fallback `displayName = Unknown user`.

---

## 4. Access Permissions

| Permission | Description |
|---|---|
| `Requestor` | Can view feedback list of own booking |
| `ServiceWorkProcessor` | Can view feedback list of bookings belonging to accessible Service Work Requests |
| `FleetOwner` | Can view feedback list of bookings from own fleets / fleets where user has Fleet Management Access |
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
| `FORBIDDEN` | User has no access to this booking feedback list |
| `NOT_FOUND` | Booking not found |
| `AUTHOR_RESOLUTION_FAILED` | Non-blocking internal author resolution problem; backend should still return fallback author representation |

HTTP codes:
- `401 Unauthorized`
- `403 Forbidden`
- `404 Not Found`

Note:
- `AUTHOR_RESOLUTION_FAILED` should normally be logged internally and should not break a successful response.

---

## 7. Method Parameters

| # | Description | Model field | Backend type | Required | Validation | Default | Location | Comment |
|---|---|---|---|---|---|---|---|---|
| 1 | Booking identifier | `id` | `uuid` | `+` | Must be a valid UUID and existing booking ID | — | Path param | |

---

## 8. Request Example

```http
GET /api/booking/v1/bookings/8c4c8b6d-7bc0-41fb-9038-422cf55d1111/feedbacks
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
| 1 | Method result | `value` | `array<object>` | `object[]` | — | backend aggregation | Collection of feedback records for one booking |
| 2 | Success flag | `isSuccess` | `bool` | `boolean` | — | backend | |
| 3 | Errors | `errors` | `array<object>` | `object[]` | `[]` | backend | |

### 9.2 Structure of `value[]`

| # | Description | Model field | Backend type | Format | Default | Data source | Comment |
|---|---|---|---|---|---|---|---|
| 1 | Feedback record ID | `id` | `uuid` | `UUID v4` | — | `EquipmentFeedbacks.id` | |
| 2 | Booking identifier | `bookingId` | `uuid` | `UUID v4` | — | `EquipmentFeedbacks.bookingId` | |
| 3 | Equipment identifier | `equipmentId` | `uuid` | `UUID v4` | — | `EquipmentFeedbacks.equipmentId` | |
| 4 | Feedback text | `feedback` | `string` | `string` | — | `EquipmentFeedbacks.feedback` | Main feedback content |
| 5 | Creation timestamp | `createdAt` | `datetime` | `ISO 8601` | — | `EquipmentFeedbacks.createdAt` | |
| 6 | Author block | `createdBy` | `object` | `object` | — | backend composition from `EquipmentFeedbacks.createdBy` + `Users` | Never omitted |
| 6.1 | Author identifier | `createdBy.userId` | `uuid \| null` | `UUID v4 \| null` | `null` | `EquipmentFeedbacks.createdBy` | |
| 6.2 | Author display name | `createdBy.displayName` | `string` | `string` | — | `Users.fullName` or backend fallback | |
| 6.3 | Author email | `createdBy.email` | `string \| null` | `email \| null` | `null` | `Users.email` | Optional |
| 6.4 | Author role caption | `createdBy.role` | `string \| null` | `string \| null` | `null` | backend composition from user roles / business context | Example: `Requestor`, `ServiceWorkProcessor` |
| 6.5 | Author department caption | `createdBy.department` | `string \| null` | `string \| null` | `null` | `Users.departmentId -> Departments.name` | Optional |

### 9.3 Response Example

```json
{
  "value": [
    {
      "id": "3e7e8f9a-a8e5-48d3-8ca2-a0cce5d40001",
      "bookingId": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
      "equipmentId": "c3b5af91-61f8-4bc0-bd88-d099d3e90001",
      "feedback": "Equipment was delivered on time and worked without issues.",
      "createdAt": "2026-05-22T17:30:00Z",
      "createdBy": {
        "userId": "8f83f79c-3d25-4f07-a17b-9dc4b9f25001",
        "displayName": "John Smith",
        "email": "john.smith@tco.example",
        "role": "Requestor",
        "department": "Maintenance Planning"
      }
    },
    {
      "id": "3e7e8f9a-a8e5-48d3-8ca2-a0cce5d40002",
      "bookingId": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
      "equipmentId": "c3b5af91-61f8-4bc0-bd88-d099d3e90001",
      "feedback": "Tracker data matched actual equipment usage during the booked period.",
      "createdAt": "2026-05-22T18:10:00Z",
      "createdBy": {
        "userId": "9d92a3f1-5f8b-4da7-bf12-123456789001",
        "displayName": "Aruzhan Bekenova",
        "email": "aruzhan.bekenova@tco.example",
        "role": "ServiceWorkProcessor",
        "department": "Service Operations"
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
- do not mix feedback from other bookings even if `equipmentId` is the same;
- keep this method read-only and independent from booking lifecycle transitions;
- if pagination is not implemented in Phase 1, return full list for one booking.

Frontend:
- primary columns / fields should be `createdBy.displayName`, `createdAt`, `feedback`;
- optional secondary fields: `createdBy.role`, `createdBy.email`, `createdBy.department`;
- render empty state when no feedback exists.

Open implementation note:
- if business later expects very large feedback collections per booking, method may evolve to paginated shape, but current recommended version is a plain collection inside result wrapper.
