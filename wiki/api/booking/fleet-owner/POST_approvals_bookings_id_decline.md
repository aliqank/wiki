# POST /approvals/bookings/{id}/decline

**Created:** 2026-05-14  
**Last updated:** 2026-06-02  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Отклонить бронь как [`Fleet Owner`](../../../requirements/Roles and Access Model.md) |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / [`Fleet Owner`](../../../requirements/Roles and Access Model.md) UI` |
| Endpoint URL | `/api/booking/v1/approvals/bookings/{id}/decline` |
| Метод запроса | `POST` |
| Связанные use cases | [`UC-FO-06 - Отклонение брони Fleet Owner`](../../../requirements/usecases/Fleet%20Owner/UC-FO-06%20-%20Отклонение%20брони%20Fleet%20Owner.md) |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Выполняет decline входящей брони.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-049 | FO can decline booking | Confirmed | BRD v13 | Прямое покрытие |
| TCO Booking Tool | FR-050 | FO provides reason/comment when declining | Confirmed | BRD v13 | Причина обязательна |
| TCO Booking Tool | FR-064 | Booking -> Closed with closure reason Declined once declined by FO | Confirmed | BRD v13 | Обновление статуса |

---

## 3. Описание логики работы метода

1. Проверить бронь и права доступа.
2. Разрешить действие только для статуса `Submitted`.
3. Потребовать непустой `reason`.
4. Обновить бронь до terminal-состояния `Closed` и зафиксировать причину decline через `closureReason = Declined`.
5. Создать запись в `BookingApprovals` с `approvalType = FoApproval`, `status = Declined`, `userId = currentUserId`, `approvalOrder = 1`, `comment = reason`.
6. Создать запись в `BookingStatuses`.
7. Пересчитать агрегированный статус родительской заявки.
8. Если отклоненная бронь была последней активной в заявке, перевести заявку в `Closed`:
   - с `requestClosureReason = Cancelled`, если заявка ни разу не была в `InProgress`;
   - с `requestClosureReason = Completed`, если заявка ранее уже была в `InProgress`.
9. Если при пересчете request status фактически изменился, создать запись в `BookingRequestStatuses`.
10. Вернуть результат.

Сущности:
- читаются: [`Bookings`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#22-bookings), [`BookingRequests`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#21-bookingrequests)
- изменяются: [`Bookings`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#22-bookings), [`BookingApprovals`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#23-bookingapprovals), [`BookingStatuses`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#24-bookingstatuses), [`BookingRequests`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#21-bookingrequests), [`BookingRequestStatuses`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#26-bookingrequeststatuses)

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| [`FleetOwner`](../../../requirements/Roles and Access Model.md) | Отклонение броней своих флотов |

---

## 5. Настройки системы, используемые в методе

| Наименование | Код | Тип значения | Описание | Значение по умолчанию |
|---|---|---|---|---|
| — | — | — | Не используются | — |

---

## 6. Ошибки, возвращаемые методом

| Код | Описание ошибки |
|---|---|
| `UNAUTHORIZED` | Пользователь не авторизован |
| `FORBIDDEN` | Нет доступа к брони |
| `NOT_FOUND` | Бронь не найдена |
| `BOOKING_NOT_DECLINABLE` | Текущий статус не позволяет decline |
| `VALIDATION_ERROR` | Не передана причина decline |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор брони | `id` | `uuid` | `+` | Должен существовать | — | Path param | |
| 2 | Причина отклонения | `reason` | `string` | `+` | Непустая строка | — | Request body | |

---

## 8. Пример запроса

```http
POST /api/booking/v1/approvals/bookings/8c4c8b6d-7bc0-41fb-9038-422cf55d1111/decline
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "reason": "Equipment reserved for higher-priority internal work."
}
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий [`result wrapper`](../../common/Result%20Wrapper.md).

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | value | object | object | — | backend aggregation |  |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | object[] | `[]` | backend |  |

### Структура `value`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | Bookings.id |  |
| 2 | Текущий статус | status | string | string | — | Bookings + BookingStatuses |  |
| 3 | Последнее записанное решение | lastApproval | object | object | — | backend composition from BookingApprovals | Последний approval step |

### Структура `value.lastApproval`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Тип шага согласования | type | string | string | — | BookingApprovals + ref_booking_approval_type | `FoApproval` |
| 2 | Результат решения | approvalStatus | string | string | — | BookingApprovals + ref_booking_approval_status | `Declined` |
| 3 | Порядок шага | order | int | integer | — | BookingApprovals.approvalOrder | `1` |
| 4 | Комментарий | comment | string | string | — | BookingApprovals.comment | Совпадает с `reason` |

## 10. Пример ответа

```json
{
  "value": {
    "id": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
    "status": "Closed",
    "lastApproval": {
      "type": "FoApproval",
      "approvalStatus": "Declined",
      "order": 1,
      "comment": "Equipment reserved for higher-priority internal work."
    }
  },
  "isSuccess": true,
  "errors": []
}
```
