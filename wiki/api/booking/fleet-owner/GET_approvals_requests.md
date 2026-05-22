# GET /approvals/requests

**Created:** 2026-05-21  
**Last updated:** 2026-05-21  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить список заявок Fleet Owner с вложенными бронями для работы |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Fleet Owner UI` |
| Endpoint URL | `/api/booking/v1/approvals/requests` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется для Fleet Owner view `Requests` как request-centric список заявок с составом броней, достаточным для первичной обработки без обязательного дополнительного detail-запроса.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-092 | Approver (FO) can view all pending and completed approvals | Confirmed | BRD v13 | Request-level view для зоны ответственности FO |
| TCO Booking Tool | FR-043 | Approver assigned automatically based on fleet ownership | Confirmed | BRD v13 | Отбор заявок по fleet-ам FO |
| TCO Booking Tool | FR-094 | System generates reports on Requests/Bookings/Equipment with filters | Confirmed | BRD v13 | Фильтрация request-level списка |

---

## 3. Описание логики работы метода

1. Определить список fleet-ов, где у текущего Fleet Owner есть assignment в `FleetManagePermissions` с типом `Owner` или `Delegated`, а также технику, доступную ему по делегированию.
2. Выбрать `BookingRequests`, в составе которых есть хотя бы один `Booking`, относящийся к этим fleet-ам и/или к технике, доступной пользователю по делегированию.
3. По умолчанию исключить из выдачи заявки со статусами `Draft`, `Closed`.
4. Применить request-level фильтры по статусу, типу, приоритету, поиску и периоду создания заявки.
5. Отсортировать заявки по `createdAt DESC`.
6. Для каждой заявки вернуть только связанные booking item-ы, которые относятся к зоне ответственности текущего Fleet Owner.
7. Для каждой такой брони собрать краткий `bookingSummary`, достаточный для принятия решения о дальнейшей работе с заявкой.
8. Вернуть paginated список.

Сущности, участвующие в методе:
- читаются: `BookingRequests`, `Bookings`, `Equipments`, `EquipmentTypes`, `EquipmentBrands`, `EquipmentModels`, `WorkCenters`, `Fleets`, `Users`, `EquipmentBookingAuthorizations`, `FleetManagePermissions`
- изменения не выполняются

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `FleetOwner` | Просмотр заявок и связанных броней по управляемым флотам |

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
| 1 | Фильтр по статусу заявки | `status` | `enum` | `-` | `Submitted / InProgress` | — | Query param | Используется агрегированный request status; `Draft` и `Closed` не должны возвращаться в этом методе |
| 2 | Фильтр по типу заявки | `type` | `enum` | `-` | `Regular / ServiceWork` | — | Query param | |
| 3 | Фильтр по приоритету | `priority` | `enum` | `-` | `P1 / P2 / P3 / P4` | — | Query param | |
| 4 | Поисковая строка | `search` | `string` | `-` | Поиск по `requestNumber`, `workOrderNumber` | — | Query param | |
| 5 | Дата создания заявки: начало диапазона | `createdFrom` | `date` | `-` | `<= createdTo`, формат `YYYY-MM-DD` | — | Query param | Фильтр по `BookingRequests.createdAt` |
| 6 | Дата создания заявки: конец диапазона | `createdTo` | `date` | `-` | `>= createdFrom`, формат `YYYY-MM-DD` | — | Query param | Фильтр по `BookingRequests.createdAt` |
| 7 | Номер страницы | `page` | `int` | `-` | >= 1 | `1` | Query param | |
| 8 | Размер страницы | `limit` | `int` | `-` | >= 1 | `20` | Query param | |

---

## 8. Пример запроса

```http
GET /api/booking/v1/approvals/requests?status=Submitted&type=Regular&createdFrom=2026-05-01&createdTo=2026-05-31&page=1&limit=20
Authorization: Bearer <token>
Content-Type: application/json
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий `result wrapper`.

Метод возвращает только активные для Fleet Owner заявки. `Draft` и `Closed` не должны попадать в выдачу текущего списка; завершенные и архивные сценарии должны обслуживаться отдельным history flow.

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
| 4 | Текущий статус заявки | status | string | string | — | BookingRequests + BookingRequestStatuses |  |
| 5 | Приоритет | priority | string | string | — | BookingRequests + ref_request_priority |  |
| 6 | Номер Work Order | workOrderNumber | string | string | — | BookingRequests.workOrderNumber |  |
| 7 | Дата и время создания заявки | createdAt | datetime | ISO 8601 | — | BookingRequests.createdAt |  |
| 8 | Данные requestor | requestor | object | object | — | Users |  |
| 9 | Количество броней, доступных текущему Fleet Owner | bookingsCount | int | integer | — | backend aggregation | Считаются только item-ы в зоне ответственности текущего FO |
| 10 | Сводный список броней для работы | bookingSummaries | array<object> | object[] | `[]` | backend composition from Bookings + Equipments + EquipmentTypes + EquipmentBrands + EquipmentModels + WorkCenters + Fleets + Users | Возвращаются только релевантные FO item-ы |

