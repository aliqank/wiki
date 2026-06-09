# GET /reference/request-priorities

**Created:** 2026-05-28  
**Last updated:** 2026-06-08  
**Автор документов:** OpenCode

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить справочник приоритетов заявки для фильтров, tooltip-описаний и цветового отображения в request-related UI |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Shared Reference UI` |
| Endpoint URL | `/api/booking/v1/reference/request-priorities` |
| Метод запроса | `GET` |
| Связанные use cases | [`UC-REQ-01 - Просмотр моих заявок`](../../../requirements/usecases/Requestor/UC-REQ-01%20-%20Просмотр%20моих%20заявок.md), [`UC-FO-01 - Просмотр списка заявок Fleet Owner`](../../../requirements/usecases/Fleet%20Owner/UC-FO-01%20-%20Просмотр%20списка%20заявок%20Fleet%20Owner.md) |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый reference API для `UC-REQ-01`. Используется для загрузки списка значений фильтра `priority`, tooltip-описаний и UI-цветов приоритета на страницах Requestor / Fleet Owner.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-091 | Requestor can search and filter own requests | Confirmed | BRD v13 | Значения фильтра `priority` должны загружаться отдельным reference API для текущего scope |
| TCO Booking Tool | FR-NEW-69 | Priority field tooltips/hints | Confirmed | BRD v13 | Метод возвращает локализованные descriptions для tooltip / hint по каждому priority value |

---

## 3. Описание логики работы метода

1. Выбрать записи из `ref_request_priority`.
2. Вернуть фиксированные seeded значения `P1`, `P2`, `P3`, `P4`.
3. Для каждой записи вернуть локализованное имя, локализованное описание для tooltip / hint и `color` для UI-отображения.
4. Отсортировать записи по `sortOrder`.

Сущности, участвующие в методе:
- читаются: [`ref_request_priority`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#reference-tables-instead-of-enums)
- изменения не выполняются

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Requestor` | Просмотр и фильтрация собственных заявок |
| `FleetOwner` | Просмотр request-level очереди и фильтров Fleet Owner |

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
| 4 | Локализованное описание | description | object | object | — | ref_request_priority | Используется для tooltip / hint в UI |
| 5 | Цвет отображения приоритета | color | string | `#RRGGBB` | — | ref_request_priority.color | Используется для цветового отображения priority badge / label |
| 6 | Порядок сортировки | sortOrder | int | integer | — | ref_request_priority.sortOrder |  |

### Структура `value[].name`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string | string | — | ref_request_priority.nameEn |  |
| 2 | Значение на русском языке | Ru | string | string | — | ref_request_priority.nameRu |  |
| 3 | Значение на казахском языке | Kz | string | string | — | ref_request_priority.nameKz |  |

### Структура `value[].description`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string \| null | string | `null` | ref_request_priority.descriptionEn |  |
| 2 | Значение на русском языке | Ru | string \| null | string | `null` | ref_request_priority.descriptionRu |  |
| 3 | Значение на казахском языке | Kz | string \| null | string | `null` | ref_request_priority.descriptionKz |  |

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
      "description": {
        "En": "Must be completed within the current shift.",
        "Ru": "Должно быть выполнено в рамках текущей смены.",
        "Kz": "Ағымдағы ауысым ішінде орындалуы тиіс."
      },
      "color": "#D92D20",
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
      "description": {
        "En": "Must be completed within the current week.",
        "Ru": "Должно быть выполнено в течение текущей недели.",
        "Kz": "Ағымдағы апта ішінде орындалуы тиіс."
      },
      "color": "#F79009",
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
      "description": {
        "En": "Planned work with a wider execution window.",
        "Ru": "Плановая работа с более широким окном исполнения.",
        "Kz": "Орындау терезесі кең жоспарлы жұмыс."
      },
      "color": "#2563EB",
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
      "description": {
        "En": "Standard priority with no urgent execution requirement.",
        "Ru": "Стандартный приоритет без срочного требования к исполнению.",
        "Kz": "Жедел орындау талабынсыз стандартты басымдық."
      },
      "color": "#667085",
      "sortOrder": 40
    }
  ],
  "isSuccess": true,
  "errors": []
}
```

---

## Замечания

1. Метод остаётся read-only: состав значений `P1`-`P4` не изменяется через booking reference API.
2. Frontend должен использовать `description` для tooltip / hint и `color` для визуального отображения priority badge / label.
