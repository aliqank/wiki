# GET /approvals/bookings

**Created:** 2026-05-14  
**Last updated:** 2026-05-14  
**Author:** OpenCode

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить очередь броней на согласование для Fleet Owner |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Fleet Owner UI` |
| Endpoint URL | `/api/booking/v1/approvals/bookings` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Наполняет список входящих броней для FO.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-092 | Approver (FO) can view all pending and completed approvals | Confirmed | BRD v13 | Список pending approvals |
| TCO Booking Tool | FR-043 | Approver assigned automatically based on fleet ownership | Confirmed | BRD v13 | Отбор по флотам FO |

---

## 3. Описание логики работы метода

1. Определить список флотов, доступных текущему FO через AAD-группы.
2. Выбрать `Bookings` по этим флотам со статусами `Submitted`, `ConfirmedByFo`, `Extended`, `TransportConfirmed` по фильтру экрана.
3. Подтянуть `BookingRequests`, `Equipments`, `EquipmentTypes`.
4. Вернуть пагинированный список.

Сущности:
- читаются: `Bookings`, `BookingRequests`, `Equipments`, `EquipmentTypes`, `Fleets`
- изменений нет

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `FleetOwner` | Просмотр броней по управляемым флотам |

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
| `FORBIDDEN` | У пользователя нет роли `FleetOwner` |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Статус брони | `status` | `enum` | `-` | Статусы approval queue | — | Query param | |
| 2 | Поисковая строка | `search` | `string` | `-` | Поиск по request number / equipment number | — | Query param | |
| 3 | Номер страницы | `page` | `int` | `-` | >= 1 | `1` | Query param | |
| 4 | Размер страницы | `limit` | `int` | `-` | >= 1 | `20` | Query param | |

---

## 8. Пример запроса

```http
GET /api/booking/v1/approvals/bookings?status=Submitted&page=1&limit=20
Authorization: Bearer <token>
Content-Type: application/json
```

---

## 9. Возвращаемые данные

В `value` возвращается `PaginatedResult<FoApprovalListItem>`.

`FoApprovalListItem`: `id`, `requestId`, `requestNumber`, `equipmentId`, `equipmentNumber`, `equipmentTypeName`, `status`, `startDt`, `endDt`, `requiresSupervisorApproval`, `justification`.

---

## 10. Пример ответа

```json
{
  "value": {
    "items": [
      {
        "id": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
        "requestId": "c777f75f-029d-4d8f-8c69-e74a1d280001",
        "requestNumber": "REQ-2026-00015",
        "equipmentId": "c3b5af91-61f8-4bc0-bd88-d099d3e90001",
        "equipmentNumber": "TCO-100245",
        "equipmentTypeName": {
          "En": "Excavator",
          "Ru": "Экскаватор",
          "Kz": "Экскаватор"
        },
        "status": "Submitted",
        "startDt": "2026-05-20T08:00:00Z",
        "endDt": "2026-05-22T18:00:00Z",
        "requiresSupervisorApproval": false,
        "justification": null
      }
    ],
    "total": 1
  },
  "isSuccess": true,
  "errors": []
}
```
