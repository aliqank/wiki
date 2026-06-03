**Created:** 2026-05-12  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

# POST /equipment-types/:id/properties

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Добавить новую характеристику к типу техники |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Admin Panel` |
| Endpoint URL | `/api/admin/v1/equipment-types/:id/properties` |
| Метод запроса | `POST` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Добавляет одну характеристику в блок B карточки AP-02 без повторного сохранения базовых параметров типа.

---

## 2. Функциональные требования

Общий перечень требований: Требования к системе DMMS

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-010 | Admin configures dynamic custom characteristics per equipment type via Admin Panel tool; characteristics appear automatically in equipment card, request form, and search filters based on equipment type | Confirmed | BRD v13 | Метод добавляет новую привязку характеристики к типу |
| TCO Booking Tool | FR-014 | Admin редактирует параметры техники и связанные справочные сущности | Confirmed | BRD v13 | Нужен для управления составом характеристик типа |
| TCO Booking Tool | FR-NEW-68 | System dynamically shows only type-specific characteristics in request creation form and equipment search filters, based on the selected equipment type | Confirmed | BRD v13 | Новая привязка влияет на выдачу UI и фильтров |

---

## 3. Описание логики работы метода

1. Найти запись в `EquipmentTypes` WHERE `id` = `:id` AND `isDeleted = false`. Если запись не найдена — вернуть `404 NOT_FOUND`.
2. Провалидировать `propertyId`: должен ссылаться на активную запись `Properties`.
3. Проверить, что активная привязка `EquipmentTypeProperties` с тем же `equipmentTypeId` и `propertyId` ещё не существует. Если существует — вернуть `409 PROPERTY_ALREADY_LINKED`.
4. Провалидировать `sortOrder`: целое число >= 1.
5. Создать запись в `EquipmentTypeProperties`; заполнить аудит-поля `createdAt` и `createdBy`.
6. Вернуть созданный объект `EquipmentTypePropertyItem` в формате общего `result wrapper` с HTTP 201.

Сущности, участвующие в методе:
- читаются: `EquipmentTypes`, `Properties`, `EquipmentTypeProperties`, `MeasurementUnits`, `PropertyEnumValues`
- изменяются: `EquipmentTypeProperties` (INSERT)
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
| `NOT_FOUND` | Тип техники с указанным `id` не найден или помечен как удалённый |
| `VALIDATION_ERROR` | Передан несуществующий или удалённый `propertyId` |
| `VALIDATION_ERROR` | Поле `sortOrder` имеет недопустимое значение |
| `PROPERTY_ALREADY_LINKED` | Характеристика уже привязана к данному типу техники |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор типа техники | `id` | `uuid` | `+` | Должен быть валидным UUID v4 | — | Path param | |
| 2 | Идентификатор характеристики | `propertyId` | `uuid` | `+` | Должен ссылаться на активный `Properties.id` | — | Request body | |
| 3 | Обязательность характеристики | `isRequired` | `bool` | `+` | Булево значение | — | Request body | |
| 4 | Используется в фильтрах поиска | `isFilterable` | `bool` | `+` | Булево значение | — | Request body | |
| 5 | Отображается в карточке техники | `isVisibleInCard` | `bool` | `+` | Булево значение | — | Request body | |
| 6 | Порядок отображения характеристики | `sortOrder` | `int` | `+` | Целое число >= 1 | — | Request body | |

---

## 8. Пример запроса

```http
POST /api/admin/v1/equipment-types/7b4f4b4d-52d4-4a77-b6b7-f2b7d7c81111/properties
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "propertyId": "p0000001-0000-4000-8000-000000000003",
  "isRequired": true,
  "isFilterable": false,
  "isVisibleInCard": true,
  "sortOrder": 3
}
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий [`result wrapper`](../../common/Result%20Wrapper.md).

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
| 2 | Наименование | name | object | object | — | backend composition from EquipmentTypes + Properties + EquipmentTypeProperties + MeasurementUnits + PropertyEnumValues |  |
| 3 | Тип данных свойства | dataType | string | string | — | backend composition from EquipmentTypes + Properties + EquipmentTypeProperties + MeasurementUnits + PropertyEnumValues |  |
| 4 | Единица измерения | unit | object | object | — | backend composition from EquipmentTypes + Properties + EquipmentTypeProperties + MeasurementUnits + PropertyEnumValues |  |
| 5 | Список enum-значений | enumValues | array<object> | object[] | `[]` | backend composition from EquipmentTypes + Properties + EquipmentTypeProperties + MeasurementUnits + PropertyEnumValues | Коллекция объектов |

### Структура `value.property.name`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string | string | — | backend composition from EquipmentTypes + Properties + EquipmentTypeProperties + MeasurementUnits + PropertyEnumValues |  |
| 2 | Значение на русском языке | Ru | string | string | — | backend composition from EquipmentTypes + Properties + EquipmentTypeProperties + MeasurementUnits + PropertyEnumValues |  |
| 3 | Значение на казахском языке | Kz | string | string | — | backend composition from EquipmentTypes + Properties + EquipmentTypeProperties + MeasurementUnits + PropertyEnumValues |  |

### Структура `value.property.unit`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | string | string | — | EquipmentTypeProperties.id |  |
| 2 | Код записи | code | string | string | — | backend composition from EquipmentTypes + Properties + EquipmentTypeProperties + MeasurementUnits + PropertyEnumValues |  |
| 3 | Отображаемое наименование | displayName | string | string | — | backend composition from EquipmentTypes + Properties + EquipmentTypeProperties + MeasurementUnits + PropertyEnumValues |  |

## 10. Пример ответа

```json
{
  "value": {
    "etpId": "a1b2c3d4-0001-4000-8000-000000000003",
    "sortOrder": 3,
    "isRequired": true,
    "isFilterable": false,
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

1. Одна и та же характеристика не может быть привязана к одному типу техники более одного раза.
2. После успешного создания frontend может либо вставить полученный объект в локальный список, либо перечитать `GET /equipment-types/:id/properties`.
