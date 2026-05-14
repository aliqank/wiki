# GET /booking-requests/history

**Created:** 2026-05-14  
**Last updated:** 2026-05-14  
**Author:** OpenCode

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить историю завершенных заявок текущего пользователя |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Requestor UI` |
| Endpoint URL | `/api/booking/v1/booking-requests/history` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется для страницы «История заявок» / `Completed Requests`.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-NEW-45 | Dedicated Completed Requests page | Confirmed | BRD v13 | Метод наполняет отдельную страницу |
| TCO Booking Tool | FR-NEW-52 | Requestor has access to Request History | Confirmed | BRD v13 | Прямое покрытие |

---

## 3. Описание логики работы метода

1. Выбрать заявки текущего пользователя.
2. Оставить только записи со статусом `Completed`.
3. Подтянуть минимальные данные по booking item-ам для отображения в списке.
4. Применить поиск и пагинацию.
5. Вернуть список в `PaginatedResult`.

Сущности, участвующие в методе:
- читаются: `BookingRequests`, `Bookings`, `Equipments`, `EquipmentTypes`
- изменения не выполняются

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Requestor` | Просмотр истории своих заявок |
| `ServiceWorkProcessor` | Просмотр истории SWR |

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
| 1 | Поисковая строка | `search` | `string` | `-` | Поиск по request number, WO, типу техники | — | Query param | |
| 2 | Номер страницы | `page` | `int` | `-` | >= 1 | `1` | Query param | |
| 3 | Размер страницы | `limit` | `int` | `-` | >= 1 | `20` | Query param | |

---

## 8. Пример запроса

```http
GET /api/booking/v1/booking-requests/history?page=1&limit=20
Authorization: Bearer <token>
Content-Type: application/json
```

---

## 9. Возвращаемые данные

В `value` возвращается `PaginatedResult<CompletedRequestHistoryItem>`.

`CompletedRequestHistoryItem`: `id`, `requestNumber`, `equipmentTypeName`, `bookingPeriodStartDt`, `bookingPeriodEndDt`, `status`, `workOrderJdeId`.

---

## 10. Пример ответа

```json
{
  "value": {
    "items": [
      {
        "id": "c777f75f-029d-4d8f-8c69-e74a1d280001",
        "requestNumber": "REQ-2026-00015",
        "equipmentTypeName": {
          "En": "Excavator",
          "Ru": "Экскаватор",
          "Kz": "Экскаватор"
        },
        "bookingPeriodStartDt": "2026-05-20T08:00:00Z",
        "bookingPeriodEndDt": "2026-05-22T18:00:00Z",
        "status": "Completed",
        "workOrderJdeId": "WO-10025"
      }
    ],
    "total": 1
  },
  "isSuccess": true,
  "errors": []
}
```
