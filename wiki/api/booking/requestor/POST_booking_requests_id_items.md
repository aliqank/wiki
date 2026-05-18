# POST /booking-requests/{id}/items

**Created:** 2026-05-14  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Добавить один или несколько booking item-ов в черновик заявки |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Requestor UI` |
| Endpoint URL | `/api/booking/v1/booking-requests/{id}/items` |
| Метод запроса | `POST` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Создает одну или несколько записей `Bookings` внутри существующей draft-заявки за один вызов.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-031 | Requestor can add/remove equipment items to a request | Confirmed | BRD v13 | Прямое покрытие |
| TCO Booking Tool | FR-038 | Each equipment item in request = separate booking | Confirmed | BRD v13 | Создается отдельная бронь |
| TCO Booking Tool | FR-040 | System validates availability before booking | Confirmed | BRD v13 | Проверка доступности обязательна |
| TCO Booking Tool | FR-NEW-71 | Justification mandatory for Long-term rented item | Confirmed | BRD v13 | Проверка justification |

---

## 3. Описание логики работы метода

1. Проверить существование draft-заявки и права доступа.
2. Принять список `items[]`; в нем должен быть минимум 1 элемент.
3. Для каждого элемента проверить существование техники и получить ее атрибуты `ownershipType`, `shareType`, `fleetId`.
4. Для каждого элемента проверить, что техника не относится к `OnDemand`.
5. Для каждого элемента проверить доступность на выбранный период.
6. Если техника `LongTermRented`, `Assigned` или `SharedWithConditions`, потребовать `justification` на уровне конкретного элемента.
7. Если техника `Assigned`, проверить `EquipmentBookingAuthorizations`.
8. Если передан `jdeWorkOrderStepRefId`, проверить существование шага WO и согласованность `workCenterId`.
9. Создать отдельную запись `Bookings` со статусом `Draft` для каждого элемента из `items[]`.
10. Вернуть список booking item-ов, созданных в текущем batch-добавлении.

Сущности, участвующие в методе:
- читаются: `BookingRequests`, `Equipments`, `EquipmentBookingAuthorizations`, `Bookings`, `JdeWorkOrderSteps`
- изменяются: `Bookings`

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Requestor` | Добавление техники в свою заявку |
| `ServiceWorkProcessor` | Добавление техники в SWR |

---

## 5. Настройки системы, используемые в методе

| Наименование | Код | Тип значения | Описание | Значение по умолчанию |
|---|---|---|---|---|
| Горизонт бронирования | `BOOKING_HORIZON_DAYS` | `int` | Проверка даты | Конфигурируется Admin |
| Максимальная длительность брони | `MAX_BOOKING_DURATION_DAYS` | `int` | Проверка периода | Конфигурируется Admin |

---

## 6. Ошибки, возвращаемые методом

| Код | Описание ошибки |
|---|---|
| `UNAUTHORIZED` | Пользователь не авторизован |
| `FORBIDDEN` | Нет доступа к заявке или технике |
| `NOT_FOUND` | Заявка, техника или шаг WO не найдены |
| `REQUEST_NOT_EDITABLE` | Заявка не в статусе `Draft` |
| `EQUIPMENT_NOT_AVAILABLE` | Техника недоступна на выбранный период |
| `VALIDATION_ERROR` | Не пройдены бизнес-валидации или передан пустой `items[]` |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор заявки | `id` | `uuid` | `+` | Должен существовать | — | Path param | |
| 2 | Список добавляемых позиций | `items` | `array<object>` | `+` | Минимум 1 элемент | `[]` | Request body | Один элемент = одна создаваемая бронь |
| 2.1 | Идентификатор техники | `items[].equipmentId` | `uuid` | `+` | Должен существовать | — | Request body | |
| 2.2 | Дата/время начала | `items[].startDt` | `datetime` | `+` | Меньше `items[].endDt` | — | Request body | |
| 2.3 | Дата/время окончания | `items[].endDt` | `datetime` | `+` | Больше `items[].startDt` | — | Request body | |
| 2.4 | Work Center | `items[].workCenterId` | `uuid` | `-` | Если передан, должен существовать | — | Request body | |
| 2.5 | Шаг WO | `items[].jdeWorkOrderStepRefId` | `uuid` | `-` | Если передан, должен существовать и относиться к WO заявки | — | Request body | |
| 2.6 | Обоснование | `items[].justification` | `string` | `-` | Обязательно для `LongTermRented`, `Assigned`, `SharedWithConditions` | — | Request body | |

