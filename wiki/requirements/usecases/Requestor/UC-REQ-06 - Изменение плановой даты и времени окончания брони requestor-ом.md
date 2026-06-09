# UC-REQ-06 - Изменение плановой даты и времени окончания брони requestor-ом

**Created:** 2026-06-02  
**Last updated:** 2026-06-03  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка use case

| Поле | Значение |
|---|---|
| Область действия | Страница `Мои заявки` / detail view заявки / карточка booking item |
| Участник | Пользователь с ролью [`Requestor`](../../Roles%20and%20Access%20Model.md) |
| Покрываемые FR (BRD) | `FR-061`, `FR-067` |
| Покрываемые FR (Additional list) | — |
| Предусловие | Пользователь авторизован; у пользователя есть доступ к родительской заявке через `BookingRequests.requestorId`; бронь находится в активном статусе `Submitted`, `Confirmed` или `InProgress`; пользователь изменяет только плановую дату и время окончания брони |
| Триггер | Пользователь меняет `plannedEndDateTime` в карточке активной брони |
| Ожидаемый результат | Запрос на изменение `plannedEndDateTime` принят системой; до `InProgress` бронь переходит/остается в `Submitted` и ожидает решения [`Fleet Owner`](../../Roles%20and%20Access%20Model.md); в `InProgress` бронь сохраняет статус `InProgress`, а изменение применяется только после отдельного FO approval |
| Используемые API | [`POST /bookings/{id}/extend`](../../../api/booking/requestor/POST_bookings_id_extend.md), [`GET /booking-requests/{id}`](../../../api/booking/requestor/GET_booking_requests_id.md), [`GET /booking-requests/my`](../../../api/booking/requestor/GET_booking_requests_my.md) |

---

## Основной сценарий

1. Пользователь открывает страницу `Мои заявки` или detail view заявки и выбирает активную бронь.
2. Пользователь изменяет `plannedEndDateTime` в карточке брони.
3. Frontend вызывает [`POST /bookings/{id}/extend`](../../../api/booking/requestor/POST_bookings_id_extend.md) и передает новое значение `plannedEndDateTime`.
4. Backend проверяет, что:
   - бронь существует;
   - родительская заявка имеет `BookingRequests.requestorId`, совпадающий с текущим пользователем;
   - новый `plannedEndDateTime` валиден;
   - отсутствуют [hard availability restrictions](../../../glossary/Glossary.md#hard-availability-restriction) на новый период;
   - наличие [booking conflict context](../../../glossary/Glossary.md#booking-conflict-context) само по себе не является автоматическим запретом.
5. Если текущий статус брони равен `Submitted` или `Confirmed`, backend:
   - обновляет `plannedEndDateTime`;
   - переводит бронь в `Submitted`;
   - создает запись в `BookingStatuses`;
   - инициирует повторное рассмотрение со стороны [`Fleet Owner`](../../Roles%20and%20Access%20Model.md).
6. Если текущий статус брони равен `InProgress`, backend:
   - не переводит бронь из `InProgress` обратно в `Submitted`;
   - создает отдельную запись [Booking Extension Approval](../../../glossary/Glossary.md#booking-extension-approval) в `BookingApprovals`;
   - не применяет новое `plannedEndDateTime` до решения [`Fleet Owner`](../../Roles%20and%20Access%20Model.md).
7. В обоих сценариях система уведомляет [`Fleet Owner`](../../Roles%20and%20Access%20Model.md) о запросе на изменение срока брони.
8. После положительного решения [`Fleet Owner`](../../Roles%20and%20Access%20Model.md):
   - если бронь была в pre-start состоянии, бронь переходит в `Confirmed`;
   - если бронь была в `InProgress`, сохраняется статус `InProgress`, а новое `plannedEndDateTime` применяется к брони.
9. Frontend обновляет карточку и показывает актуальное состояние брони после решения [`Fleet Owner`](../../Roles%20and%20Access%20Model.md).

---

## Альтернативные сценарии

1. Новый `plannedEndDateTime` невалиден.
   Backend возвращает `VALIDATION_ERROR`.

2. Новый период нарушает [hard availability restrictions](../../../glossary/Glossary.md#hard-availability-restriction).
   Backend возвращает `EQUIPMENT_NOT_AVAILABLE`.

3. По новой дате окончания есть [booking conflict context](../../../glossary/Glossary.md#booking-conflict-context).
   Запрос не должен отклоняться автоматически только из-за overlap по `Bookings`; решение принимает [`Fleet Owner`](../../Roles%20and%20Access%20Model.md).

4. Пользователь пытается изменить не свою бронь.
   Backend возвращает `FORBIDDEN`.

5. [`Fleet Owner`](../../Roles%20and%20Access%20Model.md) отклоняет запрос на изменение `plannedEndDateTime`.
   Для pre-start сценария бронь остается в прежнем согласованном состоянии и с прежней датой окончания. Для `InProgress` новое `plannedEndDateTime` не применяется.

---

## Замечания

1. Несмотря на имя текущего API `/extend`, сценарий описывает не только продление, но и любое изменение `plannedEndDateTime`, если это поддержано бизнес-правилами и контрактом метода.
2. Для статусов `Submitted` и `Confirmed` изменение `plannedEndDateTime` означает повторное ожидание решения [`Fleet Owner`](../../Roles%20and%20Access%20Model.md), поэтому бронь должна находиться в `Submitted` до принятия решения.
3. Для статуса `InProgress` бронь не должна возвращаться из `InProgress` в `Submitted`, так как фактическое исполнение уже началось.
4. Для `InProgress` сценария требуется отдельный [Booking Extension Approval](../../../glossary/Glossary.md#booking-extension-approval); до решения [`Fleet Owner`](../../Roles%20and%20Access%20Model.md) новое `plannedEndDateTime` не должно считаться примененным.
5. После положительного решения [`Fleet Owner`](../../Roles%20and%20Access%20Model.md) бронь должна иметь статус `Confirmed` для pre-start сценария и сохранять статус `InProgress` для сценария во время исполнения.
