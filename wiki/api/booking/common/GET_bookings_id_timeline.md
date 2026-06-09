# GET /bookings/{id}/timeline

**Created:** 2026-05-14  
**Last updated:** 2026-06-08  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить агрегированный timeline по брони |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Common` |
| Endpoint URL | `/api/booking/v1/bookings/{id}/timeline` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Уточненная dev-ready версия метода. Удобная агрегированная проекция audit trail для UI.

Назначение текущей версии:
- оставить timeline отдельной проекцией по отношению к strict status history;
- агрегировать lifecycle status events, approval decisions и execution milestones;
- дать frontend-у единый event stream для detail view / audit panel / activity feed.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-NEW-37 | Audit trail and history on demand | Confirmed | BRD v13 | Прямое покрытие |
| TCO Booking Tool | FR-077 | System notifies FO when request submitted | Confirmed | BRD v13 | Timeline должен отражать submit lifecycle event |
| TCO Booking Tool | FR-079 | System notifies Requestor/SWP of FO decision | Confirmed | BRD v13 | Timeline должен отражать approval decisions |
| TCO Booking Tool | FR-NEW-75 | System notifies FleetOwners' Supervisor when Long-term rented booking reaches Confirmed by FO | Confirmed | BRD v13 | Timeline должен показывать multi-step approval chain |
| TCO Booking Tool | FR-NEW-76 | System notifies Requestor and FO when FleetOwners' Supervisor makes a decision | Confirmed | BRD v13 | Timeline должен показывать supervisor decision event |

---

## 3. Описание логики работы метода

1. Проверить доступ к брони.
2. Собрать lifecycle history из `BookingStatuses`.
3. Собрать approval decision history из `BookingApprovals`.
4. Подтянуть actor-ов из `Users` для lifecycle и approval records.
5. Подтянуть справочные данные из `ref_booking_status`, `ref_booking_closure_reason`, `ref_booking_approval_type`, `ref_booking_approval_status`.
6. Построить единый event stream.
7. Добавить derived execution milestone events из `Bookings.actualStartDateTime` и `Bookings.actualEndDateTime`, только если они не дублируют already existing lifecycle event с тем же timestamp.
8. Отсортировать по `occurredAt ASC`, затем по `eventOrder ASC`.
9. Вернуть агрегированный timeline.

Сущности:
- читаются: [`Bookings`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#22-bookings), [`BookingStatuses`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#24-bookingstatuses), [`BookingApprovals`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#23-bookingapprovals), [`Users`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#19-users)

Правила композиции timeline:
- `StatusChanged` строится только из `BookingStatuses`;
- `ApprovalRecorded` строится только из `BookingApprovals`;
- `ExecutionMilestone` строится как derived projection из factual booking timestamps;
- timeline не заменяет source-of-truth таблицы и не должен использоваться как единственный источник lifecycle semantics;
- каждый timeline item относится ровно к одному типу внутренней backend-композиции: status event, approval event или execution milestone.

Правила derived milestones:
- `ExecutionStarted` добавляется, если `actualStartDateTime` заполнен и нет lifecycle event `InProgress` с идентичным timestamp;
- `ExecutionFinished` добавляется, если `actualEndDateTime` заполнен и нет lifecycle event `Closed` с идентичным timestamp.

Рекомендуемый `eventOrder` для одинакового `occurredAt`:
- `10` = `ApprovalRecorded`;
- `20` = `StatusChanged`;
- `30` = `ExecutionMilestone`.

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Requestor` | Timeline собственной брони |
| `ServiceWorkProcessor` | Timeline броней по доступным Service Work Request |
| `FleetOwner` | Timeline броней своих флотов |
| `FleetOwnersSupervisor` | Timeline long-term rented броней |
| `Admin` | Полный доступ |

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
| `FORBIDDEN` | Нет доступа к timeline |
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
GET /api/booking/v1/bookings/8c4c8b6d-7bc0-41fb-9038-422cf55d1111/timeline
Authorization: Bearer <token>
Content-Type: application/json
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий [`result wrapper`](../../common/Result%20Wrapper.md).

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | value | array<object> | object[] | — | backend aggregation |  |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | object[] | `[]` | backend |  |

