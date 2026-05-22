# POST /supervisor/bookings/{id}/decline

**Created:** 2026-05-14  
**Last updated:** 2026-05-22  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Отклонить long-term rented бронь как Supervisor |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Supervisor UI` |
| Endpoint URL | `/api/booking/v1/supervisor/bookings/{id}/decline` |
| Метод запроса | `POST` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Завершает supervisor approval отрицательным решением.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-NEW-73 | Supervisor can Decline with mandatory comment | Confirmed | BRD v13 | Прямое покрытие |
| TCO Booking Tool | FR-NEW-78 | Long-term rented booking -> Declined upon Supervisor decline | Confirmed | BRD v13 | Обновление статуса |

---

## 3. Описание логики работы метода

1. Проверить бронь и роль Supervisor.
2. Разрешить действие только для статуса `ConfirmedByFo`.
3. Потребовать непустой `comment`.
4. Обновить `status = Declined`.
5. Создать запись в `BookingApprovals` с `approvalType = SupervisorApproval`, `status = Declined`, `userId = currentUserId`, `approvalOrder = 2`, `comment = request.comment`.
6. Создать запись в `BookingStatuses`.
7. Вернуть результат.

Сущности:
- читаются: `Bookings`
- изменяются: `Bookings`, `BookingApprovals`, `BookingStatuses`

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `FleetOwnersSupervisor` | Финальное отклонение long-term rented брони |

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
| `FORBIDDEN` | Нет роли `FleetOwnersSupervisor` |
| `NOT_FOUND` | Бронь не найдена |
| `BOOKING_NOT_DECLINABLE` | Бронь не в статусе `ConfirmedByFo` |
| `VALIDATION_ERROR` | Не передан обязательный комментарий |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор брони | `id` | `uuid` | `+` | Должен существовать | — | Path param | |
| 2 | Комментарий Supervisor | `comment` | `string` | `+` | Непустая строка | — | Request body | Обязателен при decline |

---

## 8. Пример запроса

```http
POST /api/booking/v1/supervisor/bookings/8c4c8b6d-7bc0-41fb-9038-422cf55d1111/decline
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "comment": "Use available TCO-owned equipment for this task instead."
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
| 2 | Текущий статус | status | string | string | — | Bookings + BookingStatuses |  |
| 3 | Последнее записанное решение | lastApproval | object | object | — | backend composition from BookingApprovals | Последний approval step |

### Структура `value.lastApproval`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Тип шага согласования | type | string | string | — | BookingApprovals + ref_booking_approval_type | `SupervisorApproval` |
| 2 | Результат решения | approvalStatus | string | string | — | BookingApprovals + ref_booking_approval_status | `Declined` |
| 3 | Порядок шага | order | int | integer | — | BookingApprovals.approvalOrder | `2` |
| 4 | Комментарий | comment | string | string | — | BookingApprovals.comment | Совпадает с `request.comment` |

## 10. Пример ответа

```json
{
  "value": {
    "id": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
    "status": "Declined",
    "lastApproval": {
      "type": "SupervisorApproval",
      "approvalStatus": "Declined",
      "order": 2,
      "comment": "Use available TCO-owned equipment for this task instead."
    }
  },
  "isSuccess": true,
  "errors": []
}
```
