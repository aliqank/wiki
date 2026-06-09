# POST /booking-requests

**Created:** 2026-05-14  
**Last updated:** 2026-06-02  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Создать пустой черновик заявки на бронирование |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Requestor UI` |
| Endpoint URL | `/api/booking/v1/booking-requests` |
| Метод запроса | `POST` |
| Связанные use cases | [`UC-REQ-02 - Создание новой заявки (Draft-first)`](../../../requirements/usecases/Requestor/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first)/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first).md), [`UC-REQ-02.1 - Создание пустого draft заявки`](../../../requirements/usecases/Requestor/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first)/UC-REQ-02.1%20-%20Создание%20пустого%20draft%20заявки.md) |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Создает пустую шапку заявки со статусом `Draft` в момент открытия формы создания заявки.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-023 | Requestor can create a Request | Confirmed | BRD v13 | Прямое покрытие |
| TCO Booking Tool | FR-024 | Request has unique ID and metadata | Confirmed | BRD v13 | При создании draft формируются `id`, `requestNumber`, `createdAt` и базовые метаданные |
| TCO Booking Tool | FR-027 | Requestor can edit/cancel draft before submission | Confirmed | BRD v13 | Метод создает черновик |
| TCO Booking Tool | FR-030 | System supports draft saving | Confirmed | BRD v13 | Заявка создается как draft |
| TCO Booking Tool | FR-072 | Request -> Draft when saved as draft | Confirmed | BRD v13 | Метод создает заявку сразу в статусе `Draft` |

---

## 3. Описание логики работы метода

1. Принять тело запроса для создания draft-заявки.
2. Не требовать обязательного заполнения `priority`, `workDescription`, `location`, `workOrderNumber` на этапе создания draft.
3. Если передан `isDefaultWorkOrder = true`, сохранить `workOrderNumber = null`.
4. Создать запись в `BookingRequests` со статусом `Draft`, где:
   - `requestorId = currentUserId` как business-requestor заявки;
   - `createdBy = currentUserId` как audit creator.
5. Создать запись в `BookingRequestStatuses` со статусом `Draft`.
6. Вернуть созданную заявку.

Обязательные business-поля валидируются на этапе [`POST /booking-requests/{id}/submit`](POST_booking_requests_id_submit.md), а не на этапе создания draft.

Сущности, участвующие в методе:
- читаются: —
- изменяются: [`BookingRequests`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#21-bookingrequests), [`BookingRequestStatuses`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#26-bookingrequeststatuses)
- транзакционность: требуется, запись в шапку и историю должна быть атомарной

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Requestor` | Создание обычной заявки |

---

## 5. Настройки системы, используемые в методе

| Наименование | Код | Тип значения | Описание | Значение по умолчанию |
|---|---|---|---|---|
| Подсказки приоритета | `REQUEST_PRIORITY_HINTS` | `json` | Используются UI-формой; backend только хранит `priority` | Конфигурируется Admin |

---

## 6. Ошибки, возвращаемые методом

| Код | Описание ошибки |
|---|---|
| `UNAUTHORIZED` | Пользователь не авторизован |
| `FORBIDDEN` | Нет необходимой роли |
| `VALIDATION_ERROR` | Переданы невалидные данные draft-заявки |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Номер Work Order | `workOrderNumber` | `string` | `-` | Может быть пустым на этапе draft или `null` при `isDefaultWorkOrder = true` | — | Request body | Для draft может быть `null` |
| 2 | Признак использования Default Work Order | `isDefaultWorkOrder` | `bool` | `-` | `true / false` | `false` | Request body | При `true` backend сохраняет `workOrderNumber = null` |
| 3 | Локация | `location` | `string` | `-` | Валидируется на этапе submit | — | Request body | Для draft может быть пустой |
| 4 | Описание работ | `workDescription` | `string` | `-` | Валидируется на этапе submit | — | Request body | Для draft может быть пустым |
| 5 | Комментарии | `comments` | `string` | `-` | — | — | Request body | |
| 6 | Приоритет | `priority` | `string` | `-` | `P1 / P2 / P3 / P4` | — | Request body | Значения загружаются через [`GET /reference/request-priorities`](../reference/GET_reference_request_priorities.md); для draft может быть не заполнен |

---

## 8. Пример запроса

```http
POST /api/booking/v1/booking-requests
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "workOrderNumber": null,
  "isDefaultWorkOrder": false,
  "location": null,
  "workDescription": null,
  "comments": null,
  "priority": null
}
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
| 3 | Идентификатор business-requestor-а | requestorId | uuid | UUID v4 | — | BookingRequests.requestorId | В ручном draft-first flow совпадает с `currentUserId` |
| 4 | Тип сущности / заявки | type | string | string | — | BookingRequests + ref_request_type |  |
| 5 | Текущий статус | status | string | string | — | BookingRequests + BookingRequestStatuses |  |
| 6 | Причина закрытия заявки | closureReason | null | — | `null` | BookingRequests + BookingRequestStatuses + ref_request_closure_reason | Для draft всегда `null` |
| 7 | Номер Work Order | workOrderNumber | null | — | `null` | BookingRequests.workOrderNumber | Для draft может отсутствовать |
| 8 | Признак использования Default Work Order | isDefaultWorkOrder | bool | boolean | `false` | backend business rule / request payload |  |
| 9 | Локация | location | null | — | `null` | BookingRequests.location | Для draft может отсутствовать |
| 10 | Описание работ | workDescription | null | — | `null` | BookingRequests.workDescription | Для draft может отсутствовать |
| 11 | Комментарии | comments | null | — | `null` | BookingRequests.comments | Для draft может отсутствовать |
| 12 | Приоритет | priority | null | — | `null` | BookingRequests + ref_request_priority | Для draft может отсутствовать |

## 10. Пример ответа

```json
{
  "value": {
    "id": "c777f75f-029d-4d8f-8c69-e74a1d280001",
    "requestNumber": "REQ-2026-00015",
    "requestorId": "11111111-1111-1111-1111-111111111111",
    "type": "Regular",
    "status": "Draft",
    "closureReason": null,
    "workOrderNumber": null,
    "isDefaultWorkOrder": false,
    "location": null,
    "workDescription": null,
    "comments": null,
    "priority": null
  },
  "isSuccess": true,
  "errors": []
}
```

---

## Замечания

1. Метод создаёт именно пустой `Draft`, а не валидированную к отправке заявку.
2. Поля `priority`, `workDescription`, `location`, `workOrderNumber` могут оставаться пустыми до момента submit.
3. Если `isDefaultWorkOrder = true`, backend должен сохранять `workOrderNumber = null`.
4. В ручном requestor-flow поле `requestorId` должно фиксировать business-requestor-а, а `createdBy` использоваться только как audit creator.
5. Полная бизнес-валидация должна выполняться в [`POST /booking-requests/{id}/submit`](POST_booking_requests_id_submit.md).
