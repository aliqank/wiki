# Aggregated Request Status Rules

**Created:** 2026-05-20  
**Last updated:** 2026-05-20  
**Автор документов:** Telman Nurzhanov (SA)

---

## Назначение

Документ фиксирует правила пересчета агрегированного статуса `BookingRequest.status` на основе статусов связанных `Bookings`.

Документ нужен как единая reference-точка для:

- Requestor UI;
- Fleet Owner UI;
- backend-логики пересчета `BookingRequests`;
- use case-сценариев, где изменяется статус отдельной брони.

---

## Источники

Правила ниже опираются на следующие материалы:

- `BRD: FR-026` - request statuses: `Draft`, `Submitted`, `InProgress`, `Completed`;
- `BRD: FR-027` - draft request can be edited / cancelled before submission;
- `BRD: FR-NEW-17` - aggregated request status is auto-calculated from RequestItem statuses;
- `BRD: FR-074` - request becomes `InProgress` when the first booking goes `InProgress`;
- `BRD: FR-NEW-44` - request becomes `Completed` when the last active booking is finished;
- `BRD Updates: BRD-U-001` - request terminal lifecycle status is `Closed`; pre-start closure is expressed via request closure reason `Cancelled`, post-start closure via `Completed`;
- [`POST /booking-requests/{id}/cancel`](../api/booking/requestor/POST_booking_requests_id_cancel.md) - request can move to `Closed` with closure reason `Cancelled` before submit;
- [`POST /bookings/{id}/revoke`](../api/booking/requestor/POST_bookings_id_revoke.md) - booking revoke triggers request status recalculation;
- [`POST /bookings/{id}/close`](../api/booking/requestor/POST_bookings_id_close.md) - if the last active booking is closed, request moves to `Closed`.

---

## Request Statuses

Request-level статусы:

- `Draft`
- `Submitted`
- `InProgress`
- `Closed`

Request-level closure reasons:

- `Cancelled`
- `Completed`

Booking-level статусы, влияющие на агрегирование:

- `Draft`
- `Submitted`
- `Confirmed`
- `InProgress`
- `Closed`

Terminal booking closure reasons:

- `Cancelled`
- `Declined`
- `Revoked`
- `Terminated`
- `Completed`

---

## Классификация booking statuses

### 1. Pre-submit

- `Draft`

### 2. Active after submit

- `Submitted`
- `Confirmed`
- `InProgress`

### 3. Terminal item status

- `Closed`

### 4. Terminal item closure reasons

- `Cancelled`
- `Declined`
- `Revoked`
- `Terminated`
- `Completed`

---

## Правила пересчета

При пересчете `BookingRequest.status` backend должен применять правила в следующем порядке приоритета.

### Rule 1. Closed / Cancelled

Если заявка была отменена через [`POST /booking-requests/{id}/cancel`](../api/booking/requestor/POST_booking_requests_id_cancel.md) до отправки, request status = `Closed`, request closure reason = `Cancelled`.

Комментарий:
- `Cancelled` больше не является отдельным request status или request closure reason.
- Для request-level отмены draft-заявки используется `Closed + closureReason = Cancelled`.
- При request-level cancel связанные draft booking item-ы могут быть удалены либо переведены в `Booking.Closed` с `closureReason = Cancelled`.

### Rule 2. Draft

Если заявка еще не была отправлена и все ее item-ы находятся в pre-submit состоянии, request status = `Draft`.

Практически это означает:
- request создан как draft;
- request-level submit еще не выполнялся;
- booking item-ы, если уже добавлены, остаются в `Draft`.

### Rule 3. InProgress

Если хотя бы одна бронь заявки имеет статус `InProgress`, request status = `InProgress`.

Комментарий:
- это прямое правило из `FR-074`;
- наличие других item-ов в `Submitted`, `Confirmed`, `Closed` и других состояниях не отменяет `InProgress`, пока есть хотя бы одна активная бронь в `InProgress`.

### Rule 4. Submitted

