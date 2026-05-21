# Booking API summary

**Created:** 2026-05-14  
**Last updated:** 2026-05-19  
**Автор документов:** Telman Nurzhanov (SA)

---

## Назначение

Сводная таблица endpoint-ов для booking-блока на основе:
- `source/results/2026-05-05 - HDV HDE BRD v13.md`
- `source/results/2026-05-14 - Схема БД v10 (Equipments, Bookings).md`

Base URL: `/api/booking/v1`

---

## 1. Requestor / SWP

| Метод | Путь | Назначение |
|---|---|---|
| `GET` | `/equipment/search` | Поиск техники для создания заявки |
| `GET` | `/equipment/{id}` | Получить карточку техники в booking-контексте |
| `GET` | `/equipment/{id}/load-summary` | Получить загрузку техники на выбранный период |
| `GET` | `/reference/equipment-types` | Получить справочник типов техники для фильтра поиска |
| `GET` | `/reference/fleet-owners` | Получить справочник fleet owners для фильтра поиска |
| `GET` | `/reference/work-centers` | Получить справочник work centers для фильтра поиска |
| `GET` | `/reference/ownership-types` | Получить справочник ownership types для фильтра поиска |
| `GET` | `/reference/share-types` | Получить справочник share types для фильтра поиска |
| `GET` | `/reference/equipment-types/{equipmentTypeId}/properties` | Получить динамические свойства выбранного типа техники |
| `POST` | `/booking-requests` | Создать черновик заявки |
| `GET` | `/booking-requests/{id}` | Получить детали заявки |
| `GET` | `/booking-requests/my` | Получить список собственных заявок |
| `PATCH` | `/booking-requests/{id}` | Обновить черновик заявки |
| `POST` | `/booking-requests/{id}/submit` | Отправить заявку |
| `POST` | `/booking-requests/{id}/cancel` | Отменить черновик |
| `POST` | `/booking-requests/{id}/items` | Добавить booking item в черновик |
| `PATCH` | `/booking-requests/{id}/items/{bookingId}` | Обновить booking item в черновике |
| `DELETE` | `/booking-requests/{id}/items/{bookingId}` | Удалить booking item из черновика |
| `POST` | `/bookings/{id}/revoke` | Отозвать бронь до решения FO |
| `POST` | `/bookings/{id}/extend` | Запросить продление брони |
| `POST` | `/bookings/{id}/terminate` | Досрочно завершить подтвержденную бронь |
| `POST` | `/bookings/{id}/close` | Закрыть бронь вручную |
| `POST` | `/bookings/{id}/feedback` | Оставить отзыв по технике |
| `GET` | `/booking-requests/history` | Получить историю завершенных заявок |

---

## 2. Fleet Owner

| Метод | Путь | Назначение |
|---|---|---|
| `GET` | `/approvals/bookings` | Очередь броней на согласование |
| `GET` | `/approvals/requests` | Список заявок с релевантными бронями для Fleet Owner |
| `GET` | `/approvals/bookings/{id}` | Детали брони для FO |
| `GET` | `/approvals/bookings/{id}/load-summary` | Загрузка техники на даты в окне подтверждения |
| `POST` | `/approvals/bookings/{id}/confirm` | Подтвердить бронь |
| `POST` | `/approvals/bookings/{id}/decline` | Отклонить бронь |
| `POST` | `/approvals/bookings/{id}/change-equipment` | Заменить технику в брони |
| `POST` | `/approvals/bookings/{id}/change-period` | Изменить период брони |
| `POST` | `/approvals/bookings/{id}/mobilization-start` | Зафиксировать начало мобилизации |
| `POST` | `/approvals/bookings/{id}/terminate` | Досрочно завершить бронь |
| `GET` | `/approvals/completed` | История обработанных согласований |

---

## 3. FleetOwners' Supervisor

| Метод | Путь | Назначение |
|---|---|---|
| `GET` | `/supervisor/bookings` | Очередь long-term rented броней в статусе `ConfirmedByFo` |
| `GET` | `/supervisor/bookings/{id}` | Детали брони для финального решения |
| `POST` | `/supervisor/bookings/{id}/confirm` | Финально подтвердить бронь |
| `POST` | `/supervisor/bookings/{id}/decline` | Отклонить бронь с обязательным комментарием |

---

## 4. Transportation Responsible

| Метод | Путь | Назначение |
|---|---|---|
| `GET` | `/transport/bookings` | Очередь заявок на транспортировку |
| `GET` | `/transport/bookings/{id}` | Детали transport booking |
| `POST` | `/transport/bookings/{id}/confirm` | Подтвердить транспортировку |
| `POST` | `/transport/bookings/{id}/decline` | Отклонить транспортировку |

---

## 5. Audit / History / Reporting

| Метод | Путь | Назначение |
|---|---|---|
| `GET` | `/bookings/{id}/status-history` | История статусов брони |
| `GET` | `/booking-requests/{id}/status-history` | История статусов заявки |
| `GET` | `/bookings/{id}/timeline` | Агрегированный audit trail по брони |
| `GET` | `/reports/requests` | Отчет по заявкам |
| `GET` | `/reports/bookings` | Отчет по броням |
| `GET` | `/reports/usage-rate` | Usage Rate Dashboard |
| `GET` | `/reports/work-centers` | Отчет по Work Centers |
| `GET` | `/reports/closed-requests` | Отчет / страница закрытых заявок |

---

## Замечания

1. External booking workflow в Phase 1 не входит в scope и в сводку не включен.
2. Для Requestor / SWP детальная спецификация вынесена в `wiki/api/booking/requestor/`.
3. Все методы должны использовать общий `result wrapper`; для списков с пагинацией дополнительно использовать `PaginatedResult`.
