# UC-FO-09 - Согласование изменения плановой даты и времени окончания брони

**Created:** 2026-06-02  
**Last updated:** 2026-06-02  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка use case

| Поле | Значение |
|---|---|
| Область действия | Страница Fleet Owner `Approvals` -> detail / action view брони с pending extension request |
| Участник | Пользователь с ролью `FleetOwner` |
| Покрываемые FR (BRD) | `FR-061`, `FR-067`, `FR-081` |
| Покрываемые FR (Additional list) | — |
| Предусловие | Пользователь авторизован в системе; пользователь имеет роль `FleetOwner`; бронь относится к fleet-у, по которому у пользователя есть [Fleet Management Access](../../../glossary/Glossary.md#fleet-management-access); по брони существует pending [Booking Extension Approval](../../../glossary/Glossary.md#booking-extension-approval) |
| Триггер | Fleet Owner открывает бронь, по которой Requestor / SWP запросил изменение `plannedEndDateTime` |
| Ожидаемый результат | Fleet Owner принимает решение по запросу на изменение срока брони; после approve новое `plannedEndDateTime` применяется, а статус брони становится `Confirmed`, если других pending approvals не требуется, либо сохраняет `InProgress`, если исполнение уже началось |
| Используемые API | [`GET /approvals/bookings/{id}`](../../../api/booking/fleet-owner/GET_approvals_bookings_id.md), [`POST /approvals/bookings/{id}/extension-approval`](../../../api/booking/fleet-owner/POST_approvals_bookings_id_extension_approval.md) |

---

## Основной сценарий

1. Fleet Owner открывает страницу `Approvals` и переходит в detail / action view брони, по которой есть запрос на изменение `plannedEndDateTime`.
2. Frontend загружает контекст брони через [`GET /approvals/bookings/{id}`](../../../api/booking/fleet-owner/GET_approvals_bookings_id.md).
3. Система показывает:
   - текущий lifecycle status брони;
   - текущее `plannedEndDateTime`;
   - запрошенное новое `plannedEndDateTime`;
   - комментарий Requestor / SWP;
   - [booking conflict context](../../../glossary/Glossary.md#booking-conflict-context), если он есть.
4. Fleet Owner анализирует запрос и принимает решение.
5. Frontend вызывает [`POST /approvals/bookings/{id}/extension-approval`](../../../api/booking/fleet-owner/POST_approvals_bookings_id_extension_approval.md) с решением `Approved`.
6. Backend проверяет, что:
   - бронь существует;
   - по ней есть pending [Booking Extension Approval](../../../glossary/Glossary.md#booking-extension-approval);
   - текущий Fleet Owner имеет право принять решение.
7. Backend создает approval-запись `BookingExtensionApproval / Approved` в `BookingApprovals`.
8. Backend применяет новое `plannedEndDateTime` к брони.
9. Backend фиксирует запись в `BookingStatuses`, если по принятой модели требуется audit lifecycle change.
10. Если запрос был создан для pre-start брони и других обязательных pending approvals больше нет, backend переводит бронь в `Confirmed`.
11. Если запрос был создан для брони в `InProgress`, backend сохраняет статус `InProgress` и обновляет только `plannedEndDateTime`.
12. Frontend обновляет карточку и показывает новое состояние брони.

---

## Альтернативные сценарии

1. Fleet Owner отклоняет запрос на изменение `plannedEndDateTime`.
   Backend создает approval-запись `BookingExtensionApproval / Declined`; новое `plannedEndDateTime` не применяется.

2. По брони больше нет pending extension request.
   Backend возвращает бизнес-ошибку, что approval уже обработан или отсутствует.

3. Пользователь не имеет доступа к брони.
   Backend возвращает `FORBIDDEN`.

4. Бронь не найдена.
   Backend возвращает `NOT_FOUND`.

---

## Замечания

1. [Booking Extension Approval](../../../glossary/Glossary.md#booking-extension-approval) является отдельным approval step и не должен смешиваться с базовым `FoApproval` первоначального согласования брони.
2. Для pre-start сценария после approve бронь должна стать `Confirmed`, если никаких других обязательных approval steps больше не требуется.
3. Для `InProgress` сценария approve не должен откатывать бронь из `InProgress`; меняется только `plannedEndDateTime`.
4. Данный use case описывает только решение Fleet Owner по уже существующему запросу Requestor / SWP на изменение срока брони.
