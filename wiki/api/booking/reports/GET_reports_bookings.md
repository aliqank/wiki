# GET /reports/bookings

**Created:** 2026-05-14  
**Last updated:** 2026-05-14  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить отчет по броням |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Reporting` |
| Endpoint URL | `/api/booking/v1/reports/bookings` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется для отчетов по booking item-ам.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-094 | System generates reports on Requests/Bookings/Equipment with filters | Confirmed | BRD v13 | Прямое покрытие |
| TCO Booking Tool | FR-096 | FO can view/download own approval reports | Confirmed | BRD v13 | Частный случай доступа |

---

## 3. Описание логики работы метода

1. Проверить права доступа.
2. Выбрать `Bookings` по фильтрам статуса, периода, ownershipType, workCenter.
3. Подтянуть `BookingRequests`, `Equipments`, `EquipmentTypes`.
4. Вернуть пагинированный отчет.

Сущности:
- читаются: `Bookings`, `BookingRequests`, `Equipments`, `EquipmentTypes`

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Admin` | Полный отчет по всем броням |
| `FleetOwner` | Отчет по своим броням |

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
| `FORBIDDEN` | Нет прав на отчеты |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Статус брони | `status` | `enum` | `-` | Статусы брони | — | Query param | |
| 2 | Тип владения | `ownershipType` | `enum` | `-` | `TcoOwned / LongTermRented` | — | Query param | |
| 3 | Work Center | `workCenterId` | `uuid` | `-` | Если передан, должен существовать | — | Query param | |
| 4 | Номер страницы | `page` | `int` | `-` | >= 1 | `1` | Query param | |
| 5 | Размер страницы | `limit` | `int` | `-` | >= 1 | `20` | Query param | |

---

## 8. Пример запроса

```http
GET /api/booking/v1/reports/bookings?status=Confirmed&page=1&limit=20
Authorization: Bearer <token>
Content-Type: application/json
```

---

## 9. Возвращаемые данные

В `value` возвращается `PaginatedResult<BookingReportItem>`.

---

## 10. Пример ответа

```json
{
  "value": {
    "items": [
      {
        "id": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
        "requestNumber": "REQ-2026-00015",
        "status": "Confirmed",
        "ownershipType": "TcoOwned"
      }
    ],
    "total": 1
  },
  "isSuccess": true,
  "errors": []
}
```
