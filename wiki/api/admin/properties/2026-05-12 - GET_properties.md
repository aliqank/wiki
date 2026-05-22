**Created:** 2026-05-12  
**Last updated:** 2026-05-22  
**Автор документов:** Telman Nurzhanov (SA)

---

# GET /properties

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить постраничный список характеристик |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Admin Panel` |
| Endpoint URL | `/api/admin/v1/properties` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется для экрана управления справочником характеристик в Admin Panel.

---

## 2. Функциональные требования

Общий перечень требований: Требования к системе DMMS

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-010 | Admin configures dynamic custom characteristics per equipment type via Admin Panel tool | Confirmed | BRD v13 | Характеристики являются центральным справочником EAV |
| TCO Booking Tool | FR-014 | Admin редактирует параметры техники и связанные справочные сущности | Confirmed | BRD v13 | Метод нужен для управления справочником |
| TCO Booking Tool | FR-NEW-50 | Admin manages all reference/handbook values via Admin Panel | Confirmed | BRD v13 | Справочник управляется через Admin Panel |

---

## 3. Описание логики работы метода

1. Получить записи из `Properties` WHERE `isDeleted = false`.
2. Если передан `search`, применить фильтр по `Properties.code`, `Properties.name.En`, `Properties.name.Ru`, `Properties.name.Kz`.
3. Подтянуть `MeasurementUnits` по `Properties.unitId`.
4. Для каждой записи рассчитать:
   - `enumValuesCount` = COUNT(`PropertyEnumValues` WHERE `propertyId` = `Properties.id` AND `isDeleted = false`)
   - `equipmentTypesCount` = COUNT(`EquipmentTypeProperties` WHERE `propertyId` = `Properties.id` AND `isDeleted = false`)
5. Отсортировать по `name.Ru ASC`.
6. Вернуть страницу данных в формате общего `result wrapper` с `PaginatedResult` внутри `value`.

Сущности, участвующие в методе:
- читаются: `Properties`, `MeasurementUnits`, `PropertyEnumValues`, `EquipmentTypeProperties`
- изменения не выполняются
- транзакционность не требуется, метод read-only

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

HTTP-коды: `401 Unauthorized`, `403 Forbidden`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Поиск по коду или названию характеристики | `search` | `string` | `-` | Если передан, используется как фильтр по `Properties.code`, `Properties.name.En`, `Properties.name.Ru`, `Properties.name.Kz` | — | Query param | |
| 2 | Номер страницы | `page` | `int` | `-` | Целое число >= 1 | `1` | Query param | |
| 3 | Размер страницы | `limit` | `int` | `-` | Целое число >= 1 | `20` | Query param | |

---

## 8. Пример запроса

```http
GET /api/admin/v1/properties?search=глубина&page=1&limit=20
Authorization: Bearer <token>
Content-Type: application/json
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий `result wrapper`.

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | value | object | PaginatedResult | — | backend aggregation |  |
| 1.1 | Элементы текущей страницы | items | array<object> | object[] | — | backend composition from Properties + MeasurementUnits + PropertyEnumValues + EquipmentTypeProperties | Коллекция объектов |
| 1.2 | Общее количество записей | total | int | integer | — | backend |  |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | object[] | `[]` | backend |  |

### Структура `value`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Элементы текущей страницы | items | array<object> | object[] | — | backend composition from Properties + MeasurementUnits + PropertyEnumValues + EquipmentTypeProperties | Коллекция объектов |
| 2 | Общее количество записей | total | int | integer | — | backend |  |

### Структура `value.items[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | string | string | — | Properties.id |  |
| 2 | Код характеристики | code | string | string | — | Properties.code |  |
| 3 | Наименование | name | object | object | — | Properties |  |
| 4 | Тип данных свойства | dataType | string | string | — | Properties + ref_property_data_type |  |
| 5 | Единица измерения | unit | object | object | — | MeasurementUnits |  |
| 6 | Количество enum-значений | enumValuesCount | int | integer | — | COUNT(PropertyEnumValues) |  |
| 7 | Количество типов техники | equipmentTypesCount | int | integer | — | COUNT(EquipmentTypeProperties) |  |

### Структура `value.items[].name`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string | string | — | Properties |  |
| 2 | Значение на русском языке | Ru | string | string | — | Properties |  |
| 3 | Значение на казахском языке | Kz | string | string | — | Properties |  |

### Структура `value.items[].unit`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | string | string | — | Properties.id |  |
| 2 | Код записи | code | string | string | — | backend composition from Properties + MeasurementUnits + PropertyEnumValues + EquipmentTypeProperties |  |
| 3 | Отображаемое наименование | displayName | string | string | — | backend composition from Properties + MeasurementUnits + PropertyEnumValues + EquipmentTypeProperties |  |

## 10. Пример ответа

```json
{
  "value": {
    "items": [
      {
        "id": "p0000001-0000-4000-8000-000000000003",
        "code": "maximum_depth",
        "name": {
          "En": "Maximum depth",
          "Ru": "Максимальная глубина",
          "Kz": "Ең үлкен тереңдік"
        },
        "dataType": "number",
        "unit": {
          "id": "u0000001-0000-4000-8000-000000000002",
          "code": "м",
          "displayName": "Метр"
        },
        "enumValuesCount": 0,
        "equipmentTypesCount": 4
      }
    ],
    "total": 24
  },
  "isSuccess": true,
  "errors": []
}
```

---

## Замечания

1. Для `dataType = enum` состав значений управляется через `PropertyEnumValues`.
2. `equipmentTypesCount` используется для guard-логики удаления характеристики.
3. Поле `name` возвращается как локализованный объект `{ En, Ru, Kz }`.
