# PATCH /booking-requests/{id}

**Created:** 2026-05-14  
**Last updated:** 2026-06-02  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Обновить черновик заявки |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Requestor UI` |
| Endpoint URL | `/api/booking/v1/booking-requests/{id}` |
| Метод запроса | `PATCH` |
| Связанные use cases | [`UC-REQ-02 - Создание новой заявки (Draft-first)`](../../../requirements/usecases/Requestor/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first)/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first).md), [`UC-REQ-02.2 - Заполнение и редактирование шапки заявки`](../../../requirements/usecases/Requestor/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first)/UC-REQ-02.2%20-%20Заполнение%20и%20редактирование%20шапки%20заявки.md) |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется для редактирования шапки draft-заявки.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-027 | Requestor can edit/cancel draft before submission | Confirmed | BRD v13 | Прямое покрытие |
| TCO Booking Tool | FR-030 | System supports draft saving | Confirmed | BRD v13 | Изменение draft-заявки |

---

## 3. Описание логики работы метода

1. Проверить существование заявки и права доступа.
   Для роли `Requestor` доступ определяется по `BookingRequests.requestorId = currentUserId`, а не по audit-полю `createdBy`.
2. Разрешить редактирование только если `BookingRequests.status = Draft`.
3. Обновить только переданные поля.
4. Сохранить `updatedAt`, `updatedBy`.
5. Вернуть обновленную заявку.

Сущности, участвующие в методе:
- читаются: [`BookingRequests`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#21-bookingrequests)
- изменяются: [`BookingRequests`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#21-bookingrequests)

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Requestor` | Редактирование draft-заявки, где пользователь является `requestorId` |
| `ServiceWorkProcessor` | Редактирование собственной рабочей заявки |

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
| `REQUEST_NOT_EDITABLE` | Заявка уже не в статусе `Draft` |
| `VALIDATION_ERROR` | Переданы невалидные данные |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор заявки | `id` | `uuid` | `+` | Должен существовать | — | Path param | |
| 2 | Номер Work Order | `workOrderNumber` | `string` | `-` | Может быть пустым до submit или `null` при `isDefaultWorkOrder = true` | — | Request body | |
| 3 | Признак использования Default Work Order | `isDefaultWorkOrder` | `bool` | `-` | `true / false` | `false` | Request body | |
| 4 | Локация | `location` | `string` | `-` | — | — | Request body | |
| 5 | Описание работ | `workDescription` | `string` | `-` | Если передан, не должен быть пустым | — | Request body | |
| 6 | Комментарии | `comments` | `string` | `-` | — | — | Request body | |
| 7 | Приоритет | `priority` | `string` | `-` | `P1 / P2 / P3 / P4` | — | Request body | Значения загружаются через [`GET /reference/request-priorities`](../reference/GET_reference_request_priorities.md) |

---

## 8. Пример запроса

```http
PATCH /api/booking/v1/booking-requests/c777f75f-029d-4d8f-8c69-e74a1d280001
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "comments": "Updated comment",
  "priority": "P1"
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
| 3 | Текущий статус | status | string | string | — | BookingRequests + BookingRequestStatuses |  |
| 4 | Причина закрытия заявки | closureReason | null | — | `null` | BookingRequests + BookingRequestStatuses + ref_request_closure_reason | Для edit draft всегда `null` |
| 5 | Приоритет | priority | string | string | — | BookingRequests + ref_request_priority |  |
| 6 | Комментарии | comments | string | string | — | BookingRequests.comments |  |

## 10. Пример ответа

```json
{
  "value": {
    "id": "c777f75f-029d-4d8f-8c69-e74a1d280001",
    "requestNumber": "REQ-2026-00015",
    "status": "Draft",
    "closureReason": null,
    "priority": "P1",
    "comments": "Updated comment"
  },
  "isSuccess": true,
  "errors": []
}
```
