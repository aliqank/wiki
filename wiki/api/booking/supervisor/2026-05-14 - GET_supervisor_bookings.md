# GET /supervisor/bookings

**Created:** 2026-05-14  
**Last updated:** 2026-05-14  
**Author:** OpenCode

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить очередь long-term rented броней для FleetOwners' Supervisor |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Supervisor UI` |
| Endpoint URL | `/api/booking/v1/supervisor/bookings` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Наполняет очередь Supervisor по броням в статусе `ConfirmedByFo`.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-NEW-73 | Supervisor reviews Long-term rented booking in ConfirmedByFo status | Confirmed | BRD v13 | Очередь для финального решения |
| TCO Booking Tool | FR-NEW-75 | System notifies Supervisor when booking reaches ConfirmedByFo | Confirmed | BRD v13 | Экран входящих задач |

---

## 3. Описание логики работы метода

1. Проверить роль текущего пользователя `FleetOwnersSupervisor`.
2. Выбрать `Bookings` со статусом `ConfirmedByFo` и `requiresSupervisorApproval = true`.
3. Подтянуть `BookingRequests`, `Equipments`, `EquipmentTypes`.
4. Вернуть пагинированный список.

Сущности:
- читаются: `Bookings`, `BookingRequests`, `Equipments`, `EquipmentTypes`

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `FleetOwnersSupervisor` | Доступ к очереди финального согласования long-term rented броней |

---

## 5. Настройки системы, используемые в методе

| Наименование | Код | Тип значения | Описание | Значение по умолчанию |
|---|---|---|---|---|
| Таймаут ответа Supervisor | `SUPERVISOR_RESPONSE_TIMEOUT_HOURS` | `int` | Может отображаться в UI как SLA | Конфигурируется Admin |

---

## 6. Ошибки, возвращаемые методом

| Код | Описание ошибки |
|---|---|
| `UNAUTHORIZED` | Пользователь не авторизован |
| `FORBIDDEN` | У пользователя нет роли `FleetOwnersSupervisor` |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Поисковая строка | `search` | `string` | `-` | Поиск по request number / equipment number / justification | — | Query param | |
| 2 | Номер страницы | `page` | `int` | `-` | >= 1 | `1` | Query param | |
| 3 | Размер страницы | `limit` | `int` | `-` | >= 1 | `20` | Query param | |

---

## 8. Пример запроса

```http
GET /api/booking/v1/supervisor/bookings?page=1&limit=20
Authorization: Bearer <token>
Content-Type: application/json
```

---

## 9. Возвращаемые данные

В `value` возвращается `PaginatedResult<SupervisorApprovalListItem>`.

---

## 10. Пример ответа

```json
{
  "value": {
    "items": [
      {
        "id": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
        "requestNumber": "REQ-2026-00015",
        "equipmentNumber": "TCO-100245",
        "status": "ConfirmedByFo",
        "justification": "No suitable TCO-owned unit available for required window."
      }
    ],
    "total": 1
  },
  "isSuccess": true,
  "errors": []
}
```
