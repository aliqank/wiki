**Created:** 2026-05-12  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

# POST /work-centers

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Создать новый work center в справочнике [`Admin`](../../../requirements/Roles%20and%20Access%20Model.md) Panel |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Admin Panel` |
| Endpoint URL | `/api/admin/v1/work-centers` |
| Метод запроса | `POST` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Обеспечивает создание записи work center через форму или inline-create на экране справочника [`Admin`](../../../requirements/Roles%20and%20Access%20Model.md) Panel.

---

## 2. Функциональные требования

Общий перечень требований: Требования к системе DMMS

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-013 | [`Admin`](../../../requirements/Roles%20and%20Access%20Model.md) создаёт equipment types / equipment-related master data | Confirmed | BRD v13 | Work centers являются связанным справочником для типов техники |
| TCO Booking Tool | FR-014 | [`Admin`](../../../requirements/Roles%20and%20Access%20Model.md) редактирует параметры техники и связанные справочные сущности | Confirmed | BRD v13 | Метод нужен для пополнения справочника work centers |
| TCO Booking Tool | FR-NEW-50 | [`Admin`](../../../requirements/Roles%20and%20Access%20Model.md) manages all reference/handbook values via [`Admin`](../../../requirements/Roles%20and%20Access%20Model.md) Panel | Confirmed | BRD v13 | Work centers управляются через [`Admin`](../../../requirements/Roles%20and%20Access%20Model.md) Panel |

---

## 3. Описание логики работы метода

1. Принять тело запроса; провалидировать обязательные поля (`code`, `name`).
2. Проверить уникальность `code` в таблице `WorkCenters` WHERE `isDeleted = false`. Если запись с таким кодом уже существует — вернуть `422 VALIDATION_ERROR`.
3. Проверить поле `name`: должен быть передан объект `{ En, Ru, Kz }`; `name.En`, `name.Ru`, `name.Kz` обязательны.
4. Проверить уникальность `name` в таблице `WorkCenters` WHERE `isDeleted = false`. Проверка выполняется по правилу локализованной уникальности для набора `name.En`, `name.Ru`, `name.Kz`. Если запись с таким локализованным названием уже существует — вернуть `422 VALIDATION_ERROR`.
5. Создать новую запись в `WorkCenters`; заполнить аудит-поля `createdAt` и `createdBy`.
6. Рассчитать `equipmentTypesCount` = `0`.
7. Вернуть созданный объект в формате общего `result wrapper` с HTTP 201.

Сущности, участвующие в методе:
- читаются: `WorkCenters` (валидация уникальности)
- изменяются: `WorkCenters` (INSERT)
- транзакционность: не требуется

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| [`Admin`](../../../requirements/Roles%20and%20Access%20Model.md) | Доступ к административной панели и справочнику work centers |

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
| `VALIDATION_ERROR` | Поле `code` пустое или уже существует в справочнике |
| `VALIDATION_ERROR` | Поле `name` не передано, одно из полей `name.En`, `name.Ru`, `name.Kz` пустое или локализованное имя уже существует в справочнике |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Код work center | `code` | `string` | `+` | Непустая строка; уникальная среди `WorkCenters` WHERE `isDeleted = false` | — | Request body | |
| 2 | Наименование work center | `name` | `object` | `+` | Объект `{ En, Ru, Kz }`; `name.En`, `name.Ru`, `name.Kz` обязательны; локализованное имя уникально среди `WorkCenters` WHERE `isDeleted = false` | — | Request body | |

---

## 8. Пример запроса

```http
POST /api/admin/v1/work-centers
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "code": "WC-100",
  "name": {
    "En": "Drilling Operations",
    "Ru": "Буровые работы",
    "Kz": "Бұрғылау жұмыстары"
  }
}
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий [`result wrapper`](../../common/Result%20Wrapper.md).

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | value | object | object | — | backend aggregation |  |
| 1.1 | Идентификатор записи | id | uuid | UUID v4 | — | WorkCenters.id |  |
| 1.2 | Код записи | code | string | string | — | WorkCenters.code |  |
| 1.3 | Наименование | name | object | object | — | WorkCenters |  |
| 1.4 | Количество типов техники | equipmentTypesCount | int | integer | — | COUNT(EquipmentTypes) |  |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | object[] | `[]` | backend |  |

### Структура `value`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | WorkCenters.id |  |
| 2 | Код записи | code | string | string | — | WorkCenters.code |  |
| 3 | Наименование | name | object | object | — | WorkCenters |  |
| 4 | Количество типов техники | equipmentTypesCount | int | integer | — | COUNT(EquipmentTypes) |  |

### Структура `value.name`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string | string | — | WorkCenters |  |
| 2 | Значение на русском языке | Ru | string | string | — | WorkCenters |  |
| 3 | Значение на казахском языке | Kz | string | string | — | WorkCenters |  |

## 10. Пример ответа

```json
{
  "value": {
    "id": "12a8b1ce-3aaf-4f55-8ac8-f8cf5d86c222",
    "code": "WC-100",
    "name": {
      "En": "Drilling Operations",
      "Ru": "Буровые работы",
      "Kz": "Бұрғылау жұмыстары"
    },
    "equipmentTypesCount": 0
  },
  "isSuccess": true,
  "errors": []
}
```

---

## Замечания

1. Уникальность `code` и `name` проверяется только среди активных записей (`isDeleted = false`). Для `name` применяется правило локализованной уникальности.
2. После создания запись сразу доступна для выбора в `POST /equipment-types` и `PUT /equipment-types/:id`.
