# POST /supervisor/bookings/{id}/confirm

**Created:** 2026-05-14  
**Last updated:** 2026-05-22  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Финально подтвердить long-term rented бронь |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Supervisor UI` |
| Endpoint URL | `/api/booking/v1/supervisor/bookings/{id}/confirm` |
| Метод запроса | `POST` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Завершает supervisor approval положительным решением.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-NEW-73 | Supervisor can Confirm -> booking moves to Confirmed | Confirmed | BRD v13 | Прямое покрытие |
| TCO Booking Tool | FR-NEW-78 | Long-term rented booking -> Confirmed upon Supervisor approval | Confirmed | BRD v13 | Обновление статуса |

---

## 3. Описание логики работы метода

1. Проверить бронь и роль Supervisor.
2. Разрешить действие только для статуса `ConfirmedByFo`.
3. Обновить `status = Confirmed`.
4. Создать запись в `BookingApprovals` с `approvalType = SupervisorApproval`, `status = Approved`, `userId = currentUserId`, `approvalOrder = 2`, `comment = request.comment`.
5. Создать запись в `BookingStatuses`.
6. Вернуть результат.

Сущности:
- читаются: `Bookings`
- изменяются: `Bookings`, `BookingApprovals`, `BookingStatuses`

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `FleetOwnersSupervisor` | Финальное подтверждение long-term rented брони |

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
| `BOOKING_NOT_CONFIRMABLE` | Бронь не в статусе `ConfirmedByFo` |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор брони | `id` | `uuid` | `+` | Должен существовать | — | Path param | |
| 2 | Комментарий Supervisor | `comment` | `string` | `-` | — | — | Request body | Необязателен для confirm |

---

## 8. Пример запроса

```http
POST /api/booking/v1/supervisor/bookings/8c4c8b6d-7bc0-41fb-9038-422cf55d1111/confirm
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "comment": "Confirmed after reviewing business need."
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
| 2 | Результат решения | approvalStatus | string | string | — | BookingApprovals + ref_booking_approval_status | `Approved` |
| 3 | Порядок шага | order | int | integer | — | BookingApprovals.approvalOrder | `2` |
| 4 | Комментарий | comment | string | string | — | BookingApprovals.comment | Может быть `null` |

## 10. Пример ответа

```json
{
  "value": {
    "id": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
    "status": "Confirmed",
    "lastApproval": {
      "type": "SupervisorApproval",
      "approvalStatus": "Approved",
      "order": 2,
      "comment": "Confirmed after reviewing business need."
    }
  },
  "isSuccess": true,
  "errors": []
}
```
