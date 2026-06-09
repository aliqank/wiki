# UC-REQ-02.5 - Редактирование брони или замена техники в draft

**Created:** 2026-05-15  
**Last updated:** 2026-06-03  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка use case

| Поле | Значение |
|---|---|
| Область действия | Карточка booking item в draft-заявке |
| Участник | [`Requestor`](../../../Roles and Access Model.md) |
| Покрываемые FR (BRD) | `FR-027`, `FR-031`, `FR-038`, `FR-039`, `FR-040`, `FR-041`, `FR-NEW-04`, `FR-NEW-08`, `FR-NEW-71` |
| Покрываемые FR (Additional list) | — |
| Триггер | Пользователь редактирует период брони, justification или хочет заменить технику |
| Ожидаемый результат | Booking item обновлён без удаления всей draft-заявки |
| Используемые API | [`PATCH /booking-requests/{id}/items/{bookingId}`](../../../../api/booking/requestor/PATCH_booking_requests_id_items_bookingId.md), [`GET /equipment/search`](../../../../api/booking/requestor/GET_equipment_search.md) |

---

## Основной сценарий

1. Пользователь открывает booking item на редактирование.
2. Если требуется изменить только период брони, `justification` или `workCenterId`, frontend вызывает [`PATCH /booking-requests/{id}/items/{bookingId}`](../../../../api/booking/requestor/PATCH_booking_requests_id_items_bookingId.md).
3. Если пользователь редактирует inline-поле `justification`, frontend сохраняет его автоматически через [`PATCH /booking-requests/{id}/items/{bookingId}`](../../../../api/booking/requestor/PATCH_booking_requests_id_items_bookingId.md) без отдельной кнопки `Сохранить`.
4. Если требуется заменить технику, frontend повторно открывает окно выбора техники.
5. Frontend повторно вызывает [`GET /equipment/search`](../../../../api/booking/requestor/GET_equipment_search.md) с актуальным периодом и фильтрами.
6. Пользователь выбирает новую технику.
7. Frontend вызывает [`PATCH /booking-requests/{id}/items/{bookingId}`](../../../../api/booking/requestor/PATCH_booking_requests_id_items_bookingId.md) и передаёт обновлённые `equipmentId`, `plannedStartDateTime`, `plannedEndDateTime`, `justification`, `workCenterId`.
8. Backend обновляет существующий booking item и пересчитывает, требуется ли для него `justification`.

---

## UML Sequence Diagram

```mermaid
sequenceDiagram
    actor Requestor
    participant Frontend
    participant BookingAPI as Booking API
    participant BookingRequests
    participant Bookings

    Requestor->>Frontend: Редактирует booking item
    alt Изменение периода/justification/workCenter
        Frontend->>BookingAPI: PATCH /booking-requests/{id}/items/{bookingId}
    else Замена техники
        Frontend->>BookingAPI: GET /equipment/search
        BookingAPI-->>Frontend: Возвращает новую технику
        Frontend->>BookingAPI: PATCH /booking-requests/{id}/items/{bookingId}
    end
    BookingAPI->>BookingRequests: Проверяет draft request и доступ
    BookingAPI->>Bookings: Обновляет booking item
    BookingAPI-->>Frontend: Возвращает обновленное состояние item-а
    Frontend-->>Requestor: Показывает обновленные данные
```

---

## Альтернативные сценарии

1. При замене техники новая единица имеет конфликты с другими активными бронями на выбранный период.
   Backend сохраняет обновление item. Наличие [booking conflict context](../../../../glossary/Glossary.md#booking-conflict-context) не блокирует редактирование и должно быть отражено в UI как информационный признак для последующего решения [`Fleet Owner`](../../../Roles and Access Model.md).

2. Пользователь меняет период брони.
   Backend повторно валидирует итоговые значения item. [Booking conflict context](../../../../glossary/Glossary.md#booking-conflict-context) не блокирует сохранение, но должен быть рассчитан как active booking conflicts. [Hard availability restrictions](../../../../glossary/Glossary.md#hard-availability-restriction) по-прежнему блокируют обновление.

3. Для updated item требуется `justification`, но оно не заполнено.
   Item остаётся незавершённым до исправления.

4. Пользователь ввёл `justification`, но сразу не отправил заявку.
   Значение сохраняется автоматически через [`PATCH /booking-requests/{id}/items/{bookingId}`](../../../../api/booking/requestor/PATCH_booking_requests_id_items_bookingId.md) и остаётся в draft.
