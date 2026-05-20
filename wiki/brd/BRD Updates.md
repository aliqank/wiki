# BRD Updates

**Created:** 2026-05-20  
**Last updated:** 2026-05-20  
**Автор документов:** Telman Nurzhanov (SA)

---

## Назначение

Файл фиксирует бизнес-уточнения и изменения правил, появившиеся после согласования и подписания `BRD.md`.

Правило проекта:

- `BRD.md` не редактируется после согласования;
- все новые договоренности, уточнения и отклонения от BRD фиксируются только в этом файле;
- при конфликте между `BRD.md` и этим файлом для дальнейшей аналитики, UI/API-документации и реализации используется более поздняя запись из этого файла.

---

## Updates Register

| ID | Date | Area | Type | Summary | Affects BRD |
|---|---|---|---|---|---|
| `BRD-U-001` | 2026-05-20 | Request lifecycle | Business rule clarification | Request terminal status after submit is `Closed`; `Cancelled` remains draft-only | `FR-026`, `FR-NEW-17`, `FR-NEW-44` |
| `BRD-U-002` | 2026-05-20 | Booking lifecycle | Business rule clarification | Booking gets explicit `Cancelled` status when parent draft request is cancelled before submit | `FR-027`, `FR-042`, `FR-NEW-17` |
| `BRD-U-003` | 2026-05-20 | Request lifecycle | Business rule clarification | Separate request-level withdraw flow allowed for `Submitted` request before first FO confirmation in the request | `FR-025`, `FR-058`, `FR-NEW-17` |

---

## BRD-U-001 - Request terminal status semantics

### Причина

Текущая трактовка `Completed` на уровне заявки воспринимается как успешное выполнение, хотя request-level статус в системе должен отражать завершение жизненного цикла заявки, а не бизнес-успешность каждой брони.

### Новое правило

Для request-level статусов используется следующий набор:

- `Draft`
- `Submitted`
- `InProgress`
- `Closed`
- `Cancelled`

### Семантика terminal statuses

- `Cancelled`
  Используется только для request-level отмены draft-заявки до submit.

- `Closed`
  Используется если заявка была отправлена и в ней больше не осталось активных броней. Причины завершения отдельных броней могут быть разными: `Closed`, `Revoked`, `Declined`, `Terminated`.

### Примеры

1. Все брони `Revoked` / `Declined` до старта.
   Итоговый request status = `Closed`.

2. Одна бронь `Closed`, другая `Terminated`.
   Итоговый request status = `Closed`.

3. Draft-заявка отменена пользователем.
   Итоговый request status = `Cancelled`.

### Правило приоритета

Если после submit больше нет активных item-ов, request status = `Closed`.

### Влияние на документацию и реализацию

Эта запись должна использоваться при обновлении:

- request status rules;
- API-документации list/history/report endpoints;
- UI-лейблов и фильтров по request status;
- backend-логики пересчета `BookingRequest.status`;
- модели reference-статусов заявки в БД и API.

---

## BRD-U-002 - Booking Cancelled for cancelled draft request

### Причина

Если draft-заявка отменяется, связанные draft booking item-ы не должны оставаться в статусе `Draft`, потому что это затрудняет анализ черновиков и аудит их конечного состояния.

### Новое правило

При отмене draft-заявки через `POST /booking-requests/{id}/cancel`:

- `BookingRequest.status` переводится в `Cancelled`;
- все связанные booking item-ы в статусе `Draft` переводятся в `Cancelled`;
- по каждому booking item создается запись в `BookingStatuses`.

### Семантика `Booking.Cancelled`

- `Booking.Cancelled` используется только для booking item-ов, отмененных вместе с родительской draft-заявкой до submit;
- `Booking.Cancelled` не используется для submitted booking item;
- после submit requestor-initiated cancel на уровне item по-прежнему выражается статусом `Revoked`.

### Разграничение с соседними статусами

- `Draft`
  Живой, еще не отмененный draft booking item.

- `Cancelled`
  Draft booking item, завершенный отменой родительской draft-заявки.

- `Revoked`
  Submitted booking item, отозванный requestor-ом до решения FO.

### Влияние на документацию и реализацию

Эта запись должна использоваться при обновлении:

- booking status reference list;
- API `POST /booking-requests/{id}/cancel`;
- request/booking history and audit trail;
- аналитики по draft lifecycle.

---

## BRD-U-003 - Withdraw submitted request before first FO confirmation

### Причина

Requestor должен иметь отдельный request-level сценарий для отзыва уже отправленной заявки целиком до того, как Fleet Owner подтвердил первую бронь из этой заявки.

### Новое правило

Разрешается отдельный use case `Отзыв заявки requestor-ом` для заявки в статусе `Submitted`, если ни один booking item этой заявки еще не был подтвержден Fleet Owner.

### Условия допустимости

- request status = `Submitted`;
- у заявки нет booking item-ов, которые уже перешли в `Confirmed`, `ConfirmedByFo`, `TransportConfirmed`, `InProgress`, `Closed` или `Terminated`;
- requestor отзывает заявку до первого FO confirmation по любому item-у.

### Ожидаемое поведение

- request-level действие отзывает все еще не обработанные booking item-ы заявки;
- такие item-ы переходят в `Revoked`;
- request status пересчитывается после массового revoke item-ов;
- после того как активных item-ов не остается, request переходит в `Closed`.

### Разграничение с соседними сценариями

- `UC-REQ-03` - отмена draft-заявки до submit;
- `UC-REQ-04` - отзыв отдельной брони requestor-ом;
- `UC-REQ-05` - отзыв всей submitted-заявки до первого FO confirmation.

### Влияние на документацию и реализацию

Эта запись должна использоваться при обновлении:

- use case request withdrawal;
- UI-кнопки request-level withdraw для submitted requests;
- API orchestration / dedicated endpoint для массового revoke item-ов в request.
