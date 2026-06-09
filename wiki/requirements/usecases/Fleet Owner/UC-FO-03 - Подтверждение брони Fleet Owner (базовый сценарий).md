# UC-FO-03 - Подтверждение брони [`Fleet Owner`](../../Roles%20and%20Access%20Model.md) (базовый сценарий)

**Created:** 2026-05-20  
**Last updated:** 2026-06-09  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка use case

| Поле | Значение |
|---|---|
| Область действия | Страница [`Fleet Owner`](../../Roles%20and%20Access%20Model.md) `Approvals` -> view `Requests` -> detail / action view конкретной брони |
| Участник | Пользователь с ролью [`FleetOwner`](../../Roles%20and%20Access%20Model.md) |
| Покрываемые FR (BRD) | `FR-043`, `FR-044`, `FR-045`, `FR-063`, `FR-NEW-19` |
| Покрываемые FR (Additional list) | — |
| Предусловие | Пользователь авторизован в системе; пользователь имеет роль [`FleetOwner`](../../Roles%20and%20Access%20Model.md); бронь относится к fleet-у пользователя; бронь находится в статусе `Submitted`; сценарий не требует транспортировки и не требует дополнительного согласования Supervisor |
| Триггер | Нажатие кнопки `Подтвердить` в карточке брони, открытой из request view |
| Ожидаемый результат | Бронь подтверждена [`Fleet Owner`](../../Roles%20and%20Access%20Model.md) и переходит в статус `Confirmed` |
| Используемые API | [`GET /approvals/bookings/{id}`](../../../api/booking/fleet-owner/GET_approvals_bookings_id.md), [`POST /approvals/bookings/{id}/confirm`](../../../api/booking/fleet-owner/POST_approvals_bookings_id_confirm.md) |

---

## Основной сценарий

1. [`Fleet Owner`](../../Roles%20and%20Access%20Model.md) открывает страницу `Approvals` во view `Requests` и выбирает заявку из списка.
2. Пользователь открывает внутри заявки detail / action view конкретной релевантной брони.
3. Frontend загружает актуальные данные брони через [`GET /approvals/bookings/{id}`](../../../api/booking/fleet-owner/GET_approvals_bookings_id.md).
4. Пользователь проверяет ключевые данные брони:
   - requestor;
   - Work Order и приоритет;
   - технику;
   - плановый период;
   - load summary, включая `Work Order number`, и наличие конфликтов с другими активными бронями на момент подтверждения.
5. Система определяет, что бронь относится к базовому позитивному сценарию:
   - не требуется перевозка;
   - не требуется дополнительное согласование Supervisor;
   - текущий статус позволяет confirm.
6. Пользователь нажимает кнопку `Подтвердить`.
7. Frontend вызывает [`POST /approvals/bookings/{id}/confirm`](../../../api/booking/fleet-owner/POST_approvals_bookings_id_confirm.md).
8. Backend проверяет, что:
   - бронь существует;
   - бронь относится к зоне ответственности текущего [`Fleet Owner`](../../Roles%20and%20Access%20Model.md);
   - текущий статус равен `Submitted`;
   - отсутствуют [hard availability restrictions](../../../glossary/Glossary.md#hard-availability-restriction), делающие технику фактически недоступной независимо от competing bookings.
9. Backend переводит бронь в статус `Confirmed` только для брони, которая не требует дополнительного согласования Supervisor, то есть не относится к `Long-term rented` flow.
10. Backend создает запись шага согласования в `BookingApprovals` с типом `FoApproval`, результатом `Approved` и `order = 1`.
11. Backend создает запись в `BookingStatuses`.
12. Frontend обновляет карточку и показывает новый статус `Confirmed`.
13. Система обновляет request view так, чтобы подтвержденная бронь больше не отображалась как ожидающая решения [`Fleet Owner`](../../Roles%20and%20Access%20Model.md).

---

## Альтернативные сценарии

1. Текущий статус брони уже не позволяет confirm.
   Backend возвращает `BOOKING_NOT_CONFIRMABLE`, frontend показывает сообщение и обновляет данные брони.

2. У брони есть конфликты с другими активными бронями.
   Confirm не блокируется автоматически только из-за самого факта конфликта. Fleet Owner принимает решение на основании [booking conflict context](../../../glossary/Glossary.md#booking-conflict-context), load summary, контекста заявки и бизнес-приоритета.

3. Выявлено hard-ограничение доступности техники.
   Backend возвращает `EQUIPMENT_NOT_AVAILABLE`, если техника фактически недоступна по [hard availability restrictions](../../../glossary/Glossary.md#hard-availability-restriction), не связанным с competing bookings.

4. Бронь требует дополнительного согласования Supervisor.
   Данный базовый use case не покрывает эту ветку целиком; после positive decision [`Fleet Owner`](../../Roles%20and%20Access%20Model.md) сценарий продолжается отдельным Supervisor flow: [`UC-SUP-01 - Просмотр очереди long-term rented броней Supervisor`](../FleetOwners%27%20Supervisor/UC-SUP-01%20-%20Просмотр%20очереди%20long-term%20rented%20броней%20Supervisor.md), [`UC-SUP-02 - Подтверждение long-term rented брони Supervisor`](../FleetOwners%27%20Supervisor/UC-SUP-02%20-%20Подтверждение%20long-term%20rented%20брони%20Supervisor.md) и [`UC-SUP-03 - Отклонение long-term rented брони Supervisor`](../FleetOwners%27%20Supervisor/UC-SUP-03%20-%20Отклонение%20long-term%20rented%20брони%20Supervisor.md). После positive решения FO бронь уходит в pending Supervisor approval и не становится финально `Confirmed` до решения Supervisor.

5. Бронь требует транспортировки.
   Данный use case не применяется; используется отдельный сценарий с transport flow.

6. Пользователь не имеет доступа к брони.
   Backend возвращает `FORBIDDEN`.

---

## Замечания

1. Use case описывает только базовый позитивный путь для брони, которая после FO decision сразу переходит в `Confirmed`.
2. Сценарий намеренно исключает ветки pending `SupervisorApproval` и transport-specific обработку.
3. Для long-term rented booking после confirm [`Fleet Owner`](../../Roles%20and%20Access%20Model.md) бронь не становится финально `Confirmed` сразу; требуется отдельное решение [`FleetOwners' Supervisor`](../../Roles%20and%20Access%20Model.md).
4. Решение [`Fleet Owner`](../../Roles%20and%20Access%20Model.md) должно фиксироваться не только как status transition, но и как отдельный approval step в `BookingApprovals`.
5. Отдельный special status для продления не рассматривается; повторное согласование должно возвращать бронь в lifecycle `Submitted`.
