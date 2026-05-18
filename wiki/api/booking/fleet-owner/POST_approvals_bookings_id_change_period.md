# POST /approvals/bookings/{id}/change-period

**Created:** 2026-05-14  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Изменить период брони как Fleet Owner |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Fleet Owner UI` |
| Endpoint URL | `/api/booking/v1/approvals/bookings/{id}/change-period` |
| Метод запроса | `POST` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Позволяет FO менять период до и после подтверждения.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-048 | FO can adjust booking date range before or after confirmation | Confirmed | BRD v13 | Прямое покрытие |
| TCO Booking Tool | FR-079a | System notifies Requestor when FO updates booking period | Confirmed | BRD v13 | Есть side effect-уведомление |

---

## 3. Описание логики работы метода

1. Проверить бронь и права доступа.
2. Проверить, что статус брони допускает изменение периода.
3. Провалидировать новый диапазон дат и доступность техники.
4. Обновить `plannedStartDateTime`, `plannedEndDateTime`.
5. Создать запись в `BookingStatuses` с комментарием.
6. Вернуть обновленную бронь.

Сущности:
- читаются: `Bookings`
- изменяются: `Bookings`, `BookingStatuses`

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `FleetOwner` | Изменение периода броней своих флотов |

---

## 5. Настройки системы, используемые в методе

| Наименование | Код | Тип значения | Описание | Значение по умолчанию |
|---|---|---|---|---|
| Максимальная длительность брони | `MAX_BOOKING_DURATION_DAYS` | `int` | Проверка нового периода | Конфигурируется Admin |

---

## 6. Ошибки, возвращаемые методом

| Код | Описание ошибки |
|---|---|
| `UNAUTHORIZED` | Пользователь не авторизован |
| `FORBIDDEN` | Нет доступа к брони |
| `NOT_FOUND` | Бронь не найдена |
| `BOOKING_NOT_CHANGEABLE` | Бронь нельзя изменить |
| `EQUIPMENT_NOT_AVAILABLE` | Новый период конфликтует с другими бронями |
| `VALIDATION_ERROR` | Новый период невалиден |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор брони | `id` | `uuid` | `+` | Должен существовать | — | Path param | |
| 2 | Новая плановая дата/время начала | `plannedStartDateTime` | `datetime` | `+` | Меньше `plannedEndDateTime` | — | Request body | |
| 3 | Новая плановая дата/время окончания | `plannedEndDateTime` | `datetime` | `+` | Больше `plannedStartDateTime` | — | Request body | |
| 4 | Комментарий | `comment` | `string` | `-` | — | — | Request body | |

---

## 8. Пример запроса

```http
POST /api/booking/v1/approvals/bookings/8c4c8b6d-7bc0-41fb-9038-422cf55d1111/change-period
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "plannedStartDateTime": "2026-05-20T10:00:00Z",
  "plannedEndDateTime": "2026-05-22T20:00:00Z",
  "comment": "Shifted due to site access window."
}
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
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | Bookings.id |  |
| 2 | Плановая дата и время начала | plannedStartDateTime | datetime | ISO 8601 | — | backend composition from Bookings + BookingStatuses |  |
| 3 | Плановая дата и время окончания | plannedEndDateTime | datetime | ISO 8601 | — | backend composition from Bookings + BookingStatuses |  |
| 4 | Текущий статус | status | string | string | — | Bookings + BookingStatuses |  |

## 10. Пример ответа

```json
{
  "value": {
    "id": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
    "plannedStartDateTime": "2026-05-20T10:00:00Z",
    "plannedEndDateTime": "2026-05-22T20:00:00Z",
    "status": "Confirmed"
  },
  "isSuccess": true,
  "errors": []
}
```
