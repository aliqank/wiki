# POST /booking-requests

**Created:** 2026-05-14  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Создать черновик заявки на бронирование |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Requestor UI` |
| Endpoint URL | `/api/booking/v1/booking-requests` |
| Метод запроса | `POST` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Создает шапку заявки со статусом `Draft`.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-023 | Requestor can create a Request | Confirmed | BRD v13 | Прямое покрытие |
| TCO Booking Tool | FR-027 | Requestor can edit/cancel draft before submission | Confirmed | BRD v13 | Метод создает черновик |
| TCO Booking Tool | FR-030 | System supports draft saving | Confirmed | BRD v13 | Заявка создается как draft |
| TCO Booking Tool | FR-NEW-11 | Requestor selects priority P1-P4 | Confirmed | BRD v13 | Приоритет хранится на заявке |
| TCO Booking Tool | FR-NEW-13 | Work Description mandatory | Confirmed | BRD v13 | Обязательное поле |

---

## 3. Описание логики работы метода

1. Принять тело запроса и провалидировать обязательные поля.
2. Если передан `jdeWorkOrderRefId`, проверить существование записи в `JdeWorkOrders`.
3. Если не передан WO и выбран сценарий SCM Logistics, проверить наличие `location`.
4. Создать запись в `BookingRequests` со статусом `Draft`.
5. Создать запись в `BookingRequestStatuses` со статусом `Draft`.
6. Вернуть созданную заявку.

Сущности, участвующие в методе:
- читаются: `JdeWorkOrders`
- изменяются: `BookingRequests`, `BookingRequestStatuses`
- транзакционность: требуется, запись в шапку и историю должна быть атомарной

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Requestor` | Создание обычной заявки |
| `ServiceWorkProcessor` | Создание и ведение SWR |

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
| `VALIDATION_ERROR` | Не заполнены обязательные поля заявки |
| `NOT_FOUND` | WO не найден |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор WO | `jdeWorkOrderRefId` | `uuid` | `-` | Если передан, WO должен существовать | — | Request body | |
| 2 | Номер WO | `workOrderJdeId` | `string` | `-` | Должен соответствовать выбранному WO, если указан `jdeWorkOrderRefId` | — | Request body | Денормализованное поле |
| 3 | Локация | `location` | `string` | `-` | Обязательно для сценариев без WO | — | Request body | |
| 4 | Описание работ | `workDescription` | `string` | `+` | Непустая строка, около 50+ символов по бизнес-правилу | — | Request body | |
| 5 | Комментарии | `comments` | `string` | `-` | — | — | Request body | |
| 6 | Приоритет | `priority` | `enum` | `+` | `P1 / P2 / P3 / P4` | — | Request body | |

---

## 8. Пример запроса

```http
POST /api/booking/v1/booking-requests
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "jdeWorkOrderRefId": "9fda7b6b-aa83-4ec7-a7f7-899a4b430001",
  "workOrderJdeId": "WO-10025",
  "workDescription": "Excavator required for trench preparation near sector 4.",
  "comments": "Coordinate with site supervisor before mobilization.",
  "priority": "P2"
}
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий `result wrapper`.

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | value | object | object | — | backend aggregation |  |
| 1.1 | Идентификатор записи | id | uuid | UUID v4 | — | BookingRequests.id |  |
| 1.2 | Номер заявки | requestNumber | string | string | — | BookingRequests.requestNumber |  |
| 1.3 | Тип сущности / заявки | type | string | string | — | BookingRequests + ref_request_type |  |
| 1.4 | Текущий статус | status | string | string | — | BookingRequests + BookingRequestStatuses |  |
| 1.5 | Идентификатор связанного Work Order | jdeWorkOrderRefId | uuid | UUID v4 | — | backend composition from JdeWorkOrders + BookingRequests + BookingRequestStatuses |  |
| 1.6 | Номер Work Order из JDE | workOrderJdeId | string | string | — | BookingRequests.workOrderJdeId |  |
| 1.7 | Локация | location | null | — | `null` | BookingRequests.location |  |
| 1.8 | Описание работ | workDescription | string | string | — | BookingRequests.workDescription |  |
| 1.9 | Комментарии | comments | string | string | — | BookingRequests.comments |  |
| 1.10 | Приоритет | priority | string | string | — | BookingRequests + ref_request_priority |  |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | object[] | `[]` | backend |  |

### Структура `value`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | BookingRequests.id |  |
| 2 | Номер заявки | requestNumber | string | string | — | BookingRequests.requestNumber |  |
| 3 | Тип сущности / заявки | type | string | string | — | BookingRequests + ref_request_type |  |
| 4 | Текущий статус | status | string | string | — | BookingRequests + BookingRequestStatuses |  |
| 5 | Идентификатор связанного Work Order | jdeWorkOrderRefId | uuid | UUID v4 | — | backend composition from JdeWorkOrders + BookingRequests + BookingRequestStatuses |  |
| 6 | Номер Work Order из JDE | workOrderJdeId | string | string | — | BookingRequests.workOrderJdeId |  |
| 7 | Локация | location | null | — | `null` | BookingRequests.location |  |
| 8 | Описание работ | workDescription | string | string | — | BookingRequests.workDescription |  |
| 9 | Комментарии | comments | string | string | — | BookingRequests.comments |  |
| 10 | Приоритет | priority | string | string | — | BookingRequests + ref_request_priority |  |

## 10. Пример ответа

```json
{
  "value": {
    "id": "c777f75f-029d-4d8f-8c69-e74a1d280001",
    "requestNumber": "REQ-2026-00015",
    "type": "Regular",
    "status": "Draft",
    "jdeWorkOrderRefId": "9fda7b6b-aa83-4ec7-a7f7-899a4b430001",
    "workOrderJdeId": "WO-10025",
    "location": null,
    "workDescription": "Excavator required for trench preparation near sector 4.",
    "comments": "Coordinate with site supervisor before mobilization.",
    "priority": "P2"
  },
  "isSuccess": true,
  "errors": []
}
```
