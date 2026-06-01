**Created:** 2026-06-01 09:23  
**Last updated:** 2026-06-01 09:23  
**Author:** Telman Nurzhanov (SA)

---

# UC-FO-04 - Mobilization of Equipment

## Context

This use case describes the Fleet Owner action that is performed after the positive base confirmation scenario.

Relation to existing flow:

- previous use case: `UC-FO-03 - Подтверждение брони Fleet Owner (базовый сценарий)`;
- booking must already be in `Confirmed` status;
- this use case fixes the actual mobilization start and moves the booking to `InProgress`.

Sources used:

- `wiki/brd/BRD.md`;
- `wiki/requirements/usecases/Fleet Owner/UC-FO-03 - Подтверждение брони Fleet Owner (базовый сценарий).md`;
- `wiki/api/booking/fleet-owner/POST_approvals_bookings_id_mobilization_start.md`.

---

## Confirmed Facts

- BRD defines mobilization as the preparation / movement period before equipment arrives at the work site.
- Booking period starts from mobilization start, not from arrival.
- `FR-NEW-24`: Fleet Owner presses `Mobilization started` to record actual start time.
- The repository already contains a dedicated API method: `POST /approvals/bookings/{id}/mobilization-start`.
- API method logic already states:
  - action is allowed only for booking in `Confirmed` status;
  - `actualStartDateTime` is recorded;
  - booking status changes to `InProgress`.

---

## Use Case Card

| Field | Value |
|---|---|
| Scope | Fleet Owner page `Approvals` -> detail / action view of confirmed booking |
| Actor | User with role `FleetOwner` |
| Covered FR (BRD) | `FR-NEW-18`, `FR-NEW-24` |
| Covered FR (Additional) | — |
| Precondition | User is authenticated; user has role `FleetOwner`; booking belongs to fleet of current user; booking is already in `Confirmed` status as result of base positive confirmation flow; booking does not require additional approval step before start |
| Trigger | Fleet Owner presses button `Mobilization started` in confirmed booking card |
| Expected result | System records actual mobilization start time and booking moves to `InProgress` |
| Used API | `GET /approvals/bookings/{id}`, `POST /approvals/bookings/{id}/mobilization-start` |

---

## Main Scenario

1. Fleet Owner opens page `Approvals` and navigates to the detail / action view of a previously confirmed booking.
2. Frontend loads current booking data through `GET /approvals/bookings/{id}`.
3. System shows booking in status `Confirmed` and displays action `Mobilization started`.
4. Fleet Owner reviews booking context:
   - equipment;
   - request number;
   - Work Order;
   - planned period;
   - current booking status.
5. Fleet Owner presses `Mobilization started`.
6. Frontend calls `POST /approvals/bookings/{id}/mobilization-start`.
7. Backend checks that:
   - booking exists;
   - booking belongs to Fleet Owner responsibility scope;
   - booking current status is `Confirmed`;
   - booking has not already been started.
8. Backend sets `actualStartDateTime`:
   - from request body, if provided;
   - otherwise uses current backend time.
9. Backend changes booking status to `InProgress`.
10. Backend creates booking status history record in `BookingStatuses`.
11. Backend returns updated booking start data.
12. Frontend refreshes booking card and shows:
   - status `InProgress`;
   - recorded `actualStartDateTime`;
   - absence or disabled state of `Mobilization started` action for repeated execution.

---

## Alternative Scenarios

1. Booking is no longer in `Confirmed` status.
   Backend returns `BOOKING_NOT_STARTABLE`; frontend shows message and refreshes booking state.

2. Fleet Owner has no access to this booking.
   Backend returns `FORBIDDEN`.

3. Booking is not found.
   Backend returns `NOT_FOUND`.

4. Frontend does not pass `actualStartDateTime`.
   Backend uses current system time and still starts mobilization successfully.

5. User tries to execute the action again for already started booking.
   Backend must reject repeated start through `BOOKING_NOT_STARTABLE` or equivalent conflict behavior.

---

## Business Rules

1. Mobilization start is a manual action.
2. Booking lifecycle start is tied to mobilization start, not to arrival at work site.
3. Fleet Owner starts mobilization only after the booking has already reached `Confirmed`.
4. This use case is not for bookings still waiting for Supervisor approval.
5. Recording `actualStartDateTime` is mandatory at outcome level, but request body value is optional because backend may use current time.

---

## API Availability Check

### Confirmed Existing API

The API required for this use case already exists in repository documentation:

- `wiki/api/booking/fleet-owner/POST_approvals_bookings_id_mobilization_start.md`

Method summary:

- endpoint: `POST /api/booking/v1/approvals/bookings/{id}/mobilization-start`;
- permission: `FleetOwner`;
- allowed source status: `Confirmed`;
- effect: writes `actualStartDateTime`, changes booking status to `InProgress`, writes `BookingStatuses` record.

### Supporting Read API

For opening the booking card before action, the use case can rely on:

- `GET /approvals/bookings/{id}`

### Conclusion

API coverage for `UC-FO-04` is present.

No new dedicated write API is required for the basic mobilization-start scenario.

---

## Assumptions

- Button `Mobilization started` is shown in the same booking detail / action area that is used after `UC-FO-03`.
- Base scenario refers to TCO Owned or another flow where FO confirmation has already resulted in actual `Confirmed` status.

---

## Open Questions

1. Should UI allow Fleet Owner to manually edit the proposed `actualStartDateTime`, or should the action always use backend current time?
2. Should notification be sent to Requestor when mobilization starts?
3. Should the action be available for transport-related bookings separately, or only for the main booking entity?
