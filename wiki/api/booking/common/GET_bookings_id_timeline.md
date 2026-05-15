# GET /bookings/{id}/timeline

**Created:** 2026-05-14  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить агрегированный timeline по брони |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Common` |
| Endpoint URL | `/api/booking/v1/bookings/{id}/timeline` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Удобная агрегированная проекция audit trail для UI.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-NEW-37 | Audit trail and history on demand | Confirmed | BRD v13 | Альтернативное представление истории |

---

## 3. Описание логики работы метода

1. Проверить доступ к брони.
2. Собрать историю из `BookingStatuses`.
3. Добавить служебные события из основной записи `Bookings`: `actualStartDt`, `actualEndDt`, `supervisorApprovedAt` при наличии.
4. Вернуть отсортированный timeline.

Сущности:
- читаются: `Bookings`, `BookingStatuses`

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Requestor` | Timeline собственной брони |
| `ServiceWorkProcessor` | Timeline SWR-брони |
| `FleetOwner` | Timeline броней своих флотов |
| `FleetOwnersSupervisor` | Timeline long-term rented броней |
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
| `FORBIDDEN` | Нет доступа к timeline |
| `NOT_FOUND` | Бронь не найдена |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор брони | `id` | `uuid` | `+` | Должен существовать | — | Path param | |

---

## 8. Пример запроса

```http
GET /api/booking/v1/bookings/8c4c8b6d-7bc0-41fb-9038-422cf55d1111/timeline
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
| 3 | Ошибки | errors | array<object> | array | `[]` | backend |  |

### Структура `value[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Тип события | eventType | string | string | — | response DTO |  |
| 2 | Текущий статус | status | string | string | — | response DTO |  |
| 3 | Дата и время события | occurredAt | datetime | ISO 8601 | — | response DTO |  |

## 10. Пример ответа

```json
{
  "value": [
    {
      "eventType": "StatusChanged",
      "status": "Submitted",
      "occurredAt": "2026-05-14T10:00:00Z"
    },
    {
      "eventType": "ActualStartRecorded",
      "status": "InProgress",
      "occurredAt": "2026-05-20T08:05:00Z"
    }
  ],
  "isSuccess": true,
  "errors": []
}
```
