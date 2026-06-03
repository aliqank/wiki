# POST /bookings/{id}/feedback

**Created:** 2026-05-14  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Оставить отзыв по технике в рамках конкретной брони |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Requestor UI` |
| Endpoint URL | `/api/booking/v1/bookings/{id}/feedback` |
| Метод запроса | `POST` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Создает запись в `EquipmentFeedbacks`.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-022 | Requestor can submit feedback on equipment with confirmed booking | Confirmed | BRD v13 | Прямое покрытие для текущего scope |

---

## 3. Описание логики работы метода

1. Проверить существование брони и права доступа.
2. Разрешить создание отзыва только для брони, достигшей `Confirmed` и выше.
3. Проверить, что у брони есть `equipmentId`.
4. Создать запись в `EquipmentFeedbacks`.
5. Вернуть созданный отзыв.

Сущности, участвующие в методе:
- читаются: [`Bookings`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#22-bookings)
- изменяются: [`EquipmentFeedbacks`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#17-equipmentfeedbacks--equipmentbookingauthorizations)

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Requestor` | Оставление отзыва по своей брони |

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
| `BOOKING_FEEDBACK_NOT_ALLOWED` | Отзыв пока нельзя оставить |
| `VALIDATION_ERROR` | Текст отзыва пустой |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор брони | `id` | `uuid` | `+` | Должен существовать | — | Path param | |
| 2 | Текст отзыва | `feedback` | `string` | `+` | Непустая строка | — | Request body | |

---

## 8. Пример запроса

```http
POST /api/booking/v1/bookings/8c4c8b6d-7bc0-41fb-9038-422cf55d1111/feedback
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "feedback": "Equipment was delivered on time and worked without issues."
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
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | backend composition from Bookings + EquipmentFeedbacks |  |
| 2 | Идентификатор брони | bookingId | uuid | UUID v4 | — | backend composition from Bookings + EquipmentFeedbacks |  |
| 3 | Идентификатор техники | equipmentId | uuid | UUID v4 | — | backend composition from Bookings + EquipmentFeedbacks |  |
| 4 | Текст отзыва | feedback | string | string | — | backend composition from Bookings + EquipmentFeedbacks |  |
| 5 | Дата и время создания | createdAt | datetime | ISO 8601 | — | backend composition from Bookings + EquipmentFeedbacks |  |

## 10. Пример ответа

```json
{
  "value": {
    "id": "3e7e8f9a-a8e5-48d3-8ca2-a0cce5d40001",
    "bookingId": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
    "equipmentId": "c3b5af91-61f8-4bc0-bd88-d099d3e90001",
    "feedback": "Equipment was delivered on time and worked without issues.",
    "createdAt": "2026-05-22T17:30:00Z"
  },
  "isSuccess": true,
  "errors": []
}
```