### Структура `value[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Дата и время события | occurredAt | datetime | ISO 8601 | — | backend timeline aggregation | Основное поле сортировки |
| 2 | Описание события | description | string | null | `null` | backend composition | Детализация для expanded UI |
| 3 | Actor события | actor | object | object | — | backend composition from audit fields + Users |  |
| 4 | Payload статуса | status | object | null | `null` | status-based events only | Для `StatusChanged` |
| 5 | Payload approval step | approval | object | null | `null` | approval-based events only | Для `ApprovalRecorded` |
| 6 | Комментарий | comment | string | null | `null` | source record comment / backend-generated milestone text |  |

### Структура `value[].actor`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор actor-а | userId | uuid | null | `null` | audit source field |  |
| 2 | Отображаемое имя actor-а | displayName | string | string | — | Users.fullName / backend fallback |  |
| 3 | Email actor-а | email | string | null | `null` | Users.email |  |
| 4 | Тип actor-а | actorType | string | string | — | backend composition | `User`, `System`, `Unknown` |

### Структура `value[].status`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Код lifecycle статуса | code | string | string | — | ref_booking_status | Только для `StatusChanged` |
| 2 | Подпись lifecycle статуса | label | string | string | — | ref_booking_status |  |
| 3 | Код terminal closure reason | closureReason | string | null | `null` | ref_booking_closure_reason | Только для `Closed` |
| 4 | Подпись terminal closure reason | closureReasonLabel | string | null | `null` | ref_booking_closure_reason | Только для `Closed` |

### Структура `value[].approval`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Код approval type | type | string | string | — | ref_booking_approval_type | Например: `FoApproval`, `SupervisorApproval` |
| 2 | Подпись approval type | typeLabel | string | string | — | ref_booking_approval_type |  |
| 3 | Код результата решения | status | string | string | — | ref_booking_approval_status | Например: `Approved`, `Declined` |
| 4 | Подпись результата решения | statusLabel | string | string | — | ref_booking_approval_status |  |
| 5 | Порядок approval step | order | int | integer | — | BookingApprovals.approvalOrder | Позиция шага в approval chain |

## 10. Пример ответа

```json
{
  "value": [
    {
      "occurredAt": "2026-05-14T10:00:00Z",
      "description": "Booking entered the approval flow.",
      "actor": {
        "userId": "8f83f79c-3d25-4f07-a17b-9dc4b9f25001",
        "displayName": "John Smith",
        "email": "john.smith@tco.example",
        "actorType": "User"
      },
      "status": {
        "code": "Submitted",
        "label": "Submitted",
        "closureReason": null,
        "closureReasonLabel": null
      },
      "approval": null,
      "comment": null
    },
    {
      "occurredAt": "2026-05-14T11:20:00Z",
      "description": "Approval step 1 was completed with positive decision.",
      "actor": {
        "userId": "d61f5b36-c1c1-40de-a37a-11d6d1898002",
        "displayName": "Aigerim Sarsenova",
        "email": "aigerim.sarsenova@tco.example",
        "actorType": "User"
      },
      "status": null,
      "approval": {
        "type": "FoApproval",
        "typeLabel": "Fleet Owner approval",
        "status": "Approved",
        "statusLabel": "Approved",
        "order": 1
      },
      "comment": "Approved for planned work window."
    },
    {
      "occurredAt": "2026-05-14T11:20:00Z",
      "description": "All mandatory approvals were completed and booking became ready for execution.",
      "actor": {
        "userId": "d61f5b36-c1c1-40de-a37a-11d6d1898002",
        "displayName": "Aigerim Sarsenova",
        "email": "aigerim.sarsenova@tco.example",
        "actorType": "User"
      },
      "status": {
        "code": "Confirmed",
        "label": "Confirmed",
        "closureReason": null,
        "closureReasonLabel": null
      },
      "approval": null,
      "comment": null
    },
    {
      "occurredAt": "2026-05-20T08:05:00Z",
      "description": "Actual booking execution start was recorded.",
      "actor": {
        "userId": null,
        "displayName": "System",
        "email": null,
        "actorType": "System"
      },
      "status": null,
      "approval": null,
      "comment": null
    }
  ],
  "isSuccess": true,
  "errors": []
}
```

---

## 11. UI Interpretation Notes

- Использовать timeline для vertical activity feed, expandable audit panel или event stream в details view.
- Для строгой таблицы status history frontend должен использовать `GET /bookings/{id}/status-history`.
- Lifecycle transitions нельзя infer-ить только из approval events.
