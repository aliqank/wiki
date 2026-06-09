# GET /equipment/{id}/load-summary

**Created:** 2026-05-14  
**Last updated:** 2026-06-05  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить сводку загрузки техники на выбранный период для [`Requestor`](../../../requirements/Roles and Access Model.md) |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / [`Requestor`](../../../requirements/Roles and Access Model.md) UI` |
| Endpoint URL | `/api/booking/v1/equipment/{id}/load-summary` |
| Метод запроса | `GET` |
| Связанные use cases | [`UC-REQ-02.3.1 - Просмотр load summary при выборе техники`](../../../requirements/usecases/[`Requestor`](../../../requirements/Roles and Access Model.md)/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first)/UC-REQ-02.3.1%20-%20Просмотр%20load%20summary%20при%20выборе%20техники.md) |
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
6. Для каждой пересекающейся брони дополнительно вернуть:
   - фактические дата и время начала / завершения при наличии;
   - статус как объект, а не как плоскую строку;
   - `workDescription` из родительской `BookingRequest`.

Сущности, участвующие в методе:
- читаются: [`Bookings`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#22-bookings), [`BookingRequests`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#21-bookingrequests), [`Equipments`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#8-equipments)
- изменения не выполняются

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| [`Requestor`](../../../requirements/Roles and Access Model.md) | Выбор техники в заявке |

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

Возвращаемые данные обёрнуты в общий [`result wrapper`](../../common/Result%20Wrapper.md).

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
| 5 | Описание работ из родительской заявки | workDescription | string | string | — | backend composition from BookingRequests | Поле родительской заявки |
| 6 | Текущий статус | status | object | object | — | backend composition from Bookings + ref_booking_status + ref_booking_closure_reason |  |
| 7 | Плановая дата и время начала | plannedStartDateTime | datetime | ISO 8601 | — | backend composition from Bookings + BookingRequests + Equipments |  |
| 8 | Плановая дата и время окончания | plannedEndDateTime | datetime | ISO 8601 | — | backend composition from Bookings + BookingRequests + Equipments |  |
| 9 | Фактическая дата и время начала | actualStartDateTime | datetime | ISO 8601 | `null` | backend composition from Bookings | Для еще не начатой брони возвращается `null` |
| 10 | Фактическая дата и время завершения | actualEndDateTime | datetime | ISO 8601 | `null` | backend composition from Bookings | Для незавершенной брони возвращается `null` |

### Структура `value.items[].status`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Код lifecycle статуса | code | string | string | — | ref_booking_status | Например: `Submitted`, `Confirmed`, `InProgress`, `Closed` |
| 2 | Подпись lifecycle статуса | label | string | string | — | ref_booking_status | UI-readable caption |
| 3 | Код terminal closure reason | closureReason | string | null | `null` | ref_booking_closure_reason | Заполняется только если статус terminal |
| 4 | Подпись terminal closure reason | closureReasonLabel | string | null | `null` | ref_booking_closure_reason | Заполняется только если статус terminal |

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
        "workDescription": "Excavator required for trench preparation near sector 4.",
        "status": {
          "code": "Confirmed",
          "label": "Confirmed",
          "closureReason": null,
          "closureReasonLabel": null
        },
        "plannedStartDateTime": "2026-05-20T06:00:00Z",
        "plannedEndDateTime": "2026-05-21T18:00:00Z",
        "actualStartDateTime": "2026-05-20T06:15:00Z",
        "actualEndDateTime": null
      }
    ]
  },
  "isSuccess": true,
  "errors": []
}
```

## Замечания

1. Наличие записей в load summary не означает автоматический запрет на создание или отправку заявки.
2. Summary используется для информирования [`Requestor`](../../../requirements/Roles and Access Model.md) о competing bookings по выбранной технике.
