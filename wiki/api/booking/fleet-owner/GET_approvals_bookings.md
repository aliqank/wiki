# GET /approvals/bookings

**Created:** 2026-05-14  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить очередь броней на согласование для Fleet Owner |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Fleet Owner UI` |
| Endpoint URL | `/api/booking/v1/approvals/bookings` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Наполняет список входящих броней для FO.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-092 | Approver (FO) can view all pending and completed approvals | Confirmed | BRD v13 | Список pending approvals |
| TCO Booking Tool | FR-043 | Approver assigned automatically based on fleet ownership | Confirmed | BRD v13 | Отбор по флотам FO |

---

## 3. Описание логики работы метода

1. Определить список флотов, доступных текущему FO через AAD-группы.
2. Выбрать `Bookings` по этим флотам со статусами `Submitted`, `ConfirmedByFo`, `Extended`, `TransportConfirmed` по фильтру экрана.
3. Подтянуть `BookingRequests`, `Equipments`, `EquipmentTypes`.
4. Вернуть пагинированный список.

Сущности:
- читаются: `Bookings`, `BookingRequests`, `Equipments`, `EquipmentTypes`, `Fleets`
- изменений нет

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `FleetOwner` | Просмотр броней по управляемым флотам |

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
| `FORBIDDEN` | У пользователя нет роли `FleetOwner` |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Статус брони | `status` | `enum` | `-` | Статусы approval queue | — | Query param | |
| 2 | Поисковая строка | `search` | `string` | `-` | Поиск по request number / equipment number | — | Query param | |
| 3 | Номер страницы | `page` | `int` | `-` | >= 1 | `1` | Query param | |
| 4 | Размер страницы | `limit` | `int` | `-` | >= 1 | `20` | Query param | |

---

## 8. Пример запроса

```http
GET /api/booking/v1/approvals/bookings?status=Submitted&page=1&limit=20
Authorization: Bearer <token>
Content-Type: application/json
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий `result wrapper`.

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | value | object | PaginatedResult | — | backend aggregation |  |
| 1.1 | Элементы текущей страницы | items | array<object> | object[] | — | backend composition from Bookings + BookingRequests + Equipments + EquipmentTypes + Fleets | Коллекция объектов |
| 1.2 | Общее количество записей | total | int | integer | — | backend |  |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | object[] | `[]` | backend |  |

### Структура `value`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Элементы текущей страницы | items | array<object> | object[] | — | backend composition from Bookings + BookingRequests + Equipments + EquipmentTypes + Fleets | Коллекция объектов |
| 2 | Общее количество записей | total | int | integer | — | backend |  |

### Структура `value.items[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | Bookings.id |  |
| 2 | Идентификатор заявки | requestId | uuid | UUID v4 | — | backend composition from Bookings + BookingRequests + Equipments + EquipmentTypes + Fleets |  |
| 3 | Номер заявки | requestNumber | string | string | — | BookingRequests.requestNumber |  |
| 4 | Идентификатор техники | equipmentId | uuid | UUID v4 | — | backend composition from Bookings + BookingRequests + Equipments + EquipmentTypes + Fleets |  |
| 5 | Номер техники | equipmentNumber | string | string | — | Equipments.tcoId |  |
| 6 | Наименование типа техники | equipmentTypeName | object | object | — | backend composition from Bookings + BookingRequests + Equipments + EquipmentTypes + Fleets |  |
| 7 | Текущий статус | status | string | string | — | Bookings + BookingStatuses |  |
| 8 | Дата и время начала | startDt | datetime | ISO 8601 | — | backend composition from Bookings + BookingRequests + Equipments + EquipmentTypes + Fleets |  |
| 9 | Дата и время окончания | endDt | datetime | ISO 8601 | — | backend composition from Bookings + BookingRequests + Equipments + EquipmentTypes + Fleets |  |
| 10 | Признак необходимости согласования Supervisor | requiresSupervisorApproval | bool | boolean | — | Bookings.requiresSupervisorApproval |  |
| 11 | Обоснование | justification | null | — | `null` | Bookings.justification |  |

### Структура `value.items[].equipmentTypeName`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string | string | — | Bookings |  |
| 2 | Значение на русском языке | Ru | string | string | — | Bookings |  |
| 3 | Значение на казахском языке | Kz | string | string | — | Bookings |  |

## 10. Пример ответа

```json
{
  "value": {
    "items": [
      {
        "id": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
        "requestId": "c777f75f-029d-4d8f-8c69-e74a1d280001",
        "requestNumber": "REQ-2026-00015",
        "equipmentId": "c3b5af91-61f8-4bc0-bd88-d099d3e90001",
        "equipmentNumber": "TCO-100245",
        "equipmentTypeName": {
          "En": "Excavator",
          "Ru": "Экскаватор",
          "Kz": "Экскаватор"
        },
        "status": "Submitted",
        "startDt": "2026-05-20T08:00:00Z",
        "endDt": "2026-05-22T18:00:00Z",
        "requiresSupervisorApproval": false,
        "justification": null
      }
    ],
    "total": 1
  },
  "isSuccess": true,
  "errors": []
}
```
