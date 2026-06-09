# GET /bookings/{id}/status-history

**Created:** 2026-05-14  
**Last updated:** 2026-06-05  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить историю статусов брони |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Common` |
| Endpoint URL | `/api/booking/v1/bookings/{id}/status-history` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Уточненная dev-ready версия метода. Используется для audit trail на уровне booking item.

Назначение текущей версии:
- оставить метод строго в рамках lifecycle history;
- добавить данные о том, кто изменил статус;
- добавить terminal `closureReason` для `Closed` записей;
- сделать ответ пригодным для прямого отображения в UI-таблице истории статусов.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-NEW-37 | Audit trail and history on demand | Confirmed | BRD v13 | Прямое покрытие |
| TCO Booking Tool | FR-025 | [`Requestor`](../../../requirements/Roles and Access Model.md) can view request details and status | Confirmed | BRD v13 | История статусов поддерживает transparency на details view |
| TCO Booking Tool | FR-042 | Status tracking for bookings | Confirmed | BRD v13 | Метод возвращает lifecycle transitions брони |

---

## 3. Описание логики работы метода

1. Проверить существование брони и права доступа по роли пользователя.
2. Выбрать записи из `BookingStatuses` по `bookingId`.
3. Подтянуть `ref_booking_status` для machine-readable и UI-readable представления статуса.
4. Left join `ref_booking_closure_reason` для terminal `Closed` записей.
5. Left join `Users` по `BookingStatuses.createdBy` для возврата информации об actor-е.
6. Отсортировать по `BookingStatuses.createdAt ASC`, затем по `BookingStatuses.id ASC` для детерминированного порядка.
7. Вернуть одну запись на каждый фактически записанный lifecycle transition.

Сущности:
- читаются: [`Bookings`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#22-bookings), [`BookingStatuses`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#24-bookingstatuses), [`Users`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#19-users)

Правила метода:
- метод возвращает только lifecycle history и не должен включать approval decisions из `BookingApprovals`;
- повторяющиеся статусы допустимы и должны возвращаться как отдельные записи, если они были записаны в `BookingStatuses`;
- если `status != Closed`, поля `closureReason` и `closureReasonLabel` должны быть `null`;
- поле `changedAt` является business-facing alias для `BookingStatuses.createdAt`;
- поле `changedBy` должно строиться из audit actor `BookingStatuses.createdBy`;
- если actor не резолвится в `Users`, backend должен вернуть fallback-представление, а не скрывать запись истории.

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| [`Requestor`](../../../requirements/Roles and Access Model.md) | История собственной брони |
| [`ServiceWorkProcessor`](../../../requirements/Roles and Access Model.md) | История броней по доступным Service Work Request |
| [`FleetOwner`](../../../requirements/Roles and Access Model.md) | История брони своих флотов |
| [`FleetOwnersSupervisor`](../../../requirements/Roles and Access Model.md) | История long-term rented броней |
| [`Admin`](../../../requirements/Roles and Access Model.md) | Полный доступ |

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
| `FORBIDDEN` | Нет доступа к истории брони |
| `NOT_FOUND` | Бронь не найдена |
| `ACTOR_RESOLUTION_FAILED` | Невалидный или нерезолвящийся actor в audit field; метод должен отдать fallback actor representation |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`

Примечание:
- `ACTOR_RESOLUTION_FAILED` не должен приводить к blocking response error; ожидаемое поведение — internal log + fallback actor representation.

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор брони | `id` | `uuid` | `+` | Должен существовать | — | Path param | |

---

## 8. Пример запроса

```http
GET /api/booking/v1/bookings/8c4c8b6d-7bc0-41fb-9038-422cf55d1111/status-history
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
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | BookingStatuses.id |  |
| 2 | Lifecycle status code | status | string | string | — | BookingStatuses + ref_booking_status | Стабильное machine-readable значение |
| 3 | Lifecycle status label | statusLabel | string | string | — | ref_booking_status | UI-подпись статуса |
| 4 | Код terminal closure reason | closureReason | string | null | `null` | BookingStatuses + ref_booking_closure_reason | Возвращается только для `Closed` |
| 5 | Подпись terminal closure reason | closureReasonLabel | string | null | `null` | ref_booking_closure_reason | Возвращается только для `Closed` |
| 6 | Дата и время изменения статуса | changedAt | datetime | ISO 8601 | — | BookingStatuses.createdAt | Business-facing alias для audit timestamp |
| 7 | Комментарий | comment | string | null | `null` | BookingStatuses.comment |  |
| 8 | Кто изменил статус | changedBy | object | object | — | backend composition from BookingStatuses + Users |  |

### Структура `value[].changedBy`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор actor-а | userId | uuid | UUID v4 | — | BookingStatuses.createdBy | Audit actor ID |
| 2 | Отображаемое имя actor-а | displayName | string | string | — | Users.fullName / backend fallback | Например: `John Smith`, `System`, `Unknown user` |
| 3 | Email actor-а | email | string | null | `null` | Users.email | Для system/unknown actor может быть `null` |
| 4 | Тип actor-а | actorType | string | string | — | backend composition | `User`, `System`, `Unknown` |

## 10. Пример ответа

```json
{
  "value": [
    {
      "id": "11111111-2222-3333-4444-555555550001",
      "status": "Draft",
      "statusLabel": "Draft",
      "closureReason": null,
      "closureReasonLabel": null,
      "changedAt": "2026-05-14T09:15:00Z",
      "comment": null,
      "changedBy": {
        "userId": "8f83f79c-3d25-4f07-a17b-9dc4b9f25001",
        "displayName": "John Smith",
        "email": "john.smith@tco.example",
        "actorType": "User"
      }
    },
    {
      "id": "11111111-2222-3333-4444-555555550002",
      "status": "Submitted",
      "statusLabel": "Submitted",
      "closureReason": null,
      "closureReasonLabel": null,
      "changedAt": "2026-05-14T10:00:00Z",
      "comment": "Submitted by Requestor",
      "changedBy": {
        "userId": "8f83f79c-3d25-4f07-a17b-9dc4b9f25001",
        "displayName": "John Smith",
        "email": "john.smith@tco.example",
        "actorType": "User"
      }
    },
    {
      "id": "11111111-2222-3333-4444-555555550003",
      "status": "Closed",
      "statusLabel": "Closed",
      "closureReason": "Completed",
      "closureReasonLabel": "Completed",
      "changedAt": "2026-05-20T17:45:00Z",
      "comment": "Booking closed manually by Fleet Owner",
      "changedBy": {
        "userId": "d61f5b36-c1c1-40de-a37a-11d6d1898002",
        "displayName": "Aigerim Sarsenova",
        "email": "aigerim.sarsenova@tco.example",
        "actorType": "User"
      }
    }
  ],
  "isSuccess": true,
  "errors": []
}
```

---

## 11. UI Interpretation Notes

- Основные колонки для таблицы: `changedAt`, `statusLabel`, `changedBy.displayName`.
- `closureReasonLabel` показывается только если `status = Closed`.
- `comment` отображается как optional detail / tooltip и не должен быть обязательной колонкой.
- Для строгой таблицы истории статусов frontend должен использовать именно этот endpoint, а не `timeline`.
