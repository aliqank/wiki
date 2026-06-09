# UC-FO-04 - Мобилизация техники

**Created:** 2026-06-01  
**Last updated:** 2026-06-02  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка use case

| Поле | Значение |
|---|---|
| Область действия | Страница [`Fleet Owner`](../../Roles%20and%20Access%20Model.md) `Approvals` -> detail / action view подтвержденной брони |
| Участник | Пользователь с ролью [`FleetOwner`](../../Roles%20and%20Access%20Model.md) |
| Покрываемые FR (BRD) | `FR-NEW-18`, `FR-NEW-24`, `FR-074` |
| Покрываемые FR (Additional list) | — |
| Предусловие | Пользователь авторизован в системе; пользователь имеет роль [`FleetOwner`](../../Roles%20and%20Access%20Model.md); бронь относится к fleet-у пользователя; бронь уже находится в статусе `Confirmed`; сценарий выполняется после позитивного confirm-сценария и до фактического завершения брони |
| Триггер | Нажатие кнопки `Mobilization started` в карточке подтвержденной брони |
| Ожидаемый результат | Система фиксирует фактическое время начала мобилизации, а бронь переходит в статус `InProgress` |
| Используемые API | [`GET /approvals/bookings/{id}`](../../../api/booking/fleet-owner/GET_approvals_bookings_id.md), [`POST /approvals/bookings/{id}/mobilization-start`](../../../api/booking/fleet-owner/POST_approvals_bookings_id_mobilization_start.md) |

---

## Основной сценарий

1. [`Fleet Owner`](../../Roles%20and%20Access%20Model.md) открывает страницу `Approvals` и переходит в detail / action view ранее подтвержденной брони.
2. Frontend загружает актуальные данные брони через [`GET /approvals/bookings/{id}`](../../../api/booking/fleet-owner/GET_approvals_bookings_id.md).
3. Система отображает бронь в статусе `Confirmed` и показывает действие `Mobilization started`.
4. Пользователь проверяет контекст брони:
   - технику;
   - номер заявки;
   - Work Order;
   - плановый период;
   - текущий статус брони.
5. Пользователь нажимает кнопку `Mobilization started`.
6. Frontend вызывает [`POST /approvals/bookings/{id}/mobilization-start`](../../../api/booking/fleet-owner/POST_approvals_bookings_id_mobilization_start.md).
7. Backend проверяет, что:
   - бронь существует;
   - бронь относится к зоне ответственности текущего [`Fleet Owner`](../../Roles%20and%20Access%20Model.md);
   - текущий статус брони равен `Confirmed`;
   - бронь еще не была переведена в фактический старт использования.
8. Backend устанавливает `actualStartDateTime`:
   - из request body, если значение передано;
   - либо текущим backend-time, если значение не передано.
9. Backend переводит бронь в статус `InProgress`.
10. Backend создает запись в `BookingStatuses`.
11. Backend пересчитывает агрегированный статус заявки; если это первая бронь заявки, перешедшая в `InProgress`, заявка также переходит в `InProgress`.
12. Backend возвращает обновленные данные по фактическому старту брони.
13. Frontend обновляет карточку и показывает:
   - статус `InProgress`;
   - зафиксированное `actualStartDateTime`;
   - отсутствие или disabled-state для повторного выполнения действия `Mobilization started`.

---

## Альтернативные сценарии

1. Текущий статус брони уже не позволяет начать мобилизацию.
   Backend возвращает `BOOKING_NOT_STARTABLE`; frontend показывает сообщение и обновляет состояние брони.

2. Пользователь не имеет доступа к брони.
   Backend возвращает `FORBIDDEN`.

3. Бронь не найдена.
   Backend возвращает `NOT_FOUND`.

4. Frontend не передает `actualStartDateTime`.
   Backend использует текущее системное время и все равно успешно фиксирует старт мобилизации.

5. Пользователь повторно пытается выполнить действие для уже стартовавшей брони.
   Backend отклоняет повторный запуск через `BOOKING_NOT_STARTABLE` или эквивалентное conflict-поведение.

---

## Замечания

1. Данный use case выполняется после базового позитивного сценария подтверждения и не заменяет сам confirm-flow.
2. Начало периода бронирования считается от момента начала мобилизации, а не от прибытия техники на площадку.
3. Use case применим только к броням, которые уже достигли статуса `Confirmed`; для long-term rented брони, ожидающей финального Supervisor approval, данный сценарий еще не должен быть доступен.
4. `actualStartDateTime` является обязательным на уровне результата бизнес-действия, но значение может быть определено backend-ом автоматически, если frontend не передал его явно.
5. Для базового сценария отдельный новый write API не требуется: покрытие уже обеспечено методом [`POST /approvals/bookings/{id}/mobilization-start`](../../../api/booking/fleet-owner/POST_approvals_bookings_id_mobilization_start.md).
6. На уровне request lifecycle старт первой брони в `InProgress` должен приводить к пересчету `BookingRequest.status` в `InProgress` по `FR-074` и актуальным aggregated request status rules.
