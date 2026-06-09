# POST /bookings/{id}/terminate

**Created:** 2026-05-14  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Досрочно завершить подтвержденную бронь |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Requestor UI` |
| Endpoint URL | `/api/booking/v1/bookings/{id}/terminate` |
| Метод запроса | `POST` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Завершает подтвержденную бронь до планового окончания.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-060 | [`Requestor`](../../../requirements/Roles%20and%20Access%20Model.md) can terminate confirmed booking | Confirmed | BRD v13 | Прямое покрытие для текущего scope |
| TCO Booking Tool | FR-069 | Booking -> Closed with closure reason Terminated once terminated | Confirmed | BRD v13 | Обновление статуса |

---

## 3. Описание логики работы метода

1. Проверить существование брони и права доступа.
2. Разрешить terminate только для внутренней подтвержденной брони.
3. Потребовать непустой `reason`.
4. Обновить бронь до terminal-состояния и зафиксировать причину terminate в terminal audit / closure reason модели.
5. Создать запись в `BookingStatuses` с `status = Closed` и `closureReason = Terminated`.
6. Пересчитать статус заявки.

Сущности, участвующие в методе:
- читаются: [`Bookings`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#22-bookings), [`BookingRequests`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#21-bookingrequests), [`Equipments`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#8-equipments)
- изменяются: [`Bookings`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#22-bookings), [`BookingStatuses`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#24-bookingstatuses), [`BookingRequests`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#21-bookingrequests), [`BookingRequestStatuses`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#26-bookingrequeststatuses)
- транзакционность: обязательна

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| [`Requestor`](../../../requirements/Roles%20and%20Access%20Model.md) | Terminate собственной брони |

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
| `BOOKING_NOT_TERMINABLE` | Бронь нельзя terminate |
| `VALIDATION_ERROR` | Не передана причина terminate |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор брони | `id` | `uuid` | `+` | Должен существовать | — | Path param | |
| 2 | Причина досрочного завершения | `reason` | `string` | `+` | Непустая строка | — | Request body | |

---

## 8. Пример запроса

```http
POST /api/booking/v1/bookings/8c4c8b6d-7bc0-41fb-9038-422cf55d1111/terminate
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "reason": "Work completed earlier than planned."
}
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
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | backend composition from Bookings + BookingRequests + Equipments + BookingStatuses + BookingRequestStatuses |  |
| 2 | Текущий статус | status | string | string | — | backend composition from Bookings + BookingRequests + Equipments + BookingStatuses + BookingRequestStatuses |  |

## 10. Пример ответа

```json
{
  "value": {
    "id": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
    "status": "Closed"
  },
  "isSuccess": true,
  "errors": []
}
```
