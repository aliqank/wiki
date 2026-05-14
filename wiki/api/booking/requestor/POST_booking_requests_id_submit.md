# POST /booking-requests/{id}/submit

**Created:** 2026-05-14  
**Last updated:** 2026-05-14  
**Author:** OpenCode

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Отправить заявку на согласование |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Requestor UI` |
| Endpoint URL | `/api/booking/v1/booking-requests/{id}/submit` |
| Метод запроса | `POST` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Переводит заявку и ее item-ы из draft в submitted.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-023 | Requestor can create a Request | Confirmed | BRD v13 | Метод завершает создание заявки |
| TCO Booking Tool | FR-062 | Booking -> Submitted once Request submitted | Confirmed | BRD v13 | Все booking item-ы переводятся в `Submitted` |
| TCO Booking Tool | FR-043 | Approver assigned automatically based on fleet ownership | Confirmed | BRD v13 | При submit начинается approval flow |
| TCO Booking Tool | FR-NEW-71 | Justification mandatory for Long-term rented item | Confirmed | BRD v13 | Проверка перед отправкой |

---

## 3. Описание логики работы метода

1. Проверить, что заявка существует и находится в статусе `Draft`.
2. Проверить, что в заявке есть хотя бы один booking item.
3. Для каждого item выполнить валидации периода и обязательного justification.
4. Для `Assigned` техники проверить наличие активной записи в `EquipmentBookingAuthorizations`.
5. Обновить `BookingRequests.status = Submitted` и создать запись в `BookingRequestStatuses`.
6. Для каждого item обновить `Bookings.status = Submitted` и создать запись в `BookingStatuses`.
7. Вернуть обновленную заявку.

Сущности, участвующие в методе:
- читаются: `BookingRequests`, `Bookings`, `Equipments`, `EquipmentBookingAuthorizations`
- изменяются: `BookingRequests`, `Bookings`, `BookingRequestStatuses`, `BookingStatuses`
- транзакционность: обязательна

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Requestor` | Отправка собственной заявки |
| `ServiceWorkProcessor` | Отправка SWR |

---

## 5. Настройки системы, используемые в методе

| Наименование | Код | Тип значения | Описание | Значение по умолчанию |
|---|---|---|---|---|
| Горизонт бронирования | `BOOKING_HORIZON_DAYS` | `int` | Проверка дат | Конфигурируется Admin |
| Максимальная длительность брони | `MAX_BOOKING_DURATION_DAYS` | `int` | Проверка длительности | Конфигурируется Admin |

---

## 6. Ошибки, возвращаемые методом

| Код | Описание ошибки |
|---|---|
| `UNAUTHORIZED` | Пользователь не авторизован |
| `FORBIDDEN` | Нет доступа к заявке |
| `NOT_FOUND` | Заявка не найдена |
| `REQUEST_NOT_SUBMITTABLE` | Заявка не в статусе `Draft` или пуста |
| `VALIDATION_ERROR` | Не пройдены бизнес-валидации item-ов |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор заявки | `id` | `uuid` | `+` | Должен существовать | — | Path param | |

У метода нет request body.

---

## 8. Пример запроса

```http
POST /api/booking/v1/booking-requests/c777f75f-029d-4d8f-8c69-e74a1d280001/submit
Authorization: Bearer <token>
Content-Type: application/json
```

---

## 9. Возвращаемые данные

В `value` возвращается объект `BookingRequestSubmitResult` с обновленным статусом заявки и списком item-ов.

---

## 10. Пример ответа

```json
{
  "value": {
    "id": "c777f75f-029d-4d8f-8c69-e74a1d280001",
    "requestNumber": "REQ-2026-00015",
    "status": "Submitted",
    "bookings": [
      {
        "id": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
        "status": "Submitted"
      }
    ]
  },
  "isSuccess": true,
  "errors": []
}
```
