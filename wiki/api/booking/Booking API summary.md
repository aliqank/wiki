# Booking API summary

**Created:** 2026-05-14  
**Last updated:** 2026-06-02  
**Автор документов:** Telman Nurzhanov (SA)

---

## Назначение

Сводная таблица endpoint-ов для booking-блока на основе:
- [`source/results/2026-05-05 - HDV HDE BRD v13.md`](../../../source/results/2026-05-05%20-%20HDV%20HDE%20BRD%20v13.md)
- [`wiki/db/2026-05-21 - DB Schema v12 (Azure SQL, Equipments, Booking).md`](../db/2026-05-21%20-%20DB%20Schema%20v12%20(Azure%20SQL,%20Equipments,%20Booking).md)

Base URL: `/api/booking/v1`

---

## 1. Requestor / SWP

| Метод | Путь | Назначение |
|---|---|---|
| `GET` | [`/equipment/search`](requestor/GET_equipment_search.md) | Поиск техники для создания заявки |
| `GET` | [`/equipment/{id}`](requestor/GET_equipment_id.md) | Получить карточку техники в booking-контексте |
| `GET` | [`/equipment/{id}/load-summary`](requestor/GET_equipment_id_load_summary.md) | Получить загрузку техники на выбранный период |
| `GET` | [`/reference/equipment-types`](reference/GET_reference_equipment_types.md) | Получить справочник типов техники для фильтра поиска |
| `GET` | [`/reference/fleets`](reference/GET_reference_fleets.md) | Получить справочник fleet-ов для фильтра поиска |
| `GET` | [`/reference/work-centers`](reference/GET_reference_work_centers.md) | Получить справочник work centers для фильтра поиска |
| `GET` | [`/reference/ownership-types`](reference/GET_reference_ownership_types.md) | Получить справочник ownership types для фильтра поиска |
| `GET` | [`/reference/share-types`](reference/GET_reference_share_types.md) | Получить справочник share types для фильтра поиска |
| `GET` | [`/reference/request-statuses`](reference/GET_reference_request_statuses.md) | Получить справочник статусов заявки для фильтра `Мои заявки` |
| `GET` | [`/reference/requestor-request-types`](reference/GET_reference_requestor_request_types.md) | Получить справочник типов заявки для фильтра `Мои заявки` |
| `GET` | [`/reference/request-priorities`](reference/GET_reference_request_priorities.md) | Получить справочник приоритетов заявки для фильтра `Мои заявки` |
| `GET` | [`/reference/approval-request-statuses`](reference/GET_reference_approval_request_statuses.md) | Получить справочник статусов заявки для Fleet Owner view `Approvals -> Requests` |
| `GET` | [`/reference/approval-request-types`](reference/GET_reference_approval_request_types.md) | Получить справочник типов заявки для Fleet Owner view `Approvals -> Requests` |
| `GET` | [`/reference/equipment-types/{equipmentTypeId}/properties`](reference/GET_reference_equipment_types_id_properties.md) | Получить динамические свойства выбранного типа техники |
| `POST` | [`/booking-requests`](requestor/POST_booking_requests.md) | Создать черновик заявки |
| `GET` | [`/booking-requests/{id}`](requestor/GET_booking_requests_id.md) | Получить детали заявки |
| `GET` | [`/booking-requests/my`](requestor/GET_booking_requests_my.md) | Получить список собственных заявок |
| `PATCH` | [`/booking-requests/{id}`](requestor/PATCH_booking_requests_id.md) | Обновить черновик заявки |
| `POST` | [`/booking-requests/{id}/submit`](requestor/POST_booking_requests_id_submit.md) | Отправить заявку |
| `POST` | [`/booking-requests/{id}/cancel`](requestor/POST_booking_requests_id_cancel.md) | Отменить черновик |
| `POST` | [`/booking-requests/{id}/items`](requestor/POST_booking_requests_id_items.md) | Добавить booking item в черновик |
| `PATCH` | [`/booking-requests/{id}/items/{bookingId}`](requestor/PATCH_booking_requests_id_items_bookingId.md) | Обновить booking item в черновике |
| `DELETE` | [`/booking-requests/{id}/items/{bookingId}`](requestor/DELETE_booking_requests_id_items_bookingId.md) | Удалить booking item из черновика |
| `POST` | [`/bookings/{id}/revoke`](requestor/POST_bookings_id_revoke.md) | Отозвать бронь до решения FO |
| `POST` | [`/bookings/{id}/extend`](requestor/POST_bookings_id_extend.md) | Запросить изменение плановой даты и времени окончания брони |
| `POST` | [`/bookings/{id}/terminate`](requestor/POST_bookings_id_terminate.md) | Досрочно завершить подтвержденную бронь |
| `POST` | [`/bookings/{id}/close`](requestor/POST_bookings_id_close.md) | Закрыть бронь вручную |
| `POST` | [`/bookings/{id}/feedback`](requestor/POST_bookings_id_feedback.md) | Оставить отзыв по технике |
| `GET` | [`/booking-requests/history`](requestor/GET_booking_requests_history.md) | Получить историю завершенных заявок |

---

## 2. Fleet Owner

