# GET /booking-requests/{id}

**Created:** 2026-05-14  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить детальную информацию по заявке |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Requestor UI` |
| Endpoint URL | `/api/booking/v1/booking-requests/{id}` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется для открытия страницы заявки и просмотра ее item-ов.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-025 | Requestor can view request details and status | Confirmed | BRD v13 | Прямое покрытие |
| TCO Booking Tool | FR-038 | Each equipment item in request = separate booking | Confirmed | BRD v13 | Метод возвращает item-ы заявки |

---

## 3. Описание логики работы метода

1. Получить `BookingRequests` по `id`.
2. Проверить доступ: Requestor видит только собственные заявки; SWP видит свои рабочие заявки.
3. Подтянуть связанные `Bookings`, а также справочные данные по технике.
4. Вернуть агрегированную модель заявки.

Сущности, участвующие в методе:
- читаются: `BookingRequests`, `Bookings`, `Equipments`, `EquipmentTypes`
- изменения не выполняются

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Requestor` | Просмотр собственных заявок |
| `ServiceWorkProcessor` | Просмотр SWR и связанных заявок |

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
| `FORBIDDEN` | Нет доступа к заявке |
| `NOT_FOUND` | Заявка не найдена |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор заявки | `id` | `uuid` | `+` | Должен существовать | — | Path param | |

---

## 8. Пример запроса

```http
GET /api/booking/v1/booking-requests/c777f75f-029d-4d8f-8c69-e74a1d280001
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
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | BookingRequests.id |  |
| 2 | Номер заявки | requestNumber | string | string | — | BookingRequests.requestNumber |  |
| 3 | Тип сущности / заявки | type | string | string | — | BookingRequests + ref_request_type |  |
| 4 | Текущий статус | status | string | string | — | BookingRequests + BookingRequestStatuses |  |
| 5 | Приоритет | priority | string | string | — | BookingRequests + ref_request_priority |  |
| 6 | Описание работ | workDescription | string | string | — | BookingRequests.workDescription |  |
| 7 | Комментарии | comments | string | string | — | BookingRequests.comments |  |
| 8 | Список броней | bookings | array<object> | object[] | `[]` | backend composition from BookingRequests + Bookings + Equipments + EquipmentTypes | Коллекция объектов |

## 10. Пример ответа

```json
{
  "value": {
    "id": "c777f75f-029d-4d8f-8c69-e74a1d280001",
    "requestNumber": "REQ-2026-00015",
    "type": "Regular",
    "status": "Draft",
    "priority": "P2",
    "workDescription": "Excavator required for trench preparation near sector 4.",
    "comments": "Coordinate with site supervisor before mobilization.",
    "bookings": []
  },
  "isSuccess": true,
  "errors": []
}
```
