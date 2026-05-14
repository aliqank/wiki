# POST /approvals/bookings/{id}/decline

**Created:** 2026-05-14  
**Last updated:** 2026-05-14  
**Author:** OpenCode

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Отклонить бронь как Fleet Owner |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Fleet Owner UI` |
| Endpoint URL | `/api/booking/v1/approvals/bookings/{id}/decline` |
| Метод запроса | `POST` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Выполняет decline входящей брони.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-049 | FO can decline booking | Confirmed | BRD v13 | Прямое покрытие |
| TCO Booking Tool | FR-050 | FO provides reason/comment when declining | Confirmed | BRD v13 | Причина обязательна |
| TCO Booking Tool | FR-064 | Booking -> Declined once declined by FO | Confirmed | BRD v13 | Обновление статуса |

---

## 3. Описание логики работы метода

1. Проверить бронь и права доступа.
2. Разрешить действие только для статусов `Submitted` или `Extended`.
3. Потребовать непустой `reason`.
4. Обновить `Bookings.status = Declined`, записать `declineReason`.
5. Создать запись в `BookingStatuses`.
6. Вернуть результат.

Сущности:
- читаются: `Bookings`
- изменяются: `Bookings`, `BookingStatuses`

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `FleetOwner` | Отклонение броней своих флотов |

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
| `BOOKING_NOT_DECLINABLE` | Текущий статус не позволяет decline |
| `VALIDATION_ERROR` | Не передана причина decline |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор брони | `id` | `uuid` | `+` | Должен существовать | — | Path param | |
| 2 | Причина отклонения | `reason` | `string` | `+` | Непустая строка | — | Request body | |

---

## 8. Пример запроса

```http
POST /api/booking/v1/approvals/bookings/8c4c8b6d-7bc0-41fb-9038-422cf55d1111/decline
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "reason": "Equipment reserved for higher-priority internal work."
}
```

---

## 9. Возвращаемые данные

В `value` возвращается `FoBookingDecisionResult`: `id`, `status`, `declineReason`.

---

## 10. Пример ответа

```json
{
  "value": {
    "id": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
    "status": "Declined",
    "declineReason": "Equipment reserved for higher-priority internal work."
  },
  "isSuccess": true,
  "errors": []
}
```
