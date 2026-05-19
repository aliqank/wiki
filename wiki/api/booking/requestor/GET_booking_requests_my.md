# GET /booking-requests/my

**Created:** 2026-05-14  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить список собственных незавершенных заявок текущего пользователя |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Requestor UI` |
| Endpoint URL | `/api/booking/v1/booking-requests/my` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется для страницы «Мои заявки» как summary API для списка незавершенных заявок Requestor / SWP.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-025 | Requestor can view request details and status | Confirmed | BRD v13 | Частичное покрытие на уровне summary-list незавершенных заявок |
| TCO Booking Tool | FR-091 | Requestor/SWP can search and filter own requests | Confirmed | BRD v13 | Прямое покрытие |

---

## 3. Описание логики работы метода

1. Выбрать `BookingRequests` по `createdBy = currentUserId`.
2. По умолчанию исключить terminal statuses заявки: `Completed` и `Cancelled`.
3. Применить фильтры по `status`, `type`, `priority`, `search`, `createdFrom`, `createdTo`.
4. Если передан `status`, он должен относиться только к незавершённым статусам заявки.
5. Отсортировать по `createdAt DESC`.
6. Для каждой заявки собрать summary-данные по связанным `Bookings`.
7. Для каждой брони вернуть только краткий `bookingSummary`, достаточный для таблицы списка заявок.
8. Вернуть пагинированный список.

Метод не возвращает полные детали заявки или полные карточки броней. Для этого используется `GET /booking-requests/{id}`.

Сущности, участвующие в методе:
- читаются: `BookingRequests`, `Bookings`, `Equipments`, `EquipmentTypes`, `EquipmentBrands`, `EquipmentModels`, `WorkCenters`, `Fleets`, `Users`
- изменения не выполняются

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Requestor` | Просмотр своих заявок |
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
| 1 | Фильтр по статусу незавершенной заявки | `status` | `enum` | `-` | `Draft / Submitted / InProgress` | — | Query param | Terminal statuses `Completed` и `Cancelled` в этом методе не используются |
| 2 | Фильтр по типу заявки | `type` | `enum` | `-` | `Regular / ServiceWork` | — | Query param | |
| 3 | Фильтр по приоритету | `priority` | `enum` | `-` | `P1 / P2 / P3 / P4` | — | Query param | |
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

Возвращаемые данные обёрнуты в общий `result wrapper`.

Метод возвращает только незавершенные заявки. Завершенные и отмененные заявки должны запрашиваться через отдельный history/archive endpoint.

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
| 5 | Приоритет | priority | string | string | — | BookingRequests + ref_request_priority |  |
| 6 | Номер Work Order из JDE | workOrderNumber | string | string | — | BookingRequests.workOrderNumber |  |
| 7 | Дата и время создания | createdAt | datetime | ISO 8601 | — | BookingRequests.createdAt |  |
| 8 | Данные реквестора | requestor | object | object | — | Users |  |
| 9 | Количество броней | bookingsCount | int | integer | — | COUNT(Bookings) |  |
| 10 | Сводный список броней | bookingSummaries | array<object> | object[] | — | backend composition from Bookings + Equipments + EquipmentTypes + EquipmentBrands + EquipmentModels + WorkCenters + Fleets + Users | Коллекция объектов |

### Структура `value.items[].requestor`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | Users.id |  |
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
| 7 | Конфликты с опубликованными бронями | publishedConflicts | object | object | — | backend overlap check against published bookings | Учитываются только опубликованные брони с пересечением диапазона дат; черновики не учитываются |
| 8 | Код рабочего центра | workCenterCode | string | string | — | WorkCenters.code |  |
| 9 | Владелец / ответственный fleet | fleetOwner | string | string | — | Fleets + Users |  |
| 10 | Плановая дата и время начала | plannedStartDateTime | datetime | ISO 8601 | — | Bookings.plannedStartDateTime |  |
| 11 | Плановая дата и время окончания | plannedEndDateTime | datetime | ISO 8601 | — | Bookings.plannedEndDateTime |  |
| 12 | Фактическая дата и время начала | actualStartDateTime | null | — | `null` | Bookings.actualStartDateTime |  |
| 13 | Фактическая дата и время окончания | actualEndDateTime | null | — | `null` | Bookings.actualEndDateTime |  |
| 14 | Обоснование | justification | string | string | — | Bookings.justification | Для draft-item может быть пустым до позднего заполнения |
| 15 | Признак, что для item обязателен justification | requiresJustification | bool | boolean | — | backend business rule from Equipments + ref_ownership_type + ref_share_type | `true`, если `ownershipType = LongTermRented` или `shareType IN (Assigned, SharedWithConditions)` |
| 16 | Признак завершенности item | isComplete | bool | boolean | — | backend business rule | `false`, если обязательный `justification` еще не заполнен |
| 17 | Текущий статус | status | string | string | — | Bookings + BookingStatuses |  |

### Структура `value.items[].bookingSummaries[].publishedConflicts`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Наличие конфликтов с опубликованными бронями | hasPublishedConflicts | bool | boolean | `false` | backend overlap check against published bookings | `true`, если есть хотя бы одна опубликованная бронь по этой же технике с пересечением диапазона дат; черновики не учитываются |
| 2 | Количество конфликтов с опубликованными бронями | publishedConflictsCount | int | integer | `0` | backend aggregation | Количество опубликованных броней по этой же технике с пересечением диапазона дат; черновики не учитываются |

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
            "publishedConflicts": {
              "hasPublishedConflicts": true,
              "publishedConflictsCount": 2
            },
            "workCenterCode": "BHOE",
            "fleetOwner": "Maintenance Fleet",
            "plannedStartDateTime": "2026-05-20T08:00:00Z",
            "plannedEndDateTime": "2026-05-22T20:00:00Z",
            "actualStartDateTime": null,
            "actualEndDateTime": null,
            "justification": null,
            "requiresJustification": true,
            "isComplete": false,
            "status": "Submitted"
          },
          {
            "id": "b777f75f-029d-4d8f-8c69-e74a1d280002",
            "equipmentType": "Water Truck",
            "brandModel": "HOWO 6x4",
            "tcoId": "TCO-98765",
            "stateNumber": "B456CD",
            "equipmentDescription": "Water truck configured for dust suppression and site support.",
            "publishedConflicts": {
              "hasPublishedConflicts": false,
              "publishedConflictsCount": 0
            },
            "workCenterCode": "HYDR",
            "fleetOwner": "Operations Fleet",
            "plannedStartDateTime": "2026-05-21T08:00:00Z",
            "plannedEndDateTime": "2026-05-21T18:00:00Z",
            "actualStartDateTime": null,
            "actualEndDateTime": null,
            "justification": "Dust suppression required for road preparation.",
            "requiresJustification": true,
            "isComplete": true,
            "status": "Submitted"
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

1. Метод предназначен именно для страницы `Мои заявки` и возвращает только active / non-terminal requests.
2. Для истории завершённых заявок должен использоваться отдельный endpoint `GET /booking-requests/history`.
3. Поле `status` в query не должно использоваться для `Completed` и `Cancelled`.
