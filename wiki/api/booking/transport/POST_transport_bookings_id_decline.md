# POST /transport/bookings/{id}/decline

**Created:** 2026-05-14  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Отклонить транспортировку |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Transportation UI` |
| Endpoint URL | `/api/booking/v1/transport/bookings/{id}/decline` |
| Метод запроса | `POST` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Фиксирует отказ по транспортировке и причину.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-NEW-29 | Three-step Unwheeled flow | Confirmed | BRD v13 | Транспорт может отклонить запрос |
| TCO Booking Tool | FR-NEW-30 | If transport unavailable, nearest date proposed | Pending | BRD v13 | Метод поддерживает `proposedStartDateTime` как опциональное поле |

---

## 3. Описание логики работы метода

1. Проверить бронь и роль пользователя.
2. Разрешить decline только если ожидается решение транспортной роли.
3. Потребовать `reason`.
4. Не менять основную бронь на `Declined`; вернуть ее на шаг FO с записью в `BookingStatuses` и комментарием транспортной роли.
5. Если передана `proposedStartDateTime`, сохранить ее в комментарии/metadata до финального отдельного проектного решения.
6. Вернуть результат.

Сущности:
- читаются: `Bookings`
- изменяются: `BookingStatuses`

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `TransportationResponsible` | Отклонение транспортировки |

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
| `FORBIDDEN` | Нет роли `TransportationResponsible` |
| `NOT_FOUND` | Бронь не найдена |
| `BOOKING_NOT_DECLINABLE` | Бронь не ожидает транспортного решения |
| `VALIDATION_ERROR` | Не передана причина |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор брони | `id` | `uuid` | `+` | Должен существовать | — | Path param | |
| 2 | Причина отказа | `reason` | `string` | `+` | Непустая строка | — | Request body | |
| 3 | Предлагаемая ближайшая дата | `proposedStartDateTime` | `datetime` | `-` | Должна быть в будущем | — | Request body | Pending rule |

---

## 8. Пример запроса

```http
POST /api/booking/v1/transport/bookings/aaabbbcc-dddd-4444-8888-123456780001/decline
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "reason": "No transport crew available for requested date.",
  "proposedStartDateTime": "2026-05-26T08:00:00Z"
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
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | Bookings.id |  |
| 2 | Текущий статус | status | string | string | — | Bookings + BookingStatuses |  |
| 3 | Комментарий | comment | string | string | — | backend composition from Bookings + BookingStatuses |  |

## 10. Пример ответа

```json
{
  "value": {
    "id": "aaabbbcc-dddd-4444-8888-123456780001",
    "status": "Confirmed",
    "comment": "Transport declined: No transport crew available for requested date. ProposedStartDateTime=2026-05-26T08:00:00Z"
  },
  "isSuccess": true,
  "errors": []
}
```
