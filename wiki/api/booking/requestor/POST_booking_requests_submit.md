# POST /booking-requests/submit

**Created:** 2026-05-14  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Создать заявку с booking item-ами и сразу отправить ее на согласование |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Requestor UI` |
| Endpoint URL | `/api/booking/v1/booking-requests/submit` |
| Метод запроса | `POST` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Поддерживает сценарий one-shot submit без промежуточного сохранения draft через UI.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-023 | Requestor can create a Request | Confirmed | BRD v13 | Метод создает заявку |
| TCO Booking Tool | FR-031 | Requestor can add/remove equipment items to a request; availability updated | Confirmed | BRD v13 | Метод создает booking item-ы внутри запроса |
| TCO Booking Tool | FR-038 | Each equipment item in request = separate booking | Confirmed | BRD v13 | Каждый item сохраняется как отдельная бронь |
| TCO Booking Tool | FR-040 | System validates availability before booking | Confirmed | BRD v13 | Валидация всех item-ов до commit |
| TCO Booking Tool | FR-043 | Approver assigned automatically based on fleet ownership | Confirmed | BRD v13 | Submit сразу запускает approval flow |
| TCO Booking Tool | FR-062 | Booking -> Submitted once Request submitted | Confirmed | BRD v13 | Все booking item-ы создаются сразу в `Submitted` |
| TCO Booking Tool | FR-NEW-71 | Justification mandatory for Long-term rented item | Confirmed | BRD v13 | Обязательная проверка перед submit |

---

## 3. Описание логики работы метода

1. Принять тело запроса и провалидировать шапку заявки: `jdeWorkOrderRefId`, `workOrderNumber`, `location`, `workDescription`, `comments`, `priority`.
2. Проверить, что в `items[]` передан хотя бы один booking item.
3. Для каждого item проверить:
   - существование техники;
   - допустимость `ownershipType` для booking workflow;
   - валидность `plannedStartDateTime` / `plannedEndDateTime`;
   - доступность техники на период;
   - обязательность `justification` для `LongTermRented`, `Assigned`, `SharedWithConditions`;
   - наличие авторизации в `EquipmentBookingAuthorizations` для `Assigned`;
   - согласованность `jdeWorkOrderStepRefId` и `workCenterId`, если они переданы.
4. В рамках одной транзакции создать запись в `BookingRequests` со статусом `Submitted`.
5. Создать запись в `BookingRequestStatuses` со статусом `Submitted`.
6. Для каждого элемента `items[]` создать запись в `Bookings` сразу со статусом `Submitted`.
7. Для каждого созданного booking item создать запись в `BookingStatuses` со статусом `Submitted`.
8. Если хотя бы одна проверка или вставка не проходит, откатить всю транзакцию.
9. Вернуть созданную заявку с созданными booking item-ами.

Сущности, участвующие в методе:
- читаются: `JdeWorkOrders`, `JdeWorkOrderSteps`, `BookingRequests`, `Equipments`, `EquipmentBookingAuthorizations`, `Bookings`
- изменяются: `BookingRequests`, `BookingRequestStatuses`, `Bookings`, `BookingStatuses`
- транзакционность: обязательна; метод должен быть полностью atomic

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Requestor` | Создание и немедленная отправка обычной заявки |
| `ServiceWorkProcessor` | Создание и немедленная отправка SWR |

---

## 5. Настройки системы, используемые в методе

| Наименование | Код | Тип значения | Описание | Значение по умолчанию |
|---|---|---|---|---|
| Подсказки приоритета | `REQUEST_PRIORITY_HINTS` | `json` | Используются UI-формой; backend только хранит `priority` | Конфигурируется Admin |
| Горизонт бронирования | `BOOKING_HORIZON_DAYS` | `int` | Проверка дат всех item-ов | Конфигурируется Admin |
| Максимальная длительность брони | `MAX_BOOKING_DURATION_DAYS` | `int` | Проверка периода всех item-ов | Конфигурируется Admin |

---

## 6. Ошибки, возвращаемые методом

| Код | Описание ошибки |
|---|---|
| `UNAUTHORIZED` | Пользователь не авторизован |
| `FORBIDDEN` | Нет необходимой роли |
| `NOT_FOUND` | WO, шаг WO, техника или связанная сущность не найдены |
| `EQUIPMENT_NOT_AVAILABLE` | Хотя бы одна единица техники недоступна на запрошенный период |
| `VALIDATION_ERROR` | Не пройдены проверки шапки заявки или booking item-ов |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор WO | `jdeWorkOrderRefId` | `uuid` | `-` | Если передан, WO должен существовать | — | Request body | |
| 2 | Номер WO | `workOrderNumber` | `string` | `-` | Должен соответствовать выбранному WO, если передан `jdeWorkOrderRefId` | — | Request body | Денормализованное поле |
| 3 | Локация | `location` | `string` | `-` | Обязательно для сценариев без WO | — | Request body | |
| 4 | Описание работ | `workDescription` | `string` | `+` | Непустая строка, около 50+ символов по бизнес-правилу | — | Request body | |
| 5 | Комментарии | `comments` | `string` | `-` | — | — | Request body | |
| 6 | Приоритет | `priority` | `enum` | `+` | `P1 / P2 / P3 / P4` | — | Request body | |
| 7 | Список booking item-ов | `items` | `array<object>` | `+` | Минимум 1 элемент | `[]` | Request body | |
| 7.1 | Идентификатор техники | `items[].equipmentId` | `uuid` | `+` | Должен существовать | — | Request body | |
| 7.2 | Плановая дата/время начала | `items[].plannedStartDateTime` | `datetime` | `+` | Меньше `items[].plannedEndDateTime` | — | Request body | |
| 7.3 | Плановая дата/время окончания | `items[].plannedEndDateTime` | `datetime` | `+` | Больше `items[].plannedStartDateTime` | — | Request body | |
| 7.4 | Work Center | `items[].workCenterId` | `uuid` | `-` | Если передан, должен существовать | — | Request body | |
| 7.5 | Шаг WO | `items[].jdeWorkOrderStepRefId` | `uuid` | `-` | Если передан, должен существовать и относиться к WO | — | Request body | |
| 7.6 | Обоснование | `items[].justification` | `string` | `-` | Обязательно для `LongTermRented`, `Assigned`, `SharedWithConditions` | — | Request body | |

