# GET /approvals/bookings/{id}/load-summary

**Created:** 2026-05-14  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить сводку загрузки техники на даты брони для Fleet Owner |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Fleet Owner UI` |
| Endpoint URL | `/api/booking/v1/approvals/bookings/{id}/load-summary` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется в окне принятия решения FO.

Frontend открывает popup / modal `Load summary` по выбранной брони.
Заголовок popup / modal может быть собран из уже загруженного booking context на странице `Requests` или в detail view, а этот метод возвращает summary line и список competing bookings.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-NEW-64 | Display equipment loading summary by dates in FO approval window | Confirmed | BRD v13 | Прямое покрытие |

---

## 3. Описание логики работы метода

1. Получить бронь и проверить доступ FO.
2. Выбрать пересекающиеся брони по тому же `equipmentId`.
3. Исключить текущую бронь из списка пересечений.
4. Вернуть summary для окна approval.

Сущности:
- читаются: `Bookings`, `BookingRequests`

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `FleetOwner` | Просмотр загрузки техники по своим броням |

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
| `FORBIDDEN` | Нет доступа к брони |
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
GET /api/booking/v1/approvals/bookings/8c4c8b6d-7bc0-41fb-9038-422cf55d1111/load-summary
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
| 1 | Идентификатор техники | equipmentId | uuid | UUID v4 | — | backend composition from Bookings + BookingRequests |  |
| 2 | Количество активных броней в периоде (`Submitted`, `Confirmed`, `InProgress`) | activeBookingsCount | int | integer | — | backend composition from Bookings + BookingRequests | Используется для summary line и conflict state в popup / modal |
| 3 | Элементы текущей страницы | items | array<object> | object[] | — | backend composition from Bookings + BookingRequests | Коллекция объектов |

### Структура `value.items[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор брони | bookingId | uuid | UUID v4 | — | Bookings.id |  |
| 2 | Номер заявки | requestNumber | string | string | — | BookingRequests.requestNumber |  |
| 3 | Номер Work Order | workOrderNumber | string | string | — | BookingRequests.workOrderNumber | Может быть `null`, если для заявки используется `Default Work Order` |
| 4 | Текущий статус | status | string | string | — | Bookings + BookingStatuses |  |
| 5 | Плановая дата и время начала | plannedStartDateTime | datetime | ISO 8601 | — | backend composition from Bookings + BookingRequests |  |
| 6 | Плановая дата и время окончания | plannedEndDateTime | datetime | ISO 8601 | — | backend composition from Bookings + BookingRequests |  |

## 10. Пример ответа

```json
{
  "value": {
    "equipmentId": "c3b5af91-61f8-4bc0-bd88-d099d3e90001",
    "activeBookingsCount": 1,
    "items": [
      {
        "bookingId": "6e16b907-9d91-4d39-a6c5-1af22d710001",
        "requestNumber": "REQ-2026-00012",
        "workOrderNumber": "WO-2026-00421",
        "status": "Confirmed",
        "plannedStartDateTime": "2026-05-19T08:00:00Z",
        "plannedEndDateTime": "2026-05-21T18:00:00Z"
      }
    ]
  },
  "isSuccess": true,
  "errors": []
}
```

## Замечания

1. Summary предназначен для принятия решения Fleet Owner по competing bookings и не является самостоятельным hard-stop механизмом.
2. Типовой popup / modal `Load summary` показывает:
   - контекст текущей брони, уже известный frontend-у из строки списка или detail view;
   - summary line на основе `activeBookingsCount`;
   - список competing bookings из `items[]`.
