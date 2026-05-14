# GET /booking-requests/my

**Created:** 2026-05-14  
**Last updated:** 2026-05-14  
**Author:** OpenCode

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить список собственных заявок текущего пользователя |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Requestor UI` |
| Endpoint URL | `/api/booking/v1/booking-requests/my` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется для страницы «Мои заявки».

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-091 | Requestor/SWP can search and filter own requests | Confirmed | BRD v13 | Прямое покрытие |

---

## 3. Описание логики работы метода

1. Выбрать `BookingRequests` по `createdBy = currentUserId`.
2. Применить фильтры по `status`, `type`, `priority`, `search`.
3. Отсортировать по `createdAt DESC`.
4. Вернуть пагинированный список.

Сущности, участвующие в методе:
- читаются: `BookingRequests`
- изменения не выполняются

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Requestor` | Просмотр своих заявок |
| `ServiceWorkProcessor` | Просмотр своих SWR |

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
| `FORBIDDEN` | Нет необходимой роли |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Фильтр по статусу | `status` | `enum` | `-` | `Draft / Submitted / InProgress / Completed / Cancelled` | — | Query param | |
| 2 | Фильтр по типу заявки | `type` | `enum` | `-` | `Regular / ServiceWork` | — | Query param | |
| 3 | Фильтр по приоритету | `priority` | `enum` | `-` | `P1 / P2 / P3 / P4` | — | Query param | |
| 4 | Поисковая строка | `search` | `string` | `-` | Поиск по `requestNumber`, `workOrderJdeId`, `workDescription` | — | Query param | |
| 5 | Номер страницы | `page` | `int` | `-` | >= 1 | `1` | Query param | |
| 6 | Размер страницы | `limit` | `int` | `-` | >= 1 | `20` | Query param | |

---

## 8. Пример запроса

```http
GET /api/booking/v1/booking-requests/my?status=Submitted&page=1&limit=20
Authorization: Bearer <token>
Content-Type: application/json
```

---

## 9. Возвращаемые данные

В `value` возвращается `PaginatedResult<MyBookingRequestListItem>`.

`MyBookingRequestListItem`: `id`, `requestNumber`, `type`, `status`, `priority`, `workOrderJdeId`, `createdAt`.

---

## 10. Пример ответа

```json
{
  "value": {
    "items": [
      {
        "id": "c777f75f-029d-4d8f-8c69-e74a1d280001",
        "requestNumber": "REQ-2026-00015",
        "type": "Regular",
        "status": "Submitted",
        "priority": "P2",
        "workOrderJdeId": "WO-10025",
        "createdAt": "2026-05-14T09:15:00Z"
      }
    ],
    "total": 1
  },
  "isSuccess": true,
  "errors": []
}
```
