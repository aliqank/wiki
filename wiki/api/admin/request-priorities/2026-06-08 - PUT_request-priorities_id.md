**Created:** 2026-06-08  
**Last updated:** 2026-06-08  
**Автор документов:** Telman Nurzhanov (SA)

---

# PUT /request-priorities/:id

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Сохранить изменения описания и цвета приоритета заявки в справочнике [`Admin`](../../../requirements/Roles and Access Model.md) Panel |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Admin Panel` |
| Endpoint URL | `/api/admin/v1/request-priorities/:id` |
| Метод запроса | `PUT` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется для редактирования только `description` и `color` существующих priority values `P1`-`P4` в [`Admin`](../../../requirements/Roles and Access Model.md) Panel.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-NEW-50 | [`Admin`](../../../requirements/Roles and Access Model.md) manages all reference/handbook values via [`Admin`](../../../requirements/Roles and Access Model.md) Panel | Confirmed | BRD v13 | Метод используется для управления metadata справочника приоритетов |
| TCO Booking Tool | FR-NEW-69 | Priority field tooltips/hints | Confirmed | BRD v13 | Через метод администратор редактирует descriptions для tooltip / hint |

---

## 3. Описание логики работы метода

1. Найти запись в `ref_request_priority` по `id`. Если запись не найдена - вернуть `404 NOT_FOUND`.
2. Провалидировать поле `description`: должен быть передан объект `{ En, Ru, Kz }`; допускаются `null` или пустые строки, если бизнес не предоставил текст для конкретной локали.
3. Провалидировать поле `color`: обязательная строка в формате `#RRGGBB`.
4. Обновить только поля `descriptionEn`, `descriptionRu`, `descriptionKz`, `color`.
5. Не изменять `code`, `name*`, `sortOrder`.
6. Вернуть обновлённый объект справочника в общем `result wrapper`.

Сущности, участвующие в методе:
- читаются: `ref_request_priority`
- изменяются: `ref_request_priority` (UPDATE `description*`, `color`)
- транзакционность: не требуется

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| [`Admin`](../../../requirements/Roles and Access Model.md) | Доступ к административной панели и справочнику request priorities |

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
| `FORBIDDEN` | У пользователя нет роли [`Admin`](../../../requirements/Roles and Access Model.md) |
| `NOT_FOUND` | Priority с указанным `id` не найден |
| `VALIDATION_ERROR` | Поле `color` не передано или не соответствует формату `#RRGGBB` |
| `VALIDATION_ERROR` | Поле `description` не передано как объект `{ En, Ru, Kz }` |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор priority | `id` | `uuid` | `+` | Должен быть валидным UUID v4 | — | Path param | |
| 2 | Локализованное описание | `description` | `object` | `+` | Объект `{ En, Ru, Kz }` | — | Request body | Редактируемое поле |
| 3 | Значение на английском языке | `description.En` | `string \| null` | `-` | Если передано строкой, длина <= 1000 | `null` | Request body | |
| 4 | Значение на русском языке | `description.Ru` | `string \| null` | `-` | Если передано строкой, длина <= 1000 | `null` | Request body | |
| 5 | Значение на казахском языке | `description.Kz` | `string \| null` | `-` | Если передано строкой, длина <= 1000 | `null` | Request body | |
| 6 | Цвет отображения | `color` | `string` | `+` | Строка формата `#RRGGBB` | — | Request body | Редактируемое поле |

---

## 8. Пример запроса

```http
PUT /api/admin/v1/request-priorities/79a0ae7f-2b86-4e65-aad9-2087fb4d0001
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "description": {
    "En": "Must be completed within the current shift.",
    "Ru": "Должно быть выполнено в рамках текущей смены.",
    "Kz": "Ағымдағы ауысым ішінде орындалуы тиіс."
  },
  "color": "#D92D20"
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
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | ref_request_priority.id |  |
| 2 | Код приоритета | code | string | string | — | ref_request_priority.code | `P1 / P2 / P3 / P4`, read-only |
| 3 | Локализованное наименование | name | object | object | — | ref_request_priority | read-only |
| 4 | Локализованное описание | description | object | object | — | ref_request_priority | Обновляется методом |
| 5 | Цвет отображения | color | string | `#RRGGBB` | — | ref_request_priority.color | Обновляется методом |
| 6 | Порядок сортировки | sortOrder | int | integer | — | ref_request_priority.sortOrder | read-only |

### Структура `value.name`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string | string | — | ref_request_priority.nameEn |  |
| 2 | Значение на русском языке | Ru | string | string | — | ref_request_priority.nameRu |  |
| 3 | Значение на казахском языке | Kz | string | string | — | ref_request_priority.nameKz |  |

### Структура `value.description`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string \| null | string | `null` | ref_request_priority.descriptionEn |  |
| 2 | Значение на русском языке | Ru | string \| null | string | `null` | ref_request_priority.descriptionRu |  |
| 3 | Значение на казахском языке | Kz | string \| null | string | `null` | ref_request_priority.descriptionKz |  |

## 10. Пример ответа

```json
{
  "value": {
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
  "isSuccess": true,
  "errors": []
}
```

---

## Замечания

1. Метод не позволяет изменять `code`, `name`, `sortOrder` и не меняет набор значений `P1`-`P4`.
2. Изменённые `description` и `color` должны использоваться frontend-ом через [`GET /reference/request-priorities`](../../booking/reference/GET_reference_request_priorities.md).
