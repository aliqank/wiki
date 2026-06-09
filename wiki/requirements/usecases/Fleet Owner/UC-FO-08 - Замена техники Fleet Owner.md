# UC-FO-08 - Замена техники [`Fleet Owner`](../../Roles%20and%20Access%20Model.md)

**Created:** 2026-06-02  
**Last updated:** 2026-06-08  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка use case

| Поле | Значение |
|---|---|
| Область действия | Страница [`Fleet Owner`](../../Roles%20and%20Access%20Model.md) `Approvals` -> detail / action view конкретной брони |
| Участник | Пользователь с ролью [`FleetOwner`](../../Roles%20and%20Access%20Model.md) |
| Покрываемые FR (BRD) | `FR-046`, `FR-047`, `FR-NEW-19` |
| Покрываемые FR (Additional list) | — |
| Предусловие | Пользователь авторизован в системе; пользователь имеет роль [`FleetOwner`](../../Roles%20and%20Access%20Model.md); бронь относится к fleet-у, по которому у пользователя есть [Fleet Management Access](../../../glossary/Glossary.md#fleet-management-access); бронь еще не перешла в фактическое исполнение и находится в состоянии, допускающем замену техники (`Submitted` или `Confirmed`) |
| Триггер | Нажатие кнопки `Change equipment` в карточке брони |
| Ожидаемый результат | [`Fleet Owner`](../../Roles%20and%20Access%20Model.md) выбирает заменяющую технику из допустимого набора, и бронь обновляется на новую единицу техники без запуска нового approval flow |
| Используемые API | [`GET /approvals/bookings/{id}`](../../../api/booking/fleet-owner/GET_approvals_bookings_id.md), [`GET /approvals/bookings/{id}/replacement-options`](../../../api/booking/fleet-owner/GET_approvals_bookings_id_replacement_options.md), [`POST /approvals/bookings/{id}/change-equipment`](../../../api/booking/fleet-owner/POST_approvals_bookings_id_change_equipment.md) |

---

## Основной сценарий

1. [`Fleet Owner`](../../Roles%20and%20Access%20Model.md) открывает страницу `Approvals` и переходит в detail / action view нужной брони.
2. Frontend загружает актуальные данные брони через [`GET /approvals/bookings/{id}`](../../../api/booking/fleet-owner/GET_approvals_bookings_id.md).
3. Пользователь проверяет контекст брони:
   - requestor;
   - Work Order и приоритет;
   - текущую технику;
   - текущий плановый период;
   - текущий lifecycle status;
   - Work Center, который должен быть сохранен при замене.
4. Пользователь нажимает кнопку `Change equipment`.
5. Frontend открывает окно выбора replacement candidates.
6. Frontend вызывает [`GET /approvals/bookings/{id}/replacement-options`](../../../api/booking/fleet-owner/GET_approvals_bookings_id_replacement_options.md).
7. Backend формирует список replacement candidates по следующим правилам:
   - техника доступна текущему Fleet Owner по [Fleet Management Access](../../../glossary/Glossary.md#fleet-management-access);
   - техника имеет тот же `Work Center`, что и исходная бронь;
   - техника находится в допустимом business context для замены;
   - исходная единица техники не включается в список replacement candidates.
8. Backend проверяет для каждой replacement candidate:
   - отсутствие [hard availability restrictions](../../../glossary/Glossary.md#hard-availability-restriction) на период брони;
   - наличие [booking conflict context](../../../glossary/Glossary.md#booking-conflict-context), если по той же технике уже есть другие активные брони на тот же период.
9. Frontend показывает список replacement candidates с индикаторами conflict context.
10. Пользователь выбирает новую технику.
11. Frontend вызывает [`POST /approvals/bookings/{id}/change-equipment`](../../../api/booking/fleet-owner/POST_approvals_bookings_id_change_equipment.md).
12. Backend проверяет, что:
   - бронь существует;
   - бронь находится в допустимом для замены состоянии (`Submitted` или `Confirmed`);
   - новая техника существует;
   - новая техника имеет тот же `Work Center`, что и исходная бронь;
   - новая техника доступна текущему Fleet Owner по [Fleet Management Access](../../../glossary/Glossary.md#fleet-management-access);
   - [hard availability restrictions](../../../glossary/Glossary.md#hard-availability-restriction) для новой техники отсутствуют.
13. Backend не блокирует замену автоматически только из-за [booking conflict context](../../../glossary/Glossary.md#booking-conflict-context).
14. Backend обновляет `Bookings.equipmentId` и, если требуется, `Bookings.fleetId`.
15. Backend создает запись в `BookingStatuses` с комментарием о замене техники.
16. Backend не возвращает бронь в `Submitted` и не запускает повторное согласование.
17. Frontend обновляет карточку и показывает новую технику в составе брони.

---

## Альтернативные сценарии

1. Текущий статус брони уже не позволяет замену техники.
   Backend возвращает `BOOKING_NOT_CHANGEABLE`, frontend показывает сообщение и обновляет данные брони.

2. Новая техника не найдена.
   Backend возвращает `NOT_FOUND`.

3. Новая техника имеет другой `Work Center`.
   Backend возвращает `VALIDATION_ERROR` или эквивалентную бизнес-ошибку несовместимости replacement candidate.

4. Новая техника недоступна по [hard availability restrictions](../../../glossary/Glossary.md#hard-availability-restriction).
   Backend возвращает `EQUIPMENT_NOT_AVAILABLE`.

5. Для replacement candidate есть [booking conflict context](../../../glossary/Glossary.md#booking-conflict-context).
   Замена техники не блокируется автоматически только из-за overlap по `Bookings`; [`Fleet Owner`](../../Roles%20and%20Access%20Model.md) принимает решение на основании conflict context и бизнес-приоритета.

6. Пользователь не имеет доступа к броне или replacement candidate.
   Backend возвращает `FORBIDDEN`.

---

## Замечания

1. Use case описывает замену техники только до фактического старта исполнения брони.
2. Сценарий требует сохранения `Work Center` исходной брони; replacement candidate с другим `Work Center` не допускается.
3. Для замены техники должно использоваться отдельное read API с replacement candidates, а не generic [`Requestor`](../../Roles%20and%20Access%20Model.md) search flow.
4. [Booking conflict context](../../../glossary/Glossary.md#booking-conflict-context) должен быть показан [`Fleet Owner`](../../Roles%20and%20Access%20Model.md)-у как decision-support, но не как автоматический hard-stop.
5. [Hard availability restrictions](../../../glossary/Glossary.md#hard-availability-restriction) по новой технике должны блокировать замену независимо от наличия или отсутствия других броней.