Если заявка уже отправлена, ни одна бронь еще не перешла в `InProgress`, и при этом существует хотя бы одна активная бронь, request status = `Submitted`.

К активным броням для этого правила относятся:

- `Submitted`
- `Confirmed`

Комментарий:
- request status `Submitted` является агрегированным статусом ожидания / согласования / подтвержденной, но еще не начавшейся работы;
- request-level статус не дублирует approval chain и сворачивает бронь в общее `Submitted`, пока не завершены все обязательные согласования либо пока работа не началась.

### Rule 5. Closed

Если заявка была отправлена, в ней больше не осталось активных броней и ни одна бронь этой заявки еще не переходила в `InProgress`, request status = `Closed`.

Это правило срабатывает, когда все booking item-ы заявки находятся в статусе `Closed`.

Комментарий:
- `Closed` является единым terminal request status после submit;
- в этом сценарии request closure reason = `Cancelled`;
- успешность или неуспешность отдельных броней определяется `Booking.closureReason`, а не отдельным request-level статусом.

### Rule 6. Completed

Если заявка уже хотя бы один раз была в `InProgress` и после этого в ней больше не осталось активных броней, request status = `Closed`, request closure reason = `Completed`.

Комментарий:
- `Completed` используется только для заявки, которая уже была в `InProgress`.
- сам lifecycle status заявки при этом остается `Closed`.

---

## Decision Table

| Условие по request / bookings | Итоговый `BookingRequest.status` |
|---|---|
| Draft request, submit еще не выполнялся | `Draft` |
| Draft request отменен через request-level cancel | `Closed` + `closureReason = Cancelled` |
| Есть хотя бы один `InProgress` item | `InProgress` |
| Нет `InProgress`, но есть хотя бы один active submitted/confirmed item | `Submitted` |
| Нет active item-ов после submit и request ни разу не был `InProgress` | `Closed` + `closureReason = Cancelled` |
| Нет active item-ов после того, как request уже был `InProgress` | `Closed` + `closureReason = Completed` |

---

## Примеры

### Example 1. Partial revoke before FO action

Bookings:

- `Closed` (`closureReason = Revoked`)
- `Submitted`

Итог: request status = `Submitted`.

Причина:
- одна бронь отозвана;
- вторая бронь остается активной.

### Example 2. All items revoked or declined before start

Bookings:

- `Closed` (`closureReason = Revoked`)
- `Closed` (`closureReason = Declined`)

Итог: request status = `Closed`, request closure reason = `Cancelled`.

Причина:
- после submit больше нет ни одной активной брони.

### Example 3. One booking started, another still waiting

Bookings:

- `InProgress`
- `Confirmed`

Итог: request status = `InProgress`.

### Example 4. Last active booking closed

Bookings:

- `Closed` (`closureReason = Completed`)
- `Closed` (`closureReason = Terminated`)

Итог: request status = `Closed`, request closure reason = `Completed`.

### Example 5. Confirmed but not started yet

Bookings:

- `Confirmed`
- `Submitted`

Итог: request status = `Submitted`.

---

## Implementation Notes

1. Пересчет request status должен выполняться в той же транзакции, что и изменение статуса отдельной брони или request-level cancel.
2. История request status должна записываться в `BookingRequestStatuses` только при фактическом изменении агрегированного статуса.
3. [`GET /booking-requests/my`](../api/booking/requestor/GET_booking_requests_my.md) показывает только [незавершенные заявки](../glossary/Glossary.md), поэтому terminal request status для list view ограничен `Closed`; различие между pre-start cancellation и post-start completion определяется через `closureReason`.
4. Для терминальной брони бизнес-причина должна определяться через `Booking.closureReason`, а не через отдельный lifecycle status.
5. Request-level статус не хранит специальные значения вроде `Revoked` или `Declined`; такие состояния существуют только на уровне booking closure reason.
6. Request-level `closureReason` intentionally coarse-grained и ограничен значениями `Cancelled` и `Completed`.
