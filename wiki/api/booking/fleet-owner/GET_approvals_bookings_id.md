# GET /approvals/bookings/{id}

**Created:** 2026-05-14  
**Last updated:** 2026-05-22  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить детали брони для Fleet Owner |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Fleet Owner UI` |
| Endpoint URL | `/api/booking/v1/approvals/bookings/{id}` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Открывает карточку входящей брони для действий FO.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-041 | Equipment attributes displayed in booking | Confirmed | BRD v13 | Детали техники в approval окне |
| TCO Booking Tool | FR-NEW-64 | Display equipment loading summary by dates in FO approval window | Confirmed | BRD v13 | Карточка используется вместе с load summary |

---

## 3. Описание логики работы метода

1. Проверить существование брони.
2. Проверить, что бронь принадлежит одному из флотов FO.
3. Подтянуть `BookingRequests`, `Equipments`, `EquipmentTypes`, `EquipmentProperties`, `EquipmentPhotos`, `BookingApprovals`.
4. Вернуть агрегированную модель брони.

Сущности:
- читаются: `Bookings`, `BookingRequests`, `Equipments`, `EquipmentTypes`, `EquipmentProperties`, `EquipmentPhotos`, `BookingApprovals`, `Users`

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `FleetOwner` | Просмотр броней по своим флотам |

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
| `FORBIDDEN` | Бронь не относится к флотам пользователя |
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
GET /api/booking/v1/approvals/bookings/8c4c8b6d-7bc0-41fb-9038-422cf55d1111
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
| 3 | Плановая дата и время начала | plannedStartDateTime | datetime | ISO 8601 | — | backend composition from Bookings + BookingRequests + Equipments + EquipmentTypes + EquipmentProperties + EquipmentPhotos |  |
| 4 | Плановая дата и время окончания | plannedEndDateTime | datetime | ISO 8601 | — | backend composition from Bookings + BookingRequests + Equipments + EquipmentTypes + EquipmentProperties + EquipmentPhotos |  |
| 5 | Обоснование | justification | null | — | `null` | Bookings.justification |  |
| 6 | Цепочка согласования | approvalChain | array<object> | object[] | `[]` | backend composition from BookingApprovals + Users + reference tables | Коллекция approval step-ов |
| 7 | История изменений | history | array<object> | object[] | `[]` | backend composition from Bookings + BookingRequests + Equipments + EquipmentTypes + EquipmentProperties + EquipmentPhotos | Коллекция объектов |

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
    "plannedStartDateTime": "2026-05-20T08:00:00Z",
    "plannedEndDateTime": "2026-05-22T18:00:00Z",
    "justification": null,
    "approvalChain": [],
    "history": []
  },
  "isSuccess": true,
  "errors": []
}
```
