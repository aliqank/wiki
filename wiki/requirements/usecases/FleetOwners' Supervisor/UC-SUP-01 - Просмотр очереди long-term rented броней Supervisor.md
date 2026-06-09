# UC-SUP-01 - Просмотр очереди long-term rented броней [`FleetOwners' Supervisor`](../../Roles%20and%20Access%20Model.md)

**Created:** 2026-06-09  
**Last updated:** 2026-06-09  
**Автор документов:** OpenCode

---

## Карточка use case

| Поле | Значение |
|---|---|
| Область действия | Страница [`FleetOwners' Supervisor`](../../Roles%20and%20Access%20Model.md) `Approvals` / queue view long-term rented броней |
| Участник | Пользователь с ролью [`FleetOwnersSupervisor`](../../Roles%20and%20Access%20Model.md) |
| Покрываемые FR (BRD) | `FR-NEW-73`, `FR-NEW-75` |
| Покрываемые FR (Additional list) | — |
| Предусловие | Пользователь авторизован; пользователь имеет роль [`FleetOwnersSupervisor`](../../Roles%20and%20Access%20Model.md); в системе могут существовать long-term rented брони с pending шагом `SupervisorApproval` |
| Триггер | Открытие страницы входящих согласований Supervisor |
| Ожидаемый результат | Пользователь видит paginated queue long-term rented броней, ожидающих финального решения Supervisor, и может открыть карточку конкретной брони |
| Используемые API | [`GET /supervisor/bookings`](../../../api/booking/supervisor/GET_supervisor_bookings.md) |

---

## Основной сценарий

1. [`FleetOwners' Supervisor`](../../Roles%20and%20Access%20Model.md) открывает страницу входящих согласований.
2. Frontend вызывает [`GET /supervisor/bookings`](../../../api/booking/supervisor/GET_supervisor_bookings.md).
3. Backend проверяет, что текущий пользователь имеет роль [`FleetOwnersSupervisor`](../../Roles%20and%20Access%20Model.md).
4. Backend отбирает long-term rented брони, по которым ожидается шаг `SupervisorApproval`.
5. Backend возвращает paginated queue.
6. Frontend отображает список броней с ключевым контекстом для принятия решения:
   - `requestNumber`;
   - `tcoId` техники;
   - текущий lifecycle status;
   - `justification`.
7. Пользователь при необходимости использует поиск по номеру заявки, номеру техники или тексту `justification`.
8. Пользователь выбирает конкретную бронь из очереди для перехода к detail / decision view.

---

## Альтернативные сценарии

1. В очереди нет доступных броней.
   Backend возвращает пустой список; frontend показывает empty state.

2. Пользователь не имеет роли [`FleetOwnersSupervisor`](../../Roles%20and%20Access%20Model.md).
   Backend возвращает `FORBIDDEN`.

3. Пользователь применяет поисковый запрос, который не находит совпадений.
   Backend возвращает пустую страницу результатов; frontend сохраняет введенный фильтр и показывает empty state для search-result.

---

## Замечания

1. В очередь должны попадать только long-term rented брони с pending шагом `SupervisorApproval`.
2. TCO Owned и иные booking flow не должны отображаться в этом списке.
3. Queue view является входной точкой для финального решения Supervisor, но не заменяет detail / decision сценарии.
4. Наличие записи в очереди означает, что positive decision [`Fleet Owner`](../../Roles%20and%20Access%20Model.md) уже зафиксирован как approval step, но финальное согласование еще не завершено.
