# GET /reference/approval-request-types

**Created:** 2026-05-28  
**Last updated:** 2026-06-02  
**Автор документов:** OpenCode

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить справочник типов заявки для фильтра [`Fleet Owner`](../../../requirements/Roles and Access Model.md) view `Approvals -> Requests` |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / [`Fleet Owner`](../../../requirements/Roles and Access Model.md) UI` |
| Endpoint URL | `/api/booking/v1/reference/approval-request-types` |
| Метод запроса | `GET` |
| Связанные use cases | [`UC-FO-01 - Просмотр списка заявок Fleet Owner`](../../../requirements/usecases/Fleet%20Owner/UC-FO-01%20-%20Просмотр%20списка%20заявок%20Fleet%20Owner.md) |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый reference API для `UC-FO-03`. Используется для загрузки списка значений фильтра `type` во view [`Fleet Owner`](../../../requirements/Roles and Access Model.md) `Approvals -> Requests`.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-094 | System generates reports on Requests/Bookings/Equipment with filters | Confirmed | BRD v13 | Значения фильтра `type` должны загружаться отдельным reference API |

---

## 3. Описание логики работы метода

1. Выбрать активные записи из `ref_request_type`.
2. Вернуть значения `Regular` и `ServiceWork`.
3. Отсортировать записи по `sortOrder`.

Сущности, участвующие в методе:
- читаются: [`ref_request_type`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#reference-tables-instead-of-enums)
- изменения не выполняются

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| [`FleetOwner`](../../../requirements/Roles and Access Model.md) | Просмотр и фильтрация заявок в approval request view |

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
GET /api/booking/v1/reference/approval-request-types
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
| 1 | Идентификатор значения | id | uuid | UUID v4 | — | ref_request_type.id |  |
| 2 | Код типа заявки | code | string | string | — | ref_request_type.code | `Regular / ServiceWork` |
| 3 | Локализованное наименование | name | object | object | — | ref_request_type |  |
| 4 | Порядок сортировки | sortOrder | int | integer | — | ref_request_type.sortOrder |  |

### Структура `value[].name`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string | string | — | ref_request_type.nameEn |  |
| 2 | Значение на русском языке | Ru | string | string | — | ref_request_type.nameRu |  |
| 3 | Значение на казахском языке | Kz | string | string | — | ref_request_type.nameKz |  |

## 10. Пример ответа

```json
{
  "value": [
    {
      "id": "9b4bf1f1-cc8f-4fbf-a1e5-9d4c9ae20001",
      "code": "Regular",
      "name": {
        "En": "Regular Request",
        "Ru": "Обычная заявка",
        "Kz": "Қалыпты өтінім"
      },
      "sortOrder": 10
    },
    {
      "id": "9b4bf1f1-cc8f-4fbf-a1e5-9d4c9ae20002",
      "code": "ServiceWork",
      "name": {
        "En": "Service Work Request",
        "Ru": "Сервисная заявка",
        "Kz": "Сервистік өтінім"
      },
      "sortOrder": 20
    }
  ],
  "isSuccess": true,
  "errors": []
}
```
