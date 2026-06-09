# GET /approvals/bookings

**Created:** 2026-05-14  
**Last updated:** 2026-05-19  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить очередь броней на согласование для [`Fleet Owner`](../../../requirements/Roles and Access Model.md) |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / [`Fleet Owner`](../../../requirements/Roles and Access Model.md) UI` |
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

1. Определить список флотов, где у текущего FO есть [Fleet Management Access](../../../glossary/Glossary.md#fleet-management-access), а также технику, доступную ему в рамках этого доступа.
2. Выбрать `Bookings` по этим флотам и/или по доступной технике со статусами, допустимыми для view `Bookings`, по фильтру экрана.
3. Подтянуть `BookingRequests`, `Equipments`, `EquipmentTypes`, `EquipmentBrands`, `EquipmentModels`, `Users`.
4. Для каждого booking вычислить таймер с момента submit.
5. Вернуть пагинированный список.

Сущности:
- читаются: [`Bookings`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#22-bookings), [`BookingRequests`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#21-bookingrequests), [`Equipments`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#8-equipments), [`EquipmentTypes`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#1-equipmenttypes), [`EquipmentBrands`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#4-equipmentbrands), [`EquipmentModels`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#5-equipmentmodels), [`Fleets`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#2-fleets), [`Users`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#19-users), [`FleetManagePermissions`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#7-fleetmanagepermissions)
- изменений нет

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| [`FleetOwner`](../../../requirements/Roles and Access Model.md) | Просмотр броней по управляемым флотам |

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
| `FORBIDDEN` | У пользователя нет роли [`FleetOwner`](../../../requirements/Roles and Access Model.md) |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Статус брони | `status` | `string` | `-` | Статусы approval queue для view `Bookings` | — | Query param | Значения соответствуют кодам `ref_booking_status`; отдельный reference API для approval queue statuses не зафиксирован |
| 2 | Тип заявки | `requestType` | `string` | `-` | Допустимые типы `BookingRequest` | — | Query param | Значения загружаются через [`GET /reference/approval-request-types`](../reference/GET_reference_approval_request_types.md) |
| 3 | Период заявок: начало | `createdFrom` | `date` | `-` | Если передан, должен быть <= `createdTo` | — | Query param | Фильтр по дате создания брони |
| 4 | Период заявок: конец | `createdTo` | `date` | `-` | Если передан, должен быть >= `createdFrom` | — | Query param | Фильтр по дате создания брони |
| 5 | Номер Work Order | `workOrderNumber` | `string` | `-` | Partial search | — | Query param | Поиск только по `workOrderNumber` |
| 6 | Номер страницы | `page` | `int` | `-` | >= 1 | `1` | Query param | |
| 7 | Размер страницы | `limit` | `int` | `-` | >= 1 | `20` | Query param | |

---

## 8. Пример запроса

```http
GET /api/booking/v1/approvals/bookings?status=Submitted&requestType=Regular&workOrderNumber=WO-10025&page=1&limit=20
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
| 1 | Элементы текущей страницы | items | array<object> | object[] | — | backend composition from Bookings + BookingRequests + Equipments + EquipmentTypes + Fleets | Коллекция объектов |
| 2 | Общее количество записей | total | int | integer | — | backend |  |

### Структура `value.items[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | Bookings.id |  |
| 2 | Идентификатор заявки | requestId | uuid | UUID v4 | — | backend composition from Bookings + BookingRequests + Equipments + EquipmentTypes + Fleets |  |
| 3 | Номер заявки | requestNumber | string | string | — | BookingRequests.requestNumber |  |
| 4 | [`Requestor`](../../../requirements/Roles and Access Model.md) | requestor | string | string | — | BookingRequests.requestorId + Users.fullName | Business-requestor заявки |
| 5 | Номер Work Order | workOrderNumber | string | string | — | BookingRequests.workOrderNumber |  |
| 6 | Приоритет | priority | string | string | — | BookingRequests + ref_request_priority |  |
| 7 | Дата создания брони | bookingCreatedAt | datetime | ISO 8601 | — | Bookings.createdAt |  |
| 8 | Идентификатор техники | equipmentId | uuid | UUID v4 | — | backend composition from Bookings + BookingRequests + Equipments + EquipmentTypes + Fleets |  |
| 9 | Подвижность техники | mobilityType | string | string | — | EquipmentTypes.mobilityType |  |
| 10 | Тип техники | equipmentTypeName | object | object | — | backend composition from Bookings + BookingRequests + Equipments + EquipmentTypes + Fleets |  |
| 11 | Марка и модель | brandModel | string | string | — | EquipmentBrands + EquipmentModels |  |
| 12 | ТШО-номер техники | tcoId | string | string | — | Equipments.tcoId |  |
| 13 | ГРНЗ | stateNumber | string | string | — | Equipments.stateNumber |  |
| 14 | Текущий статус брони | status | string | string | — | Bookings + BookingStatuses |  |
| 15 | Плановая дата и время начала | plannedStartDateTime | datetime | ISO 8601 | — | backend composition from Bookings + BookingRequests + Equipments + EquipmentTypes + Fleets |  |
| 16 | Плановая дата и время окончания | plannedEndDateTime | datetime | ISO 8601 | — | backend composition from Bookings + BookingRequests + Equipments + EquipmentTypes + Fleets |  |
| 17 | Таймер с момента submit | timeSinceSubmitSec | int | integer | — | backend calculation from BookingStatuses / submit timestamp | Возраст брони в очереди FO в секундах |
| 18 | Обоснование | justification | null | — | `null` | Bookings.justification |  |

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
        "requestor": "Telman Nurzhanov",
        "workOrderNumber": "WO-10025",
        "priority": "P2",
        "bookingCreatedAt": "2026-05-14T09:20:00Z",
        "equipmentId": "c3b5af91-61f8-4bc0-bd88-d099d3e90001",
        "mobilityType": "SelfPropelled",
        "tcoId": "TCO-100245",
        "equipmentTypeName": {
          "En": "Excavator",
          "Ru": "Экскаватор",
          "Kz": "Экскаватор"
        },
        "brandModel": "CAT 320D",
        "stateNumber": "KZ 123 ABC 02",
        "status": "Submitted",
        "plannedStartDateTime": "2026-05-20T08:00:00Z",
        "plannedEndDateTime": "2026-05-22T18:00:00Z",
        "timeSinceSubmitSec": 86400,
        "justification": null
      }
    ],
    "total": 1
  },
  "isSuccess": true,
  "errors": []
}
```
