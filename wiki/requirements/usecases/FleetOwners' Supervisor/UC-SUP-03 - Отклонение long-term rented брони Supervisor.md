# UC-SUP-03 - Отклонение long-term rented брони [`FleetOwners' Supervisor`](../../Roles%20and%20Access%20Model.md)

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
| Триггер | Нажатие кнопки `Decline` в detail / decision view брони из очереди Supervisor |
| Ожидаемый результат | Supervisor отклоняет бронь; система фиксирует шаг `SupervisorApproval / Declined`, переводит бронь в `Closed` с `closureReason = Declined` и инициирует уведомления Requestor и FO |
| Используемые API | [`GET /supervisor/bookings/{id}`](../../../api/booking/supervisor/GET_supervisor_bookings_id.md), [`POST /supervisor/bookings/{id}/decline`](../../../api/booking/supervisor/POST_supervisor_bookings_id_decline.md) |

---

## Основной сценарий

1. [`FleetOwners' Supervisor`](../../Roles%20and%20Access%20Model.md) открывает очередь long-term rented броней.
2. Пользователь выбирает нужную бронь из списка.
3. Frontend загружает detail / decision view через [`GET /supervisor/bookings/{id}`](../../../api/booking/supervisor/GET_supervisor_bookings_id.md).
4. Backend проверяет:
   - что бронь существует;
   - что текущий пользователь имеет роль [`FleetOwnersSupervisor`](../../Roles%20and%20Access%20Model.md);
   - что по брони ожидается шаг `SupervisorApproval`.
5. Frontend показывает Supervisor контекст решения:
   - request-level данные;
   - технику и плановый период;
   - `justification` от Requestor;
   - уже записанное решение [`Fleet Owner`](../../Roles%20and%20Access%20Model.md);
   - history / timeline context, если он доступен в detail model.
6. Пользователь нажимает `Decline`.
7. Frontend открывает форму decline и требует обязательный комментарий.
8. Пользователь вводит комментарий и подтверждает действие.
9. Frontend вызывает [`POST /supervisor/bookings/{id}/decline`](../../../api/booking/supervisor/POST_supervisor_bookings_id_decline.md).
10. Backend проверяет, что по брони все еще ожидается шаг `SupervisorApproval` и что передан непустой `comment`.
11. Backend переводит бронь в terminal state `Closed` с `closureReason = Declined`.
12. Backend создает запись в `BookingApprovals` с типом `SupervisorApproval`, результатом `Declined` и `order = 2`.
13. Backend создает запись в `BookingStatuses`.
14. Backend инициирует уведомления для Requestor и [`Fleet Owner`](../../Roles%20and%20Access%20Model.md) о Supervisor decline.
15. Frontend обновляет карточку и показывает:
   - статус `Closed`;
   - closure reason `Declined`;
   - сохраненный комментарий Supervisor.

---

## Альтернативные сценарии

1. Пользователь не передал обязательный комментарий.
   Backend возвращает `VALIDATION_ERROR`; frontend подсвечивает обязательность поля `comment`.

2. По брони больше не ожидается шаг `SupervisorApproval`.
   Backend возвращает `BOOKING_NOT_DECLINABLE`; frontend показывает сообщение и обновляет данные брони.

3. Пользователь не имеет роли [`FleetOwnersSupervisor`](../../Roles%20and%20Access%20Model.md).
   Backend возвращает `FORBIDDEN`.

4. Бронь не найдена.
   Backend возвращает `NOT_FOUND`.

---

## Замечания

1. Сценарий применим только к long-term rented броням на финальном шаге Supervisor.
2. Для отрицательного решения комментарий Supervisor обязателен.
3. Supervisor decline завершает approval chain отрицательным исходом и переводит бронь в terminal state `Closed + Declined`.
4. Requestor и [`Fleet Owner`](../../Roles%20and%20Access%20Model.md) должны получать notification outcome этого решения через отдельный notification flow.
