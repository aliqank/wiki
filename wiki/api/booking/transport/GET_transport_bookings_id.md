# GET /transport/bookings/{id}

**Created:** 2026-05-14  
**Last updated:** 2026-05-22  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить детали transport booking |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Transportation UI` |
| Endpoint URL | `/api/booking/v1/transport/bookings/{id}` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Открывает карточку транспортной задачи.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-NEW-29 | Three-step Unwheeled flow | Confirmed | BRD v13 | Карточка транспортной задачи |

---

## 3. Описание логики работы метода

1. Проверить бронь и роль пользователя.
2. Подтянуть заявку, технику, тип техники, период, FO-комментарии.
3. Подтянуть transport linkage из `BookingTransportations`.
4. Вернуть агрегированную карточку для решения по транспортировке.

Сущности:
- читаются: [`Bookings`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#22-bookings), [`BookingRequests`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#21-bookingrequests), [`Equipments`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#8-equipments), [`EquipmentTypes`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#1-equipmenttypes), [`BookingTransportations`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#25-bookingtransportations)

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| [`TransportationResponsible`](../../../requirements/Roles%20and%20Access%20Model.md) | Просмотр транспортных задач |

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
| `FORBIDDEN` | Нет роли [`TransportationResponsible`](../../../requirements/Roles%20and%20Access%20Model.md) |
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
GET /api/booking/v1/transport/bookings/aaabbbcc-dddd-4444-8888-123456780001
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
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | Bookings.id |  |
| 2 | Текущий статус | status | string | string | — | Bookings + BookingStatuses |  |
| 3 | Плановая дата и время начала | plannedStartDateTime | datetime | ISO 8601 | — | backend composition from Bookings + BookingRequests + Equipments + EquipmentTypes |  |
| 4 | Плановая дата и время окончания | plannedEndDateTime | datetime | ISO 8601 | — | backend composition from Bookings + BookingRequests + Equipments + EquipmentTypes |  |
| 5 | Идентификатор транспортирующей брони | transportingBookingId | uuid | UUID v4 | — | BookingTransportations.transportingBookingId | Может быть `null` |

## 10. Пример ответа

```json
{
  "value": {
    "id": "aaabbbcc-dddd-4444-8888-123456780001",
    "status": "Confirmed",
    "plannedStartDateTime": "2026-05-25T08:00:00Z",
    "plannedEndDateTime": "2026-05-25T18:00:00Z",
    "transportingBookingId": null
  },
  "isSuccess": true,
  "errors": []
}
```
