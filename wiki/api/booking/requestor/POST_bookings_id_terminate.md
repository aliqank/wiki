# POST /bookings/{id}/terminate

**Created:** 2026-05-14  
**Last updated:** 2026-05-14  
**Author:** OpenCode

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Досрочно завершить подтвержденную бронь |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Requestor UI` |
| Endpoint URL | `/api/booking/v1/bookings/{id}/terminate` |
| Метод запроса | `POST` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Завершает подтвержденную бронь до планового окончания.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-060 | Requestor/SWP can terminate confirmed booking | Confirmed | BRD v13 | Прямое покрытие |
| TCO Booking Tool | FR-069 | Booking -> Terminated once terminated | Confirmed | BRD v13 | Обновление статуса |

---

## 3. Описание логики работы метода

1. Проверить существование брони и права доступа.
2. Разрешить terminate только для внутренней подтвержденной брони.
3. Потребовать непустой `reason`.
4. Обновить `Bookings.status = Terminated`, записать `terminateReason`.
5. Создать запись в `BookingStatuses`.
6. Пересчитать статус заявки.

Сущности, участвующие в методе:
- читаются: `Bookings`, `BookingRequests`, `Equipments`
- изменяются: `Bookings`, `BookingStatuses`, `BookingRequests`, `BookingRequestStatuses`
- транзакционность: обязательна

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Requestor` | Terminate собственной брони |
| `ServiceWorkProcessor` | Terminate брони в SWR |

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
| `BOOKING_NOT_TERMINABLE` | Бронь нельзя terminate |
| `VALIDATION_ERROR` | Не передана причина terminate |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор брони | `id` | `uuid` | `+` | Должен существовать | — | Path param | |
| 2 | Причина досрочного завершения | `reason` | `string` | `+` | Непустая строка | — | Request body | |

---

## 8. Пример запроса

```http
POST /api/booking/v1/bookings/8c4c8b6d-7bc0-41fb-9038-422cf55d1111/terminate
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "reason": "Work completed earlier than planned."
}
```

---

## 9. Возвращаемые данные

В `value` возвращается объект `BookingTerminateResult`: `id`, `status`, `terminateReason`.

---

## 10. Пример ответа

```json
{
  "value": {
    "id": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
    "status": "Terminated",
    "terminateReason": "Work completed earlier than planned."
  },
  "isSuccess": true,
  "errors": []
}
```
