# GET /booking-requests/{id}

**Created:** 2026-05-14  
**Last updated:** 2026-05-19  
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

Новый метод. Используется для открытия и обновления страницы `Новая заявка / Редактировать заявку`, включая sidebar request-level полей и полный список booking item-ов.

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
4. Вернуть агрегированную модель заявки, достаточную для отображения обновленного состояния страницы `Новая заявка / Редактировать заявку` после добавления техники.

Сущности, участвующие в методе:
- читаются: `BookingRequests`, `Bookings`, `Equipments`, `EquipmentTypes`, `EquipmentBrands`, `EquipmentModels`, `EquipmentPhotos`, `Users`, `Fleets`, `EquipmentProperties`, `Properties`, `PropertyEnumValues`, `MeasurementUnits`
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
| 5 | Номер Work Order | workOrderNumber | string | string | — | BookingRequests.workOrderNumber | Поле sidebar |
| 6 | Признак использования Default Work Order | isDefaultWorkOrder | bool | boolean | `false` | BookingRequests.isDefaultWorkOrder / business rule | Поле sidebar |
| 7 | Приоритет | priority | string | string | — | BookingRequests + ref_request_priority | Поле sidebar |
| 8 | Локация | location | string | string | — | BookingRequests.location | Поле sidebar |
| 9 | Описание работ | workDescription | string | string | — | BookingRequests.workDescription | Поле sidebar |
| 10 | Комментарии | comments | string | string | — | BookingRequests.comments | Поле sidebar |
| 11 | Общее количество броней в заявке | bookingsCount | int | integer | `0` | COUNT(Bookings) | Для центральной части страницы |
| 12 | Список броней | bookings | array<object> | object[] | `[]` | backend composition from BookingRequests + Bookings + Equipments + EquipmentTypes + EquipmentBrands + EquipmentModels + EquipmentPhotos + Fleets + Users + EquipmentProperties | Полный список броней заявки |

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
| 9 | Данные Fleet Owner | fleetOwner | object | object | — | Fleets + Users |  |
| 10 | Плановая дата и время начала | plannedStartDateTime | datetime | ISO 8601 | — | Bookings.plannedStartDateTime |  |
| 11 | Плановая дата и время окончания | plannedEndDateTime | datetime | ISO 8601 | — | Bookings.plannedEndDateTime |  |
| 12 | Конфликты с опубликованными бронями | publishedConflicts | object | object | — | backend overlap check against published bookings | Учитываются только опубликованные брони с пересечением диапазона дат; черновики не учитываются |
| 13 | Код рабочего центра | workCenterCode | string | string | — | WorkCenters.code |  |
| 14 | Характеристики техники | properties | array<object> | object[] | `[]` | backend composition from EquipmentProperties + Properties + PropertyEnumValues + MeasurementUnits | Список `ключ - значение` |
| 15 | Обоснование | justification | string | string | — | Bookings.justification | Пользователь редактирует это поле в строке/карточке брони |
| 16 | Признак необходимости согласования Supervisor | requiresSupervisorApproval | bool | boolean | — | backend business rule |  |
| 17 | Текущий статус брони | status | string | string | — | Bookings + ref_booking_status |  |

### Структура `value.bookings[].fleetOwner`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор пользователя | userId | uuid | UUID v4 | — | Users.id |  |
| 2 | Полное имя | fullName | string | string | — | Users.fullName |  |
| 3 | Email | email | string | string | — | Users.email |  |
| 4 | Наименование fleet | fleetName | string | string | — | Fleets.nameEn / localized projection |  |

### Структура `value.bookings[].publishedConflicts`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Наличие конфликтов с опубликованными бронями | hasPublishedConflicts | bool | boolean | `false` | backend overlap check against published bookings | `true`, если есть хотя бы одна опубликованная бронь по этой же технике с пересечением диапазона дат; черновики не учитываются |
| 2 | Количество конфликтов с опубликованными бронями | publishedConflictsCount | int | integer | `0` | backend aggregation | Количество опубликованных броней по этой же технике с пересечением диапазона дат; черновики не учитываются |

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
        "publishedConflicts": {
          "hasPublishedConflicts": true,
          "publishedConflictsCount": 2
        },
        "workCenterCode": "BHOE",
        "fleetOwner": {
          "userId": "4c9ad2d2-6df8-4f7b-87fe-36cefc100001",
          "fullName": "Nurlan Sarsenov",
          "email": "nurlan.sarsenov@tco.example",
          "fleetName": "Maintenance Fleet"
        },
        "plannedStartDateTime": "2026-05-20T08:00:00Z",
        "plannedEndDateTime": "2026-05-22T18:00:00Z",
        "properties": [
          {
            "key": "Engine volume",
            "value": "1.8 L"
          }
        ],
        "justification": "Required specialized bucket setup for this trench segment.",
        "requiresSupervisorApproval": false,
        "status": "Draft"
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
        "publishedConflicts": {
          "hasPublishedConflicts": false,
          "publishedConflictsCount": 0
        },
        "workCenterCode": "HYDR",
        "fleetOwner": {
          "userId": "4c9ad2d2-6df8-4f7b-87fe-36cefc100001",
          "fullName": "Nurlan Sarsenov",
          "email": "nurlan.sarsenov@tco.example",
          "fleetName": "Maintenance Fleet"
        },
        "plannedStartDateTime": "2026-05-20T08:00:00Z",
        "plannedEndDateTime": "2026-05-22T18:00:00Z",
        "properties": [
          {
            "key": "Engine volume",
            "value": "2.0 L"
          }
        ],
        "justification": "Required for parallel work on adjacent segment.",
        "requiresSupervisorApproval": false,
        "status": "Draft"
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
