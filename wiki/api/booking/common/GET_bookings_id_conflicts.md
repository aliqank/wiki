# GET /bookings/{id}/conflicts

**Created:** 2026-05-25  
**Last updated:** 2026-05-25  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить список броней, конфликтующих с выбранной бронью |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Common` |
| Endpoint URL | `/api/booking/v1/bookings/{id}/conflicts` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется по клику на индикатор конфликтов у booking item, чтобы показать пользователю список броней, пересекающихся с текущей бронью по той же технике.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-040 | System validates availability before booking | Confirmed | BRD v13 | Метод раскрывает детали конфликтов, уже учитываемых в availability check |
| TCO Booking Tool | FR-025 | Requestor can view request details and status | Confirmed | BRD v13 | Позволяет детализировать конфликты из карточки брони |

---

## 3. Описание логики работы метода

1. Проверить существование брони и права доступа к ней.
2. Получить у исходной брони `equipmentId`, `plannedStartDateTime`, `plannedEndDateTime`.
3. Выбрать из `Bookings` другие записи по той же технике (`equipmentId` совпадает).
4. Исключить из выборки саму исходную бронь (`Bookings.id != :id`).
5. Включить только брони со статусами `Submitted`, `Confirmed`, `InProgress`.
6. Оставить только брони, у которых период пересекается с периодом исходной брони.
7. Для каждой конфликтующей брони дополнительно подтянуть номер заявки, тип заявки, статус заявки, requestor, fleet и `workCenterCode`.
8. Для каждой конфликтующей брони рассчитать фактический диапазон пересечения: `overlapStartDateTime` / `overlapEndDateTime`.
9. Вернуть список конфликтующих броней и общее количество конфликтов.

Сущности:
- читаются: `Bookings`, `BookingRequests`, `BookingStatuses`, `BookingRequestStatuses`, `Users`, `Fleets`, `WorkCenters`
- изменения не выполняются

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Requestor` | Просмотр конфликтов собственной брони |
| `ServiceWorkProcessor` | Просмотр конфликтов брони в SWR |
| `FleetOwner` | Просмотр конфликтов броней своих флотов |
| `FleetOwnersSupervisor` | Просмотр конфликтов long-term rented броней |
| `Admin` | Полный доступ |

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
| `FORBIDDEN` | Нет доступа к конфликтам этой брони |
| `NOT_FOUND` | Бронь не найдена |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор брони | `id` | `uuid` | `+` | Должен существовать | — | Path param | |

---

## 8. Пример запроса

