# GET /approvals/requests

**Created:** 2026-05-21  
**Last updated:** 2026-06-02  
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
| Связанные use cases | [`UC-FO-01 - Просмотр списка заявок Fleet Owner`](../../../requirements/usecases/Fleet%20Owner/UC-FO-01%20-%20Просмотр%20списка%20заявок%20Fleet%20Owner.md) |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется для Fleet Owner view `Requests` как request-centric список заявок с составом броней, достаточным для первичной обработки без обязательного дополнительного detail-запроса.

Для загрузки значений фильтров UI использует отдельные reference API: `GET /reference/approval-request-statuses`, `GET /reference/approval-request-types`, `GET /reference/request-priorities`.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-092 | Approver (FO) can view all pending and completed approvals | Confirmed | BRD v13 | Request-level view для зоны ответственности FO |
| TCO Booking Tool | FR-043 | Approver assigned automatically based on fleet ownership | Confirmed | BRD v13 | Отбор заявок по fleet-ам FO |
| TCO Booking Tool | FR-094 | System generates reports on Requests/Bookings/Equipment with filters | Confirmed | BRD v13 | Фильтрация request-level списка |

---

## 3. Описание логики работы метода

1. Определить список fleet-ов, где у текущего Fleet Owner есть [Fleet Management Access](../../../glossary/Glossary.md#fleet-management-access), а также технику, доступную ему в рамках этого доступа.
2. Выбрать `BookingRequests`, в составе которых есть хотя бы один `Booking`, относящийся к этим fleet-ам и/или к технике, доступной пользователю по делегированию.
3. По умолчанию исключить из выдачи заявки со статусами `Draft`, `Closed`.
4. Применить request-level фильтры по статусу, типу, приоритету, поиску и периоду создания заявки.
5. Отсортировать заявки по `createdAt DESC`.
6. Для каждой заявки вернуть только связанные booking item-ы, которые относятся к зоне ответственности текущего Fleet Owner.
7. Для каждой такой брони собрать краткий `bookingSummary`, достаточный для принятия решения о дальнейшей работе с заявкой, включая краткий conflict context для UI.
8. Вернуть paginated список.

Сущности, участвующие в методе:
- читаются: [`BookingRequests`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#21-bookingrequests), [`Bookings`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#22-bookings), [`Equipments`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#8-equipments), [`EquipmentTypes`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#1-equipmenttypes), [`EquipmentBrands`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#4-equipmentbrands), [`EquipmentModels`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#5-equipmentmodels), [`WorkCenters`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#3-workcenters), [`Fleets`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#2-fleets), [`Users`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#19-users), [`EquipmentBookingAuthorizations`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#17-equipmentfeedbacks--equipmentbookingauthorizations), [`FleetManagePermissions`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#7-fleetmanagepermissions)
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
| 1 | Фильтр по статусу заявки | `status` | `string` | `-` | `Submitted / InProgress` | — | Query param | Значения загружаются через `GET /reference/approval-request-statuses`; используется агрегированный request status; `Draft` и `Closed` не должны возвращаться в этом методе |
| 2 | Фильтр по типу заявки | `type` | `string` | `-` | `Regular / ServiceWork` | — | Query param | Значения загружаются через `GET /reference/approval-request-types` |
| 3 | Фильтр по приоритету | `priority` | `string` | `-` | `P1 / P2 / P3 / P4` | — | Query param | Значения загружаются через `GET /reference/request-priorities` |
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

Возвращаемые данные обёрнуты в общий [`result wrapper`](../../common/Result%20Wrapper.md).

Метод возвращает только активные для Fleet Owner заявки со статусами `Submitted` и `InProgress`. `Draft` и `Closed` не должны попадать в выдачу текущего списка; завершенные и архивные сценарии должны обслуживаться отдельным history flow.

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
| 5 | Кто обновил текущий статус заявки | statusUpdatedBy | object | object | — | last BookingRequestStatuses + Users | Автор последнего status transition |
| 6 | Приоритет | priority | string | string | — | BookingRequests + ref_request_priority |  |
| 7 | Номер Work Order | workOrderNumber | string | string | — | BookingRequests.workOrderNumber |  |
| 8 | Дата и время создания заявки | createdAt | datetime | ISO 8601 | — | BookingRequests.createdAt |  |
| 9 | Данные requestor | requestor | object | object | — | BookingRequests.requestorId + Users | Business-requestor заявки |
| 10 | Количество броней, доступных текущему Fleet Owner | bookingsCount | int | integer | — | backend aggregation | Считаются только item-ы в зоне ответственности текущего FO |
| 11 | Сводный список броней для работы | bookingSummaries | array<object> | object[] | `[]` | backend composition from Bookings + Equipments + EquipmentTypes + EquipmentBrands + EquipmentModels + WorkCenters + Fleets + Users | Возвращаются только релевантные FO item-ы |

### Структура `value.items[].requestor`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор пользователя | id | uuid | UUID v4 | — | Users.id |  |
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
| 11 | Fleet | fleet | object | object | — | Fleets | Базовый контекст флота техники |
| 12 | Список Fleet Owners | fleetOwners | array<object> | object[] | `[]` | Fleets + FleetManagePermissions + Users | Только owner-assignment'ы для флота |
| 13 | Плановая дата и время начала | plannedStartDateTime | datetime | ISO 8601 | — | Bookings.plannedStartDateTime |  |
| 14 | Плановая дата и время окончания | plannedEndDateTime | datetime | ISO 8601 | — | Bookings.plannedEndDateTime |  |
| 15 | Текущий статус брони | status | string | string | — | Bookings + BookingStatuses |  |
| 16 | Кто обновил текущий статус брони | statusUpdatedBy | object | object | — | last BookingStatuses + Users | Автор последнего status transition |
| 17 | Обоснование | justification | string | string | — | Bookings.justification |  |
| 18 | Есть ли активные конфликты с другими бронями по той же технике | hasConflicts | bool | boolean | `false` | backend overlap calculation from Bookings | Используется для отображения conflict indicator / quick action в строке брони |
| 19 | Количество конфликтующих броней | conflictsCount | int | integer | `0` | backend overlap calculation from Bookings | Количество активных пересечений по статусам `Submitted`, `Confirmed`, `InProgress` |

### Структура `value.items[].bookingSummaries[].fleet`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор флота | id | uuid | UUID v4 | — | Fleets.id |  |
| 2 | Наименование флота | name | string | string | — | Fleets.nameEn / localized projection |  |

### Структура `value.items[].bookingSummaries[].fleetOwners[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор пользователя | userId | uuid | UUID v4 | — | Users.id | Только owner-assignment'ы из `FleetManagePermissions` |
| 2 | Полное имя | fullName | string | string | — | Users.fullName |  |
| 3 | Email | email | string | string | — | Users.email |  |

### Структура `value.items[].bookingSummaries[].statusUpdatedBy`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор пользователя | userId | uuid | UUID v4 | — | Users.id | Автор последнего status transition брони |
| 2 | Полное имя | fullName | string | string | — | Users.fullName |  |
| 3 | Email | email | string | string | — | Users.email |  |

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
            "fleet": {
              "id": "f0000001-0000-4000-8000-000000000001",
              "name": "Maintenance Fleet"
            },
            "fleetOwners": [
              {
                "userId": "4c9ad2d2-6df8-4f7b-87fe-36cefc100001",
                "fullName": "Nurlan Sarsenov",
                "email": "nurlan.sarsenov@tco.example"
              },
              {
                "userId": "4c9ad2d2-6df8-4f7b-87fe-36cefc100002",
                "fullName": "Aidos Beketov",
                "email": "aidos.beketov@tco.example"
              }
            ],
            "plannedStartDateTime": "2026-05-20T08:00:00Z",
            "plannedEndDateTime": "2026-05-22T18:00:00Z",
            "status": "Submitted",
            "statusUpdatedBy": {
              "userId": "11111111-1111-1111-1111-111111111111",
              "fullName": "Telman Nurzhanov",
              "email": "telman.nurzhanov@example.com"
            },
            "justification": null,
            "hasConflicts": true,
            "conflictsCount": 2
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
            "fleet": {
              "id": "f0000001-0000-4000-8000-000000000002",
              "name": "Operations Fleet"
            },
            "fleetOwners": [
              {
                "userId": "4c9ad2d2-6df8-4f7b-87fe-36cefc100003",
                "fullName": "Marat Ibragimov",
                "email": "marat.ibragimov@tco.example"
              }
            ],
            "plannedStartDateTime": "2026-05-21T08:00:00Z",
            "plannedEndDateTime": "2026-05-21T18:00:00Z",
            "status": "Submitted",
            "statusUpdatedBy": {
              "userId": "11111111-1111-1111-1111-111111111111",
              "fullName": "Telman Nurzhanov",
              "email": "telman.nurzhanov@example.com"
            },
            "justification": "Dust suppression required for road preparation.",
            "hasConflicts": false,
            "conflictsCount": 0
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
