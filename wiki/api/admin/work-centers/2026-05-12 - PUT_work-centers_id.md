**Created:** 2026-05-12  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

# PUT /work-centers/:id

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Сохранить изменения work center в справочнике [`Admin`](../../../requirements/Roles%20and%20Access%20Model.md) Panel |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Admin Panel` |
| Endpoint URL | `/api/admin/v1/work-centers/:id` |
| Метод запроса | `PUT` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Обрабатывает редактирование строки work center в справочнике [`Admin`](../../../requirements/Roles%20and%20Access%20Model.md) Panel.

---

## 2. Функциональные требования

Общий перечень требований: Требования к системе DMMS

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-013 | [`Admin`](../../../requirements/Roles%20and%20Access%20Model.md) создаёт equipment types / equipment-related master data | Confirmed | BRD v13 | Work centers являются связанным справочником для типов техники |
| TCO Booking Tool | FR-014 | [`Admin`](../../../requirements/Roles%20and%20Access%20Model.md) редактирует параметры техники и связанные справочные сущности | Confirmed | BRD v13 | Метод нужен для редактирования справочника work centers |
| TCO Booking Tool | FR-NEW-50 | [`Admin`](../../../requirements/Roles%20and%20Access%20Model.md) manages all reference/handbook values via [`Admin`](../../../requirements/Roles%20and%20Access%20Model.md) Panel | Confirmed | BRD v13 | Work centers управляются через [`Admin`](../../../requirements/Roles%20and%20Access%20Model.md) Panel |

---

## 3. Описание логики работы метода

1. Найти запись в `WorkCenters` WHERE `id` = `:id` AND `isDeleted = false`. Если запись не найдена — вернуть `404 NOT_FOUND`.
2. Провалидировать поле `code`: непустая строка; уникальная в `WorkCenters` WHERE `id` ≠ `:id` AND `isDeleted = false`.
3. Провалидировать поле `name`: должен быть передан объект `{ En, Ru, Kz }`; `name.En`, `name.Ru`, `name.Kz` обязательны; локализованное имя уникально в `WorkCenters` WHERE `id` ≠ `:id` AND `isDeleted = false`.
4. Обновить запись в `WorkCenters`; заполнить аудит-поля `updatedAt` и `updatedBy`.
5. Рассчитать `equipmentTypesCount` = COUNT(`EquipmentTypes` WHERE `workCenterId` = `:id` AND `isDeleted = false`).
6. Вернуть обновлённый объект `WorkCenter` в формате общего `result wrapper` с HTTP 200.

Сущности, участвующие в методе:
- читаются: `WorkCenters`, `EquipmentTypes` (агрегат)
- изменяются: `WorkCenters` (UPDATE)
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
| `NOT_FOUND` | Work center с указанным `id` не найден или помечен как удалённый |
| `VALIDATION_ERROR` | Поле `code` пустое или уже существует в справочнике |
| `VALIDATION_ERROR` | Поле `name` не передано, одно из полей `name.En`, `name.Ru`, `name.Kz` пустое или локализованное имя уже существует в справочнике |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор work center | `id` | `uuid` | `+` | Должен быть валидным UUID v4 | — | Path param | |
| 2 | Код work center | `code` | `string` | `+` | Непустая строка; уникальная среди `WorkCenters` WHERE `id` ≠ `:id` AND `isDeleted = false` | — | Request body | |
| 3 | Наименование work center | `name` | `object` | `+` | Объект `{ En, Ru, Kz }`; `name.En`, `name.Ru`, `name.Kz` обязательны; локализованное имя уникально среди `WorkCenters` WHERE `id` ≠ `:id` AND `isDeleted = false` | — | Request body | |

---

## 8. Пример запроса

```http
PUT /api/admin/v1/work-centers/12a8b1ce-3aaf-4f55-8ac8-f8cf5d86c222
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "code": "WC-100",
  "name": {
    "En": "Drilling and Wells",
    "Ru": "Бурение и скважины",
    "Kz": "Бұрғылау және ұңғымалар"
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
      "En": "Drilling and Wells",
      "Ru": "Бурение и скважины",
      "Kz": "Бұрғылау және ұңғымалар"
    },
    "equipmentTypesCount": 6
  },
  "isSuccess": true,
  "errors": []
}
```

---

## Замечания

1. Уникальность `code` и `name` при обновлении проверяется с исключением самой редактируемой записи (`id ≠ :id`). Для `name` применяется правило локализованной уникальности.
2. Изменение `code` или `name` не требует обновления `EquipmentTypes`, так как тип техники хранит только `workCenterId`.
