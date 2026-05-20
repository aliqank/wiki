# POST /approvals/bookings/{id}/confirm

**Created:** 2026-05-14  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Подтвердить бронь как Fleet Owner |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Fleet Owner UI` |
| Endpoint URL | `/api/booking/v1/approvals/bookings/{id}/confirm` |
| Метод запроса | `POST` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Выполняет FO approval.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-045 | FO can confirm incoming bookings | Confirmed | BRD v13 | Прямое покрытие |
| TCO Booking Tool | FR-063 | TCO Owned -> Confirmed; Long-term rented -> ConfirmedByFo | Confirmed | BRD v13 | Разный результат по ownershipType |
| TCO Booking Tool | FR-NEW-72 | Long-term rented routed to Supervisor after FO confirmation | Confirmed | BRD v13 | Ветка `ConfirmedByFo` |

---

## 3. Описание логики работы метода

1. Проверить бронь и права доступа.
2. Разрешить действие только для статуса `Submitted`.
3. Проверить, не изменилась ли доступность техники.
4. Если `ownershipType = LongTermRented`, установить `status = ConfirmedByFo`, `requiresSupervisorApproval = true`.
5. Иначе установить `status = Confirmed`.
6. Создать запись в `BookingStatuses`.
7. При long-term rented инициировать уведомление Supervisor.
8. Вернуть результат.

Сущности:
- читаются: `Bookings`, `Equipments`
- изменяются: `Bookings`, `BookingStatuses`
- транзакционность: обязательна

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `FleetOwner` | Подтверждение броней своих флотов |

---

## 5. Настройки системы, используемые в методе

| Наименование | Код | Тип значения | Описание | Значение по умолчанию |
|---|---|---|---|---|
| Таймаут ответа Supervisor | `SUPERVISOR_RESPONSE_TIMEOUT_HOURS` | `int` | Используется после перехода в `ConfirmedByFo` | Конфигурируется Admin |

---

## 6. Ошибки, возвращаемые методом

| Код | Описание ошибки |
|---|---|
| `UNAUTHORIZED` | Пользователь не авторизован |
| `FORBIDDEN` | Нет доступа к брони |
| `NOT_FOUND` | Бронь не найдена |
| `BOOKING_NOT_CONFIRMABLE` | Текущий статус не позволяет confirm |
| `EQUIPMENT_NOT_AVAILABLE` | Техника уже недоступна |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор брони | `id` | `uuid` | `+` | Должен существовать | — | Path param | |
| 2 | Комментарий FO | `comment` | `string` | `-` | — | — | Request body | Необязательный служебный комментарий |

---

## 8. Пример запроса

```http
POST /api/booking/v1/approvals/bookings/8c4c8b6d-7bc0-41fb-9038-422cf55d1111/confirm
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "comment": "Approved by FO."
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
| 3 | Признак необходимости согласования Supervisor | requiresSupervisorApproval | bool | boolean | — | Bookings.requiresSupervisorApproval |  |

## 10. Пример ответа

```json
{
  "value": {
    "id": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
    "status": "Confirmed",
    "requiresSupervisorApproval": false
  },
  "isSuccess": true,
  "errors": []
}
```
