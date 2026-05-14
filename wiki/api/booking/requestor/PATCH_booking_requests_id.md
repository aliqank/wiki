# PATCH /booking-requests/{id}

**Created:** 2026-05-14  
**Last updated:** 2026-05-14  
**Author:** OpenCode

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Обновить черновик заявки |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Requestor UI` |
| Endpoint URL | `/api/booking/v1/booking-requests/{id}` |
| Метод запроса | `PATCH` |
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
2. Разрешить редактирование только если `BookingRequests.status = Draft`.
3. Обновить только переданные поля.
4. Сохранить `updatedAt`, `updatedBy`.
5. Вернуть обновленную заявку.

Сущности, участвующие в методе:
- читаются: `BookingRequests`, `JdeWorkOrders`
- изменяются: `BookingRequests`

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Requestor` | Редактирование собственной draft-заявки |
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
| 2 | Идентификатор WO | `jdeWorkOrderRefId` | `uuid` | `-` | Если передан, WO должен существовать | — | Request body | |
| 3 | Номер WO | `workOrderJdeId` | `string` | `-` | Должен соответствовать WO | — | Request body | |
| 4 | Локация | `location` | `string` | `-` | — | — | Request body | |
| 5 | Описание работ | `workDescription` | `string` | `-` | Если передан, не должен быть пустым | — | Request body | |
| 6 | Комментарии | `comments` | `string` | `-` | — | — | Request body | |
| 7 | Приоритет | `priority` | `enum` | `-` | `P1 / P2 / P3 / P4` | — | Request body | |

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

В `value` возвращается обновленный объект `BookingRequest`.

---

## 10. Пример ответа

```json
{
  "value": {
    "id": "c777f75f-029d-4d8f-8c69-e74a1d280001",
    "requestNumber": "REQ-2026-00015",
    "status": "Draft",
    "priority": "P1",
    "comments": "Updated comment"
  },
  "isSuccess": true,
  "errors": []
}
```
