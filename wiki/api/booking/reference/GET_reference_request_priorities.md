# GET /reference/request-priorities

**Created:** 2026-05-28  
**Last updated:** 2026-06-02  
**Автор документов:** OpenCode

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить справочник приоритетов заявки для фильтра страницы `Мои заявки` |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Requestor UI` |
| Endpoint URL | `/api/booking/v1/reference/request-priorities` |
| Метод запроса | `GET` |
| Связанные use cases | [`UC-REQ-01 - Просмотр моих заявок`](../../../requirements/usecases/Requestor/UC-REQ-01%20-%20Просмотр%20моих%20заявок.md), [`UC-FO-01 - Просмотр списка заявок Fleet Owner`](../../../requirements/usecases/Fleet%20Owner/UC-FO-01%20-%20Просмотр%20списка%20заявок%20Fleet%20Owner.md) |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый reference API для `UC-REQ-01`. Используется для загрузки списка значений фильтра `priority` на странице `Мои заявки`.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-091 | Requestor/SWP can search and filter own requests | Confirmed | BRD v13 | Значения фильтра `priority` должны загружаться отдельным reference API |

---

## 3. Описание логики работы метода

1. Выбрать активные записи из `ref_request_priority`.
2. Вернуть значения `P1`, `P2`, `P3`, `P4`.
3. Отсортировать записи по `sortOrder`.

Сущности, участвующие в методе:
- читаются: [`ref_request_priority`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#reference-tables-instead-of-enums)
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
GET /api/booking/v1/reference/request-priorities
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
| 1 | Идентификатор значения | id | uuid | UUID v4 | — | ref_request_priority.id |  |
| 2 | Код приоритета | code | string | string | — | ref_request_priority.code | `P1 / P2 / P3 / P4` |
| 3 | Локализованное наименование | name | object | object | — | ref_request_priority |  |
| 4 | Порядок сортировки | sortOrder | int | integer | — | ref_request_priority.sortOrder |  |

### Структура `value[].name`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string | string | — | ref_request_priority.nameEn |  |
| 2 | Значение на русском языке | Ru | string | string | — | ref_request_priority.nameRu |  |
| 3 | Значение на казахском языке | Kz | string | string | — | ref_request_priority.nameKz |  |

## 10. Пример ответа

```json
{
  "value": [
    {
      "id": "79a0ae7f-2b86-4e65-aad9-2087fb4d0001",
      "code": "P1",
      "name": {
        "En": "Priority 1",
        "Ru": "Приоритет 1",
        "Kz": "1-басымдық"
      },
      "sortOrder": 10
    },
    {
      "id": "79a0ae7f-2b86-4e65-aad9-2087fb4d0002",
      "code": "P2",
      "name": {
        "En": "Priority 2",
        "Ru": "Приоритет 2",
        "Kz": "2-басымдық"
      },
      "sortOrder": 20
    },
    {
      "id": "79a0ae7f-2b86-4e65-aad9-2087fb4d0003",
      "code": "P3",
      "name": {
        "En": "Priority 3",
        "Ru": "Приоритет 3",
        "Kz": "3-басымдық"
      },
      "sortOrder": 30
    },
    {
      "id": "79a0ae7f-2b86-4e65-aad9-2087fb4d0004",
      "code": "P4",
      "name": {
        "En": "Priority 4",
        "Ru": "Приоритет 4",
        "Kz": "4-басымдық"
      },
      "sortOrder": 40
    }
  ],
  "isSuccess": true,
  "errors": []
}
```
