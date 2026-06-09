# UC-SUP-02 - Подтверждение long-term rented брони [`FleetOwners' Supervisor`](../../Roles%20and%20Access%20Model.md)

**Created:** 2026-06-09  
**Last updated:** 2026-06-09  
**Автор документов:** OpenCode

---

## Карточка use case

| Поле | Значение |
|---|---|
| Область действия | Страница [`FleetOwners' Supervisor`](../../Roles%20and%20Access%20Model.md) queue -> detail / decision view конкретной long-term rented брони |
| Участник | Пользователь с ролью [`FleetOwnersSupervisor`](../../Roles%20and%20Access%20Model.md) |
| Покрываемые FR (BRD) | `FR-NEW-73`, `FR-NEW-76`, `FR-NEW-78` |
| Покрываемые FR (Additional list) | — |
| Предусловие | Пользователь авторизован; пользователь имеет роль [`FleetOwnersSupervisor`](../../Roles%20and%20Access%20Model.md); бронь является long-term rented; по брони ожидается шаг `SupervisorApproval`; positive step [`Fleet Owner`](../../Roles%20and%20Access%20Model.md) уже зафиксирован |
| Триггер | Нажатие кнопки `Confirm` в detail / decision view брони из очереди Supervisor |
| Ожидаемый результат | Supervisor финально подтверждает бронь; система фиксирует шаг `SupervisorApproval / Approved`, переводит бронь в `Confirmed` и инициирует уведомления Requestor и FO |
| Используемые API | [`GET /supervisor/bookings/{id}`](../../../api/booking/supervisor/GET_supervisor_bookings_id.md), [`POST /supervisor/bookings/{id}/confirm`](../../../api/booking/supervisor/POST_supervisor_bookings_id_confirm.md) |

---

## Основной сценарий

1. [`FleetOwners' Supervisor`](../../Roles%20and%20Access%20Model.md) открывает очередь long-term rented броней.
2. Пользователь выбирает нужную бронь из списка.
3. Frontend загружает detail / decision view через [`GET /supervisor/bookings/{id}`](../../../api/booking/supervisor/GET_supervisor_bookings_id.md).
4. Backend проверяет:
   - что бронь существует;
   - что текущий пользователь имеет роль [`FleetOwnersSupervisor`](../../Roles%20and%20Access%20Model.md);
   - что по брони ожидается шаг `SupervisorApproval`.
5. Frontend показывает Supervisor полный контекст решения:
   - request-level данные;
   - технику и плановый период;
   - `justification` от Requestor;
   - уже записанное решение [`Fleet Owner`](../../Roles%20and%20Access%20Model.md) в approval chain;
   - history / timeline context, если он доступен в detail model.
6. Пользователь принимает положительное решение и нажимает `Confirm`.
7. Frontend вызывает [`POST /supervisor/bookings/{id}/confirm`](../../../api/booking/supervisor/POST_supervisor_bookings_id_confirm.md).
8. Backend проверяет, что по брони все еще ожидается шаг `SupervisorApproval`.
9. Backend переводит бронь в статус `Confirmed`.
10. Backend создает запись в `BookingApprovals` с типом `SupervisorApproval`, результатом `Approved` и `order = 2`.
11. Backend создает запись в `BookingStatuses`.
12. Backend инициирует уведомления для Requestor и [`Fleet Owner`](../../Roles%20and%20Access%20Model.md) о финальном положительном решении Supervisor.
13. Frontend обновляет карточку и показывает финальный статус `Confirmed`.

---

## Альтернативные сценарии

1. По брони больше не ожидается шаг `SupervisorApproval`.
   Backend возвращает `BOOKING_NOT_CONFIRMABLE`; frontend показывает сообщение и обновляет данные брони.

2. Пользователь не имеет роли [`FleetOwnersSupervisor`](../../Roles%20and%20Access%20Model.md).
   Backend возвращает `FORBIDDEN`.

3. Бронь не найдена.
   Backend возвращает `NOT_FOUND`.

4. Supervisor добавляет комментарий при confirm.
   Комментарий сохраняется как служебный комментарий approval step, но не является обязательным для положительного решения.

---

## Замечания

1. Сценарий применим только к long-term rented броням и не должен использоваться для TCO Owned booking flow.
2. Positive решение Supervisor завершает approval chain и только после этого бронь становится `Confirmed`.
3. После финального confirm становятся допустимыми следующие execution-сценарии, например мобилизация.
4. Supervisor decision должен фиксироваться и как approval event, и как lifecycle transition.