### Структура `value.items[].requestor`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор пользователя | id | uuid | UUID v4 | — | Users.id |  |
| 2 | Полное имя пользователя | fullName | string | string | — | Users.fullName |  |
| 3 | Email | email | string | string | — | Users.email |  |

### Структура `value.items[].bookingSummaries[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор брони | id | uuid | UUID v4 | — | Bookings.id |  |
| 2 | Тип техники | equipmentType | string | string | — | EquipmentTypes |  |
| 3 | Марка и модель техники | brandModel | string | string | — | EquipmentBrands + EquipmentModels |  |
| 4 | ТШО-номер техники | tcoId | string | string | — | Equipments.tcoId |  |
| 5 | Государственный регистрационный номер | stateNumber | string | string | — | Equipments.stateNumber |  |
| 6 | Описание техники | equipmentDescription | string | string | — | Equipments.description |  |
| 7 | Требуется ли транспортировка | requiresTransport | bool | boolean | — | EquipmentTypes.requiresTransport |  |
| 8 | Тип доступности техники | shareType | string | string | — | Equipments + ref_share_type |  |
| 9 | Тип владения техникой | ownershipType | string | string | — | Equipments + ref_ownership_type |  |
| 10 | Код рабочего центра | workCenterCode | string | string | — | WorkCenters.code |  |
| 11 | Данные Fleet Owner | fleetOwner | object | object | — | Fleets + Users |  |
| 12 | Плановая дата и время начала | plannedStartDateTime | datetime | ISO 8601 | — | Bookings.plannedStartDateTime |  |
| 13 | Плановая дата и время окончания | plannedEndDateTime | datetime | ISO 8601 | — | Bookings.plannedEndDateTime |  |
| 14 | Текущий статус брони | status | string | string | — | Bookings + BookingStatuses |  |
| 15 | Обоснование | justification | string | string | — | Bookings.justification |  |

### Структура `value.items[].bookingSummaries[].fleetOwner`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор пользователя | userId | uuid | UUID v4 | — | Users.id |  |
| 2 | Полное имя | fullName | string | string | — | Users.fullName |  |
| 3 | Email | email | string | string | — | Users.email |  |
| 4 | Наименование fleet | fleetName | string | string | — | Fleets.nameEn / localized projection |  |

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
            "id": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
            "equipmentType": "Excavator",
            "brandModel": "CAT 320D",
            "tcoId": "TCO-100245",
            "stateNumber": "KZ 123 ABC 02",
            "equipmentDescription": "Tracked excavator with trenching bucket and reinforced undercarriage.",
            "requiresTransport": false,
            "shareType": "SharedWithConditions",
            "ownershipType": "TcoOwned",
            "workCenterCode": "BHOE",
            "fleetOwner": {
              "userId": "4c9ad2d2-6df8-4f7b-87fe-36cefc100001",
              "fullName": "Nurlan Sarsenov",
              "email": "nurlan.sarsenov@tco.example",
              "fleetName": "Maintenance Fleet"
            },
            "plannedStartDateTime": "2026-05-20T08:00:00Z",
            "plannedEndDateTime": "2026-05-22T18:00:00Z",
            "status": "Submitted",
            "justification": null
          },
          {
            "id": "8c4c8b6d-7bc0-41fb-9038-422cf55d2222",
            "equipmentType": "Water Truck",
            "brandModel": "HOWO 6x4",
            "tcoId": "TCO-98765",
            "stateNumber": "B456CD",
            "equipmentDescription": "Water truck configured for dust suppression and site support.",
            "requiresTransport": true,
            "shareType": "Assigned",
            "ownershipType": "LongTermRented",
            "workCenterCode": "HYDR",
            "fleetOwner": {
              "userId": "4c9ad2d2-6df8-4f7b-87fe-36cefc100001",
              "fullName": "Nurlan Sarsenov",
              "email": "nurlan.sarsenov@tco.example",
              "fleetName": "Operations Fleet"
            },
            "plannedStartDateTime": "2026-05-21T08:00:00Z",
            "plannedEndDateTime": "2026-05-21T18:00:00Z",
            "status": "Submitted",
            "justification": "Dust suppression required for road preparation."
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

1. Метод является request-centric: одна строка списка соответствует одной заявке, но внутри строки возвращается релевантный набор связанных броней.
2. В `bookingSummaries` не должны попадать чужие booking item-ы, не относящиеся к зоне ответственности текущего Fleet Owner.
3. Если в заявке есть брони по нескольким fleet-ам, Fleet Owner видит только те item-ы, по которым он вправе принимать решение.
4. Текущая версия метода должна содержать достаточно данных для первичной работы с заявкой и её релевантными booking item-ами без обязательного отдельного detail API.
