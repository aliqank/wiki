# POST /approvals/bookings/{id}/mobilization-start

**Created:** 2026-05-14  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Зафиксировать начало мобилизации |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Fleet Owner UI` |
| Endpoint URL | `/api/booking/v1/approvals/bookings/{id}/mobilization-start` |
| Метод запроса | `POST` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Переводит бронь в фактический старт использования.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-NEW-24 | FO presses Mobilization started to record actual start time | Confirmed | BRD v13 | Прямое покрытие |

---

## 3. Описание логики работы метода

1. Проверить бронь и права доступа.
2. Разрешить действие только для брони в статусе `Confirmed` или `TransportConfirmed`.
3. Установить `actualStartDt` = текущее время или значение из запроса.
4. Обновить `Bookings.status = InProgress`.
5. Создать запись в `BookingStatuses`.

Сущности:
- читаются: `Bookings`
- изменяются: `Bookings`, `BookingStatuses`

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `FleetOwner` | Фиксация начала мобилизации |

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
| `FORBIDDEN` | Нет доступа к брони |
| `NOT_FOUND` | Бронь не найдена |
| `BOOKING_NOT_STARTABLE` | Текущий статус не позволяет запуск |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор брони | `id` | `uuid` | `+` | Должен существовать | — | Path param | |
| 2 | Фактическая дата/время старта | `actualStartDt` | `datetime` | `-` | Если не передано, backend использует текущее время | now | Request body | |

---

## 8. Пример запроса

```http
POST /api/booking/v1/approvals/bookings/8c4c8b6d-7bc0-41fb-9038-422cf55d1111/mobilization-start
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "actualStartDt": "2026-05-20T08:05:00Z"
}
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий `result wrapper`.

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | value | object | object | — | backend aggregation |  |
| 1.1 | Идентификатор записи | id | uuid | UUID v4 | — | Bookings.id |  |
| 1.2 | Текущий статус | status | string | string | — | Bookings + BookingStatuses |  |
| 1.3 | Фактическая дата и время начала | actualStartDt | datetime | ISO 8601 | — | Bookings.actualStartDt |  |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | object[] | `[]` | backend |  |

### Структура `value`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | Bookings.id |  |
| 2 | Текущий статус | status | string | string | — | Bookings + BookingStatuses |  |
| 3 | Фактическая дата и время начала | actualStartDt | datetime | ISO 8601 | — | Bookings.actualStartDt |  |

## 10. Пример ответа

```json
{
  "value": {
    "id": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
    "status": "InProgress",
    "actualStartDt": "2026-05-20T08:05:00Z"
  },
  "isSuccess": true,
  "errors": []
}
```
