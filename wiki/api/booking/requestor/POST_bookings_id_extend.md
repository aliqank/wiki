# POST /bookings/{id}/extend

**Created:** 2026-05-14  
**Last updated:** 2026-06-02  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Запросить изменение `plannedEndDateTime` активной брони |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Requestor UI` |
| Endpoint URL | `/api/booking/v1/bookings/{id}/extend` |
| Метод запроса | `POST` |
| Связанные use cases | [`UC-REQ-06 - Изменение плановой даты и времени окончания брони requestor-ом`](../../../requirements/usecases/Requestor/UC-REQ-06%20-%20Изменение%20плановой%20даты%20и%20времени%20окончания%20брони%20requestor-ом.md) |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Обрабатывает запрос Requestor на изменение `plannedEndDateTime` с разным поведением для pre-start и `InProgress` сценариев.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-061 | Requestor can extend confirmed booking | Confirmed | BRD v13 | Базовое покрытие change-end-date сценария для текущего scope |
| TCO Booking Tool | FR-067 | Booking -> Submitted once extended | Confirmed | BRD v13 | Для pre-start сценария после изменения срок снова требует решения FO |
| TCO Booking Tool | FR-081 | System notifies FO when Requestor extends booking | Confirmed | BRD v13 | Уведомление FO обязательно |

---

## 3. Описание логики работы метода

1. Проверить существование брони и права доступа.
2. Разрешить изменение `plannedEndDateTime` только для активной брони в статусе `Submitted`, `Confirmed` или `InProgress`.
3. Провалидировать новый `newPlannedEndDateTime`, [hard availability restrictions](../../../glossary/Glossary.md#hard-availability-restriction) и [booking conflict context](../../../glossary/Glossary.md#booking-conflict-context) на новый период.
4. Если текущий статус равен `Submitted` или `Confirmed`, обновить `Bookings.plannedEndDateTime`, перевести бронь в `Submitted` и создать запись в `BookingStatuses`.
5. Если текущий статус равен `InProgress`, не менять статус брони, создать отдельную запись [Booking Extension Approval](../../../glossary/Glossary.md#booking-extension-approval) в `BookingApprovals` для запроса на изменение срока и не применять новое `plannedEndDateTime` до решения Fleet Owner.
6. После положительного решения Fleet Owner:
   - для pre-start сценария вернуть бронь в `Confirmed`;
   - для `InProgress` сценария сохранить статус `InProgress` и применить новое `plannedEndDateTime`.
7. Уведомить Fleet Owner о запросе на изменение срока брони.
8. Вернуть обновленную или pending-to-approval бронь в зависимости от текущего сценария.

Сущности, участвующие в методе:
- читаются: [`Bookings`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#22-bookings)
- изменяются: [`Bookings`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#22-bookings), [`BookingStatuses`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#24-bookingstatuses), [`BookingApprovals`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#23-bookingapprovals)
- транзакционность: обязательна

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Requestor` | Изменение `plannedEndDateTime` собственной брони |

---

## 5. Настройки системы, используемые в методе

| Наименование | Код | Тип значения | Описание | Значение по умолчанию |
|---|---|---|---|---|
| Максимальная длительность брони | `MAX_BOOKING_DURATION_DAYS` | `int` | Проверка нового периода | Конфигурируется Admin |

---

## 6. Ошибки, возвращаемые методом

| Код | Описание ошибки |
|---|---|
| `UNAUTHORIZED` | Пользователь не авторизован |
| `FORBIDDEN` | Нет доступа к брони |
| `NOT_FOUND` | Бронь не найдена |
| `BOOKING_NOT_EXTENDABLE` | Бронь нельзя перевести в extension/change-end-date flow |
| `EQUIPMENT_NOT_AVAILABLE` | Техника недоступна по [hard availability restrictions](../../../glossary/Glossary.md#hard-availability-restriction) на новый период; competing bookings сами по себе не вызывают эту ошибку |
| `VALIDATION_ERROR` | Невалидный `newPlannedEndDateTime` |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор брони | `id` | `uuid` | `+` | Должен существовать | — | Path param | |
| 2 | Новая плановая дата/время окончания | `newPlannedEndDateTime` | `datetime` | `+` | Должна быть валидна для текущего lifecycle-сценария брони | — | Request body | Для `InProgress` применяется только после approval |
| 3 | Комментарий к изменению срока | `comment` | `string` | `-` | — | — | Request body | |

---

## 8. Пример запроса

```http
POST /api/booking/v1/bookings/8c4c8b6d-7bc0-41fb-9038-422cf55d1111/extend
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "newPlannedEndDateTime": "2026-05-23T18:00:00Z",
  "comment": "Need one additional shift for trench completion."
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
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | backend composition from Bookings + BookingStatuses |  |
| 2 | Текущий статус | status | string | string | — | backend composition from Bookings + BookingStatuses | Для pre-start сценария может стать `Submitted`, для `InProgress` сохраняется `InProgress` |
| 3 | Плановая дата и время начала | plannedStartDateTime | datetime | ISO 8601 | — | backend composition from Bookings + BookingStatuses |  |
| 4 | Плановая дата и время окончания | plannedEndDateTime | datetime | ISO 8601 | — | backend composition from Bookings + BookingStatuses |  |

## 10. Пример ответа

```json
{
  "value": {
    "id": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
    "status": "Submitted",
    "plannedStartDateTime": "2026-05-20T08:00:00Z",
    "plannedEndDateTime": "2026-05-23T18:00:00Z"
  },
  "isSuccess": true,
  "errors": []
}
```
