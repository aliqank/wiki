# UC-FO-01 - Просмотр списка заявок Fleet Owner

**Created:** 2026-05-21  
**Last updated:** 2026-06-08  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка use case

| Поле | Значение |
|---|---|
| Область действия | Страница Fleet Owner `Approvals` -> view `Requests` |
| Участник | Пользователь с ролью `FleetOwner` |
| Покрываемые FR (BRD) | `FR-092`, `FR-043` |
| Покрываемые FR (Additional list) | `BRD-U-001` |
| Предусловие | Пользователь авторизован в системе; пользователь имеет роль `FleetOwner`; пользователю доступны один или несколько fleet-ов |
| Триггер | Вход на страницу Fleet Owner `Approvals` c активной view `Requests` |
| Ожидаемый результат | Отображается paginated список заявок, относящихся к зоне ответственности Fleet Owner, с request-level статусом и базовыми фильтрами |
| Используемые API | [`GET /approvals/requests`](../../../api/booking/fleet-owner/GET_approvals_requests.md), [`GET /reference/approval-request-statuses`](../../../api/booking/reference/GET_reference_approval_request_statuses.md), [`GET /reference/approval-request-types`](../../../api/booking/reference/GET_reference_approval_request_types.md), [`GET /reference/request-priorities`](../../../api/booking/reference/GET_reference_request_priorities.md) |

---

## Основной сценарий

1. Пользователь открывает страницу Fleet Owner `Approvals` и переключается во view `Requests`.
2. Frontend загружает справочники фильтров через [`GET /reference/approval-request-statuses`](../../../api/booking/reference/GET_reference_approval_request_statuses.md), [`GET /reference/approval-request-types`](../../../api/booking/reference/GET_reference_approval_request_types.md), [`GET /reference/request-priorities`](../../../api/booking/reference/GET_reference_request_priorities.md); для `priority` справочник также содержит `description` и `color`.
3. Frontend вызывает [`GET /approvals/requests`](../../../api/booking/fleet-owner/GET_approvals_requests.md).
4. Backend проверяет, что текущий пользователь имеет роль `FleetOwner`.
5. Backend определяет список fleet-ов, где у текущего Fleet Owner есть [Fleet Management Access](../../../glossary/Glossary.md#fleet-management-access).
6. Backend выбирает только те `BookingRequests`, в составе которых есть booking item-ы по этим fleet-ам.
7. Backend применяет request-level фильтры экрана по статусу, типу заявки и периоду.
8. Backend рассчитывает и возвращает paginated список заявок вместе с релевантными `bookingSummaries` внутри каждой заявки.
9. Frontend отображает таблицу заявок.
10. Для каждой строки отображаются request-level поля и краткий состав связанных броней, доступных текущему Fleet Owner:
   - номер заявки;
   - текущий статус заявки;
   - тип заявки;
   - приоритет с цветом, соответствующим настройке справочника.
11. Для каждой релевантной брони система также может показывать conflict indicator и quick action для открытия `load summary` прямо в строке брони.
12. Пользователь получает в составе ответа всю информацию, необходимую для первичной работы со связанными бронями без обязательного дополнительного detail-запроса.

---

## Альтернативные сценарии

1. У Fleet Owner нет заявок в зоне ответственности.
   Система отображает пустое состояние без строк таблицы.

2. Пользователь применяет фильтр по статусу заявки.
   Frontend повторно вызывает [`GET /approvals/requests`](../../../api/booking/fleet-owner/GET_approvals_requests.md) с query-параметром `status`.

3. Пользователь применяет фильтр по типу заявки.
   Frontend повторно вызывает [`GET /approvals/requests`](../../../api/booking/fleet-owner/GET_approvals_requests.md) с query-параметром `type`.

4. Пользователь задает период выборки.
   Frontend повторно вызывает [`GET /approvals/requests`](../../../api/booking/fleet-owner/GET_approvals_requests.md) с query-параметрами `createdFrom` / `createdTo`.

5. Backend возвращает ошибку авторизации или доступа.
   Frontend отображает сообщение об ошибке и не показывает данные списка.

---

## Замечания

1. View `Requests` является request-centric: одна строка списка соответствует одной заявке, а не отдельной брони.
2. В список не должны попадать заявки, не имеющие booking item-ов по fleet-ам текущего Fleet Owner.
3. Для request-level списка используется агрегированный статус заявки, а не статус отдельной брони.
4. Для terminal request statuses должна использоваться актуальная семантика: request lifecycle использует единый статус `Closed`, а различие между pre-start cancellation и post-start completion хранится в `requestClosureReason`.
5. Текущая версия use case предполагает, что для первичной работы со связанными booking item-ами достаточно данных из [`GET /approvals/requests`](../../../api/booking/fleet-owner/GET_approvals_requests.md).
6. Для фильтра `status` используется [`GET /reference/approval-request-statuses`](../../../api/booking/reference/GET_reference_approval_request_statuses.md), для фильтра `type` используется [`GET /reference/approval-request-types`](../../../api/booking/reference/GET_reference_approval_request_types.md), для фильтра `priority` используется [`GET /reference/request-priorities`](../../../api/booking/reference/GET_reference_request_priorities.md), который также возвращает `description` и `color`.
7. Признак конфликта в строке брони является short summary; детальный состав конфликтов / competing bookings раскрывается через `load summary`.
