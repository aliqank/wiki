# GET /reports/requests

**Created:** 2026-05-14  
**Last updated:** 2026-05-20  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить отчет по заявкам |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Reporting` |
| Endpoint URL | `/api/booking/v1/reports/requests` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется для фильтруемого списка заявок в отчетных экранах.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-094 | System generates reports on Requests/Bookings/Equipment with filters | Confirmed | BRD v13 | Прямое покрытие |
| TCO Booking Tool | FR-095 | Admin can view/download all reports | Confirmed | BRD v13 | Полный доступ |

---

## 3. Описание логики работы метода

1. Проверить права доступа отчетной роли.
2. Выбрать `BookingRequests` по фильтрам периода, статуса, причины закрытия, типа, приоритета.
3. Подтянуть агрегаты по количеству item-ов и статусам.
4. Вернуть пагинированный отчет.

Сущности:
- читаются: [`BookingRequests`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#21-bookingrequests), [`Bookings`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#22-bookings)

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Admin` | Все заявки и все отчеты |
| `FleetOwner` | Только заявки по своим флотам |
| `ServiceWorkProcessor` | Только релевантные SWR |

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
| `FORBIDDEN` | Нет прав на отчеты |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Статус заявки | `status` | `string` | `-` | `Draft / Submitted / InProgress / Closed` | — | Query param | Значения соответствуют кодам `ref_booking_request_status`; отдельный report-specific reference API не зафиксирован |
| 2 | Причина закрытия заявки | `closureReason` | `string` | `-` | `Cancelled / Completed` | — | Query param | Значения соответствуют кодам `ref_request_closure_reason`; используется только вместе со `status = Closed` или для неявной фильтрации по закрытым заявкам |
| 3 | Тип заявки | `type` | `string` | `-` | `Regular / ServiceWork` | — | Query param | Значения соответствуют кодам `ref_request_type`; отдельный report-specific reference API не зафиксирован |
| 4 | Дата начала периода | `from` | `date` | `-` | — | — | Query param | |
| 5 | Дата окончания периода | `to` | `date` | `-` | — | — | Query param | |
| 6 | Номер страницы | `page` | `int` | `-` | >= 1 | `1` | Query param | |
| 7 | Размер страницы | `limit` | `int` | `-` | >= 1 | `20` | Query param | |

---

## 8. Пример запроса

```http
GET /api/booking/v1/reports/requests?status=Closed&page=1&limit=20
Authorization: Bearer <token>
Content-Type: application/json
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий [`result wrapper`](../../common/Result%20Wrapper.md).

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | value | object | PaginatedResult | — | backend aggregation |  |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | object[] | `[]` | backend |  |

### Структура `value`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Элементы текущей страницы | items | array<object> | object[] | — | backend composition from BookingRequests + Bookings | Коллекция объектов |
| 2 | Общее количество записей | total | int | integer | — | backend |  |

### Структура `value.items[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | BookingRequests.id |  |
| 2 | Номер заявки | requestNumber | string | string | — | BookingRequests.requestNumber |  |
| 3 | Текущий статус | status | string | string | — | BookingRequests + BookingRequestStatuses |  |
| 4 | Причина закрытия заявки | closureReason | string | null | `null` | BookingRequests + BookingRequestStatuses + ref_request_closure_reason | Для незакрытых заявок возвращается `null` |
| 5 | Тип сущности / заявки | type | string | string | — | BookingRequests + ref_request_type |  |

## 10. Пример ответа

```json
{
  "value": {
    "items": [
      {
        "id": "c777f75f-029d-4d8f-8c69-e74a1d280001",
        "requestNumber": "REQ-2026-00015",
        "status": "Closed",
        "closureReason": "Completed",
        "type": "Regular"
      }
    ],
    "total": 1
  },
  "isSuccess": true,
  "errors": []
}
```
