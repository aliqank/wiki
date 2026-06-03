# GET /booking-requests/my

**Created:** 2026-05-14  
**Last updated:** 2026-06-02  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить список [незавершенных заявок](../../../glossary/Glossary.md), где текущий пользователь является business-requestor-ом |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Requestor UI` |
| Endpoint URL | `/api/booking/v1/booking-requests/my` |
| Метод запроса | `GET` |
| Связанные use cases | [`UC-REQ-01 - Просмотр моих заявок`](../../../requirements/usecases/Requestor/UC-REQ-01%20-%20Просмотр%20моих%20заявок.md), [`UC-REQ-03 - Отмена draft-заявки requestor-ом`](../../../requirements/usecases/Requestor/UC-REQ-03%20-%20Отмена%20draft-заявки%20requestor-ом.md), [`UC-REQ-04 - Отзыв брони requestor-ом`](../../../requirements/usecases/Requestor/UC-REQ-04%20-%20Отзыв%20брони%20requestor-ом.md), [`UC-REQ-05 - Отзыв submitted-заявки requestor-ом`](../../../requirements/usecases/Requestor/UC-REQ-05%20-%20Отзыв%20submitted-заявки%20requestor-ом.md) |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется для страницы «Мои заявки» как summary API для списка [незавершенных заявок](../../../glossary/Glossary.md) Requestor / SWP.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-025 | Requestor can view request details and status | Confirmed | BRD v13 | Частичное покрытие на уровне summary-list [незавершенных заявок](../../../glossary/Glossary.md) |
| TCO Booking Tool | FR-091 | Requestor/SWP can search and filter own requests | Confirmed | BRD v13 | Прямое покрытие |
| TCO Booking Tool | BRD-U-001 | Request terminal status semantics | Confirmed | BRD Updates | Определяет, какие request statuses считаются non-terminal для [`GET /booking-requests/my`](GET_booking_requests_my.md) |

---

## 3. Описание логики работы метода

1. Для Requestor выбрать `BookingRequests` по `requestorId = currentUserId`.
2. Для ServiceWorkProcessor применять отдельные правила видимости SWR; выборка не должна опираться только на audit-поле `createdBy`.
3. По умолчанию исключить terminal status заявки: `Closed`.
4. Применить фильтры по `status`, `type`, `priority`, `search`, `createdFrom`, `createdTo`.
5. Если передан `status`, он должен относиться только к статусам [незавершенной заявки](../../../glossary/Glossary.md): `Draft`, `Submitted`, `InProgress`.
6. Отсортировать по `createdAt DESC`.
7. Для каждой заявки собрать summary-данные по связанным `Bookings`.
8. Для каждой брони вернуть только краткий `bookingSummary`, достаточный для таблицы списка заявок.
9. Вернуть пагинированный список.

Метод не возвращает полные детали заявки или полные карточки броней. Для этого используется [`GET /booking-requests/{id}`](GET_booking_requests_id.md).

Сущности, участвующие в методе:
- читаются: [`BookingRequests`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#21-bookingrequests), [`Bookings`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#22-bookings), [`Equipments`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#8-equipments), [`EquipmentTypes`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#1-equipmenttypes), [`EquipmentBrands`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#4-equipmentbrands), [`EquipmentModels`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#5-equipmentmodels), [`WorkCenters`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#3-workcenters), [`Fleets`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#2-fleets), [`Users`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#19-users)
- изменения не выполняются

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Requestor` | Просмотр заявок, где пользователь является `requestorId` |
| `ServiceWorkProcessor` | Просмотр своих SWR |

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
| `FORBIDDEN` | Нет необходимой роли |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Фильтр по статусу [незавершенной заявки](../../../glossary/Glossary.md) | `status` | `string` | `-` | `Draft / Submitted / InProgress` | — | Query param | Значения загружаются через [`GET /reference/request-statuses`](../reference/GET_reference_request_statuses.md); terminal status `Closed` в этом методе не используется |
| 2 | Фильтр по типу заявки | `type` | `string` | `-` | `Regular / ServiceWork` | — | Query param | Значения загружаются через [`GET /reference/requestor-request-types`](../reference/GET_reference_requestor_request_types.md) |
| 3 | Фильтр по приоритету | `priority` | `string` | `-` | `P1 / P2 / P3 / P4` | — | Query param | Значения загружаются через [`GET /reference/request-priorities`](../reference/GET_reference_request_priorities.md) |
| 4 | Поисковая строка | `search` | `string` | `-` | Поиск по `requestNumber`, `workOrderNumber`, `workDescription` | — | Query param | |
| 5 | Дата создания заявки: начало диапазона | `createdFrom` | `date` | `-` | `<= createdTo`, формат `YYYY-MM-DD` | — | Query param | Фильтр по `BookingRequests.createdAt` |
| 6 | Дата создания заявки: конец диапазона | `createdTo` | `date` | `-` | `>= createdFrom`, формат `YYYY-MM-DD` | — | Query param | Фильтр по `BookingRequests.createdAt` |
| 7 | Номер страницы | `page` | `int` | `-` | >= 1 | `1` | Query param | |
| 8 | Размер страницы | `limit` | `int` | `-` | >= 1 | `20` | Query param | |

