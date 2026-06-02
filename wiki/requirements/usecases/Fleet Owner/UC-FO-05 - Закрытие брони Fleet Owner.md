# UC-FO-05 - Закрытие брони Fleet Owner

**Created:** 2026-06-01  
**Last updated:** 2026-06-02  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка use case

| Поле | Значение |
|---|---|
| Область действия | Страница Fleet Owner `Approvals` -> detail / action view активной брони |
| Участник | Пользователь с ролью `FleetOwner` |
| Покрываемые FR (BRD) | `FR-NEW-32`, `FR-NEW-33` |
| Покрываемые FR (Additional list) | `BRD-U-001` |
| Предусловие | Пользователь авторизован в системе; пользователь имеет роль `FleetOwner`; бронь относится к fleet-у пользователя; бронь находится в активном статусе `Confirmed` или `InProgress`; сценарий выполняется для ручного завершения брони |
| Триггер | Нажатие кнопки `Close` в карточке активной брони |
| Ожидаемый результат | Бронь вручную закрыта, фактические даты использования зафиксированы, статус брони переходит в `Closed` с причиной `Completed` |
| Используемые API | [`GET /approvals/bookings/{id}`](../../../api/booking/fleet-owner/GET_approvals_bookings_id.md), [`POST /approvals/bookings/{id}/close`](../../../api/booking/fleet-owner/POST_approvals_bookings_id_close.md) |

---

## Основной сценарий

1. Fleet Owner открывает страницу `Approvals` и переходит в detail / action view активной брони.
2. Frontend загружает актуальные данные брони через [`GET /approvals/bookings/{id}`](../../../api/booking/fleet-owner/GET_approvals_bookings_id.md).
3. Система отображает бронь в активном статусе и показывает действие `Close`.
4. Пользователь проверяет контекст брони:
   - технику;
   - номер заявки;
   - Work Order;
   - текущий статус брони;
   - уже известное или рассчитанное фактическое время начала использования.
5. Пользователь инициирует действие `Close`.
6. Frontend открывает форму ручного закрытия и передает в backend:
   - `actualStartDateTime`;
   - `actualEndDateTime`.
7. Frontend вызывает [`POST /approvals/bookings/{id}/close`](../../../api/booking/fleet-owner/POST_approvals_bookings_id_close.md).
8. Backend проверяет, что:
   - бронь существует;
   - бронь относится к зоне ответственности текущего Fleet Owner;
   - текущий статус позволяет close;
   - `actualStartDateTime <= actualEndDateTime`.
9. Backend переводит бронь в `Closed` с terminal причиной `Completed`.
10. Backend сохраняет `actualStartDateTime` и `actualEndDateTime`.
11. Backend создает запись в `BookingStatuses`.
12. Backend пересчитывает агрегированный статус заявки; если это была последняя активная бронь, заявка также переходит в `Closed` с `requestClosureReason = Completed`.
13. Frontend обновляет карточку и показывает:
   - статус `Closed`;
   - closure reason `Completed`;
   - фактические даты использования.

---

## Альтернативные сценарии

1. Текущий статус брони уже не позволяет close.
   Backend возвращает `BOOKING_NOT_CLOSABLE`; frontend показывает сообщение и обновляет состояние брони.

2. Пользователь не имеет доступа к брони.
   Backend возвращает `FORBIDDEN`.

3. Бронь не найдена.
   Backend возвращает `NOT_FOUND`.

4. Переданы невалидные фактические даты.
   Backend возвращает `VALIDATION_ERROR`.

5. Это последняя активная бронь заявки.
   После close backend дополнительно переводит заявку в `Closed` с `requestClosureReason = Completed`.

---

## Замечания

1. Use case описывает ручное завершение брони Fleet Owner-ом и является следующим базовым шагом после `UC-FO-04`, если работа по брони фактически завершена.
2. Автоматическое закрытие по datetime не допускается.
3. На уровне booking lifecycle завершение выражается через `status = Closed` и `closureReason = Completed`.
4. Если бронь уже стартовала через `UC-FO-04`, `actualStartDateTime` обычно уже известен системе, но в close-flow все равно должен быть передан и/или подтвержден для аналитики.
5. Если закрываемая бронь является последней активной бронью заявки после фактического старта работ, request-level terminal результат должен быть `Closed` + `requestClosureReason = Completed`.
