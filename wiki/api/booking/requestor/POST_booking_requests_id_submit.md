# POST /booking-requests/{id}/submit

**Created:** 2026-05-14  
**Last updated:** 2026-05-19  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Отправить заявку на согласование |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Requestor UI` |
| Endpoint URL | `/api/booking/v1/booking-requests/{id}/submit` |
| Метод запроса | `POST` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Переводит заявку и ее item-ы из draft в submitted.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-023 | Requestor can create a Request | Confirmed | BRD v13 | Метод завершает создание заявки |
| TCO Booking Tool | FR-062 | Booking -> Submitted once Request submitted | Confirmed | BRD v13 | Все booking item-ы переводятся в `Submitted` |
| TCO Booking Tool | FR-043 | Approver assigned automatically based on fleet ownership | Confirmed | BRD v13 | При submit начинается approval flow |
| TCO Booking Tool | FR-NEW-71 | Justification mandatory for Long-term rented item | Confirmed | BRD v13 | Проверка перед отправкой |

---

## 3. Описание логики работы метода

1. Проверить, что заявка существует и находится в статусе `Draft`.
2. Проверить, что в заявке есть хотя бы один booking item.
3. Для каждого item выполнить валидации на основании уже сохраненных в `Bookings` данных и связанных атрибутов техники:
   - проверить, что `plannedStartDateTime` и `plannedEndDateTime` заполнены;
   - проверить, что `plannedStartDateTime < plannedEndDateTime`;
   - проверить, что период не нарушает ограничения `BOOKING_HORIZON_DAYS` и `MAX_BOOKING_DURATION_DAYS`;
   - вычислить `requiresJustification` по правилу: `ownershipType = LongTermRented` или `shareType IN (Assigned, SharedWithConditions)`;
   - если `requiresJustification = true`, проверить, что `justification` заполнен и не является пустой / whitespace-only строкой;
   - если `requiresJustification = true` и `justification` не заполнен, считать item незавершенным и отклонять submit;
   - если техника `Assigned`, проверить наличие активной записи в `EquipmentBookingAuthorizations` для текущего пользователя и периода.
4. Если хотя бы один item не прошел перечисленные проверки, вернуть `VALIDATION_ERROR` и не переводить заявку в `Submitted`.
5. Обновить `BookingRequests.status = Submitted` и создать запись в `BookingRequestStatuses`.
6. Для каждого item обновить `Bookings.status = Submitted` и создать запись в `BookingStatuses`.
7. Вернуть обновленную заявку.

Сущности, участвующие в методе:
- читаются: `BookingRequests`, `Bookings`, `Equipments`, `EquipmentBookingAuthorizations`
- изменяются: `BookingRequests`, `Bookings`, `BookingRequestStatuses`, `BookingStatuses`
- транзакционность: обязательна

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Requestor` | Отправка собственной заявки |
| `ServiceWorkProcessor` | Отправка SWR |

---

## 5. Настройки системы, используемые в методе

| Наименование | Код | Тип значения | Описание | Значение по умолчанию |
|---|---|---|---|---|
| Горизонт бронирования | `BOOKING_HORIZON_DAYS` | `int` | Проверка дат | Конфигурируется Admin |
| Максимальная длительность брони | `MAX_BOOKING_DURATION_DAYS` | `int` | Проверка длительности | Конфигурируется Admin |

---

## 6. Ошибки, возвращаемые методом

| Код | Описание ошибки |
|---|---|
| `UNAUTHORIZED` | Пользователь не авторизован |
| `FORBIDDEN` | Нет доступа к заявке |
| `NOT_FOUND` | Заявка не найдена |
| `REQUEST_NOT_SUBMITTABLE` | Заявка не в статусе `Draft` или пуста |
| `VALIDATION_ERROR` | Не пройдены бизнес-валидации item-ов |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор заявки | `id` | `uuid` | `+` | Должен существовать | — | Path param | |

У метода нет request body. Метод не принимает `justification` или другие незасейвленные item-level поля в теле запроса и валидирует только уже сохраненное состояние draft-заявки.

Frontend перед вызовом `POST /booking-requests/{id}/submit` должен завершить autosave всех несохраненных изменений booking item-ов.

---

## 8. Пример запроса

```http
POST /api/booking/v1/booking-requests/c777f75f-029d-4d8f-8c69-e74a1d280001/submit
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
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | BookingRequests.id |  |
| 2 | Номер заявки | requestNumber | string | string | — | BookingRequests.requestNumber |  |
| 3 | Текущий статус | status | string | string | — | BookingRequests + BookingRequestStatuses |  |
| 4 | Список броней | bookings | array<object> | object[] | — | backend composition from BookingRequests + Bookings + Equipments + EquipmentBookingAuthorizations + BookingRequestStatuses + BookingStatuses | Коллекция объектов |

### Структура `value.bookings[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | BookingRequests.id |  |
| 2 | Текущий статус | status | string | string | — | BookingRequests + BookingRequestStatuses |  |

## 10. Пример ответа

```json
{
  "value": {
    "id": "c777f75f-029d-4d8f-8c69-e74a1d280001",
    "requestNumber": "REQ-2026-00015",
    "status": "Submitted",
    "bookings": [
      {
        "id": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
        "status": "Submitted"
      }
    ]
  },
  "isSuccess": true,
  "errors": []
}
```
