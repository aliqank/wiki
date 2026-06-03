# POST /approvals/bookings/{id}/close

**Created:** 2026-06-01  
**Last updated:** 2026-06-02  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Закрыть бронь вручную как Fleet Owner и зафиксировать фактическое время использования |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Fleet Owner UI` |
| Endpoint URL | `/api/booking/v1/approvals/bookings/{id}/close` |
| Метод запроса | `POST` |
| Связанные use cases | [`UC-FO-05 - Закрытие брони Fleet Owner`](../../../requirements/usecases/Fleet%20Owner/UC-FO-05%20-%20Закрытие%20брони%20Fleet%20Owner.md) |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Реализует manual close брони на стороне Fleet Owner.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-NEW-32 | Booking closure is manual only — "Close" button by Requestor or FO | Confirmed | BRD v13 | Прямое покрытие для Fleet Owner сценария |
| TCO Booking Tool | FR-NEW-33 | At close: Requestor inputs actual start/end time for usage rate analytics | Confirmed | BRD v13 | Для FO close используются те же фактические даты в аналитических целях |

---

## 3. Описание логики работы метода

1. Проверить существование брони и права доступа Fleet Owner.
2. Разрешить close только для активной подтвержденной / in-progress брони в зоне ответственности текущего Fleet Owner.
3. Провалидировать `actualStartDateTime` и `actualEndDateTime`.
4. Обновить `Bookings.status = Closed`, `closureReason = Completed`, заполнить `actualStartDateTime`, `actualEndDateTime`.
5. Создать запись в `BookingStatuses` с `status = Closed` и `closureReason = Completed`.
6. Пересчитать статус заявки; если это последняя активная бронь, перевести заявку в `Closed` с `requestClosureReason = Completed`.

Сущности, участвующие в методе:
- читаются: [`Bookings`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#22-bookings), [`BookingRequests`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#21-bookingrequests)
- изменяются: [`Bookings`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#22-bookings), [`BookingStatuses`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#24-bookingstatuses), [`BookingRequests`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#21-bookingrequests), [`BookingRequestStatuses`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#26-bookingrequeststatuses)
- транзакционность: обязательна

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `FleetOwner` | Закрытие броней своих флотов |

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
| `BOOKING_NOT_CLOSABLE` | Бронь нельзя закрыть |
| `VALIDATION_ERROR` | Фактические даты невалидны |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор брони | `id` | `uuid` | `+` | Должен существовать | — | Path param | |
| 2 | Фактическая дата/время начала | `actualStartDateTime` | `datetime` | `+` | Меньше либо равно `actualEndDateTime` | — | Request body | |
| 3 | Фактическая дата/время окончания | `actualEndDateTime` | `datetime` | `+` | Больше либо равно `actualStartDateTime` | — | Request body | |

---

## 8. Пример запроса

```http
POST /api/booking/v1/approvals/bookings/8c4c8b6d-7bc0-41fb-9038-422cf55d1111/close
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "actualStartDateTime": "2026-05-20T08:10:00Z",
  "actualEndDateTime": "2026-05-22T17:25:00Z"
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
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | backend composition from Bookings + BookingRequests + BookingStatuses + BookingRequestStatuses |  |
| 2 | Текущий статус | status | string | string | — | backend composition from Bookings + BookingRequests + BookingStatuses + BookingRequestStatuses |  |
| 3 | Фактическая дата и время начала | actualStartDateTime | datetime | ISO 8601 | — | backend composition from Bookings + BookingRequests + BookingStatuses + BookingRequestStatuses |  |
| 4 | Фактическая дата и время окончания | actualEndDateTime | datetime | ISO 8601 | — | backend composition from Bookings + BookingRequests + BookingStatuses + BookingRequestStatuses |  |

## 10. Пример ответа

```json
{
  "value": {
    "id": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
    "status": "Closed",
    "actualStartDateTime": "2026-05-20T08:10:00Z",
    "actualEndDateTime": "2026-05-22T17:25:00Z"
  },
  "isSuccess": true,
  "errors": []
}
```
