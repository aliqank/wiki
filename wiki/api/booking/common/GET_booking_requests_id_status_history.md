# GET /booking-requests/{id}/status-history

**Created:** 2026-05-14  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить историю статусов заявки |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Common` |
| Endpoint URL | `/api/booking/v1/booking-requests/{id}/status-history` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется для audit trail на уровне заявки.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-NEW-37 | Audit trail and history on demand | Confirmed | BRD v13 | Прямое покрытие |

---

## 3. Описание логики работы метода

1. Проверить существование заявки и права доступа.
2. Выбрать `BookingRequestStatuses` по `requestId`.
3. Отсортировать по `createdAt ASC`.
4. Вернуть историю.

Сущности:
- читаются: [`BookingRequests`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#21-bookingrequests), [`BookingRequestStatuses`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#26-bookingrequeststatuses)

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Requestor` | История собственной заявки |
| `ServiceWorkProcessor` | История SWR |
| `FleetOwner` | История заявок по своим броням |
| `Admin` | Полный доступ |

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
| `FORBIDDEN` | Нет доступа к истории заявки |
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
GET /api/booking/v1/booking-requests/c777f75f-029d-4d8f-8c69-e74a1d280001/status-history
Authorization: Bearer <token>
Content-Type: application/json
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий `result wrapper`.

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | value | array<object> | object[] | — | backend aggregation |  |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | object[] | `[]` | backend |  |

### Структура `value[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | BookingRequestStatuses.id |  |
| 2 | Текущий статус | status | string | string | — | BookingRequestStatuses + ref_booking_request_status |  |
| 3 | Причина закрытия заявки | closureReason | string | null | `null` | BookingRequestStatuses + ref_request_closure_reason | Для non-terminal статусов `Draft`, `Submitted`, `InProgress` возвращается `null` |
| 4 | Дата и время создания записи статуса | createdAt | datetime | ISO 8601 | — | BookingRequestStatuses.createdAt |  |
| 5 | Комментарий | comment | null | — | `null` | BookingRequestStatuses.comment |  |

## 10. Пример ответа

```json
{
  "value": [
    {
      "id": "66666666-7777-8888-9999-000000000001",
      "status": "Draft",
      "closureReason": null,
      "createdAt": "2026-05-14T09:15:00Z",
      "comment": null
    },
    {
      "id": "66666666-7777-8888-9999-000000000002",
      "status": "Submitted",
      "closureReason": null,
      "createdAt": "2026-05-14T10:00:00Z",
      "comment": null
    }
  ],
  "isSuccess": true,
  "errors": []
}
```
