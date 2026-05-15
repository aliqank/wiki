# GET /equipment/{id}/load-summary

**Created:** 2026-05-14  
**Last updated:** 2026-05-14  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить сводку загрузки техники на выбранный период для Requestor |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Requestor UI` |
| Endpoint URL | `/api/booking/v1/equipment/{id}/load-summary` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Реализует отображение load summary при выборе техники в форме заявки.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-NEW-70 | System displays equipment load summary for selected period | Confirmed | BRD v13 | Прямое покрытие требования |

---

## 3. Описание логики работы метода

1. Проверить существование техники.
2. Провалидировать `startDt` и `endDt`.
3. Выбрать из `Bookings` записи по `equipmentId = :id`, пересекающиеся с заданным периодом.
4. Включить только статусы, релевантные для анализа загрузки: `Submitted`, `ConfirmedByFo`, `Confirmed`, `TransportConfirmed`, `InProgress`, `Extended`.
5. Вернуть summary с кратким списком пересечений и агрегатами по количеству записей.

Сущности, участвующие в методе:
- читаются: `Bookings`, `BookingRequests`, `Equipments`
- изменения не выполняются

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Requestor` | Выбор техники в заявке |
| `ServiceWorkProcessor` | Выбор техники в SWR |

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
| `NOT_FOUND` | Техника не найдена |
| `VALIDATION_ERROR` | Период невалиден |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор техники | `id` | `uuid` | `+` | Должен существовать | — | Path param | |
| 2 | Дата/время начала периода | `startDt` | `datetime` | `+` | Меньше `endDt` | — | Query param | |
| 3 | Дата/время окончания периода | `endDt` | `datetime` | `+` | Больше `startDt` | — | Query param | |

---

## 8. Пример запроса

```http
GET /api/booking/v1/equipment/c3b5af91-61f8-4bc0-bd88-d099d3e90001/load-summary?startDt=2026-05-20T08:00:00Z&endDt=2026-05-22T18:00:00Z
Authorization: Bearer <token>
Content-Type: application/json
```

---

## 9. Возвращаемые данные

В `value` возвращается объект `EquipmentLoadSummary` с полями: `equipmentId`, `periodStartDt`, `periodEndDt`, `activeBookingsCount`, `items[]`.

Элемент `items[]`: `bookingId`, `requestId`, `requestNumber`, `status`, `startDt`, `endDt`.

---

## 10. Пример ответа

```json
{
  "value": {
    "equipmentId": "c3b5af91-61f8-4bc0-bd88-d099d3e90001",
    "periodStartDt": "2026-05-20T08:00:00Z",
    "periodEndDt": "2026-05-22T18:00:00Z",
    "activeBookingsCount": 2,
    "items": [
      {
        "bookingId": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
        "requestId": "c777f75f-029d-4d8f-8c69-e74a1d280001",
        "requestNumber": "REQ-2026-00015",
        "status": "Confirmed",
        "startDt": "2026-05-20T06:00:00Z",
        "endDt": "2026-05-21T18:00:00Z"
      }
    ]
  },
  "isSuccess": true,
  "errors": []
}
```
