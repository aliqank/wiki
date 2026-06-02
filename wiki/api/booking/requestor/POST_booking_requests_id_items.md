# POST /booking-requests/{id}/items

**Created:** 2026-05-14  
**Last updated:** 2026-06-02  
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
| Связанные use cases | [`UC-REQ-02 - Создание новой заявки (Draft-first)`](../../../requirements/usecases/Requestor/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first)/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first).md), [`UC-REQ-02.4 - Добавление техники в draft-заявку`](../../../requirements/usecases/Requestor/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first)/UC-REQ-02.4%20-%20Добавление%20техники%20в%20draft-заявку.md) |
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
| TCO Booking Tool | FR-NEW-71 | Justification mandatory for Long-term rented item | Confirmed | BRD v13 | Проверка justification выполняется позже, перед submit |

---

## 3. Описание логики работы метода

1. Проверить существование draft-заявки и права доступа.
2. Принять список `items[]`; в нем должен быть минимум 1 элемент.
3. Для каждого элемента проверить существование техники и получить ее атрибуты `ownershipType`, `shareType`, `fleetId`, `mobilityType`.
4. Для каждого элемента проверить, что техника не относится к `OnDemand`.
5. Для каждого элемента проверить, что техника не является стационарной. Если `mobilityType = Stationary`, вернуть `422 VALIDATION_ERROR`.
6. Для каждого элемента проверить [hard availability restrictions](../../../glossary/Glossary.md#hard-availability-restriction) на выбранный период.
   Под [hard availability restrictions](../../../glossary/Glossary.md#hard-availability-restriction) в рамках текущего базового сценария понимается, что техника не заблокирована причинами, не связанными с competing bookings, например активными записями в `EquipmentStatuses`.
7. Отдельно рассчитать наличие [booking conflict context](../../../glossary/Glossary.md#booking-conflict-context) с активными записями `Bookings` со статусами `Submitted`, `Confirmed`, `InProgress`, если их период пересекается с периодом создаваемой брони.
   Такой [booking conflict context](../../../glossary/Glossary.md#booking-conflict-context) не блокирует создание item и используется только для информирования пользователя и последующего решения Fleet Owner.
8. Если техника `LongTermRented`, `Assigned` или `SharedWithConditions`, определить, что для item потребуется `justification` на этапе последующего редактирования или перед submit.
9. Если техника `Assigned`, проверить `EquipmentBookingAuthorizations`.
10. Создать отдельную запись `Bookings` со статусом `Draft` для каждого элемента из `items[]`; `justification` на этом этапе не передается и может оставаться пустым до отдельного сохранения через редактирование item.
11. Для каждого созданного item определить, требуется ли `justification`, и вычислить признак наличия обязательного justification.
12. Вернуть список booking item-ов, созданных в текущем batch-добавлении.

Сущности, участвующие в методе:
- читаются: [`BookingRequests`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#21-bookingrequests), [`Equipments`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#8-equipments), [`EquipmentBookingAuthorizations`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#17-equipmentfeedbacks--equipmentbookingauthorizations), [`Bookings`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#22-bookings)
- изменяются: [`Bookings`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#22-bookings)

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
| `NOT_FOUND` | Заявка или техника не найдены |
| `REQUEST_NOT_EDITABLE` | Заявка не в статусе `Draft` |
| `EQUIPMENT_NOT_AVAILABLE` | Техника недоступна по [hard availability restrictions](../../../glossary/Glossary.md#hard-availability-restriction) на выбранный период; competing bookings сами по себе не вызывают эту ошибку |
| `VALIDATION_ERROR` | Не пройдены бизнес-валидации, передан пустой `items[]` или сделана попытка добавить стационарную технику (`mobilityType = Stationary`) |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор заявки | `id` | `uuid` | `+` | Должен существовать | — | Path param | |
| 2 | Список добавляемых позиций | `items` | `array<object>` | `+` | Минимум 1 элемент | `[]` | Request body | Один элемент = одна создаваемая бронь |
| 2.1 | Идентификатор техники | `items[].equipmentId` | `uuid` | `+` | Должен существовать | — | Request body | |
| 2.2 | Плановая дата/время начала | `items[].plannedStartDateTime` | `datetime` | `+` | Меньше `items[].plannedEndDateTime` | — | Request body | |
| 2.3 | Плановая дата/время окончания | `items[].plannedEndDateTime` | `datetime` | `+` | Больше `items[].plannedStartDateTime` | — | Request body | |
| 2.4 | Work Center | `items[].workCenterId` | `uuid` | `-` | Если передан, должен существовать | — | Request body | |

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
      "plannedStartDateTime": "2026-05-20T08:00:00Z",
      "plannedEndDateTime": "2026-05-22T18:00:00Z",
      "workCenterId": "12a8b1ce-3aaf-4f55-8ac8-f8cf5d86c222"
    },
    {
      "equipmentId": "c3b5af91-61f8-4bc0-bd88-d099d3e90002",
      "plannedStartDateTime": "2026-05-20T08:00:00Z",
      "plannedEndDateTime": "2026-05-22T18:00:00Z",
      "workCenterId": "12a8b1ce-3aaf-4f55-8ac8-f8cf5d86c222"
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
| 3 | Идентификатор техники | equipmentId | uuid | UUID v4 | — | backend composition from BookingRequests + Equipments + EquipmentBookingAuthorizations + Bookings |  |
| 4 | Текущий статус | status | string | string | — | Bookings + ref_booking_status |  |
| 5 | Плановая дата и время начала | plannedStartDateTime | datetime | ISO 8601 | — | Bookings.plannedStartDateTime |  |
| 6 | Плановая дата и время окончания | plannedEndDateTime | datetime | ISO 8601 | — | Bookings.plannedEndDateTime |  |
| 7 | Обоснование | justification | string | string | — | Bookings.justification | На этапе создания item может быть пустым и заполняется позже через редактирование item |
| 8 | Признак, что для item обязателен justification | requiresJustification | bool | boolean | — | backend business rule from Equipments + ref_ownership_type + ref_share_type | `true`, если `ownershipType = LongTermRented` или `shareType IN (Assigned, SharedWithConditions)` |
| 9 | Признак наличия обязательного justification | hasRequiredJustification | bool | boolean | — | backend business rule | `false`, если для item обязателен `justification`, но он еще не заполнен |

## 10. Пример ответа

```json
{
  "value": [
    {
      "id": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
      "requestId": "c777f75f-029d-4d8f-8c69-e74a1d280001",
      "equipmentId": "c3b5af91-61f8-4bc0-bd88-d099d3e90001",
      "status": "Draft",
      "plannedStartDateTime": "2026-05-20T08:00:00Z",
      "plannedEndDateTime": "2026-05-22T18:00:00Z",
      "justification": null,
      "requiresJustification": true,
      "hasRequiredJustification": false,
    },
    {
      "id": "8c4c8b6d-7bc0-41fb-9038-422cf55d2222",
      "requestId": "c777f75f-029d-4d8f-8c69-e74a1d280001",
      "equipmentId": "c3b5af91-61f8-4bc0-bd88-d099d3e90002",
      "status": "Draft",
      "plannedStartDateTime": "2026-05-20T08:00:00Z",
      "plannedEndDateTime": "2026-05-22T18:00:00Z",
      "justification": null,
      "requiresJustification": true,
      "hasRequiredJustification": false,
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
4. Под недоступностью в базовом сценарии понимаются [hard availability restrictions](../../../glossary/Glossary.md#hard-availability-restriction) по состоянию техники и другим обязательным бизнес-правилам, не связанным с competing bookings.
5. Пересечения с другими активными бронями должны рассчитываться отдельно как booking conflicts и не блокируют создание item.
6. Стационарная техника не может быть добавлена в заявку: если `mobilityType = Stationary`, метод должен вернуть `422 VALIDATION_ERROR`.
7. Поле `justification` не передается в `POST /booking-requests/{id}/items`; оно заполняется позже через редактирование конкретного item.
8. Система должна обозначить item как требующий `justification` уже в ответе `POST /booking-requests/{id}/items` через поля `requiresJustification` и `hasRequiredJustification`.
