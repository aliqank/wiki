# GET /approvals/bookings/{id}

**Created:** 2026-05-14  
**Last updated:** 2026-05-14  
**Author:** OpenCode

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить детали брони для Fleet Owner |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Fleet Owner UI` |
| Endpoint URL | `/api/booking/v1/approvals/bookings/{id}` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Открывает карточку входящей брони для действий FO.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-041 | Equipment attributes displayed in booking | Confirmed | BRD v13 | Детали техники в approval окне |
| TCO Booking Tool | FR-NEW-64 | Display equipment loading summary by dates in FO approval window | Confirmed | BRD v13 | Карточка используется вместе с load summary |

---

## 3. Описание логики работы метода

1. Проверить существование брони.
2. Проверить, что бронь принадлежит одному из флотов FO.
3. Подтянуть `BookingRequests`, `Equipments`, `EquipmentTypes`, `EquipmentProperties`, `EquipmentPhotos`.
4. Вернуть агрегированную модель брони.

Сущности:
- читаются: `Bookings`, `BookingRequests`, `Equipments`, `EquipmentTypes`, `EquipmentProperties`, `EquipmentPhotos`

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `FleetOwner` | Просмотр броней по своим флотам |

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
| `FORBIDDEN` | Бронь не относится к флотам пользователя |
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
GET /api/booking/v1/approvals/bookings/8c4c8b6d-7bc0-41fb-9038-422cf55d1111
Authorization: Bearer <token>
Content-Type: application/json
```

---

## 9. Возвращаемые данные

В `value` возвращается объект `FoBookingApprovalDetails`.

Основные поля: `id`, `request`, `equipment`, `status`, `startDt`, `endDt`, `justification`, `requiresSupervisorApproval`, `history[]`.

---

## 10. Пример ответа

```json
{
  "value": {
    "id": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
    "status": "Submitted",
    "startDt": "2026-05-20T08:00:00Z",
    "endDt": "2026-05-22T18:00:00Z",
    "justification": null,
    "requiresSupervisorApproval": false,
    "history": []
  },
  "isSuccess": true,
  "errors": []
}
```
