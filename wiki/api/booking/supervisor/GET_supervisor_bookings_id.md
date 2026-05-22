# GET /supervisor/bookings/{id}

**Created:** 2026-05-14  
**Last updated:** 2026-05-22  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить детали брони для финального решения Supervisor |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Supervisor UI` |
| Endpoint URL | `/api/booking/v1/supervisor/bookings/{id}` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Открывает карточку брони с pending шагом `SupervisorApproval`.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-NEW-73 | Supervisor reviews booking including Requestor justification | Confirmed | BRD v13 | Карточка решения |

---

## 3. Описание логики работы метода

1. Проверить бронь и роль Supervisor.
2. Проверить, что по брони ожидается шаг `SupervisorApproval`.
3. Подтянуть `BookingRequests`, `Equipments`, `EquipmentTypes`, `BookingApprovals`, историю статусов.
4. Вернуть агрегированную модель для review.

Сущности:
- читаются: `Bookings`, `BookingRequests`, `Equipments`, `EquipmentTypes`, `BookingApprovals`, `BookingStatuses`, `Users`

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `FleetOwnersSupervisor` | Просмотр карточек long-term rented броней на финальном шаге |

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

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор брони | `id` | `uuid` | `+` | Должен существовать | — | Path param | |

---

## 8. Пример запроса

```http
GET /api/booking/v1/supervisor/bookings/8c4c8b6d-7bc0-41fb-9038-422cf55d1111
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
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | Bookings.id |  |
| 2 | Текущий статус | status | string | string | — | Bookings + BookingStatuses |  |
| 3 | Обоснование | justification | string | string | — | Bookings.justification |  |
| 4 | Цепочка согласования | approvalChain | array<object> | object[] | `[]` | backend composition from BookingApprovals + Users + reference tables | Коллекция approval step-ов |
| 5 | История изменений | history | array<object> | object[] | `[]` | backend composition from Bookings + BookingRequests + Equipments + EquipmentTypes + BookingStatuses | Коллекция объектов |

### Структура `value.approvalChain[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор шага согласования | id | uuid | UUID v4 | — | BookingApprovals.id |  |
| 2 | Тип шага согласования | type | string | string | — | BookingApprovals + ref_booking_approval_type | `FoApproval` / `SupervisorApproval` |
| 3 | Результат решения | approvalStatus | string | string | — | BookingApprovals + ref_booking_approval_status | `Approved` / `Declined` |
| 4 | Порядок шага | order | int | integer | — | BookingApprovals.approvalOrder |  |
| 5 | Идентификатор пользователя | userId | uuid | UUID v4 | — | BookingApprovals.userId |  |
| 6 | Комментарий | comment | string | string | — | BookingApprovals.comment |  |
| 7 | Дата и время решения | createdAt | datetime | ISO 8601 | — | BookingApprovals.createdAt |  |

## 10. Пример ответа

```json
{
  "value": {
    "id": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
    "status": "Submitted",
    "justification": "No suitable TCO-owned unit available for required window.",
    "approvalChain": [
      {
        "id": "9a4c8b6d-7bc0-41fb-9038-422cf55d0001",
        "type": "FoApproval",
        "approvalStatus": "Approved",
        "order": 1,
        "userId": "4c9ad2d2-6df8-4f7b-87fe-36cefc100001",
        "comment": "Approved by FO.",
        "createdAt": "2026-05-18T11:45:00Z"
      }
    ],
    "history": []
  },
  "isSuccess": true,
  "errors": []
}
```