---

## 8. Пример запроса

```http
POST /api/booking/v1/booking-requests/submit
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "jdeWorkOrderRefId": "9fda7b6b-aa83-4ec7-a7f7-899a4b430001",
  "workOrderNumber": "WO-10025",
  "workDescription": "Excavator required for trench preparation near sector 4.",
  "comments": "Coordinate with site supervisor before mobilization.",
  "priority": "P2",
  "items": [
    {
      "equipmentId": "c3b5af91-61f8-4bc0-bd88-d099d3e90001",
      "plannedStartDateTime": "2026-05-20T08:00:00Z",
      "plannedEndDateTime": "2026-05-22T18:00:00Z",
      "workCenterId": "12a8b1ce-3aaf-4f55-8ac8-f8cf5d86c222",
      "jdeWorkOrderStepRefId": "d8975a3d-a1a0-4f78-878c-e854ff560001",
      "justification": "Required specialized bucket setup for this trench segment."
    }
  ]
}
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
| 5 | Идентификатор связанного Work Order | jdeWorkOrderRefId | uuid | UUID v4 | — | backend composition from JdeWorkOrders + JdeWorkOrderSteps + BookingRequests + Equipments + EquipmentBookingAuthorizations + Bookings + BookingRequestStatuses + BookingStatuses |  |
| 6 | Номер Work Order из JDE | workOrderNumber | string | string | — | BookingRequests.workOrderNumber |  |
| 7 | Локация | location | null | — | `null` | BookingRequests.location |  |
| 8 | Описание работ | workDescription | string | string | — | BookingRequests.workDescription |  |
| 9 | Комментарии | comments | string | string | — | BookingRequests.comments |  |
| 10 | Приоритет | priority | string | string | — | BookingRequests + ref_request_priority |  |
| 11 | Список броней | bookings | array<object> | object[] | — | backend composition from JdeWorkOrders + JdeWorkOrderSteps + BookingRequests + Equipments + EquipmentBookingAuthorizations + Bookings + BookingRequestStatuses + BookingStatuses | Коллекция объектов |

### Структура `value.bookings[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | BookingRequests.id |  |
| 2 | Идентификатор техники | equipmentId | uuid | UUID v4 | — | backend composition from JdeWorkOrders + JdeWorkOrderSteps + BookingRequests + Equipments + EquipmentBookingAuthorizations + Bookings + BookingRequestStatuses + BookingStatuses |  |
| 3 | Текущий статус | status | string | string | — | BookingRequests + BookingRequestStatuses |  |
| 4 | Плановая дата и время начала | plannedStartDateTime | datetime | ISO 8601 | — | backend composition from JdeWorkOrders + JdeWorkOrderSteps + BookingRequests + Equipments + EquipmentBookingAuthorizations + Bookings + BookingRequestStatuses + BookingStatuses |  |
| 5 | Плановая дата и время окончания | plannedEndDateTime | datetime | ISO 8601 | — | backend composition from JdeWorkOrders + JdeWorkOrderSteps + BookingRequests + Equipments + EquipmentBookingAuthorizations + Bookings + BookingRequestStatuses + BookingStatuses |  |
| 6 | Обоснование | justification | string | string | — | backend composition from JdeWorkOrders + JdeWorkOrderSteps + BookingRequests + Equipments + EquipmentBookingAuthorizations + Bookings + BookingRequestStatuses + BookingStatuses |  |
| 7 | Признак необходимости согласования Supervisor | requiresSupervisorApproval | bool | boolean | — | backend composition from JdeWorkOrders + JdeWorkOrderSteps + BookingRequests + Equipments + EquipmentBookingAuthorizations + Bookings + BookingRequestStatuses + BookingStatuses |  |

## 10. Пример ответа

```json
{
  "value": {
    "id": "c777f75f-029d-4d8f-8c69-e74a1d280001",
    "requestNumber": "REQ-2026-00015",
    "type": "Regular",
    "status": "Submitted",
    "jdeWorkOrderRefId": "9fda7b6b-aa83-4ec7-a7f7-899a4b430001",
    "workOrderNumber": "WO-10025",
    "location": null,
    "workDescription": "Excavator required for trench preparation near sector 4.",
    "comments": "Coordinate with site supervisor before mobilization.",
    "priority": "P2",
    "bookings": [
      {
        "id": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
        "equipmentId": "c3b5af91-61f8-4bc0-bd88-d099d3e90001",
        "status": "Submitted",
        "plannedStartDateTime": "2026-05-20T08:00:00Z",
        "plannedEndDateTime": "2026-05-22T18:00:00Z",
        "justification": "Required specialized bucket setup for this trench segment.",
        "requiresSupervisorApproval": false
      }
    ]
  },
  "isSuccess": true,
  "errors": []
}
```

---

## Замечания

1. Метод не заменяет draft flow, а дополняет его для UI-сценария «создать и сразу отправить».
2. Рекомендуется использовать этот endpoint, когда пользователю не нужен промежуточно сохраняемый черновик.
3. В отличие от связки `POST /booking-requests` + `POST /booking-requests/{id}/items` + `POST /booking-requests/{id}/submit`, этот метод должен быть полностью atomic.
