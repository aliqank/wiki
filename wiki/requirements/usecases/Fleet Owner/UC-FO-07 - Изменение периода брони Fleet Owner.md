# UC-FO-07 - Изменение периода брони [`Fleet Owner`](../../Roles%20and%20Access%20Model.md)

**Created:** 2026-06-02  
**Last updated:** 2026-06-08  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка use case

| Поле | Значение |
|---|---|
| Область действия | Страница [`Fleet Owner`](../../Roles%20and%20Access%20Model.md) `Approvals` -> detail / action view конкретной брони |
| Участник | Пользователь с ролью [`FleetOwner`](../../Roles%20and%20Access%20Model.md) |
| Покрываемые FR (BRD) | `FR-048`, `FR-079a`, `FR-NEW-19` |
| Покрываемые FR (Additional list) | — |
| Предусловие | Пользователь авторизован в системе; пользователь имеет роль [`FleetOwner`](../../Roles%20and%20Access%20Model.md); бронь относится к fleet-у, по которому у пользователя есть [Fleet Management Access](../../../glossary/Glossary.md#fleet-management-access); бронь находится в lifecycle состоянии, в котором изменение периода еще допустимо: `Submitted`, `Confirmed` или `InProgress`; для `InProgress` разрешено изменять только `plannedEndDateTime`, а `plannedStartDateTime` больше не редактируется |
| Триггер | Нажатие кнопки `Change period` в карточке брони |
| Ожидаемый результат | Плановый период брони изменен [`Fleet Owner`](../../Roles%20and%20Access%20Model.md)-ом; бронь сохраняет допустимый lifecycle status; [`Requestor`](../../Roles%20and%20Access%20Model.md) получает уведомление об изменении периода |
| Используемые API | [`GET /approvals/bookings/{id}`](../../../api/booking/fleet-owner/GET_approvals_bookings_id.md), [`POST /approvals/bookings/{id}/change-period`](../../../api/booking/fleet-owner/POST_approvals_bookings_id_change_period.md) |

---

## Основной сценарий

1. [`Fleet Owner`](../../Roles%20and%20Access%20Model.md) открывает страницу `Approvals` и переходит в detail / action view нужной брони.
2. Frontend загружает актуальные данные брони через [`GET /approvals/bookings/{id}`](../../../api/booking/fleet-owner/GET_approvals_bookings_id.md).
3. Пользователь проверяет контекст брони:
   - requestor;
   - Work Order и приоритет;
   - технику;
   - текущий плановый период;
   - текущий lifecycle status.
4. Пользователь нажимает кнопку `Change period`.
5. Frontend открывает форму редактирования периода.
6. Пользователь вводит новые значения периода:
   - для `Submitted` / `Confirmed` разрешено изменять `plannedStartDateTime` и `plannedEndDateTime`;
   - для `InProgress` разрешено изменять только `plannedEndDateTime`.
7. Frontend вызывает [`POST /approvals/bookings/{id}/change-period`](../../../api/booking/fleet-owner/POST_approvals_bookings_id_change_period.md).
8. Backend проверяет, что:
   - бронь существует;
   - бронь относится к зоне ответственности текущего [`Fleet Owner`](../../Roles%20and%20Access%20Model.md);
   - текущий статус брони равен `Submitted`, `Confirmed` или `InProgress`;
   - если статус равен `InProgress`, backend не принимает изменение `plannedStartDateTime` и разрешает изменять только `plannedEndDateTime`;
   - новый диапазон дат валиден: для `Submitted` / `Confirmed` выполняется `plannedStartDateTime < plannedEndDateTime`; для `InProgress` новое `plannedEndDateTime` должно оставаться больше фактического или уже зафиксированного начала выполнения и не нарушать системные ограничения горизонта / длительности;
   - [hard availability restrictions](../../../glossary/Glossary.md#hard-availability-restriction) не нарушены.
9. Backend отдельно рассчитывает пересечения с другими активными бронями как [booking conflict context](../../../glossary/Glossary.md#booking-conflict-context), но сами по себе такие пересечения не блокируют изменение периода.
10. Для backend-валидации различаются два класса ограничений:
   - [hard availability restriction](../../../glossary/Glossary.md#hard-availability-restriction): техника фактически недоступна независимо от competing bookings; такое изменение должно быть отклонено;
   - [booking conflict context](../../../glossary/Glossary.md#booking-conflict-context): на новый период уже существуют другие активные брони той же техники; это должно быть показано [`Fleet Owner`](../../Roles%20and%20Access%20Model.md)-у как контекст для решения, но не является автоматическим запретом.
11. Примеры hard-ограничений на уровне таблиц:
   - в `EquipmentStates` есть актуальная запись со статусом `Decommissioned`, `Frozen` или `InRepair`, покрывающая новый диапазон;
   - в `Equipments` / reference-атрибутах техника относится к типу, который не допускается для данного сценария использования;
   - для связанного `equipmentId` действуют ограничения доступности, которые backend трактует как абсолютный запрет на бронирование независимо от конкурирующих заявок.
12. Примеры conflict context на уровне таблиц:
   - в `Bookings` уже есть другая запись по тому же `equipmentId` со статусом `Submitted`, `Confirmed` или `InProgress`, и ее диапазон пересекается с новым периодом;
   - в заявке может существовать несколько competing bookings той же техники, но пока это только overlap по данным `Bookings`, а не hard-stop по `EquipmentStates`.
13. Backend обновляет `plannedStartDateTime` и `plannedEndDateTime`.
14. Backend создает запись в `BookingStatuses` с комментарием об изменении периода.
15. Backend не переводит бронь в новый approval lifecycle status и не запускает повторное согласование.
16. Backend инициирует уведомление [`Requestor`](../../Roles%20and%20Access%20Model.md)-а об изменении периода.
17. Frontend обновляет карточку и показывает новый плановый диапазон брони.

---

## Альтернативные сценарии

1. Текущий статус брони уже не позволяет менять период.
   Backend возвращает `BOOKING_NOT_CHANGEABLE`, frontend показывает сообщение и обновляет данные брони.

2. Новый диапазон дат невалиден.
   Backend возвращает `VALIDATION_ERROR`.

3. Бронь находится в `InProgress`, и пользователь пытается изменить `plannedStartDateTime`.
   Backend возвращает `VALIDATION_ERROR` или `BOOKING_NOT_CHANGEABLE` в зависимости от принятой ошибки контракта.

4. Новый диапазон нарушает [hard availability restrictions](../../../glossary/Glossary.md#hard-availability-restriction).
   Backend возвращает `EQUIPMENT_NOT_AVAILABLE`.

5. У брони есть конфликты с другими активными бронями на новый период.
   Изменение периода не блокируется автоматически только из-за competing bookings. Fleet Owner принимает решение на основании [booking conflict context](../../../glossary/Glossary.md#booking-conflict-context) и бизнес-приоритета.

6. Пользователь не имеет доступа к брони.
   Backend возвращает `FORBIDDEN`.

7. Бронь не найдена.
   Backend возвращает `NOT_FOUND`.

---

## Замечания

1. Use case покрывает изменение периода до подтверждения, после подтверждения и частично во время исполнения: для `Submitted` / `Confirmed` меняется весь плановый диапазон, для `InProgress` меняется только плановая дата окончания.
2. Изменение периода не должно требовать повторного подтверждения [`Fleet Owner`](../../Roles%20and%20Access%20Model.md)-ом и не должно возвращать бронь в lifecycle `Submitted`.
3. Уведомление [`Requestor`](../../Roles%20and%20Access%20Model.md)-а об изменении периода является обязательным side effect согласно `FR-079a`.
4. Для данного сценария важно различать [booking conflict context](../../../glossary/Glossary.md#booking-conflict-context) и [hard availability restriction](../../../glossary/Glossary.md#hard-availability-restriction): overlap в `Bookings` сам по себе не блокирует действие, а абсолютная недоступность, зафиксированная бизнес-правилами и/или `EquipmentStates`, должна блокировать изменение периода.
