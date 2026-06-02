# PATCH /booking-requests/{id}/items/{bookingId}

**Created:** 2026-05-19  
**Last updated:** 2026-06-02  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Обновить booking item в черновике заявки |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Requestor UI` |
| Endpoint URL | `/api/booking/v1/booking-requests/{id}/items/{bookingId}` |
| Метод запроса | `PATCH` |
| Связанные use cases | [`UC-REQ-02.5 - Редактирование брони или замена техники в draft`](../../../requirements/usecases/Requestor/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first)/UC-REQ-02.5%20-%20Редактирование%20брони%20или%20замена%20техники%20в%20draft.md) |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется для редактирования периода, `justification`, `workCenterId` и/или замены `equipmentId` у существующего booking item в draft-заявке без удаления и повторного создания всей заявки.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-027 | Requestor can edit/cancel draft before submission | Confirmed | BRD v13 | Редактирование item в draft |
| TCO Booking Tool | FR-031 | Requestor can add/remove equipment items to a request | Confirmed | BRD v13 | Замена техники внутри item |
| TCO Booking Tool | FR-038 | Each equipment item in request = separate booking | Confirmed | BRD v13 | Обновляется одна существующая бронь |
| TCO Booking Tool | FR-040 | System validates availability before booking | Confirmed | BRD v13 | Проверка доступности при изменении обязательна |
| TCO Booking Tool | FR-NEW-71 | Justification mandatory for Long-term rented item | Confirmed | BRD v13 | Поддерживает позднее сохранение justification через autosave |

---

## 3. Описание логики работы метода

1. Проверить существование draft-заявки, booking item и права доступа.
2. Разрешить редактирование только если `BookingRequests.status = Draft` и `Bookings.status = Draft`.
3. Обновить только переданные поля; для непереданных полей использовать текущие значения booking item.
4. Если меняется `equipmentId`, проверить существование новой техники и получить ее атрибуты `ownershipType`, `shareType`, `fleetId`.
5. Для итогового набора значений проверить, что техника не относится к `OnDemand`.
6. Для итогового набора значений проверить hard-ограничения доступности техники на выбранный период.
   Под hard-ограничениями в рамках текущего базового сценария понимается, что техника не заблокирована причинами, не связанными с competing bookings, например активными записями в `EquipmentStatuses`.
7. Отдельно рассчитать наличие конфликтов с активными записями `Bookings` со статусами `Submitted`, `Confirmed`, `InProgress`, если их период пересекается с итоговым периодом item.
   Такие конфликты не блокируют patch и используются только для информирования пользователя и последующего решения Fleet Owner.
8. Если итоговая техника относится к `LongTermRented`, `Assigned` или `SharedWithConditions`, определить, что для item обязателен `justification`; при его отсутствии признак `hasRequiredJustification` остается `false` до последующего заполнения.
9. Если итоговая техника `Assigned`, проверить `EquipmentBookingAuthorizations`.
10. Если обновляется `justification`, метод может вызываться frontend-ом как autosave без отдельной кнопки сохранения.
11. Сохранить изменения в существующей записи `Bookings`.
12. Пересчитать признаки `requiresJustification` и `hasRequiredJustification` для итогового состояния item.
13. Вернуть обновленный booking item.

Сущности, участвующие в методе:
- читаются: `BookingRequests`, `Bookings`, `Equipments`, `EquipmentBookingAuthorizations`
- изменяются: `Bookings`

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Requestor` | Редактирование item в своей draft-заявке |
| `ServiceWorkProcessor` | Редактирование item в draft SWR |

---

## 5. Настройки системы, используемые в методе

| Наименование | Код | Тип значения | Описание | Значение по умолчанию |
|---|---|---|---|---|
| Горизонт бронирования | `BOOKING_HORIZON_DAYS` | `int` | Проверка даты | Конфигурируется Admin |
| Максимальная длительность брони | `MAX_BOOKING_DURATION_DAYS` | `int` | Проверка периода | Конфигурируется Admin |

---

## 6. Ошибки, возвращаемые методом

