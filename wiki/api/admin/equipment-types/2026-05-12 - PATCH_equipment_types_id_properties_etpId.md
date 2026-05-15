**Created:** 2026-05-12  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

# PATCH /equipment-types/:id/properties/:etpId

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Изменить параметры привязки характеристики к типу техники |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Admin Panel` |
| Endpoint URL | `/api/admin/v1/equipment-types/:id/properties/:etpId` |
| Метод запроса | `PATCH` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Обрабатывает изменение настроек строки характеристики в блоке B карточки AP-02: обязательность, участие в фильтрах, отображение в карточке и порядок.

---

## 2. Функциональные требования

Общий перечень требований: Требования к системе DMMS

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-010 | Admin configures dynamic custom characteristics per equipment type via Admin Panel tool; characteristics appear automatically in equipment card, request form, and search filters based on equipment type | Confirmed | BRD v13 | Метод меняет правила использования характеристики внутри типа |
| TCO Booking Tool | FR-014 | Admin редактирует параметры техники и связанные справочные сущности | Confirmed | BRD v13 | Нужен для редактирования блока B |
| TCO Booking Tool | FR-NEW-68 | System dynamically shows only type-specific characteristics in request creation form and equipment search filters, based on the selected equipment type | Confirmed | BRD v13 | Изменение флагов влияет на поведение UI |

---

## 3. Описание логики работы метода

1. Найти запись в `EquipmentTypes` WHERE `id` = `:id` AND `isDeleted = false`. Если запись не найдена — вернуть `404 NOT_FOUND`.
2. Найти запись в `EquipmentTypeProperties` WHERE `id` = `:etpId` AND `equipmentTypeId` = `:id` AND `isDeleted = false`. Если запись не найдена — вернуть `404 NOT_FOUND`.
3. Провалидировать переданные поля. Допускаются к изменению: `isRequired`, `isFilterable`, `isVisibleInCard`, `sortOrder`. `sortOrder`, если передан, должен быть целым числом >= 1.
4. Обновить запись; заполнить аудит-поля `updatedAt` и `updatedBy`.
5. Вернуть обновлённый объект `EquipmentTypePropertyItem` в формате общего `result wrapper` с HTTP 200.

Сущности, участвующие в методе:
- читаются: `EquipmentTypes`, `EquipmentTypeProperties`, `Properties`, `MeasurementUnits`, `PropertyEnumValues`
- изменяются: `EquipmentTypeProperties` (UPDATE)
- транзакционность: не требуется

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Admin` | Доступ к административной панели и управлению справочниками типов техники |

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
| `NOT_FOUND` | Тип техники или запись привязки характеристики не найдена |
| `VALIDATION_ERROR` | Поле `sortOrder` имеет недопустимое значение |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор типа техники | `id` | `uuid` | `+` | Должен быть валидным UUID v4 | — | Path param | |
| 2 | Идентификатор привязки характеристики | `etpId` | `uuid` | `+` | Должен быть валидным UUID v4 | — | Path param | |
| 3 | Обязательность характеристики | `isRequired` | `bool` | `-` | Булево значение | — | Request body | |
| 4 | Используется в фильтрах поиска | `isFilterable` | `bool` | `-` | Булево значение | — | Request body | |
| 5 | Отображается в карточке техники | `isVisibleInCard` | `bool` | `-` | Булево значение | — | Request body | |
| 6 | Порядок отображения характеристики | `sortOrder` | `int` | `-` | Целое число >= 1, если передано | — | Request body | |

---

## 8. Пример запроса

