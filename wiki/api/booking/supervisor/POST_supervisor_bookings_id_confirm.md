# POST /supervisor/bookings/{id}/confirm

**Created:** 2026-05-14  
**Last updated:** 2026-05-14  
**Author:** OpenCode

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Финально подтвердить long-term rented бронь |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Supervisor UI` |
| Endpoint URL | `/api/booking/v1/supervisor/bookings/{id}/confirm` |
| Метод запроса | `POST` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Завершает supervisor approval положительным решением.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-NEW-73 | Supervisor can Confirm -> booking moves to Confirmed | Confirmed | BRD v13 | Прямое покрытие |
| TCO Booking Tool | FR-NEW-78 | Long-term rented booking -> Confirmed upon Supervisor approval | Confirmed | BRD v13 | Обновление статуса |

---

## 3. Описание логики работы метода

1. Проверить бронь и роль Supervisor.
2. Разрешить действие только для статуса `ConfirmedByFo`.
3. Обновить `status = Confirmed`.
4. Записать `supervisorApprovedBy`, `supervisorApprovedAt`.
5. Создать запись в `BookingStatuses`.
6. Вернуть результат.

Сущности:
- читаются: `Bookings`
- изменяются: `Bookings`, `BookingStatuses`

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `FleetOwnersSupervisor` | Финальное подтверждение long-term rented брони |

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
| `FORBIDDEN` | Нет роли `FleetOwnersSupervisor` |
| `NOT_FOUND` | Бронь не найдена |
| `BOOKING_NOT_CONFIRMABLE` | Бронь не в статусе `ConfirmedByFo` |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор брони | `id` | `uuid` | `+` | Должен существовать | — | Path param | |
| 2 | Комментарий Supervisor | `comment` | `string` | `-` | — | — | Request body | Необязателен для confirm |

---

## 8. Пример запроса

```http
POST /api/booking/v1/supervisor/bookings/8c4c8b6d-7bc0-41fb-9038-422cf55d1111/confirm
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "comment": "Confirmed after reviewing business need."
}
```

---

## 9. Возвращаемые данные

В `value` возвращается `SupervisorDecisionResult`: `id`, `status`, `supervisorApprovedAt`.

---

## 10. Пример ответа

```json
{
  "value": {
    "id": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
    "status": "Confirmed",
    "supervisorApprovedAt": "2026-05-18T12:00:00Z"
  },
  "isSuccess": true,
  "errors": []
}
```
