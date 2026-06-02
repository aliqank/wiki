# POST /bookings/{id}/extend

**Created:** 2026-05-14  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Запросить продление подтвержденной брони |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Requestor UI` |
| Endpoint URL | `/api/booking/v1/bookings/{id}/extend` |
| Метод запроса | `POST` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Переводит бронь в повторное согласование с новым периодом.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-061 | Requestor/SWP can extend confirmed booking | Confirmed | BRD v13 | Прямое покрытие |
| TCO Booking Tool | FR-067 | Booking -> Submitted once extended | Confirmed | BRD v13 | После продления статус снова `Submitted` |

---

## 3. Описание логики работы метода

1. Проверить существование брони и права доступа.
2. Разрешить продление только для подтвержденной активной брони.
3. Провалидировать новый `newPlannedEndDateTime` и hard-ограничения доступности техники на добавляемый период.
   Пересечения с другими активными бронями не должны автоматически блокировать продление; они передаются в approval flow для решения Fleet Owner.
4. Обновить `Bookings.plannedEndDateTime`.
5. Обновить `Bookings.status = Submitted` и создать запись в `BookingStatuses` с комментарием о продлении.
6. Вернуть обновленную бронь.

Сущности, участвующие в методе:
- читаются: [`Bookings`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#22-bookings)
- изменяются: [`Bookings`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#22-bookings), [`BookingStatuses`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#24-bookingstatuses)
- транзакционность: обязательна

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Requestor` | Продление собственной брони |
| `ServiceWorkProcessor` | Продление брони в SWR |

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
| `BOOKING_NOT_EXTENDABLE` | Бронь нельзя продлить |
| `EQUIPMENT_NOT_AVAILABLE` | Техника недоступна по hard-ограничениям на новый период; competing bookings сами по себе не вызывают эту ошибку |
| `VALIDATION_ERROR` | Невалидный `newPlannedEndDateTime` |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор брони | `id` | `uuid` | `+` | Должен существовать | — | Path param | |
| 2 | Новая плановая дата/время окончания | `newPlannedEndDateTime` | `datetime` | `+` | Должна быть больше текущего `plannedEndDateTime` | — | Request body | |
| 3 | Комментарий к продлению | `comment` | `string` | `-` | — | — | Request body | |

---

## 8. Пример запроса

```http
POST /api/booking/v1/bookings/8c4c8b6d-7bc0-41fb-9038-422cf55d1111/extend
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "newPlannedEndDateTime": "2026-05-23T18:00:00Z",
  "comment": "Need one additional shift for trench completion."
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
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | backend composition from Bookings + BookingStatuses |  |
| 2 | Текущий статус | status | string | string | — | backend composition from Bookings + BookingStatuses |  |
| 3 | Плановая дата и время начала | plannedStartDateTime | datetime | ISO 8601 | — | backend composition from Bookings + BookingStatuses |  |
| 4 | Плановая дата и время окончания | plannedEndDateTime | datetime | ISO 8601 | — | backend composition from Bookings + BookingStatuses |  |

## 10. Пример ответа

```json
{
  "value": {
    "id": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
    "status": "Submitted",
    "plannedStartDateTime": "2026-05-20T08:00:00Z",
    "plannedEndDateTime": "2026-05-23T18:00:00Z"
  },
  "isSuccess": true,
  "errors": []
}
```