```http
PATCH /api/admin/v1/equipment-types/7b4f4b4d-52d4-4a77-b6b7-f2b7d7c81111/properties/a1b2c3d4-0001-4000-8000-000000000003
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "isRequired": false,
  "isFilterable": true,
  "isVisibleInCard": true,
  "sortOrder": 2
}
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий `result wrapper`.

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | value | object | object | — | backend aggregation |  |
| 1.1 | Идентификатор привязки свойства к типу техники | etpId | uuid | UUID v4 | — | EquipmentTypeProperties.id |  |
| 1.2 | Порядок сортировки | sortOrder | int | integer | — | EquipmentTypeProperties.sortOrder |  |
| 1.3 | Признак обязательности поля | isRequired | bool | boolean | — | EquipmentTypeProperties.isRequired |  |
| 1.4 | Признак доступности свойства в фильтрах | isFilterable | bool | boolean | — | EquipmentTypeProperties.isFilterable |  |
| 1.5 | Признак отображения в карточке | isVisibleInCard | bool | boolean | — | EquipmentTypeProperties.isVisibleInCard |  |
| 1.6 | Наименование свойства | property | object | object | — | Properties |  |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | object[] | `[]` | backend |  |

### Структура `value`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор привязки свойства к типу техники | etpId | uuid | UUID v4 | — | EquipmentTypeProperties.id |  |
| 2 | Порядок сортировки | sortOrder | int | integer | — | EquipmentTypeProperties.sortOrder |  |
| 3 | Признак обязательности поля | isRequired | bool | boolean | — | EquipmentTypeProperties.isRequired |  |
| 4 | Признак доступности свойства в фильтрах | isFilterable | bool | boolean | — | EquipmentTypeProperties.isFilterable |  |
| 5 | Признак отображения в карточке | isVisibleInCard | bool | boolean | — | EquipmentTypeProperties.isVisibleInCard |  |
| 6 | Наименование свойства | property | object | object | — | Properties |  |

### Структура `value.property`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | string | string | — | EquipmentTypeProperties.id |  |
| 2 | Наименование | name | object | object | — | backend composition from EquipmentTypes + EquipmentTypeProperties + Properties + MeasurementUnits + PropertyEnumValues |  |
| 3 | Тип данных свойства | dataType | string | string | — | backend composition from EquipmentTypes + EquipmentTypeProperties + Properties + MeasurementUnits + PropertyEnumValues |  |
| 4 | Единица измерения | unit | object | object | — | backend composition from EquipmentTypes + EquipmentTypeProperties + Properties + MeasurementUnits + PropertyEnumValues |  |
| 5 | Список enum-значений | enumValues | array<object> | object[] | `[]` | backend composition from EquipmentTypes + EquipmentTypeProperties + Properties + MeasurementUnits + PropertyEnumValues | Коллекция объектов |

### Структура `value.property.name`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string | string | — | backend composition from EquipmentTypes + EquipmentTypeProperties + Properties + MeasurementUnits + PropertyEnumValues |  |
| 2 | Значение на русском языке | Ru | string | string | — | backend composition from EquipmentTypes + EquipmentTypeProperties + Properties + MeasurementUnits + PropertyEnumValues |  |
| 3 | Значение на казахском языке | Kz | string | string | — | backend composition from EquipmentTypes + EquipmentTypeProperties + Properties + MeasurementUnits + PropertyEnumValues |  |

### Структура `value.property.unit`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | string | string | — | EquipmentTypeProperties.id |  |
| 2 | Код записи | code | string | string | — | backend composition from EquipmentTypes + EquipmentTypeProperties + Properties + MeasurementUnits + PropertyEnumValues |  |
| 3 | Отображаемое наименование | displayName | string | string | — | backend composition from EquipmentTypes + EquipmentTypeProperties + Properties + MeasurementUnits + PropertyEnumValues |  |

## 10. Пример ответа

```json
{
  "value": {
    "etpId": "a1b2c3d4-0001-4000-8000-000000000003",
    "sortOrder": 2,
    "isRequired": false,
    "isFilterable": true,
    "isVisibleInCard": true,
    "property": {
      "id": "p0000001-0000-4000-8000-000000000003",
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
      "enumValues": []
    }
  },
  "isSuccess": true,
  "errors": []
}
```

---

## Замечания

1. Метод не меняет `propertyId` и `equipmentTypeId`; он редактирует только параметры существующей привязки.
2. Для массовой перестановки нескольких характеристик frontend может вызывать метод последовательно для каждой строки.
