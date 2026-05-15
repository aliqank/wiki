**Created:** 2026-05-12  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

# POST /properties

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Создать новую характеристику |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Admin Panel` |
| Endpoint URL | `/api/admin/v1/properties` |
| Метод запроса | `POST` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется для создания новой записи в справочнике характеристик.

---

## 2. Функциональные требования

Общий перечень требований: Требования к системе DMMS

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-010 | Admin configures dynamic custom characteristics per equipment type via Admin Panel tool | Confirmed | BRD v13 | Нужен для пополнения EAV-справочника |
| TCO Booking Tool | FR-014 | Admin редактирует параметры техники и связанные справочные сущности | Confirmed | BRD v13 | Нужен для пополнения справочника |

---

## 3. Описание логики работы метода

1. Принять тело запроса; провалидировать обязательные поля `name`, `dataType`.
2. Проверить объект `name`: должны быть переданы локализованные поля `En`, `Ru`, `Kz`; `name.Ru` обязателен.
3. Проверить уникальность `name` среди `Properties` WHERE `isDeleted = false`. Проверка выполняется по правилу локализованной уникальности для набора `name.En`, `name.Ru`, `name.Kz`.
4. Проверить `dataType`: одно из `number / double / text / boolean / enum`.
5. Если `unitId` передан, то он допустим только для `dataType = number` или `double`; также проверить существование активной записи в `MeasurementUnits`.
6. Если передан массив `enumValues`, он допустим только для `dataType = enum`; значения не должны дублироваться внутри одного запроса.
7. Создать запись в `Properties`.
8. Для `enumValues[]`, если они переданы, создать записи в `PropertyEnumValues`.
9. Вернуть созданный объект `PropertyDetail`.

Сущности, участвующие в методе:
- читаются: `Properties`, `MeasurementUnits`
- изменяются: `Properties`, `PropertyEnumValues`
- транзакционность: требуется единая транзакция

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
| `VALIDATION_ERROR` | Поле `name` не передано, `name.Ru` пустое или локализованное имя уже существует |
| `VALIDATION_ERROR` | Поле `dataType` содержит недопустимое значение |
| `VALIDATION_ERROR` | Передан несуществующий или удалённый `unitId` |
| `VALIDATION_ERROR` | `unitId` допустим только для `number` и `double` |
| `VALIDATION_ERROR` | `enumValues[]` допустим только для `dataType = enum` |
| `VALIDATION_ERROR` | В `enumValues[]` обнаружены дубли значения |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Наименование характеристики | `name` | `object` | `+` | Объект `{ En, Ru, Kz }`; `name.Ru` обязателен; локализованное имя уникально среди активных записей | — | Request body | |
| 2 | Тип данных | `dataType` | `enum` | `+` | `number / double / text / boolean / enum` | — | Request body | |
| 3 | Идентификатор единицы измерения | `unitId` | `uuid` | `-` | Только для `number` / `double`; должен ссылаться на активную `MeasurementUnits` | `null` | Request body | |
| 4 | Список enum-значений | `enumValues` | `array<object>` | `-` | Допустим только для `dataType = enum` | `[]` | Request body | |
| 4.1 | Значение enum | `value` | `string` | `+` | Непустая строка; без дублей внутри запроса | — | Request body / enumValues[] | |
| 4.2 | Порядок отображения | `sortOrder` | `int` | `+` | Целое число >= 1 | — | Request body / enumValues[] | |

---

## 8. Пример запроса

```http
POST /api/admin/v1/properties
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "name": {
    "En": "Drive type",
    "Ru": "Тип привода",
    "Kz": "Жетек түрі"
  },
  "dataType": "enum",
  "unitId": null,
  "enumValues": [
    { "value": "Дизельный", "sortOrder": 1 },
    { "value": "Электрический", "sortOrder": 2 }
  ]
}
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий `result wrapper`.

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | value | object | object | — | backend aggregation |  |
| 1.1 | Идентификатор записи | id | string | string | — | Properties.id |  |
| 1.2 | Наименование | name | object | object | — | Properties |  |
| 1.3 | Тип данных свойства | dataType | string | string | — | Properties + ref_property_data_type |  |
| 1.4 | Единица измерения | unit | null | — | `null` | MeasurementUnits |  |
| 1.5 | Список enum-значений | enumValues | array<object> | object[] | — | PropertyEnumValues | Коллекция объектов |
| 1.6 | Количество типов техники | equipmentTypesCount | int | integer | — | COUNT(EquipmentTypeProperties) |  |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | object[] | `[]` | backend |  |

### Структура `value`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | string | string | — | Properties.id |  |
| 2 | Наименование | name | object | object | — | Properties |  |
| 3 | Тип данных свойства | dataType | string | string | — | Properties + ref_property_data_type |  |
| 4 | Единица измерения | unit | null | — | `null` | MeasurementUnits |  |
| 5 | Список enum-значений | enumValues | array<object> | object[] | — | PropertyEnumValues | Коллекция объектов |
| 6 | Количество типов техники | equipmentTypesCount | int | integer | — | COUNT(EquipmentTypeProperties) |  |

### Структура `value.name`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string | string | — | Properties |  |
| 2 | Значение на русском языке | Ru | string | string | — | Properties |  |
| 3 | Значение на казахском языке | Kz | string | string | — | Properties |  |

### Структура `value.enumValues[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | Properties.id |  |
| 2 | Результат выполнения метода | value | string | string | — | backend aggregation |  |
| 3 | Порядок сортировки | sortOrder | int | integer | — | Properties.sortOrder |  |

## 10. Пример ответа

```json
{
  "value": {
    "id": "p0000001-0000-4000-8000-000000000002",
    "name": {
      "En": "Drive type",
      "Ru": "Тип привода",
      "Kz": "Жетек түрі"
    },
    "dataType": "enum",
    "unit": null,
    "enumValues": [
      { "id": "e0000001-0000-4000-8000-000000000001", "value": "Дизельный", "sortOrder": 1 },
      { "id": "e0000001-0000-4000-8000-000000000002", "value": "Электрический", "sortOrder": 2 }
    ],
    "equipmentTypesCount": 0
  },
  "isSuccess": true,
  "errors": []
}
```

---

## Замечания

1. Для `dataType = enum` список `enumValues` можно передать сразу при создании.
2. Для остальных типов `enumValues` должен отсутствовать или быть пустым.