| Метод | Путь | Назначение |
|---|---|---|
| `GET` | [`/approvals/bookings`](fleet-owner/GET_approvals_bookings.md) | Очередь броней на согласование |
| `GET` | [`/approvals/requests`](fleet-owner/GET_approvals_requests.md) | Список заявок с релевантными бронями для Fleet Owner |
| `GET` | [`/reference/approval-request-statuses`](reference/GET_reference_approval_request_statuses.md) | Получить справочник статусов заявки для Fleet Owner view `Requests` |
| `GET` | [`/reference/approval-request-types`](reference/GET_reference_approval_request_types.md) | Получить справочник типов заявки для Fleet Owner view `Requests` |
| `GET` | [`/approvals/bookings/{id}`](fleet-owner/GET_approvals_bookings_id.md) | Детали брони для FO |
| `GET` | [`/approvals/bookings/{id}/replacement-options`](fleet-owner/GET_approvals_bookings_id_replacement_options.md) | Получить replacement candidates для замены техники |
| `GET` | [`/approvals/bookings/{id}/load-summary`](fleet-owner/GET_approvals_bookings_id_load_summary.md) | Загрузка техники на даты в окне подтверждения |
| `POST` | [`/approvals/bookings/{id}/confirm`](fleet-owner/POST_approvals_bookings_id_confirm.md) | Подтвердить бронь |
| `POST` | [`/approvals/bookings/{id}/decline`](fleet-owner/POST_approvals_bookings_id_decline.md) | Отклонить бронь |
| `POST` | [`/approvals/bookings/{id}/extension-approval`](fleet-owner/POST_approvals_bookings_id_extension_approval.md) | Принять решение по изменению `plannedEndDateTime` |
| `POST` | [`/approvals/bookings/{id}/change-equipment`](fleet-owner/POST_approvals_bookings_id_change_equipment.md) | Заменить технику в брони |
| `POST` | [`/approvals/bookings/{id}/change-period`](fleet-owner/POST_approvals_bookings_id_change_period.md) | Изменить период брони |
| `POST` | [`/approvals/bookings/{id}/mobilization-start`](fleet-owner/POST_approvals_bookings_id_mobilization_start.md) | Зафиксировать начало мобилизации |
| `POST` | [`/approvals/bookings/{id}/close`](fleet-owner/POST_approvals_bookings_id_close.md) | Закрыть бронь вручную как Fleet Owner |
| `POST` | [`/approvals/bookings/{id}/terminate`](fleet-owner/POST_approvals_bookings_id_terminate.md) | Досрочно завершить бронь |
| `GET` | [`/approvals/closed`](fleet-owner/GET_approvals_closed.md) | История closed / обработанных согласований |

---

## 3. FleetOwners' Supervisor

| Метод | Путь | Назначение |
|---|---|---|
| `GET` | [`/supervisor/bookings`](supervisor/GET_supervisor_bookings.md) | Очередь long-term rented броней с pending шагом `SupervisorApproval` |
| `GET` | [`/supervisor/bookings/{id}`](supervisor/GET_supervisor_bookings_id.md) | Детали брони для финального решения |
| `POST` | [`/supervisor/bookings/{id}/confirm`](supervisor/POST_supervisor_bookings_id_confirm.md) | Финально подтвердить бронь |
| `POST` | [`/supervisor/bookings/{id}/decline`](supervisor/POST_supervisor_bookings_id_decline.md) | Отклонить бронь с обязательным комментарием |

---

## 4. Transportation Responsible

| Метод | Путь | Назначение |
|---|---|---|
| `GET` | [`/transport/bookings`](transport/GET_transport_bookings.md) | Очередь заявок на транспортировку |
| `GET` | [`/transport/bookings/{id}`](transport/GET_transport_bookings_id.md) | Детали transport booking |
| `POST` | [`/transport/bookings/{id}/confirm`](transport/POST_transport_bookings_id_confirm.md) | Подтвердить транспортировку |
| `POST` | [`/transport/bookings/{id}/decline`](transport/POST_transport_bookings_id_decline.md) | Отклонить транспортировку |

---

## 5. Audit / History / Reporting

| Метод | Путь | Назначение |
|---|---|---|
| `GET` | [`/bookings/{id}/status-history`](common/GET_bookings_id_status_history.md) | История статусов брони |
| `GET` | [`/booking-requests/{id}/status-history`](common/GET_booking_requests_id_status_history.md) | История статусов заявки |
| `GET` | [`/bookings/{id}/timeline`](common/GET_bookings_id_timeline.md) | Агрегированный audit trail по брони |
| `GET` | [`/reports/requests`](reports/GET_reports_requests.md) | Отчет по заявкам |
| `GET` | [`/reports/bookings`](reports/GET_reports_bookings.md) | Отчет по броням |
| `GET` | [`/reports/usage-rate`](reports/GET_reports_usage_rate.md) | Usage Rate Dashboard |
| `GET` | [`/reports/work-centers`](reports/GET_reports_work_centers.md) | Отчет по Work Centers |
| `GET` | [`/reports/closed-requests`](reports/GET_reports_closed_requests.md) | Отчет / страница закрытых заявок |

---

## Замечания

1. External booking workflow в Phase 1 не входит в scope и в сводку не включен.
2. Для Requestor / SWP детальная спецификация вынесена в [`wiki/api/booking/requestor/`](requestor/).
3. Все методы должны использовать общий `result wrapper`; для списков с пагинацией дополнительно использовать `PaginatedResult`.
