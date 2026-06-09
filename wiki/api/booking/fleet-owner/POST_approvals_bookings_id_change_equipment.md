# POST /approvals/bookings/{id}/change-equipment

**Created:** 2026-05-14  
**Last updated:** 2026-06-02  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Заменить технику в брони |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Fleet Owner UI` |
| Endpoint URL | `/api/booking/v1/approvals/bookings/{id}/change-equipment` |
| Метод запроса | `POST` |
| Связанные use cases | [`UC-FO-08 - Замена техники Fleet Owner`](../../../requirements/usecases/Fleet%20Owner/UC-FO-08%20-%20Замена%20техники%20Fleet%20Owner.md) |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Позволяет FO заменить технику в еще не начавшейся брони после выбора replacement candidate.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-046 | FO can replace equipment before/after confirmation if booking not yet started | Confirmed | BRD v13 | Прямое покрытие |
| TCO Booking Tool | FR-047 | FO cannot replace if booking revoked, terminated, or past end date | Confirmed | BRD v13 | Guard-условия |

---

## 3. Описание логики работы метода

1. Проверить бронь и права доступа.
2. Проверить, что бронь еще не началась и ее статус допускает замену: `Submitted` или `Confirmed`.
3. Проверить новую технику: тот же `Work Center`, допустимый бизнес-контекст, отсутствие [hard availability restrictions](../../../glossary/Glossary.md#hard-availability-restriction) на период, не `OnDemand`.
   Пересечения с другими активными бронями должны быть доступны Fleet Owner как [booking conflict context](../../../glossary/Glossary.md#booking-conflict-context) и не блокируют замену автоматически.
4. Проверить, что новая техника доступна текущему Fleet Owner по [Fleet Management Access](../../../glossary/Glossary.md#fleet-management-access).
5. Обновить `Bookings.equipmentId` и при необходимости `fleetId`.
6. Создать запись в `BookingStatuses` с комментарием о замене техники без перевода брони в новый approval lifecycle status.
7. Вернуть обновленную бронь.

Сущности:
- читаются: [`Bookings`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#22-bookings), [`Equipments`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#8-equipments)
- изменяются: [`Bookings`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#22-bookings), [`BookingStatuses`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#24-bookingstatuses)

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| [`FleetOwner`](../../../requirements/Roles%20and%20Access%20Model.md) | Замена техники в бронях fleet-ов, по которым у пользователя есть [Fleet Management Access](../../../glossary/Glossary.md#fleet-management-access) |

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
| `NOT_FOUND` | Бронь или новая техника не найдены |
| `BOOKING_NOT_CHANGEABLE` | Бронь нельзя изменить |
| `EQUIPMENT_NOT_AVAILABLE` | Новая техника недоступна по [hard availability restrictions](../../../glossary/Glossary.md#hard-availability-restriction); competing bookings сами по себе не вызывают эту ошибку |
| `VALIDATION_ERROR` | Новая техника несовместима по `Work Center` или другому обязательному business-критерию |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор брони | `id` | `uuid` | `+` | Должен существовать | — | Path param | |
| 2 | Идентификатор новой техники | `newEquipmentId` | `uuid` | `+` | Должен существовать | — | Request body | |
| 3 | Комментарий | `comment` | `string` | `-` | — | — | Request body | |

---

## 8. Пример запроса

```http
POST /api/booking/v1/approvals/bookings/8c4c8b6d-7bc0-41fb-9038-422cf55d1111/change-equipment
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "newEquipmentId": "7a5af8f7-6d0e-4b6f-ae68-f72906f70001",
  "comment": "Replacement due to maintenance on initially reserved unit."
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
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | Bookings.id |  |
| 2 | Идентификатор техники | equipmentId | uuid | UUID v4 | — | backend composition from Bookings + Equipments + BookingStatuses |  |
| 3 | Текущий статус | status | string | string | — | Bookings + BookingStatuses | Бронь сохраняет исходный допустимый lifecycle status (`Submitted` или `Confirmed`) |

## 10. Пример ответа

```json
{
  "value": {
    "id": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
    "equipmentId": "7a5af8f7-6d0e-4b6f-ae68-f72906f70001",
    "status": "Confirmed"
  },
  "isSuccess": true,
  "errors": []
}
```
