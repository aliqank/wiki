# UC-REQ-02.7 - Удаление draft-брони

**Created:** 2026-05-20  
**Last updated:** 2026-06-02  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка use case

| Поле | Значение |
|---|---|
| Область действия | Карточка booking item в draft-заявке |
| Участник | `Requestor`, `ServiceWorkProcessor` |
| Покрываемые FR (BRD) | `FR-027`, `FR-031`, `FR-038` |
| Покрываемые FR (Additional list) | — |
| Предусловие | Пользователь авторизован; открыта собственная draft-заявка; в заявке есть booking item в статусе `Draft` |
| Триггер | Нажатие кнопки `Удалить` в карточке draft booking item |
| Ожидаемый результат | Booking item удален из draft-заявки и больше не участвует в ее составе |
| Используемые API | [`DELETE /booking-requests/{id}/items/{bookingId}`](../../../../api/booking/requestor/DELETE_booking_requests_id_items_bookingId.md), [`GET /booking-requests/{id}`](../../../../api/booking/requestor/GET_booking_requests_id.md) |

---

## Основной сценарий

1. Пользователь открывает draft-заявку.
2. Frontend отображает список booking item-ов в составе заявки.
3. Пользователь выбирает draft booking item, который больше не нужен.
4. Пользователь нажимает `Удалить`.
5. Frontend может показать подтверждающий диалог перед удалением item-а из draft.
6. После подтверждения frontend вызывает `DELETE /booking-requests/{id}/items/{bookingId}`.
7. Backend проверяет, что:
   - заявка существует и принадлежит текущему пользователю;
   - заявка находится в статусе `Draft`;
   - booking item принадлежит этой заявке;
   - booking item находится в статусе `Draft`.
8. Backend выполняет soft delete booking item-а.
9. Frontend обновляет состав заявки через `GET /booking-requests/{id}` или локальное обновление state.
10. Удаленный item больше не отображается в составе draft-заявки.

---

## Альтернативные сценарии

1. Пользователь передумал удалять draft-бронь.
   Пользователь закрывает подтверждающий диалог, вызов API не выполняется.

2. Заявка или booking item уже не в `Draft`.
   Backend возвращает `REQUEST_NOT_EDITABLE`, frontend показывает сообщение об ошибке и не меняет UI.

3. Booking item не найден или не принадлежит заявке.
   Backend возвращает `NOT_FOUND`.

4. Пользователь удалил последний booking item из draft-заявки.
   Draft-заявка остается существовать как пустой draft и может быть позже дополнена новой техникой или полностью отменена request-level сценарием.

---

## Замечания

1. Сценарий относится только к draft booking item и не применяется после submit заявки.
2. Это не item-level status transition, а удаление item-а из состава draft-заявки через `DELETE`.
3. Если пользователь хочет отменить всю draft-заявку целиком, используется отдельный сценарий `UC-REQ-03`.
