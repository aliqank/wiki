# Use Cases

**Created:** 2026-05-15  
**Last updated:** 2026-06-02  
**Автор документов:** Telman Nurzhanov (SA)

---

Раздел содержит финальные use case-сценарии, подготовленные для разработки и детализации UI/API-потока.

Связанный документ с общими правилами статусов:

- [`../Aggregated Request Status Rules.md`](../Aggregated%20Request%20Status%20Rules.md)

## Список use cases

| ID | Наименование | Область действия | Покрываемые FR | Основные API |
|---|---|---|---|---|
| `UC-REQ-01` | Просмотр моих заявок | Страница `Мои заявки` | `BRD: FR-025, FR-091` | [`GET /booking-requests/my`](../../api/booking/requestor/GET_booking_requests_my.md), [`GET /booking-requests/{id}`](../../api/booking/requestor/GET_booking_requests_id.md) |
| `UC-REQ-02` | Создание новой заявки (Draft-first) | Модальное окно `Новая заявка` | `BRD: FR-023, FR-027, FR-030, FR-031, FR-038, FR-040, FR-NEW-04, FR-NEW-08, FR-NEW-11, FR-NEW-12, FR-NEW-13, FR-NEW-14, FR-NEW-15, FR-NEW-16, FR-NEW-17, FR-NEW-38, FR-NEW-39, FR-NEW-48, FR-NEW-50, FR-NEW-51, FR-NEW-68, FR-NEW-71` | [`POST /booking-requests`](../../api/booking/requestor/POST_booking_requests.md), [`PATCH /booking-requests/{id}`](../../api/booking/requestor/PATCH_booking_requests_id.md), [`GET /equipment/search`](../../api/booking/requestor/GET_equipment_search.md), [`POST /booking-requests/{id}/items`](../../api/booking/requestor/POST_booking_requests_id_items.md), [`POST /booking-requests/{id}/submit`](../../api/booking/requestor/POST_booking_requests_id_submit.md) |
| `UC-REQ-02.1` | Создание пустого draft заявки | Кнопка `Создать заявку` | `BRD: FR-023, FR-027, FR-030` | [`POST /booking-requests`](../../api/booking/requestor/POST_booking_requests.md) |
| `UC-REQ-02.2` | Заполнение и редактирование шапки заявки | Модальное окно `Новая заявка` | `BRD: FR-027, FR-NEW-11, FR-NEW-12, FR-NEW-13, FR-NEW-14, FR-NEW-48, FR-NEW-51` | [`PATCH /booking-requests/{id}`](../../api/booking/requestor/PATCH_booking_requests_id.md) |
| `UC-REQ-02.3` | Поиск техники для добавления в заявку | Окно `Добавить технику` | `BRD: FR-031, FR-040, FR-NEW-38, FR-NEW-39, FR-NEW-50, FR-NEW-68` | [`GET /equipment/search`](../../api/booking/requestor/GET_equipment_search.md), [`GET /reference/equipment-types`](../../api/booking/reference/GET_reference_equipment_types.md), [`GET /reference/fleets`](../../api/booking/reference/GET_reference_fleets.md), [`GET /reference/work-centers`](../../api/booking/reference/GET_reference_work_centers.md), [`GET /reference/ownership-types`](../../api/booking/reference/GET_reference_ownership_types.md), [`GET /reference/share-types`](../../api/booking/reference/GET_reference_share_types.md), [`GET /reference/equipment-types/{equipmentTypeId}/properties`](../../api/booking/reference/GET_reference_equipment_types_id_properties.md) |
| `UC-REQ-02.3.1` | Просмотр load summary при выборе техники | Окно `Добавить технику` | `BRD: FR-NEW-70` | [`GET /equipment/{id}/load-summary`](../../api/booking/requestor/GET_equipment_id_load_summary.md) |
| `UC-REQ-02.4` | Добавление техники в draft-заявку | Окно `Добавить технику` | `BRD: FR-031, FR-038, FR-NEW-04, FR-NEW-08, FR-NEW-15, FR-NEW-71` | [`POST /booking-requests/{id}/items`](../../api/booking/requestor/POST_booking_requests_id_items.md) |
| `UC-REQ-02.5` | Редактирование брони или замена техники в draft | Карточка booking item в draft | `BRD: FR-027, FR-031, FR-038, FR-040, FR-NEW-04, FR-NEW-08, FR-NEW-71` | [`PATCH /booking-requests/{id}/items/{bookingId}`](../../api/booking/requestor/PATCH_booking_requests_id_items_bookingId.md), [`GET /equipment/search`](../../api/booking/requestor/GET_equipment_search.md) |
| `UC-REQ-02.6` | Отправка draft-заявки | Модальное окно `Новая заявка` | `BRD: FR-023, FR-027, FR-NEW-08, FR-NEW-16, FR-NEW-17, FR-NEW-71` | [`POST /booking-requests/{id}/submit`](../../api/booking/requestor/POST_booking_requests_id_submit.md) |
| `UC-REQ-02.7` | Отмена draft-брони | Карточка booking item в draft | `BRD: FR-027, FR-031, FR-038` | [`DELETE /booking-requests/{id}/items/{bookingId}`](../../api/booking/requestor/DELETE_booking_requests_id_items_bookingId.md), [`GET /booking-requests/{id}`](../../api/booking/requestor/GET_booking_requests_id.md) |
| `UC-REQ-03` | Отмена draft-заявки requestor-ом | Страница `Мои заявки` / detail view draft-заявки | `BRD: FR-025, FR-027` | [`POST /booking-requests/{id}/cancel`](../../api/booking/requestor/POST_booking_requests_id_cancel.md), [`GET /booking-requests/my`](../../api/booking/requestor/GET_booking_requests_my.md), [`GET /booking-requests/{id}`](../../api/booking/requestor/GET_booking_requests_id.md) |
| `UC-REQ-04` | Отзыв брони requestor-ом | Страница `Мои заявки` / карточка заявки / карточка booking item в отправленной заявке | `BRD: FR-025, FR-042, FR-058, FR-068, FR-078, FR-NEW-17` | [`GET /booking-requests/my`](../../api/booking/requestor/GET_booking_requests_my.md), [`POST /bookings/{id}/revoke`](../../api/booking/requestor/POST_bookings_id_revoke.md), [`GET /booking-requests/{id}`](../../api/booking/requestor/GET_booking_requests_id.md) |
| `UC-REQ-05` | Отзыв submitted-заявки requestor-ом | Страница `Мои заявки` / карточка submitted-заявки | `BRD: FR-025, FR-058, FR-NEW-17; Additional: BRD-U-003` | [`GET /booking-requests/my`](../../api/booking/requestor/GET_booking_requests_my.md), [`GET /booking-requests/{id}`](../../api/booking/requestor/GET_booking_requests_id.md), request-level withdraw API / orchestration over [`POST /bookings/{id}/revoke`](../../api/booking/requestor/POST_bookings_id_revoke.md) *(TBD)* |
| `UC-REQ-06` | Изменение плановой даты и времени окончания брони requestor-ом | Страница `Мои заявки` / detail view заявки / карточка booking item | `BRD: FR-061, FR-067, FR-081` | [`POST /bookings/{id}/extend`](../../api/booking/requestor/POST_bookings_id_extend.md), [`GET /booking-requests/{id}`](../../api/booking/requestor/GET_booking_requests_id.md), [`GET /booking-requests/my`](../../api/booking/requestor/GET_booking_requests_my.md) |
| `UC-REQ-07` | Отмена draft-брони requestor-ом со страницы Мои заявки | Страница `Мои заявки` / summary-блок броней внутри draft-заявки | `BRD: FR-025, FR-027, FR-031, FR-038` | [`GET /booking-requests/my`](../../api/booking/requestor/GET_booking_requests_my.md), [`DELETE /booking-requests/{id}/items/{bookingId}`](../../api/booking/requestor/DELETE_booking_requests_id_items_bookingId.md), [`GET /booking-requests/{id}`](../../api/booking/requestor/GET_booking_requests_id.md) |
| `UC-FO-01` | Просмотр списка заявок Fleet Owner | Страница `Approvals` → view `Requests` | `BRD: FR-092, FR-094, FR-043; Additional: BRD-U-001` | [`GET /approvals/requests`](../../api/booking/fleet-owner/GET_approvals_requests.md) |
| `UC-FO-02` | Просмотр load summary Fleet Owner | Страница `Approvals` → view `Requests` → detail / action view брони | `BRD: FR-NEW-64` | [`GET /approvals/bookings/{id}/load-summary`](../../api/booking/fleet-owner/GET_approvals_bookings_id_load_summary.md) |
| `UC-FO-03` | Подтверждение брони Fleet Owner (базовый сценарий) | Страница `Approvals` → view `Requests` → detail / action view брони | `BRD: FR-043, FR-045, FR-063` | [`GET /approvals/bookings/{id}`](../../api/booking/fleet-owner/GET_approvals_bookings_id.md), [`POST /approvals/bookings/{id}/confirm`](../../api/booking/fleet-owner/POST_approvals_bookings_id_confirm.md) |
| `UC-FO-04` | Мобилизация техники | Страница `Approvals` → detail / action view подтвержденной брони | `BRD: FR-NEW-18, FR-NEW-24` | [`GET /approvals/bookings/{id}`](../../api/booking/fleet-owner/GET_approvals_bookings_id.md), [`POST /approvals/bookings/{id}/mobilization-start`](../../api/booking/fleet-owner/POST_approvals_bookings_id_mobilization_start.md) |
| `UC-FO-05` | Закрытие брони Fleet Owner | Страница `Approvals` → detail / action view активной брони | `BRD: FR-NEW-32, FR-NEW-33; Additional: BRD-U-001` | [`GET /approvals/bookings/{id}`](../../api/booking/fleet-owner/GET_approvals_bookings_id.md), [`POST /approvals/bookings/{id}/close`](../../api/booking/fleet-owner/POST_approvals_bookings_id_close.md) |
| `UC-FO-06` | Отклонение брони Fleet Owner | Страница `Approvals` → view `Requests` → detail / action view брони | `BRD: FR-043, FR-049, FR-050, FR-064; Additional: BRD-U-001` | [`GET /approvals/bookings/{id}`](../../api/booking/fleet-owner/GET_approvals_bookings_id.md), [`POST /approvals/bookings/{id}/decline`](../../api/booking/fleet-owner/POST_approvals_bookings_id_decline.md) |
| `UC-FO-07` | Изменение периода брони Fleet Owner | Страница `Approvals` → detail / action view брони | `BRD: FR-048, FR-079a` | [`GET /approvals/bookings/{id}`](../../api/booking/fleet-owner/GET_approvals_bookings_id.md), [`POST /approvals/bookings/{id}/change-period`](../../api/booking/fleet-owner/POST_approvals_bookings_id_change_period.md) |
| `UC-FO-08` | Замена техники Fleet Owner | Страница `Approvals` → detail / action view брони | `BRD: FR-046, FR-047` | [`GET /approvals/bookings/{id}`](../../api/booking/fleet-owner/GET_approvals_bookings_id.md), [`GET /approvals/bookings/{id}/replacement-options`](../../api/booking/fleet-owner/GET_approvals_bookings_id_replacement_options.md), [`POST /approvals/bookings/{id}/change-equipment`](../../api/booking/fleet-owner/POST_approvals_bookings_id_change_equipment.md) |
| `UC-FO-09` | Согласование изменения плановой даты и времени окончания брони | Страница `Approvals` → detail / action view брони с pending extension request | `BRD: FR-061, FR-067, FR-081` | [`GET /approvals/bookings/{id}`](../../api/booking/fleet-owner/GET_approvals_bookings_id.md), [`POST /approvals/bookings/{id}/extension-approval`](../../api/booking/fleet-owner/POST_approvals_bookings_id_extension_approval.md) |

