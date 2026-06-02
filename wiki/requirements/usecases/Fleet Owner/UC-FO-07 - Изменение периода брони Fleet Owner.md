# UC-FO-07 - Изменение периода брони Fleet Owner

**Created:** 2026-06-02  
**Last updated:** 2026-06-02  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка use case

| Поле | Значение |
|---|---|
| Область действия | Страница Fleet Owner `Approvals` -> detail / action view конкретной брони |
| Участник | Пользователь с ролью `FleetOwner` |
| Покрываемые FR (BRD) | `FR-048`, `FR-079a` |
| Покрываемые FR (Additional list) | — |
| Предусловие | Пользователь авторизован в системе; пользователь имеет роль `FleetOwner`; бронь относится к fleet-у, по которому у пользователя есть доступ через `FleetManagePermissions` с типом `Owner` или `Delegated`; текущий статус брони допускает изменение периода |
| Триггер | Нажатие кнопки `Change period` в карточке брони |
| Ожидаемый результат | Плановый период брони изменен Fleet Owner-ом; бронь сохраняет допустимый lifecycle status; Requestor получает уведомление об изменении периода |
| Используемые API | [`GET /approvals/bookings/{id}`](../../../api/booking/fleet-owner/GET_approvals_bookings_id.md), [`POST /approvals/bookings/{id}/change-period`](../../../api/booking/fleet-owner/POST_approvals_bookings_id_change_period.md) |

---

## Основной сценарий

1. Fleet Owner открывает страницу `Approvals` и переходит в detail / action view нужной брони.
2. Frontend загружает актуальные данные брони через [`GET /approvals/bookings/{id}`](../../../api/booking/fleet-owner/GET_approvals_bookings_id.md).
3. Пользователь проверяет контекст брони:
   - requestor;
   - Work Order и приоритет;
   - технику;
   - текущий плановый период;
   - текущий lifecycle status.
4. Пользователь нажимает кнопку `Change period`.
5. Frontend открывает форму редактирования периода.
6. Пользователь вводит новые значения `plannedStartDateTime` и `plannedEndDateTime`.
7. Frontend вызывает [`POST /approvals/bookings/{id}/change-period`](../../../api/booking/fleet-owner/POST_approvals_bookings_id_change_period.md).
8. Backend проверяет, что:
   - бронь существует;
   - бронь относится к зоне ответственности текущего Fleet Owner;
   - текущий статус брони допускает изменение периода;
   - новый диапазон дат валиден;
   - hard-ограничения доступности техники не нарушены.
9. Backend рассчитывает пересечения с другими активными бронями как conflict context, но сами по себе такие пересечения не блокируют изменение периода.
10. Backend обновляет `plannedStartDateTime` и `plannedEndDateTime`.
11. Backend создает запись в `BookingStatuses` с комментарием об изменении периода.
12. Backend не переводит бронь в новый approval lifecycle status и не запускает повторное согласование.
13. Backend инициирует уведомление Requestor-а об изменении периода.
14. Frontend обновляет карточку и показывает новый плановый диапазон брони.

---

## Альтернативные сценарии

1. Текущий статус брони уже не позволяет менять период.
   Backend возвращает `BOOKING_NOT_CHANGEABLE`, frontend показывает сообщение и обновляет данные брони.

2. Новый диапазон дат невалиден.
   Backend возвращает `VALIDATION_ERROR`.

3. Новый диапазон нарушает hard-ограничения доступности техники.
   Backend возвращает `EQUIPMENT_NOT_AVAILABLE`.

4. У брони есть конфликты с другими активными бронями на новый период.
   Изменение периода не блокируется автоматически только из-за competing bookings. Fleet Owner принимает решение на основании conflict context и бизнес-приоритета.

5. Пользователь не имеет доступа к брони.
   Backend возвращает `FORBIDDEN`.

6. Бронь не найдена.
   Backend возвращает `NOT_FOUND`.

---

## Замечания

1. Use case покрывает изменение периода как до подтверждения, так и после подтверждения, если бронь еще находится в допустимом для изменения состоянии.
2. Изменение периода не должно требовать повторного подтверждения Fleet Owner-ом и не должно возвращать бронь в lifecycle `Submitted`.
3. Уведомление Requestor-а об изменении периода является обязательным side effect согласно `FR-079a`.
4. Пересечения с другими активными бронями должны показываться как conflict context, но не являются автоматическим hard-stop для действия `Change period`.
