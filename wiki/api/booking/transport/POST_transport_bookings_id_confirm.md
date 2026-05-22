# POST /transport/bookings/{id}/confirm

**Created:** 2026-05-14  
**Last updated:** 2026-05-22  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Подтвердить транспортировку |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Transportation UI` |
| Endpoint URL | `/api/booking/v1/transport/bookings/{id}/confirm` |
| Метод запроса | `POST` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Подтверждает availability транспорта.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-NEW-29 | Three-step Unwheeled flow | Confirmed | BRD v13 | Прямое покрытие |

---

## 3. Описание логики работы метода

1. Проверить бронь и роль пользователя.
2. Разрешить confirm только если ожидается решение транспортной роли.
3. Обновить `Bookings.status = TransportConfirmed`.
4. При наличии отдельной транспортирующей брони сохранить/обновить связь в `BookingTransportations`.
5. Создать запись в `BookingStatuses`.
6. Вернуть результат.

Сущности:
- читаются: `Bookings`, `BookingTransportations`
- изменяются: `Bookings`, `BookingTransportations`, `BookingStatuses`

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `TransportationResponsible` | Подтверждение транспортировки |

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
| `BOOKING_NOT_CONFIRMABLE` | Бронь не ожидает транспортного решения |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор брони | `id` | `uuid` | `+` | Должен существовать | — | Path param | |
| 2 | Комментарий | `comment` | `string` | `-` | — | — | Request body | |
| 3 | Идентификатор транспортирующей брони | `transportingBookingId` | `uuid` | `-` | Если передан, должен существовать | — | Request body | Сохраняется в `BookingTransportations.transportingBookingId` |

---

## 8. Пример запроса

```http
POST /api/booking/v1/transport/bookings/aaabbbcc-dddd-4444-8888-123456780001/confirm
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "comment": "Transport slot reserved.",
  "transportingBookingId": "bbbcbbcc-dddd-4444-8888-123456780099"
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
| 3 | Идентификатор транспортирующей брони | transportingBookingId | uuid | UUID v4 | — | BookingTransportations.transportingBookingId | Может быть `null` |

## 10. Пример ответа

```json
{
  "value": {
    "id": "aaabbbcc-dddd-4444-8888-123456780001",
    "status": "TransportConfirmed",
    "transportingBookingId": "bbbcbbcc-dddd-4444-8888-123456780099"
  },
  "isSuccess": true,
  "errors": []
}
```
