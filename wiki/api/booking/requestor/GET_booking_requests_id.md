# GET /booking-requests/{id}

**Created:** 2026-05-14  
**Last updated:** 2026-06-02  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить детальную информацию по заявке |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / [`Requestor`](../../../requirements/Roles and Access Model.md) UI` |
| Endpoint URL | `/api/booking/v1/booking-requests/{id}` |
| Метод запроса | `GET` |
| Связанные use cases | [`UC-REQ-01 - Просмотр моих заявок`](../../../requirements/usecases/Requestor/UC-REQ-01%20-%20Просмотр%20моих%20заявок.md), [`UC-REQ-03 - Отмена draft-заявки requestor-ом`](../../../requirements/usecases/Requestor/UC-REQ-03%20-%20Отмена%20draft-заявки%20requestor-ом.md), [`UC-REQ-04 - Отзыв брони requestor-ом`](../../../requirements/usecases/Requestor/UC-REQ-04%20-%20Отзыв%20брони%20requestor-ом.md), [`UC-REQ-05 - Отзыв submitted-заявки requestor-ом`](../../../requirements/usecases/[`Requestor`](../../../requirements/Roles and Access Model.md)/UC-REQ-05%20-%20Отзыв%20submitted-заявки%20requestor-ом.md) |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется для открытия и обновления страницы `Новая заявка / Редактировать заявку`, включая sidebar request-level полей и полный список booking item-ов.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-024 | Request has unique ID and metadata | Confirmed | BRD v13 | Метод возвращает идентификатор и метаданные заявки |
| TCO Booking Tool | FR-025 | [`Requestor`](../../../requirements/Roles and Access Model.md) can view request details and status | Confirmed | BRD v13 | Прямое покрытие |
| TCO Booking Tool | FR-026 | Request statuses: Draft, Submitted, In Progress, Completed | Confirmed | BRD v13 | Метод возвращает текущий статус заявки |
| TCO Booking Tool | FR-038 | Each equipment item in request = separate booking | Confirmed | BRD v13 | Метод возвращает item-ы заявки |
| TCO Booking Tool | FR-039 | Each booking has unique ID, start/end datetimes | Confirmed | BRD v13 | Метод возвращает идентификаторы и плановый период booking item-ов |
| TCO Booking Tool | FR-041 | Equipment attributes displayed in booking | Confirmed | BRD v13 | Метод возвращает атрибуты техники для отображения booking item |

---

## 3. Описание логики работы метода

1. Получить `BookingRequests` по `id`.
2. Проверить доступ: [`Requestor`](../../../requirements/Roles and Access Model.md) видит только заявки, где `BookingRequests.requestorId = currentUserId`.
3. Подтянуть связанные `Bookings`, а также справочные данные по технике.
4. Вернуть агрегированную модель заявки, достаточную для отображения обновленного состояния страницы `Новая заявка / Редактировать заявку` после добавления техники.

Сущности, участвующие в методе:
- читаются: [`BookingRequests`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#21-bookingrequests), [`Bookings`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#22-bookings), [`Equipments`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#8-equipments), [`EquipmentTypes`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#1-equipmenttypes), [`EquipmentBrands`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#4-equipmentbrands), [`EquipmentModels`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#5-equipmentmodels), [`EquipmentPhotos`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#9-equipmentphotos), [`Users`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#19-users), [`Fleets`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#2-fleets), [`EquipmentProperties`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#15-equipmentproperties), [`Properties`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#12-properties), [`PropertyEnumValues`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#13-propertyenumvalues), [`MeasurementUnits`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#11-measurementunits)
- изменения не выполняются

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| [`Requestor`](../../../requirements/Roles and Access Model.md) | Просмотр собственных заявок |

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

Возвращаемые данные обёрнуты в общий [`result wrapper`](../../common/Result%20Wrapper.md).

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
| 5 | Кто обновил текущий статус заявки | statusUpdatedBy | object | object | — | last BookingRequestStatuses + Users | Автор последнего status transition |
| 6 | Причина закрытия заявки | closureReason | string | null | `null` | BookingRequests + BookingRequestStatuses + ref_request_closure_reason | Для незакрытой заявки возвращается `null` |
| 7 | Номер Work Order | workOrderNumber | string | string | — | BookingRequests.workOrderNumber | Поле sidebar |
| 8 | Признак использования Default Work Order | isDefaultWorkOrder | bool | boolean | `false` | BookingRequests.isDefaultWorkOrder / business rule | Поле sidebar |
| 9 | Приоритет | priority | string | string | — | BookingRequests + ref_request_priority | Поле sidebar |
| 10 | Локация | location | string | string | — | BookingRequests.location | Поле sidebar |
| 11 | Описание работ | workDescription | string | string | — | BookingRequests.workDescription | Поле sidebar |
| 12 | Комментарии | comments | string | string | — | BookingRequests.comments | Поле sidebar |
| 13 | Общее количество броней в заявке | bookingsCount | int | integer | `0` | COUNT(Bookings) | Для центральной части страницы |
| 14 | Список броней | bookings | array<object> | object[] | `[]` | backend composition from BookingRequests + Bookings + Equipments + EquipmentTypes + EquipmentBrands + EquipmentModels + EquipmentPhotos + Fleets + Users + EquipmentProperties | Полный список броней заявки |

### Структура `value.statusUpdatedBy`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор пользователя | userId | uuid | UUID v4 | — | Users.id | Автор последнего status transition заявки |
| 2 | Полное имя | fullName | string | string | — | Users.fullName |  |
| 3 | Email | email | string | string | — | Users.email |  |

### Структура `value.bookings[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор брони | id | uuid | UUID v4 | — | Bookings.id |  |
| 2 | Идентификатор техники | equipmentId | uuid | UUID v4 | — | Equipments.id |  |
| 3 | Фото техники | photoUrl | string | string | — | EquipmentPhotos | Превью/основное фото |
| 4 | Тип техники | equipmentType | string | string | — | EquipmentTypes |  |
| 5 | Марка и модель | brandModel | string | string | — | EquipmentBrands + EquipmentModels |  |
| 6 | Номер ТШО | tcoId | string | string | — | Equipments.tcoId |  |
| 7 | ГРНЗ | stateNumber | string | string | — | Equipments.stateNumber |  |
| 8 | Описание техники | equipmentDescription | string | string | — | Equipments.description | Описание/комментарий по единице техники |
| 9 | Fleet | fleet | object | object | — | Fleets | Базовый контекст флота техники |
| 10 | Плановая дата и время начала | plannedStartDateTime | datetime | ISO 8601 | — | Bookings.plannedStartDateTime |  |
| 11 | Плановая дата и время окончания | plannedEndDateTime | datetime | ISO 8601 | — | Bookings.plannedEndDateTime |  |
| 12 | Конфликты с бронями в активном статусе (`Submitted`, `Confirmed`, `InProgress`) | activeBookingConflicts | object | object | — | backend overlap check against active bookings | Учитываются только брони со статусами `Submitted`, `Confirmed`, `InProgress` и с пересечением диапазона дат; `Draft` и `Closed` не учитываются |
| 13 | Код рабочего центра | workCenterCode | string | string | — | WorkCenters.code |  |
| 14 | Характеристики техники | properties | array<object> | object[] | `[]` | backend composition from EquipmentProperties + Properties + PropertyEnumValues + MeasurementUnits | Список `ключ - значение` |
| 15 | Объект обоснования | justification | object | object | — | backend composition from Bookings + business rules | Объединяет текст justification и его обязательность / заполненность |
| 16 | Текущий статус брони | status | string | string | — | Bookings + ref_booking_status |  |
| 17 | Кто обновил текущий статус брони | statusUpdatedBy | object | object | — | last BookingStatuses + Users | Автор последнего status transition |

### Структура `value.bookings[].fleet`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор флота | id | uuid | UUID v4 | — | Fleets.id |  |
| 2 | Наименование флота | name | string | string | — | Fleets.nameEn / localized projection |  |

### Структура `value.bookings[].justification`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Текст обоснования | value | string | string | — | Bookings.justification | Пользователь редактирует это поле в строке/карточке брони |
| 2 | Признак обязательности обоснования | isRequired | bool | boolean | — | backend business rule from Equipments + ref_ownership_type + ref_share_type | `true`, если `ownershipType = LongTermRented` или `shareType IN (Assigned, SharedWithConditions)` |
| 3 | Признак наличия обязательного обоснования | isSatisfied | bool | boolean | — | backend business rule | `false`, если justification обязателен, но еще не заполнен |

### Структура `value.bookings[].statusUpdatedBy`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор пользователя | userId | uuid | UUID v4 | — | Users.id | Автор последнего status transition брони |
| 2 | Полное имя | fullName | string | string | — | Users.fullName |  |
| 3 | Email | email | string | string | — | Users.email |  |

### Структура `value.bookings[].activeBookingConflicts`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Наличие конфликтов с бронями в активном статусе (`Submitted`, `Confirmed`, `InProgress`) | hasActiveBookingConflicts | bool | boolean | `false` | backend overlap check against active bookings | `true`, если есть хотя бы одна бронь по этой же технике со статусом `Submitted`, `Confirmed` или `InProgress` и с пересечением диапазона дат; `Draft` и `Closed` не учитываются |
| 2 | Количество конфликтов с бронями в активном статусе (`Submitted`, `Confirmed`, `InProgress`) | activeBookingConflictsCount | int | integer | `0` | backend aggregation | Количество броней по этой же технике со статусом `Submitted`, `Confirmed` или `InProgress` и с пересечением диапазона дат; `Draft` и `Closed` не учитываются |

### Структура `value.bookings[].properties[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Ключ характеристики | key | string | string | — | Properties.code / localized projection |  |
| 2 | Значение характеристики | value | string | string | — | EquipmentProperties + PropertyEnumValues + MeasurementUnits | Отображается как `ключ - значение` |

## 10. Пример ответа

```json
{
  "value": {
    "id": "c777f75f-029d-4d8f-8c69-e74a1d280001",
    "requestNumber": "REQ-2026-00015",
    "type": "Regular",
    "status": "Draft",
    "statusUpdatedBy": {
      "userId": "11111111-1111-1111-1111-111111111111",
      "fullName": "Telman Nurzhanov",
      "email": "telman.nurzhanov@example.com"
    },
    "workOrderNumber": "WO-10025",
    "isDefaultWorkOrder": false,
    "priority": "P2",
    "location": "Warehouse 3",
    "workDescription": "Excavator required for trench preparation near sector 4.",
    "comments": "Coordinate with site supervisor before mobilization.",
    "bookingsCount": 2,
    "bookings": [
      {
        "id": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
        "equipmentId": "c3b5af91-61f8-4bc0-bd88-d099d3e90001",
        "photoUrl": "https://cdn.example.com/equipment/c3b5af91-61f8-4bc0-bd88-d099d3e90001/preview.jpg",
        "equipmentType": "Excavator",
        "brandModel": "CAT 320D",
        "tcoId": "TCO-100245",
        "stateNumber": "KZ 123 ABC 02",
        "equipmentDescription": "Tracked excavator with trenching bucket and reinforced undercarriage.",
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
        "plannedEndDateTime": "2026-05-22T18:00:00Z",
        "properties": [
          {
            "key": "Engine volume",
            "value": "1.8 L"
          }
        ],
        "justification": {
          "value": "Required specialized bucket setup for this trench segment.",
          "isRequired": true,
          "isSatisfied": true
        },
        "status": "Draft",
        "statusUpdatedBy": {
          "userId": "11111111-1111-1111-1111-111111111111",
          "fullName": "Telman Nurzhanov",
          "email": "telman.nurzhanov@example.com"
        }
      },
      {
        "id": "8c4c8b6d-7bc0-41fb-9038-422cf55d2222",
        "equipmentId": "c3b5af91-61f8-4bc0-bd88-d099d3e90002",
        "photoUrl": "https://cdn.example.com/equipment/c3b5af91-61f8-4bc0-bd88-d099d3e90002/preview.jpg",
        "equipmentType": "Excavator",
        "brandModel": "Komatsu PC200",
        "tcoId": "TCO-100246",
        "stateNumber": "KZ 456 DEF 02",
        "equipmentDescription": "Hydraulic excavator configured for parallel earthworks on adjacent segment.",
        "activeBookingConflicts": {
          "hasActiveBookingConflicts": false,
          "activeBookingConflictsCount": 0
        },
        "workCenterCode": "HYDR",
        "fleet": {
          "id": "f0000001-0000-4000-8000-000000000002",
          "name": "Operations Fleet"
        },
        "plannedStartDateTime": "2026-05-20T08:00:00Z",
        "plannedEndDateTime": "2026-05-22T18:00:00Z",
        "properties": [
          {
            "key": "Engine volume",
            "value": "2.0 L"
          }
        ],
        "justification": {
          "value": "Required for parallel work on adjacent segment.",
          "isRequired": true,
          "isSatisfied": true
        },
        "status": "Draft",
        "statusUpdatedBy": {
          "userId": "11111111-1111-1111-1111-111111111111",
          "fullName": "Telman Nurzhanov",
          "email": "telman.nurzhanov@example.com"
        }
      }
    ]
  },
  "isSuccess": true,
  "errors": []
}
```

---

## Замечания

1. Метод должен возвращать полное текущее состояние заявки, а не только результат последнего действия над бронями.
2. Метод используется как основной источник данных для страницы `Новая заявка / Редактировать заявку` после добавления техники.
3. `bookingsCount` и `bookings[]` должны отражать все брони заявки целиком.
4. По каждому item метод должен возвращать объект `justification`, чтобы frontend мог сразу получить текст обоснования и признаки его обязательности / заполненности без разнесенных полей.
