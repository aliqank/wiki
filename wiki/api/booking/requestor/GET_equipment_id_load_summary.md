# GET /equipment/{id}/load-summary

**Created:** 2026-05-14  
**Last updated:** 2026-06-02  
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
| Связанные use cases | [`UC-REQ-02.3.1 - Просмотр load summary при выборе техники`](../../../requirements/usecases/Requestor/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first)/UC-REQ-02.3.1%20-%20Просмотр%20load%20summary%20при%20выборе%20техники.md) |
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
2. Провалидировать `plannedStartDateTime` и `plannedEndDateTime`.
3. Выбрать из `Bookings` записи по `equipmentId = :id`, пересекающиеся с заданным периодом.
4. Включить только статусы, релевантные для анализа загрузки: `Submitted`, `Confirmed`, `InProgress`.
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
| 2 | Плановая дата/время начала периода | `plannedStartDateTime` | `datetime` | `+` | Меньше `plannedEndDateTime` | — | Query param | |
| 3 | Плановая дата/время окончания периода | `plannedEndDateTime` | `datetime` | `+` | Больше `plannedStartDateTime` | — | Query param | |

---

## 8. Пример запроса

```http
GET /api/booking/v1/equipment/c3b5af91-61f8-4bc0-bd88-d099d3e90001/load-summary?plannedStartDateTime=2026-05-20T08:00:00Z&plannedEndDateTime=2026-05-22T18:00:00Z
Authorization: Bearer <token>
Content-Type: application/json
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий `result wrapper`.

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | value | object | object | — | backend aggregation |  |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | object[] | `[]` | backend |  |

### Структура `value`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор техники | equipmentId | uuid | UUID v4 | — | backend composition from Bookings + BookingRequests + Equipments |  |
| 2 | Дата и время начала периода | periodStartDateTime | datetime | ISO 8601 | — | backend composition from Bookings + BookingRequests + Equipments |  |
| 3 | Дата и время окончания периода | periodEndDateTime | datetime | ISO 8601 | — | backend composition from Bookings + BookingRequests + Equipments |  |
| 4 | Количество активных броней в периоде (`Submitted`, `Confirmed`, `InProgress`) | activeBookingsCount | int | integer | — | backend composition from Bookings + BookingRequests + Equipments |  |
| 5 | Элементы текущей страницы | items | array<object> | object[] | — | backend composition from Bookings + BookingRequests + Equipments | Коллекция объектов |

### Структура `value.items[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор брони | bookingId | uuid | UUID v4 | — | backend composition from Bookings + BookingRequests + Equipments |  |
| 2 | Идентификатор заявки | requestId | uuid | UUID v4 | — | backend composition from Bookings + BookingRequests + Equipments |  |
| 3 | Номер заявки | requestNumber | string | string | — | backend composition from Bookings + BookingRequests + Equipments |  |
| 4 | Номер Work Order | workOrderNumber | string | string | — | backend composition from Bookings + BookingRequests + Equipments | Может быть `null`, если для заявки используется `Default Work Order` |
| 5 | Текущий статус | status | string | string | — | backend composition from Bookings + BookingRequests + Equipments |  |
| 6 | Плановая дата и время начала | plannedStartDateTime | datetime | ISO 8601 | — | backend composition from Bookings + BookingRequests + Equipments |  |
| 7 | Плановая дата и время окончания | plannedEndDateTime | datetime | ISO 8601 | — | backend composition from Bookings + BookingRequests + Equipments |  |

## 10. Пример ответа

```json
{
  "value": {
    "equipmentId": "c3b5af91-61f8-4bc0-bd88-d099d3e90001",
    "periodStartDateTime": "2026-05-20T08:00:00Z",
    "periodEndDateTime": "2026-05-22T18:00:00Z",
    "activeBookingsCount": 2,
    "items": [
      {
        "bookingId": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
        "requestId": "c777f75f-029d-4d8f-8c69-e74a1d280001",
        "requestNumber": "REQ-2026-00015",
        "workOrderNumber": "WO-2026-00421",
        "status": "Confirmed",
        "plannedStartDateTime": "2026-05-20T06:00:00Z",
        "plannedEndDateTime": "2026-05-21T18:00:00Z"
      }
    ]
  },
  "isSuccess": true,
  "errors": []
}
```

## Замечания

1. Наличие записей в load summary не означает автоматический запрет на создание или отправку заявки.
2. Summary используется для информирования Requestor о competing bookings по выбранной технике.
