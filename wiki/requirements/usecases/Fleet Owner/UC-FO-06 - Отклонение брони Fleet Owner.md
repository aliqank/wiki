# UC-FO-06 - Отклонение брони Fleet Owner

**Created:** 2026-06-02  
**Last updated:** 2026-06-02  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка use case

| Поле | Значение |
|---|---|
| Область действия | Страница Fleet Owner `Approvals` -> view `Requests` -> detail / action view конкретной брони |
| Участник | Пользователь с ролью `FleetOwner` |
| Покрываемые FR (BRD) | `FR-043`, `FR-049`, `FR-050`, `FR-064` |
| Покрываемые FR (Additional list) | `BRD-U-001` |
| Предусловие | Пользователь авторизован в системе; пользователь имеет роль `FleetOwner`; бронь относится к fleet-у, по которому у пользователя есть [Fleet Management Access](../../../glossary/Glossary.md#fleet-management-access); бронь находится в статусе `Submitted`; сценарий выполняется до фактического старта работ |
| Триггер | Нажатие кнопки `Decline` в карточке брони, открытой из request view |
| Ожидаемый результат | Бронь отклонена Fleet Owner-ом и переходит в terminal status `Closed` с `closureReason = Declined`; агрегированный статус заявки пересчитан |
| Используемые API | [`GET /approvals/bookings/{id}`](../../../api/booking/fleet-owner/GET_approvals_bookings_id.md), [`POST /approvals/bookings/{id}/decline`](../../../api/booking/fleet-owner/POST_approvals_bookings_id_decline.md) |

---

## Основной сценарий

1. Fleet Owner открывает страницу `Approvals` во view `Requests` и выбирает заявку из списка.
2. Пользователь открывает внутри заявки detail / action view конкретной релевантной брони.
3. Frontend загружает актуальные данные брони через [`GET /approvals/bookings/{id}`](../../../api/booking/fleet-owner/GET_approvals_bookings_id.md).
4. Пользователь проверяет ключевые данные брони:
   - requestor;
   - Work Order и приоритет;
   - технику;
   - плановый период;
   - контекст заявки и причину, по которой бронь должна быть отклонена.
5. Пользователь нажимает кнопку `Decline`.
6. Frontend открывает форму decline и требует обязательную причину отклонения.
7. Пользователь вводит `reason` и подтверждает действие.
8. Frontend вызывает [`POST /approvals/bookings/{id}/decline`](../../../api/booking/fleet-owner/POST_approvals_bookings_id_decline.md).
9. Backend проверяет, что:
   - бронь существует;
   - бронь относится к зоне ответственности текущего Fleet Owner;
   - текущий статус равен `Submitted`;
   - передана непустая причина decline.
10. Backend переводит бронь в terminal state `Closed` с `closureReason = Declined`.
11. Backend создает запись шага согласования в `BookingApprovals` с типом `FoApproval`, результатом `Declined`, `order = 1` и комментарием, равным `reason`.
12. Backend создает запись в `BookingStatuses`.
13. Backend пересчитывает агрегированный статус родительской заявки:
   - если после decline в заявке остаются активные booking item-ы, request status пересчитывается по общим aggregated rules и остается активным;
   - если отклоненная бронь была последней активной и заявка ни разу не переходила в `InProgress`, заявка переходит в `Closed` с `requestClosureReason = Cancelled`;
   - если отклоненная бронь была последней активной и заявка ранее уже была в `InProgress`, заявка переходит в `Closed` с `requestClosureReason = Completed`.
14. Frontend обновляет карточку и показывает:
   - статус брони `Closed`;
   - closure reason `Declined`;
   - зафиксированную причину decline.
15. Система обновляет request view так, чтобы отклоненная бронь больше не отображалась как ожидающая решения Fleet Owner.

---

## Альтернативные сценарии

1. Текущий статус брони уже не позволяет decline.
   Backend возвращает `BOOKING_NOT_DECLINABLE`, frontend показывает сообщение и обновляет данные брони.

2. Пользователь не передал причину decline.
   Backend возвращает `VALIDATION_ERROR`, frontend подсвечивает обязательность поля `reason`.

3. Пользователь не имеет доступа к брони.
   Backend возвращает `FORBIDDEN`.

4. Бронь не найдена.
   Backend возвращает `NOT_FOUND`.

5. Отклоняемая бронь является последней активной в заявке.
   После decline backend дополнительно пересчитывает request-level terminal результат по aggregated rules: `Closed + Cancelled` или `Closed + Completed` в зависимости от того, была ли заявка ранее в `InProgress`.

---

## Замечания

1. Use case покрывает именно decline-решение со стороны Fleet Owner на этапе `Submitted` и не применяется после перехода брони в `Confirmed` или `InProgress`.
2. Для long-term rented брони отказ Fleet Owner завершает сценарий сразу: Supervisor approval step не создается.
3. На уровне booking lifecycle отклонение выражается через `status = Closed` и `closureReason = Declined`.
4. На уровне request lifecycle отклонение одной брони не означает автоматическое закрытие всей заявки; ключевым условием является наличие или отсутствие других активных booking item-ов.
5. Если после decline активных booking item-ов не осталось, request-level результат должен определяться по `Aggregated Request Status Rules.md`, а не выводиться только из closure reason отклоненной брони.