| Код | Описание ошибки |
|---|---|
| `UNAUTHORIZED` | Пользователь не авторизован |
| `FORBIDDEN` | Нет доступа к заявке или технике |
| `NOT_FOUND` | Заявка, booking item или техника не найдены |
| `REQUEST_NOT_EDITABLE` | Заявка или booking item не в статусе `Draft` |
| `EQUIPMENT_NOT_AVAILABLE` | Итоговая техника недоступна по hard-ограничениям на выбранный период; competing bookings сами по себе не вызывают эту ошибку |
| `VALIDATION_ERROR` | Не пройдены бизнес-валидации |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор заявки | `id` | `uuid` | `+` | Должен существовать | — | Path param | |
| 2 | Идентификатор booking item | `bookingId` | `uuid` | `+` | Должен существовать и принадлежать заявке | — | Path param | |
| 3 | Идентификатор техники | `equipmentId` | `uuid` | `-` | Если передан, должен существовать | — | Request body | Если не передан, сохраняется текущее значение |
| 4 | Плановая дата/время начала | `plannedStartDateTime` | `datetime` | `-` | Для итогового набора значений должно быть меньше `plannedEndDateTime` | — | Request body | Если не передан, сохраняется текущее значение |
| 5 | Плановая дата/время окончания | `plannedEndDateTime` | `datetime` | `-` | Для итогового набора значений должно быть больше `plannedStartDateTime` | — | Request body | Если не передан, сохраняется текущее значение |
| 6 | Work Center | `workCenterId` | `uuid` | `-` | Если передан, должен существовать | — | Request body | Если не передан, сохраняется текущее значение |
| 7 | Обоснование | `justification` | `string` | `-` | Может сохраняться позднее через autosave; для итоговой техники `LongTermRented`, `Assigned`, `SharedWithConditions` должно быть заполнено к моменту submit | — | Request body | Если не передан, сохраняется текущее значение |

---

## 8. Пример запроса

```http
PATCH /api/booking/v1/booking-requests/c777f75f-029d-4d8f-8c69-e74a1d280001/items/8c4c8b6d-7bc0-41fb-9038-422cf55d1111
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "equipmentId": "c3b5af91-61f8-4bc0-bd88-d099d3e90002",
  "plannedStartDateTime": "2026-05-21T08:00:00Z",
  "plannedEndDateTime": "2026-05-23T18:00:00Z",
  "workCenterId": "12a8b1ce-3aaf-4f55-8ac8-f8cf5d86c222",
  "justification": "Updated due to equipment replacement for the same work scope."
}
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий `result wrapper`.

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | value | object | object | — | backend aggregation | Обновленный booking item |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | object[] | `[]` | backend |  |

### Структура `value`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | Bookings.id |  |
| 2 | Идентификатор заявки | requestId | uuid | UUID v4 | — | BookingRequests.id |  |
| 3 | Идентификатор техники | equipmentId | uuid | UUID v4 | — | backend composition from BookingRequests + Bookings + Equipments + EquipmentBookingAuthorizations |  |
| 4 | Текущий статус | status | string | string | — | Bookings + ref_booking_status |  |
| 5 | Плановая дата и время начала | plannedStartDateTime | datetime | ISO 8601 | — | Bookings.plannedStartDateTime |  |
| 6 | Плановая дата и время окончания | plannedEndDateTime | datetime | ISO 8601 | — | Bookings.plannedEndDateTime |  |
| 7 | Обоснование | justification | string | string | — | Bookings.justification |  |
| 8 | Признак, что для item обязателен justification | requiresJustification | bool | boolean | — | backend business rule from Equipments + ref_ownership_type + ref_share_type | `true`, если для итоговой техники `ownershipType = LongTermRented` или `shareType IN (Assigned, SharedWithConditions)` |
| 9 | Признак наличия обязательного justification | hasRequiredJustification | bool | boolean | — | backend business rule | `false`, если для item обязателен `justification`, но он еще не заполнен |

## 10. Пример ответа

```json
{
  "value": {
    "id": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
    "requestId": "c777f75f-029d-4d8f-8c69-e74a1d280001",
    "equipmentId": "c3b5af91-61f8-4bc0-bd88-d099d3e90002",
    "status": "Draft",
    "plannedStartDateTime": "2026-05-21T08:00:00Z",
    "plannedEndDateTime": "2026-05-23T18:00:00Z",
    "justification": "Updated due to equipment replacement for the same work scope.",
    "requiresJustification": true,
    "hasRequiredJustification": true,
  },
  "isSuccess": true,
  "errors": []
}
```

## Замечания

1. Метод обновляет существующий booking item, а не создает новый.
2. Замена техники выполняется тем же методом через передачу нового `equipmentId`.
3. Для валидации используются итоговые значения item после применения patch.
4. Под недоступностью в базовом сценарии понимаются hard-ограничения по состоянию техники и другим обязательным бизнес-правилам, не связанным с competing bookings.
5. Пересечения с другими активными бронями должны рассчитываться отдельно как booking conflicts и не блокируют обновление item.
6. Inline-поле `justification` на странице draft может сохраняться этим методом автоматически, без отдельной кнопки `Сохранить`.
