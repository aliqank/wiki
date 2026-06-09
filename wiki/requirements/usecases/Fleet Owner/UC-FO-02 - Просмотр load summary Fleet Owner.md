# UC-FO-02 - Просмотр load summary [`Fleet Owner`](../../Roles and Access Model.md)

**Created:** 2026-05-28  
**Last updated:** 2026-06-02  
**Автор документов:** OpenCode

---

## Карточка use case

| Поле | Значение |
|---|---|
| Область действия | Страница [`Fleet Owner`](../../Roles and Access Model.md) `Approvals` -> view `Requests` -> booking row action `Load summary`; optional detail / action view конкретной брони |
| Участник | Пользователь с ролью [`FleetOwner`](../../Roles and Access Model.md) |
| Покрываемые FR (BRD) | `FR-NEW-64` |
| Покрываемые FR (Additional list) | — |
| Предусловие | Пользователь авторизован в системе; пользователь имеет роль [`FleetOwner`](../../Roles and Access Model.md); бронь относится к зоне ответственности пользователя; пользователь находится на странице `Approvals` во view `Requests` |
| Триггер | Нажатие кнопки `Load summary` в строке брони на странице `Requests` или открытие блока `Load summary` в detail / action view брони |
| Ожидаемый результат | Пользователь видит сводку пересекающихся активных броней по той же технике на даты текущей брони без обязательного перехода в отдельную карточку брони |
| Используемые API | [`GET /approvals/bookings/{id}/load-summary`](../../../api/booking/fleet-owner/GET_approvals_bookings_id_load_summary.md) |

---

## Основной сценарий

1. [`Fleet Owner`](../../Roles and Access Model.md) открывает страницу `Approvals` во view `Requests` и выбирает заявку из списка.
2. Пользователь находит нужную бронь внутри заявки и нажимает кнопку `Load summary` в строке этой брони.
3. Frontend определяет `bookingId` выбранной брони из строки списка.
4. Frontend вызывает [`GET /approvals/bookings/{id}/load-summary`](../../../api/booking/fleet-owner/GET_approvals_bookings_id_load_summary.md).
5. Backend проверяет, что:
   - бронь существует;
   - текущий пользователь имеет доступ к брони как [`Fleet Owner`](../../Roles and Access Model.md);
   - бронь относится к зоне ответственности пользователя.
6. Backend находит пересекающиеся активные брони по тому же `equipmentId` на даты текущей брони.
7. Backend исключает из результата текущую бронь.
8. Backend возвращает summary для approval window:
   - `equipmentId`;
   - `activeBookingsCount`;
   - `workOrderNumber` для каждой пересекающейся брони;
   - список пересекающихся броней.
9. Frontend отображает load summary в popup / modal на той же странице `Requests`.
10. В popup / modal пользователь видит:
   - контекст текущей брони из выбранной строки списка: технику и плановый период;
   - summary line с количеством активных пересекающихся броней;
   - список пересекающихся броней с `requestNumber`, `workOrderNumber`, `status`, `plannedStartDateTime`, `plannedEndDateTime`.
11. Пользователь использует load summary как входные данные для дальнейшего решения по брони.

---

## Альтернативные сценарии

1. Для выбранной техники нет пересекающихся активных броней.
   Backend возвращает `activeBookingsCount = 0` и пустой список `items`; frontend показывает пустое состояние блока load summary.

2. Пользователь не имеет доступа к брони.
   Backend возвращает `FORBIDDEN`; frontend не показывает данные load summary и отображает сообщение об ошибке.

3. Бронь не найдена.
   Backend возвращает `NOT_FOUND`; frontend показывает сообщение об ошибке.

---

## Замечания

1. Load summary используется как информационный блок для принятия решения [`Fleet Owner`](../../Roles and Access Model.md) и сам по себе не является hard-stop механизмом.
2. В summary должны попадать только пересекающиеся активные брони по той же технике, а не вся история бронирований.
3. Данный use case является вспомогательным по отношению к `UC-FO-03` и может вызываться до confirm / decline решения.
4. Открытие отдельного detail / action view не является обязательным для просмотра load summary.
5. Заголовок popup / modal может быть собран frontend-ом из данных выбранной строки брони без отдельного detail API.
