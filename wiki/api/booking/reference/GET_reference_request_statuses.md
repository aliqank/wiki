# GET /reference/request-statuses

**Created:** 2026-05-28  
**Last updated:** 2026-06-02  
**Автор документов:** OpenCode

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить справочник статусов заявки для фильтра страницы `Мои заявки` |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Requestor UI` |
| Endpoint URL | `/api/booking/v1/reference/request-statuses` |
| Метод запроса | `GET` |
| Связанные use cases | [`UC-REQ-01 - Просмотр моих заявок`](../../../requirements/usecases/Requestor/UC-REQ-01%20-%20Просмотр%20моих%20заявок.md) |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый reference API для `UC-REQ-01`. Используется для загрузки списка значений фильтра `status` на странице `Мои заявки`.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-025 | Requestor can view request details and status | Confirmed | BRD v13 | Список заявок должен использовать согласованные request statuses |
| TCO Booking Tool | FR-091 | Requestor/SWP can search and filter own requests | Confirmed | BRD v13 | Значения фильтра `status` должны загружаться отдельным reference API |
| TCO Booking Tool | BRD-U-001 | Request terminal status semantics | Confirmed | BRD Updates | Для active list используются только non-terminal request statuses |

---

## 3. Описание логики работы метода

1. Выбрать записи из `ref_booking_request_status`.
2. Вернуть только значения, допустимые для фильтра `GET /booking-requests/my`: `Draft`, `Submitted`, `InProgress`.
3. Не возвращать `Closed`, так как endpoint `GET /booking-requests/my` показывает только незавершенные заявки.
4. Отсортировать записи по `sortOrder`.

Сущности, участвующие в методе:
- читаются: [`ref_booking_request_status`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#reference-tables-instead-of-enums)
- изменения не выполняются

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Requestor` | Просмотр и фильтрация собственных заявок |
| `ServiceWorkProcessor` | Просмотр и фильтрация собственных SWR |

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
GET /api/booking/v1/reference/request-statuses
Authorization: Bearer <token>
Content-Type: application/json
```

---

## 9. Возвращаемые данные

Возвращаемые данные обернуты в общий [`result wrapper`](../../common/Result%20Wrapper.md).

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
| 2 | Код статуса | code | string | string | — | ref_booking_request_status.code | `Draft / Submitted / InProgress` |
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
      "id": "8f6af6c0-5f09-4e61-a8ea-4f7f09e50001",
      "code": "Draft",
      "name": {
        "En": "Draft",
        "Ru": "Черновик",
        "Kz": "Нобай"
      },
      "sortOrder": 10
    },
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
