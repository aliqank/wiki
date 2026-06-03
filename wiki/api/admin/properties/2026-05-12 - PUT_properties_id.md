**Created:** 2026-05-12  
**Last updated:** 2026-05-22  
**Автор документов:** Telman Nurzhanov (SA)

---

# PUT /properties/:id

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Изменить характеристику |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Admin Panel` |
| Endpoint URL | `/api/admin/v1/properties/:id` |
| Метод запроса | `PUT` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется для редактирования карточки характеристики.

---

## 2. Функциональные требования

Общий перечень требований: Требования к системе DMMS

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-010 | Admin configures dynamic custom characteristics per equipment type via Admin Panel tool | Confirmed | BRD v13 | Редактирует EAV-справочник |
| TCO Booking Tool | FR-014 | Admin редактирует параметры техники и связанные справочные сущности | Confirmed | BRD v13 | Нужен для редактирования |

---

## 3. Описание логики работы метода

1. Найти запись в `Properties` WHERE `id` = `:id` AND `isDeleted = false`. Если запись не найдена — вернуть `404 NOT_FOUND`.
2. Провалидировать `code`: непустая строка; код уникален среди `Properties` WHERE `id` != `:id` AND `isDeleted = false`.
3. Провалидировать `name`: должен быть передан объект `{ En, Ru, Kz }`; `name.Ru` обязателен; локализованное имя уникально среди `Properties` WHERE `id` != `:id` AND `isDeleted = false`.
4. Провалидировать `dataType` и `unitId` по тем же правилам, что и при создании.
5. Обновить запись `Properties`, заполнить `updatedAt`, `updatedBy`.
6. Получить актуальные `enumValues` и `equipmentTypesCount` для ответа.
7. Вернуть обновлённый объект `PropertyDetail`.

Сущности, участвующие в методе:
- читаются: `Properties`, `MeasurementUnits`, `PropertyEnumValues`, `EquipmentTypeProperties`
- изменяются: `Properties` (UPDATE)
- транзакционность: не требуется

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Admin` | Доступ к административной панели и справочнику характеристик |

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
| `FORBIDDEN` | У пользователя нет роли `Admin` |
| `NOT_FOUND` | Характеристика не найдена |
| `VALIDATION_ERROR` | Поле `code` не передано, пустое или код уже существует |
| `VALIDATION_ERROR` | Поле `name` не передано, `name.Ru` пустое или локализованное имя уже существует |
| `VALIDATION_ERROR` | Поле `dataType` содержит недопустимое значение |
| `VALIDATION_ERROR` | Передан несуществующий или удалённый `unitId` |
| `VALIDATION_ERROR` | `unitId` допустим только для `number` и `double` |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор характеристики | `id` | `uuid` | `+` | Валидный UUID v4 | — | Path param | |
| 2 | Код характеристики | `code` | `string` | `+` | Непустая строка; уникален среди активных записей кроме текущей | — | Request body | |
| 3 | Наименование характеристики | `name` | `object` | `+` | Объект `{ En, Ru, Kz }`; `name.Ru` обязателен; локализованное имя уникально среди активных записей кроме текущей | — | Request body | |
| 4 | Тип данных | `dataType` | `enum` | `+` | `number / double / text / boolean / enum` | — | Request body | |
| 5 | Идентификатор единицы измерения | `unitId` | `uuid` | `-` | Только для `number` / `double` | `null` | Request body | |

---

## 8. Пример запроса

```http
PUT /api/admin/v1/properties/p0000001-0000-4000-8000-000000000003
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "code": "maximum_digging_depth",
  "name": {
    "En": "Maximum digging depth",
    "Ru": "Максимальная глубина копания",
    "Kz": "Қазу тереңдігінің максимумы"
  },
  "dataType": "number",
  "unitId": "u0000001-0000-4000-8000-000000000002"
}
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий [`result wrapper`](../../common/Result%20Wrapper.md).

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | value | object | object | — | backend aggregation |  |
| 1.1 | Идентификатор записи | id | string | string | — | Properties.id |  |
| 1.2 | Наименование | name | object | object | — | Properties |  |
| 1.3 | Тип данных свойства | dataType | string | string | — | Properties + ref_property_data_type |  |
| 1.4 | Единица измерения | unit | object | object | — | MeasurementUnits |  |
| 1.5 | Список enum-значений | enumValues | array<object> | object[] | `[]` | PropertyEnumValues | Коллекция объектов |
| 1.6 | Количество типов техники | equipmentTypesCount | int | integer | — | COUNT(EquipmentTypeProperties) |  |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | object[] | `[]` | backend |  |

### Структура `value`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | string | string | — | Properties.id |  |
| 2 | Код характеристики | code | string | string | — | Properties.code |  |
| 3 | Наименование | name | object | object | — | Properties |  |
| 4 | Тип данных свойства | dataType | string | string | — | Properties + ref_property_data_type |  |
| 5 | Единица измерения | unit | object | object | — | MeasurementUnits |  |
| 6 | Список enum-значений | enumValues | array<object> | object[] | `[]` | PropertyEnumValues | Коллекция объектов |
| 7 | Количество типов техники | equipmentTypesCount | int | integer | — | COUNT(EquipmentTypeProperties) |  |

### Структура `value.name`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string | string | — | Properties |  |
| 2 | Значение на русском языке | Ru | string | string | — | Properties |  |
| 3 | Значение на казахском языке | Kz | string | string | — | Properties |  |

### Структура `value.unit`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | string | string | — | Properties.id |  |
| 2 | Код записи | code | string | string | — | backend composition from Properties + MeasurementUnits + PropertyEnumValues + EquipmentTypeProperties |  |
| 3 | Отображаемое наименование | displayName | string | string | — | backend composition from Properties + MeasurementUnits + PropertyEnumValues + EquipmentTypeProperties |  |

## 10. Пример ответа

```json
{
  "value": {
    "id": "p0000001-0000-4000-8000-000000000003",
    "code": "maximum_digging_depth",
    "name": {
      "En": "Maximum digging depth",
      "Ru": "Максимальная глубина копания",
      "Kz": "Қазу тереңдігінің максимумы"
    },
    "dataType": "number",
    "unit": {
      "id": "u0000001-0000-4000-8000-000000000002",
      "code": "м",
      "displayName": "Метр"
    },
    "enumValues": [],
    "equipmentTypesCount": 4
  },
  "isSuccess": true,
  "errors": []
}
```

---

## Замечания

1. Метод не управляет составом `enumValues`; он меняет только базовые параметры характеристики.
