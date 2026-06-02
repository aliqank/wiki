# GET /reference/approval-request-statuses

**Created:** 2026-05-28  
**Last updated:** 2026-06-02  
**Автор документов:** OpenCode

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить справочник статусов заявки для фильтра Fleet Owner view `Approvals -> Requests` |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Fleet Owner UI` |
| Endpoint URL | `/api/booking/v1/reference/approval-request-statuses` |
| Метод запроса | `GET` |
| Связанные use cases | [`UC-FO-01 - Просмотр списка заявок Fleet Owner`](../../../requirements/usecases/Fleet%20Owner/UC-FO-01%20-%20Просмотр%20списка%20заявок%20Fleet%20Owner.md) |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый reference API для `UC-FO-03`. Используется для загрузки списка значений фильтра `status` во view Fleet Owner `Approvals -> Requests`.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-092 | Approver (FO) can view all pending and completed approvals | Confirmed | BRD v13 | FO view использует собственный набор допустимых request statuses |
| TCO Booking Tool | FR-094 | System generates reports on Requests/Bookings/Equipment with filters | Confirmed | BRD v13 | Значения фильтра `status` должны загружаться отдельным reference API |
| TCO Booking Tool | BRD-U-001 | Request terminal status semantics | Confirmed | BRD Updates | Для active approval list используются только `Submitted` и `InProgress` |

---

## 3. Описание логики работы метода

1. Выбрать записи из `ref_booking_request_status`.
2. Вернуть только значения, допустимые для фильтра `GET /approvals/requests`: `Submitted`, `InProgress`.
3. Не возвращать `Draft` и `Closed`, так как они не должны использоваться во Fleet Owner request list.
4. Отсортировать записи по `sortOrder`.

Сущности, участвующие в методе:
- читаются: [`ref_booking_request_status`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#reference-tables-instead-of-enums)
- изменения не выполняются

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `FleetOwner` | Просмотр и фильтрация заявок в approval request view |

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
| `FORBIDDEN` | Нет необходимой роли |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`

---

## 7. Параметры метода

Метод не принимает параметров.

---

## 8. Пример запроса

```http
GET /api/booking/v1/reference/approval-request-statuses
Authorization: Bearer <token>
Content-Type: application/json
```

---

## 9. Возвращаемые данные

Возвращаемые данные обернуты в общий `result wrapper`.

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | value | array<object> | object[] | — | backend aggregation |  |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | object[] | `[]` | backend |  |

### Структура `value[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор значения | id | uuid | UUID v4 | — | ref_booking_request_status.id |  |
| 2 | Код статуса | code | string | string | — | ref_booking_request_status.code | `Submitted / InProgress` |
| 3 | Локализованное наименование | name | object | object | — | ref_booking_request_status |  |
| 4 | Порядок сортировки | sortOrder | int | integer | — | ref_booking_request_status.sortOrder |  |

### Структура `value[].name`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string | string | — | ref_booking_request_status.nameEn |  |
| 2 | Значение на русском языке | Ru | string | string | — | ref_booking_request_status.nameRu |  |
| 3 | Значение на казахском языке | Kz | string | string | — | ref_booking_request_status.nameKz |  |

## 10. Пример ответа

```json
{
  "value": [
    {
      "id": "8f6af6c0-5f09-4e61-a8ea-4f7f09e50002",
      "code": "Submitted",
      "name": {
        "En": "Submitted",
        "Ru": "Отправлена",
        "Kz": "Жiберiлген"
      },
      "sortOrder": 20
    },
    {
      "id": "8f6af6c0-5f09-4e61-a8ea-4f7f09e50003",
      "code": "InProgress",
      "name": {
        "En": "In Progress",
        "Ru": "В работе",
        "Kz": "Жұмыста"
      },
      "sortOrder": 30
    }
  ],
  "isSuccess": true,
  "errors": []
}
```
