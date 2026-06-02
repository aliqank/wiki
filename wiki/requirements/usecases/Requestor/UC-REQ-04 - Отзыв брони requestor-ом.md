# UC-REQ-04 - Отзыв брони requestor-ом

**Created:** 2026-05-20  
**Last updated:** 2026-06-02  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка use case

| Поле | Значение |
|---|---|
| Область действия | Страница `Мои заявки` / карточка заявки / карточка booking item в отправленной заявке |
| Участник | Пользователь с ролью `Requestor` или `ServiceWorkProcessor` |
| Покрываемые FR (BRD) | `FR-025`, `FR-042`, `FR-058`, `FR-068`, `FR-078`, `FR-NEW-17` |
| Покрываемые FR (Additional list) | — |
| Предусловие | Пользователь авторизован; у пользователя есть собственная незавершенная заявка; в заявке есть бронь в статусе `Submitted`, которая еще не обработана Fleet Owner |
| Триггер | Нажатие кнопки `Отозвать бронь` |
| Ожидаемый результат | Выбранная бронь переведена в статус `Closed` с причиной `Revoked`; агрегированный статус заявки пересчитан; Fleet Owner получает уведомление |
| Используемые API | [`GET /booking-requests/my`](../../../api/booking/requestor/GET_booking_requests_my.md), [`POST /bookings/{id}/revoke`](../../../api/booking/requestor/POST_bookings_id_revoke.md), [`GET /booking-requests/{id}`](../../../api/booking/requestor/GET_booking_requests_id.md) |

---

## Основной сценарий

1. Пользователь открывает страницу `Мои заявки`.
2. Frontend загружает список заявок через [`GET /booking-requests/my`](../../../api/booking/requestor/GET_booking_requests_my.md) и показывает summary по броням внутри каждой заявки.
3. Для booking summary / booking item в статусе `Submitted` frontend показывает действие `Отозвать бронь`.
4. Пользователь выбирает конкретную бронь и нажимает `Отозвать бронь`.
5. Frontend может показать подтверждающий диалог перед отзывом.
6. После подтверждения frontend вызывает [`POST /bookings/{id}/revoke`](../../../api/booking/requestor/POST_bookings_id_revoke.md).
7. Backend проверяет, что:
   - бронь существует;
   - родительская заявка брони имеет `BookingRequests.requestorId`, совпадающий с текущим пользователем;
   - бронь еще не обработана Fleet Owner;
   - текущий статус брони равен `Submitted`.
8. Backend переводит booking item в статус `Closed` с terminal причиной `Revoked`.
9. Backend сохраняет запись в истории статусов брони и пересчитывает агрегированный статус родительской заявки.
10. Система отправляет уведомление Fleet Owner об отзыве брони.
11. Frontend обновляет строку заявки на странице `Мои заявки`.
12. При необходимости пользователь открывает детальный просмотр заявки через [`GET /booking-requests/{id}`](../../../api/booking/requestor/GET_booking_requests_id.md) для просмотра полного актуального состава booking item-ов.

---

## Альтернативные сценарии

1. Пользователь передумал отзывать бронь.
   Пользователь закрывает подтверждающий диалог, вызов API не выполняется.

2. Fleet Owner уже обработал бронь.
   Backend возвращает `BOOKING_NOT_REVOCABLE`, frontend показывает сообщение об ошибке и не меняет статус.

3. Пользователь пытается отозвать не свою бронь.
   Backend возвращает `FORBIDDEN`.

4. Бронь не найдена.
   Backend возвращает `NOT_FOUND`, frontend предлагает обновить данные заявки.

5. Пользователь выполняет отзыв из detail view заявки, а не из списка `Мои заявки`.
   Frontend использует [`GET /booking-requests/{id}`](../../../api/booking/requestor/GET_booking_requests_id.md) для отображения item-ов и после успешного `revoke` обновляет детальную карточку заявки.

6. В заявке несколько booking item-ов.
   Отзыв одной брони не отменяет остальные item-ы; система пересчитывает request status на основании всех item statuses.

---

## Замечания

1. Use case относится только к отзыву брони до решения Fleet Owner; после решения FO применяются другие сценарии жизненного цикла, например termination.
2. Отзыв выполняется на уровне отдельного booking item, а не всей заявки.
3. `Revoked` трактуется как terminal причина закрытия брони, инициируемая Requestor до FO action.
4. Правила пересчета request-level статуса после `revoke` и других item-level переходов зафиксированы в отдельном документе `wiki/requirements/Aggregated Request Status Rules.md`.