---

## 8. Пример запроса

```http
POST /api/booking/v1/booking-requests/c777f75f-029d-4d8f-8c69-e74a1d280001/items
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "items": [
    {
      "equipmentId": "c3b5af91-61f8-4bc0-bd88-d099d3e90001",
      "startDt": "2026-05-20T08:00:00Z",
      "endDt": "2026-05-22T18:00:00Z",
      "workCenterId": "12a8b1ce-3aaf-4f55-8ac8-f8cf5d86c222",
      "jdeWorkOrderStepRefId": "d8975a3d-a1a0-4f78-878c-e854ff560001",
      "justification": "Required specialized bucket setup for this trench segment."
    },
    {
      "equipmentId": "c3b5af91-61f8-4bc0-bd88-d099d3e90002",
      "startDt": "2026-05-20T08:00:00Z",
      "endDt": "2026-05-22T18:00:00Z",
      "workCenterId": "12a8b1ce-3aaf-4f55-8ac8-f8cf5d86c222",
      "jdeWorkOrderStepRefId": "d8975a3d-a1a0-4f78-878c-e854ff560002",
      "justification": "Required for parallel work on adjacent segment."
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
| 1 | Результат выполнения метода | value | array<object> | object[] | — | backend aggregation | Список booking item-ов, созданных в текущем запросе |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | object[] | `[]` | backend |  |

### Структура `value[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | Bookings.id |  |
| 2 | Идентификатор заявки | requestId | uuid | UUID v4 | — | BookingRequests.id |  |
| 3 | Идентификатор техники | equipmentId | uuid | UUID v4 | — | backend composition from BookingRequests + Equipments + EquipmentBookingAuthorizations + Bookings + JdeWorkOrderSteps |  |
| 4 | Текущий статус | status | string | string | — | Bookings + ref_booking_status |  |
| 5 | Дата и время начала | startDt | datetime | ISO 8601 | — | Bookings.startDt |  |
| 6 | Дата и время окончания | endDt | datetime | ISO 8601 | — | Bookings.endDt |  |
| 7 | Обоснование | justification | string | string | — | Bookings.justification |  |
| 8 | Признак необходимости согласования Supervisor | requiresSupervisorApproval | bool | boolean | — | backend composition from Equipments + EquipmentBookingAuthorizations + Bookings + JdeWorkOrderSteps |  |

## 10. Пример ответа

```json
{
  "value": [
    {
      "id": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
      "requestId": "c777f75f-029d-4d8f-8c69-e74a1d280001",
      "equipmentId": "c3b5af91-61f8-4bc0-bd88-d099d3e90001",
      "status": "Draft",
      "startDt": "2026-05-20T08:00:00Z",
      "endDt": "2026-05-22T18:00:00Z",
      "justification": "Required specialized bucket setup for this trench segment.",
      "requiresSupervisorApproval": false
    },
    {
      "id": "8c4c8b6d-7bc0-41fb-9038-422cf55d2222",
      "requestId": "c777f75f-029d-4d8f-8c69-e74a1d280001",
      "equipmentId": "c3b5af91-61f8-4bc0-bd88-d099d3e90002",
      "status": "Draft",
      "startDt": "2026-05-20T08:00:00Z",
      "endDt": "2026-05-22T18:00:00Z",
      "justification": "Required for parallel work on adjacent segment.",
      "requiresSupervisorApproval": false
    }
  ],
  "isSuccess": true,
  "errors": []
}
```

## Замечания

1. Контракт метода описан как batch create: один вызов может добавить несколько единиц техники.
2. Один элемент в `items[]` соответствует одной создаваемой записи в `Bookings`.
3. В `value` возвращаются только брони, созданные в текущем вызове метода, а не полный список всех броней заявки.