```http
GET /api/booking/v1/bookings/8c4c8b6d-7bc0-41fb-9038-422cf55d1111/conflicts
Authorization: Bearer <token>
Content-Type: application/json
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
| 1 | Идентификатор исходной брони | bookingId | uuid | UUID v4 | — | backend aggregation from Bookings |  |
| 2 | Количество конфликтующих броней | conflictsCount | int | integer | `0` | backend aggregation |  |
| 3 | Список конфликтующих броней | items | array<object> | object[] | `[]` | backend aggregation from Bookings + BookingRequests + Users + Fleets + WorkCenters | Коллекция объектов |

### Структура `value.items[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор брони | bookingId | uuid | UUID v4 | — | Bookings.id |  |
| 2 | Идентификатор заявки | requestId | uuid | UUID v4 | — | BookingRequests.id |  |
| 3 | Номер заявки | requestNumber | string | string | — | BookingRequests.requestNumber |  |
| 4 | Тип заявки | requestType | string | string | — | BookingRequests + ref_request_type |  |
| 5 | Текущий статус заявки | requestStatus | string | string | — | last BookingRequestStatuses |  |
| 6 | Текущий статус брони | bookingStatus | string | string | — | last BookingStatuses |  |
| 7 | Плановая дата и время начала конфликтующей брони | plannedStartDateTime | datetime | ISO 8601 | — | Bookings.plannedStartDateTime |  |
| 8 | Плановая дата и время окончания конфликтующей брони | plannedEndDateTime | datetime | ISO 8601 | — | Bookings.plannedEndDateTime |  |
| 9 | Дата и время начала пересечения | overlapStartDateTime | datetime | ISO 8601 | — | backend overlap calculation | `MAX(source.plannedStartDateTime, conflicting.plannedStartDateTime)` |
| 10 | Дата и время окончания пересечения | overlapEndDateTime | datetime | ISO 8601 | — | backend overlap calculation | `MIN(source.plannedEndDateTime, conflicting.plannedEndDateTime)` |
| 11 | Код рабочего центра | workCenterCode | string | string | — | WorkCenters.code |  |
| 12 | Fleet | fleet | object | object | — | Fleets | Базовый контекст флота конфликтующей брони |
| 13 | Requestor | requestor | object | object | — | Users | Автор конфликтующей заявки |

### Структура `value.items[].fleet`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор флота | id | uuid | UUID v4 | — | Fleets.id |  |
| 2 | Наименование флота | name | string | string | — | Fleets.nameEn / localized projection |  |

### Структура `value.items[].requestor`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор пользователя | userId | uuid | UUID v4 | — | Users.id |  |
| 2 | Полное имя пользователя | fullName | string | string | — | Users.fullName |  |
| 3 | Email | email | string | string | — | Users.email |  |

## 10. Пример ответа

```json
{
  "value": {
    "bookingId": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
    "conflictsCount": 2,
    "items": [
      {
        "bookingId": "8c4c8b6d-7bc0-41fb-9038-422cf55d3333",
        "requestId": "c777f75f-029d-4d8f-8c69-e74a1d280010",
        "requestNumber": "REQ-2026-00021",
        "requestType": "Regular",
        "requestStatus": "Submitted",
        "bookingStatus": "Confirmed",
        "plannedStartDateTime": "2026-05-20T06:00:00Z",
        "plannedEndDateTime": "2026-05-21T18:00:00Z",
        "overlapStartDateTime": "2026-05-20T08:00:00Z",
        "overlapEndDateTime": "2026-05-21T18:00:00Z",
        "workCenterCode": "BHOE",
        "fleet": {
          "id": "f0000001-0000-4000-8000-000000000001",
          "name": "Maintenance Fleet"
        },
        "requestor": {
          "userId": "11111111-1111-1111-1111-111111111111",
          "fullName": "Telman Nurzhanov",
          "email": "telman.nurzhanov@example.com"
        }
      },
      {
        "bookingId": "8c4c8b6d-7bc0-41fb-9038-422cf55d4444",
        "requestId": "c777f75f-029d-4d8f-8c69-e74a1d280011",
        "requestNumber": "REQ-2026-00022",
        "requestType": "ServiceWorkRequest",
        "requestStatus": "Submitted",
        "bookingStatus": "InProgress",
        "plannedStartDateTime": "2026-05-20T10:00:00Z",
        "plannedEndDateTime": "2026-05-22T18:00:00Z",
        "overlapStartDateTime": "2026-05-20T10:00:00Z",
        "overlapEndDateTime": "2026-05-22T18:00:00Z",
        "workCenterCode": "HYDR",
        "fleet": {
          "id": "f0000001-0000-4000-8000-000000000002",
          "name": "Operations Fleet"
        },
        "requestor": {
          "userId": "22222222-2222-2222-2222-222222222222",
          "fullName": "Aidos Beketov",
          "email": "aidos.beketov@example.com"
        }
      }
    ]
  },
  "isSuccess": true,
  "errors": []
}
```

## Замечания

1. Метод возвращает только активные конфликты по статусам `Submitted`, `Confirmed`, `InProgress`.
2. Брони в статусах `Draft` и `Closed` не должны попадать в список конфликтов.
3. Метод не возвращает саму исходную бронь в `items[]`.
4. Метод предназначен для UI-детализации уже рассчитанного признака `activeBookingConflicts` и не заменяет основной availability check.
