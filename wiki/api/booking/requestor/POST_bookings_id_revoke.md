# POST /bookings/{id}/revoke

**Created:** 2026-05-14  
**Last updated:** 2026-06-02  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Отозвать бронь до решения Fleet Owner |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Requestor UI` |
| Endpoint URL | `/api/booking/v1/bookings/{id}/revoke` |
| Метод запроса | `POST` |
| Связанные use cases | [`UC-REQ-04 - Отзыв брони requestor-ом`](../../../requirements/usecases/Requestor/UC-REQ-04%20-%20Отзыв%20брони%20requestor-ом.md), [`UC-REQ-05 - Отзыв submitted-заявки requestor-ом`](../../../requirements/usecases/Requestor/UC-REQ-05%20-%20Отзыв%20submitted-заявки%20requestor-ом.md) |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Закрывает бронь с причиной `Revoked`.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-058 | Requestor/SWP can revoke booking if not yet processed by FO | Confirmed | BRD v13 | Прямое покрытие |
| TCO Booking Tool | FR-068 | Booking -> Closed with closure reason Revoked once revoked | Confirmed | BRD v13 | Обновление статуса |

---

## 3. Описание логики работы метода

1. Проверить права доступа к booking item.
2. Разрешить отзыв только если текущий статус `Submitted`.
3. Обновить `Bookings.status = Closed`, `closureReason = Revoked`.
4. Создать запись в `BookingStatuses` с `status = Closed` и `closureReason = Revoked`.
5. Пересчитать агрегированный статус заявки.

Сущности, участвующие в методе:
- читаются: `Bookings`, `BookingRequests`
- изменяются: `Bookings`, `BookingStatuses`, `BookingRequests`, `BookingRequestStatuses`
- транзакционность: обязательна

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Requestor` | Отзыв собственной брони |
| `ServiceWorkProcessor` | Отзыв брони в SWR |

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
| `NOT_FOUND` | Бронь не найдена |
| `BOOKING_NOT_REVOCABLE` | Бронь уже обработана FO |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор брони | `id` | `uuid` | `+` | Должен существовать | — | Path param | |

У метода нет request body.

---

## 8. Пример запроса

```http
POST /api/booking/v1/bookings/8c4c8b6d-7bc0-41fb-9038-422cf55d1111/revoke
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
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | backend composition from Bookings + BookingRequests + BookingStatuses + BookingRequestStatuses |  |
| 2 | Текущий статус | status | string | string | — | backend composition from Bookings + BookingRequests + BookingStatuses + BookingRequestStatuses |  |

## 10. Пример ответа

```json
{
  "value": {
    "id": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
    "status": "Closed"
  },
  "isSuccess": true,
  "errors": []
}
```
