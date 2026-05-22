# POST /bookings/{id}/close

**Created:** 2026-05-14  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Закрыть бронь вручную и зафиксировать фактическое время использования |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Requestor UI` |
| Endpoint URL | `/api/booking/v1/bookings/{id}/close` |
| Метод запроса | `POST` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Реализует manual close и запись фактических дат использования.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-NEW-32 | Booking closure is manual only | Confirmed | BRD v13 | Прямое покрытие |
| TCO Booking Tool | FR-NEW-33 | At close: Requestor inputs actual start/end time | Confirmed | BRD v13 | Заполняются `actualStartDateTime`, `actualEndDateTime` |

---

## 3. Описание логики работы метода

1. Проверить существование брони и права доступа.
2. Разрешить close только для активной подтвержденной / in-progress брони.
3. Провалидировать `actualStartDateTime` и `actualEndDateTime`.
4. Обновить `Bookings.status = Closed`, `closureReason = Completed`, заполнить `actualStartDateTime`, `actualEndDateTime`.
5. Создать запись в `BookingStatuses` с `status = Closed` и `closureReason = Completed`.
6. Пересчитать статус заявки; если это последняя активная бронь, перевести заявку в `Closed`.

Сущности, участвующие в методе:
- читаются: `Bookings`, `BookingRequests`
- изменяются: `Bookings`, `BookingStatuses`, `BookingRequests`, `BookingRequestStatuses`
- транзакционность: обязательна

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Requestor` | Закрытие собственной брони |
| `ServiceWorkProcessor` | Закрытие брони в SWR |

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
POST /api/booking/v1/bookings/8c4c8b6d-7bc0-41fb-9038-422cf55d1111/close
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

Возвращаемые данные обёрнуты в общий `result wrapper`.

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
