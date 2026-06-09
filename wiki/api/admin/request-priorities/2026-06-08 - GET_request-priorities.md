**Created:** 2026-06-08  
**Last updated:** 2026-06-08  
**Автор документов:** Telman Nurzhanov (SA)

---

# GET /request-priorities

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить список приоритетов заявки для справочника [`Admin`](../../../requirements/Roles%20and%20Access%20Model.md) Panel |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Admin Panel` |
| Endpoint URL | `/api/admin/v1/request-priorities` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется для экрана [`Admin`](../../../requirements/Roles%20and%20Access%20Model.md) Panel, где администратор просматривает фиксированный справочник приоритетов `P1`-`P4` и редактирует для них `description` и `color`.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-NEW-50 | [`Admin`](../../../requirements/Roles%20and%20Access%20Model.md) manages all reference/handbook values via [`Admin`](../../../requirements/Roles%20and%20Access%20Model.md) Panel | Confirmed | BRD v13 | Метод используется для управления metadata справочника приоритетов |
| TCO Booking Tool | FR-NEW-69 | Priority field tooltips/hints | Confirmed | BRD v13 | Метод возвращает descriptions, используемые в tooltip / hint |

---

## 3. Описание логики работы метода

1. Выбрать записи из `ref_request_priority`.
2. Вернуть фиксированный набор priority values `P1`, `P2`, `P3`, `P4`.
3. Для каждой записи вернуть `code`, `name`, `description`, `color`, `sortOrder`.
4. Отсортировать записи по `sortOrder ASC`.
5. Вернуть список в общем `result wrapper`.

Сущности, участвующие в методе:
- читаются: `ref_request_priority`
- изменения не выполняются
- транзакционность не требуется, метод read-only

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| [`Admin`](../../../requirements/Roles%20and%20Access%20Model.md) | Доступ к административной панели и справочнику request priorities |

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
| `FORBIDDEN` | У пользователя нет роли [`Admin`](../../../requirements/Roles%20and%20Access%20Model.md) |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`

---

## 7. Параметры метода

Метод не принимает параметров.

---

## 8. Пример запроса

```http
GET /api/admin/v1/request-priorities
Authorization: Bearer <token>
Content-Type: application/json
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий [`result wrapper`](../../common/Result%20Wrapper.md).

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | value | array<object> | object[] | — | backend aggregation |  |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | object[] | `[]` | backend |  |

### Структура `value[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | ref_request_priority.id |  |
| 2 | Код приоритета | code | string | string | — | ref_request_priority.code | `P1 / P2 / P3 / P4`, read-only |
| 3 | Локализованное наименование | name | object | object | — | ref_request_priority | read-only |
| 4 | Локализованное описание | description | object | object | — | ref_request_priority | Редактируется в [`Admin`](../../../requirements/Roles%20and%20Access%20Model.md) Panel |
| 5 | Цвет отображения | color | string | `#RRGGBB` | — | ref_request_priority.color | Редактируется в [`Admin`](../../../requirements/Roles%20and%20Access%20Model.md) Panel |
| 6 | Порядок сортировки | sortOrder | int | integer | — | ref_request_priority.sortOrder | read-only |

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
    }
  ],
  "isSuccess": true,
  "errors": []
}
```

---

## Замечания

1. Метод возвращает фиксированный seeded справочник приоритетов; создание и удаление приоритетов не входят в текущий scope.
2. Поля `code`, `name`, `sortOrder` в [`Admin`](../../../requirements/Roles%20and%20Access%20Model.md) UI отображаются как read-only.