---

## 8. Пример запроса

```http
GET /api/booking/v1/booking-requests/my?status=Submitted&createdFrom=2026-05-01&createdTo=2026-05-31&page=1&limit=20
Authorization: Bearer <token>
Content-Type: application/json
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий [`result wrapper`](../../common/Result%20Wrapper.md).

Метод возвращает только [незавершенные заявки](../../../glossary/Glossary.md) со статусами `Draft`, `Submitted`, `InProgress`. Terminal request statuses должны запрашиваться через отдельный history/archive endpoint.

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | value | object | PaginatedResult | — | backend aggregation |  |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | object[] | `[]` | backend |  |

### Структура `value`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Элементы текущей страницы | items | array<object> | object[] | — | backend composition from BookingRequests + Bookings + Equipments + EquipmentTypes + EquipmentBrands + EquipmentModels + WorkCenters + Fleets + Users | Коллекция объектов |
| 2 | Общее количество записей | total | int | integer | — | backend |  |

### Структура `value.items[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | BookingRequests.id |  |
| 2 | Номер заявки | requestNumber | string | string | — | BookingRequests.requestNumber |  |
| 3 | Тип сущности / заявки | type | string | string | — | BookingRequests + ref_request_type |  |
| 4 | Текущий статус | status | string | string | — | BookingRequests + BookingRequestStatuses |  |
| 5 | Кто обновил текущий статус заявки | statusUpdatedBy | object | object | — | last BookingRequestStatuses + Users | Автор последнего status transition |
| 6 | Приоритет | priority | string | string | — | BookingRequests + ref_request_priority |  |
| 7 | Номер Work Order | workOrderNumber | string | string | — | BookingRequests.workOrderNumber |  |
| 8 | Дата и время создания | createdAt | datetime | ISO 8601 | — | BookingRequests.createdAt |  |
| 9 | Данные реквестора | requestor | object | object | — | BookingRequests.requestorId + Users | Business-requestor заявки |
| 10 | Количество броней | bookingsCount | int | integer | — | COUNT(Bookings) |  |
| 11 | Сводный список броней | bookingSummaries | array<object> | object[] | — | backend composition from Bookings + Equipments + EquipmentTypes + EquipmentBrands + EquipmentModels + WorkCenters + Fleets + Users | Коллекция объектов |

### Структура `value.items[].requestor`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | Users.id |  |
| 2 | Полное имя пользователя | fullName | string | string | — | Users.fullName |  |
| 3 | Email | email | string | string | — | Users.email |  |

### Структура `value.items[].statusUpdatedBy`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор пользователя | userId | uuid | UUID v4 | — | Users.id | Автор последнего status transition заявки |
| 2 | Полное имя пользователя | fullName | string | string | — | Users.fullName |  |
| 3 | Email | email | string | string | — | Users.email |  |

### Структура `value.items[].bookingSummaries[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | Bookings.id |  |
| 2 | Тип техники | equipmentType | string | string | — | EquipmentTypes |  |
| 3 | Марка и модель техники | brandModel | string | string | — | Equipments + EquipmentBrands + EquipmentModels |  |
| 4 | ТШО-номер техники | tcoId | string | string | — | Equipments.tcoId |  |
| 5 | Государственный регистрационный номер | stateNumber | string | string | — | Equipments.stateNumber |  |
| 6 | Описание техники | equipmentDescription | string | string | — | Equipments.description | Описание/комментарий по единице техники |
| 7 | Конфликты с бронями в активном статусе (`Submitted`, `Confirmed`, `InProgress`) | activeBookingConflicts | object | object | — | backend overlap check against active bookings | Учитываются только брони со статусами `Submitted`, `Confirmed`, `InProgress` и с пересечением диапазона дат; `Draft` и `Closed` не учитываются |
| 8 | Код рабочего центра | workCenterCode | string | string | — | WorkCenters.code |  |
| 9 | Fleet | fleet | object | object | — | Fleets | Базовый контекст флота техники |
| 10 | Плановая дата и время начала | plannedStartDateTime | datetime | ISO 8601 | — | Bookings.plannedStartDateTime |  |
| 11 | Плановая дата и время окончания | plannedEndDateTime | datetime | ISO 8601 | — | Bookings.plannedEndDateTime |  |
| 12 | Фактическая дата и время начала | actualStartDateTime | null | — | `null` | Bookings.actualStartDateTime |  |
| 13 | Фактическая дата и время окончания | actualEndDateTime | null | — | `null` | Bookings.actualEndDateTime |  |
| 14 | Обоснование | justification | string | string | — | Bookings.justification | Для draft-item может быть пустым до позднего заполнения |
| 15 | Признак, что для item обязателен justification | requiresJustification | bool | boolean | — | backend business rule from Equipments + ref_ownership_type + ref_share_type | `true`, если `ownershipType = LongTermRented` или `shareType IN (Assigned, SharedWithConditions)` |
| 16 | Признак наличия обязательного justification | hasRequiredJustification | bool | boolean | — | backend business rule | `false`, если для item обязателен `justification`, но он еще не заполнен |
| 17 | Текущий статус | status | string | string | — | Bookings + BookingStatuses |  |
| 18 | Кто обновил текущий статус брони | statusUpdatedBy | object | object | — | last BookingStatuses + Users | Автор последнего status transition |

### Структура `value.items[].bookingSummaries[].fleet`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор флота | id | uuid | UUID v4 | — | Fleets.id |  |
| 2 | Наименование флота | name | string | string | — | Fleets.nameEn / localized projection |  |

### Структура `value.items[].bookingSummaries[].statusUpdatedBy`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор пользователя | userId | uuid | UUID v4 | — | Users.id | Автор последнего status transition брони |
| 2 | Полное имя пользователя | fullName | string | string | — | Users.fullName |  |
| 3 | Email | email | string | string | — | Users.email |  |

### Структура `value.items[].bookingSummaries[].activeBookingConflicts`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Наличие конфликтов с бронями в активном статусе (`Submitted`, `Confirmed`, `InProgress`) | hasActiveBookingConflicts | bool | boolean | `false` | backend overlap check against active bookings | `true`, если есть хотя бы одна бронь по этой же технике со статусом `Submitted`, `Confirmed` или `InProgress` и с пересечением диапазона дат; `Draft` и `Closed` не учитываются |
| 2 | Количество конфликтов с бронями в активном статусе (`Submitted`, `Confirmed`, `InProgress`) | activeBookingConflictsCount | int | integer | `0` | backend aggregation | Количество броней по этой же технике со статусом `Submitted`, `Confirmed` или `InProgress` и с пересечением диапазона дат; `Draft` и `Closed` не учитываются |

## 10. Пример ответа

```json
{
  "value": {
    "items": [
      {
        "id": "c777f75f-029d-4d8f-8c69-e74a1d280001",
        "requestNumber": "REQ-2026-00015",
        "type": "Regular",
        "status": "Submitted",
        "statusUpdatedBy": {
          "userId": "11111111-1111-1111-1111-111111111111",
          "fullName": "Telman Nurzhanov",
          "email": "telman.nurzhanov@example.com"
        },
        "priority": "P2",
        "workOrderNumber": "WO-10025",
        "createdAt": "2026-05-14T09:15:00Z",
        "requestor": {
          "id": "11111111-1111-1111-1111-111111111111",
          "fullName": "Telman Nurzhanov",
          "email": "telman.nurzhanov@example.com"
        },
        "bookingsCount": 2,
        "bookingSummaries": [
          {
            "id": "b777f75f-029d-4d8f-8c69-e74a1d280001",
            "equipmentType": "Excavator",
            "brandModel": "CAT 320",
            "tcoId": "TCO-12345",
            "stateNumber": "A123BC",
            "equipmentDescription": "Tracked excavator with standard bucket for trench preparation works.",
            "activeBookingConflicts": {
              "hasActiveBookingConflicts": true,
              "activeBookingConflictsCount": 2
            },
            "workCenterCode": "BHOE",
            "fleet": {
              "id": "f0000001-0000-4000-8000-000000000001",
              "name": "Maintenance Fleet"
            },
            "plannedStartDateTime": "2026-05-20T08:00:00Z",
            "plannedEndDateTime": "2026-05-22T20:00:00Z",
            "actualStartDateTime": null,
            "actualEndDateTime": null,
            "justification": null,
            "requiresJustification": true,
            "hasRequiredJustification": false,
            "status": "Submitted",
            "statusUpdatedBy": {
              "userId": "11111111-1111-1111-1111-111111111111",
              "fullName": "Telman Nurzhanov",
              "email": "telman.nurzhanov@example.com"
            }
          },
          {
            "id": "b777f75f-029d-4d8f-8c69-e74a1d280002",
            "equipmentType": "Water Truck",
            "brandModel": "HOWO 6x4",
            "tcoId": "TCO-98765",
            "stateNumber": "B456CD",
            "equipmentDescription": "Water truck configured for dust suppression and site support.",
            "activeBookingConflicts": {
              "hasActiveBookingConflicts": false,
              "activeBookingConflictsCount": 0
            },
            "workCenterCode": "HYDR",
            "fleet": {
              "id": "f0000001-0000-4000-8000-000000000002",
              "name": "Operations Fleet"
            },
            "plannedStartDateTime": "2026-05-21T08:00:00Z",
            "plannedEndDateTime": "2026-05-21T18:00:00Z",
            "actualStartDateTime": null,
            "actualEndDateTime": null,
            "justification": "Dust suppression required for road preparation.",
            "requiresJustification": true,
            "hasRequiredJustification": true,
            "status": "Submitted",
            "statusUpdatedBy": {
              "userId": "11111111-1111-1111-1111-111111111111",
              "fullName": "Telman Nurzhanov",
              "email": "telman.nurzhanov@example.com"
            }
          }
        ]
      }
    ],
    "total": 1
  },
  "isSuccess": true,
  "errors": []
}
```

---

## Замечания

1. Метод предназначен именно для страницы `Мои заявки` и возвращает только active / non-terminal requests со статусами `Draft`, `Submitted`, `InProgress`.
2. Для истории завершённых заявок должен использоваться отдельный endpoint [`GET /booking-requests/history`](GET_booking_requests_history.md).
3. Поле `status` в query не должно использоваться для `Closed`.