---

## Файлы

- [`Requestor/UC-REQ-01 - Просмотр моих заявок.md`](Requestor/UC-REQ-01%20-%20Просмотр%20моих%20заявок.md)
- [`Requestor/UC-REQ-02 - Создание новой заявки (Draft-first)/UC-REQ-02 - Создание новой заявки (Draft-first).md`](Requestor/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first)/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first).md)
- [`Requestor/UC-REQ-02 - Создание новой заявки (Draft-first)/UC-REQ-02.1 - Создание пустого draft заявки.md`](Requestor/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first)/UC-REQ-02.1%20-%20Создание%20пустого%20draft%20заявки.md)
- [`Requestor/UC-REQ-02 - Создание новой заявки (Draft-first)/UC-REQ-02.2 - Заполнение и редактирование шапки заявки.md`](Requestor/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first)/UC-REQ-02.2%20-%20Заполнение%20и%20редактирование%20шапки%20заявки.md)
- [`Requestor/UC-REQ-02 - Создание новой заявки (Draft-first)/UC-REQ-02.3 - Поиск техники для добавления в заявку.md`](Requestor/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first)/UC-REQ-02.3%20-%20Поиск%20техники%20для%20добавления%20в%20заявку.md)
- [`Requestor/UC-REQ-02 - Создание новой заявки (Draft-first)/UC-REQ-02.3.1 - Просмотр load summary при выборе техники.md`](Requestor/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first)/UC-REQ-02.3.1%20-%20Просмотр%20load%20summary%20при%20выборе%20техники.md)
- [`Requestor/UC-REQ-02 - Создание новой заявки (Draft-first)/UC-REQ-02.4 - Добавление техники в draft-заявку.md`](Requestor/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first)/UC-REQ-02.4%20-%20Добавление%20техники%20в%20draft-заявку.md)
- [`Requestor/UC-REQ-02 - Создание новой заявки (Draft-first)/UC-REQ-02.5 - Редактирование брони или замена техники в draft.md`](Requestor/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first)/UC-REQ-02.5%20-%20Редактирование%20брони%20или%20замена%20техники%20в%20draft.md)
- [`Requestor/UC-REQ-02 - Создание новой заявки (Draft-first)/UC-REQ-02.6 - Отправка draft-заявки.md`](Requestor/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first)/UC-REQ-02.6%20-%20Отправка%20draft-заявки.md)
- [`Requestor/UC-REQ-02 - Создание новой заявки (Draft-first)/UC-REQ-02.7 - Отмена draft-брони.md`](Requestor/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first)/UC-REQ-02.7%20-%20Отмена%20draft-брони.md)
- [`Requestor/UC-REQ-03 - Отмена draft-заявки requestor-ом.md`](Requestor/UC-REQ-03%20-%20Отмена%20draft-заявки%20requestor-ом.md)
- [`Requestor/UC-REQ-04 - Отзыв брони requestor-ом.md`](Requestor/UC-REQ-04%20-%20Отзыв%20брони%20requestor-ом.md)
- [`Requestor/UC-REQ-05 - Отзыв submitted-заявки requestor-ом.md`](Requestor/UC-REQ-05%20-%20Отзыв%20submitted-заявки%20requestor-ом.md)
- [`Requestor/UC-REQ-06 - Изменение плановой даты и времени окончания брони requestor-ом.md`](Requestor/UC-REQ-06%20-%20Изменение%20плановой%20даты%20и%20времени%20окончания%20брони%20requestor-ом.md)
- [`Requestor/UC-REQ-07 - Отмена draft-брони requestor-ом со страницы Мои заявки.md`](Requestor/UC-REQ-07%20-%20Отмена%20draft-брони%20requestor-ом%20со%20страницы%20Мои%20заявки.md)
- [`Fleet Owner/UC-FO-01 - Просмотр списка заявок Fleet Owner.md`](Fleet%20Owner/UC-FO-01%20-%20Просмотр%20списка%20заявок%20Fleet%20Owner.md)
- [`Fleet Owner/UC-FO-02 - Просмотр load summary Fleet Owner.md`](Fleet%20Owner/UC-FO-02%20-%20Просмотр%20load%20summary%20Fleet%20Owner.md)
- [`Fleet Owner/UC-FO-03 - Подтверждение брони Fleet Owner (базовый сценарий).md`](Fleet%20Owner/UC-FO-03%20-%20Подтверждение%20брони%20Fleet%20Owner%20(базовый%20сценарий).md)
- [`Fleet Owner/UC-FO-04 - Мобилизация техники.md`](Fleet%20Owner/UC-FO-04%20-%20Мобилизация%20техники.md)
- [`Fleet Owner/UC-FO-05 - Закрытие брони Fleet Owner.md`](Fleet%20Owner/UC-FO-05%20-%20Закрытие%20брони%20Fleet%20Owner.md)
- [`Fleet Owner/UC-FO-06 - Отклонение брони Fleet Owner.md`](Fleet%20Owner/UC-FO-06%20-%20Отклонение%20брони%20Fleet%20Owner.md)
- [`Fleet Owner/UC-FO-07 - Изменение периода брони Fleet Owner.md`](Fleet%20Owner/UC-FO-07%20-%20Изменение%20периода%20брони%20Fleet%20Owner.md)
- [`Fleet Owner/UC-FO-08 - Замена техники Fleet Owner.md`](Fleet%20Owner/UC-FO-08%20-%20Замена%20техники%20Fleet%20Owner.md)
- [`Fleet Owner/UC-FO-09 - Согласование изменения плановой даты и времени окончания брони.md`](Fleet%20Owner/UC-FO-09%20-%20Согласование%20изменения%20плановой%20даты%20и%20времени%20окончания%20брони.md)
