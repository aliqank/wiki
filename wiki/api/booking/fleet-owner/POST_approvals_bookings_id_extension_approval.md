# POST /approvals/bookings/{id}/extension-approval

**Created:** 2026-06-02  
**Last updated:** 2026-06-02  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Принять решение Fleet Owner по запросу на изменение `plannedEndDateTime` |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Fleet Owner UI` |
| Endpoint URL | `/api/booking/v1/approvals/bookings/{id}/extension-approval` |
| Метод запроса | `POST` |
| Связанные use cases | [`UC-FO-09 - Согласование изменения плановой даты и времени окончания брони`](../../../requirements/usecases/Fleet%20Owner/UC-FO-09%20-%20Согласование%20изменения%20плановой%20даты%20и%20времени%20окончания%20брони.md) |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется Fleet Owner-ом для approve/decline запроса Requestor / SWP на изменение `plannedEndDateTime`.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-061 | Requestor/SWP can extend confirmed booking | Confirmed | BRD v13 | Метод завершает approval-часть flow |
| TCO Booking Tool | FR-067 | Booking -> Submitted once extended | Confirmed | BRD v13 | Для pre-start сценария request возвращается в approval cycle |
| TCO Booking Tool | FR-081 | System notifies FO when Requestor extends booking | Confirmed | BRD v13 | Метод является ответом на такое уведомление |

---

## 3. Описание логики работы метода

1. Проверить существование брони и права доступа Fleet Owner.
2. Проверить, что по брони есть pending [Booking Extension Approval](../../../glossary/Glossary.md#booking-extension-approval).
3. Принять решение `Approved` или `Declined`.
4. Создать запись в `BookingApprovals` с `approvalType = BookingExtensionApproval`.
5. Если решение равно `Approved`:
   - применить новое `plannedEndDateTime`;
   - если бронь была в `Submitted` или `Confirmed` и других обязательных approvals больше нет, перевести бронь в `Confirmed`;
   - если бронь была в `InProgress`, сохранить `InProgress` и обновить только `plannedEndDateTime`.
6. Если решение равно `Declined`, не применять новое `plannedEndDateTime`.
7. Зафиксировать lifecycle / audit изменения в `BookingStatuses`, если они возникают по итогам решения.
8. Вернуть обновленную бронь.

Сущности:
- читаются: [`Bookings`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#22-bookings), [`BookingApprovals`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#23-bookingapprovals)
- изменяются: [`Bookings`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#22-bookings), [`BookingApprovals`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#23-bookingapprovals), [`BookingStatuses`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#24-bookingstatuses)

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `FleetOwner` | Принятие решения по [Booking Extension Approval](../../../glossary/Glossary.md#booking-extension-approval) для броней fleet-ов, по которым у пользователя есть [Fleet Management Access](../../../glossary/Glossary.md#fleet-management-access) |

---

## 5. Настройки системы, используемые в методе

| Наименование | Код | Тип значения | Описание | Значение по умолчанию |
|---|---|---|---|---|
| — | — | — | Не используются отдельно от общих booking rules | — |

---

## 6. Ошибки, возвращаемые методом

| Код | Описание ошибки |
|---|---|
| `UNAUTHORIZED` | Пользователь не авторизован |
| `FORBIDDEN` | Нет доступа к брони |
| `NOT_FOUND` | Бронь не найдена |
| `EXTENSION_APPROVAL_NOT_PENDING` | По брони нет pending extension approval |
| `VALIDATION_ERROR` | Передано невалидное решение |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор брони | `id` | `uuid` | `+` | Должен существовать | — | Path param | |
| 2 | Решение Fleet Owner | `decision` | `string` | `+` | `Approved / Declined` | — | Request body | |
| 3 | Комментарий | `comment` | `string` | `-` | — | — | Request body | |

---

## 8. Пример запроса

```http
POST /api/booking/v1/approvals/bookings/8c4c8b6d-7bc0-41fb-9038-422cf55d1111/extension-approval
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "decision": "Approved",
  "comment": "Extension approved for one more shift."
}
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий `result wrapper`.

### Структура `value`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | Bookings.id |  |
| 2 | Текущий статус | status | string | string | — | Bookings + BookingStatuses | `Confirmed` или `InProgress` после approve; без изменения при decline |
| 3 | Плановая дата и время окончания | plannedEndDateTime | datetime | ISO 8601 | — | Bookings.plannedEndDateTime | Обновляется только после approve |

## 10. Пример ответа

```json
{
  "value": {
    "id": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
    "status": "Confirmed",
    "plannedEndDateTime": "2026-05-23T18:00:00Z"
  },
  "isSuccess": true,
  "errors": []
}
```
