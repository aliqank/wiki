**Created:** 2026-05-08  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

# GET /equipment-types/:id

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить карточку типа техники с полным списком его характеристик для экрана AP-02 в [`Admin`](../../../requirements/Roles and Access Model.md) Panel |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Admin Panel` |
| Endpoint URL | `/api/admin/v1/equipment-types/:id` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Возвращает полную карточку типа техники: базовые параметры (блок A) и список привязанных характеристик (блок B) за один HTTP round-trip. Используется при открытии экрана AP-02.

---

## 2. Функциональные требования

Общий перечень требований: Требования к системе DMMS

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-010 | [`Admin`](../../../requirements/Roles and Access Model.md) configures dynamic custom characteristics per equipment type via [`Admin`](../../../requirements/Roles and Access Model.md) Panel tool; characteristics appear automatically in equipment card, request form, and search filters based on equipment type | Confirmed | BRD v13 | Метод загружает карточку типа вместе с привязанными характеристиками для редактирования в AP-02 |
| TCO Booking Tool | FR-NEW-50 | [`Admin`](../../../requirements/Roles and Access Model.md) manages all reference/handbook values via [`Admin`](../../../requirements/Roles and Access Model.md) Panel | Confirmed | BRD v13 | Типы техники — справочная сущность, управляемая [`Admin`](../../../requirements/Roles and Access Model.md) Panel |
| TCO Booking Tool | FR-NEW-68 | System dynamically shows only type-specific characteristics in request creation form and equipment search filters, based on the selected equipment type | Confirmed | BRD v13 | Требует корректно настроенных характеристик на уровне типа |

---

## 3. Описание логики работы метода

1. Найти запись в `EquipmentTypes` WHERE `id` = `:id` AND `isDeleted = false`. Если запись не найдена — вернуть `404 NOT_FOUND`.
2. Рассчитать `equipmentsCount` = COUNT(`Equipments` WHERE `equipmentTypeId` = `:id` AND `isDeleted = false`).
3. Получить список характеристик типа: выбрать записи из `EquipmentTypeProperties` WHERE `equipmentTypeId` = `:id` AND `isDeleted = false`, JOIN `Properties` (WHERE `isDeleted = false`), LEFT JOIN `MeasurementUnits`, LEFT JOIN `PropertyEnumValues` (WHERE `isDeleted = false`). Отсортировать по `EquipmentTypeProperties.sortOrder ASC`.
4. Собрать объект `EquipmentTypeDetail` и вернуть его в формате общего `result wrapper` с HTTP 200.

Сущности, участвующие в методе:
- читаются: `EquipmentTypes`, `Equipments` (агрегат), `EquipmentTypeProperties`, `Properties`, `MeasurementUnits`, `PropertyEnumValues`
- изменения не выполняются
- транзакционность: не требуется, метод read-only

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| [`Admin`](../../../requirements/Roles and Access Model.md) | Доступ к административной панели и управлению справочниками типов техники |

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
| `NOT_FOUND` | Тип техники с указанным `id` не найден или помечен как удалённый |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор типа техники | `id` | `uuid` | `+` | Должен быть валидным UUID v4 | — | Path param | |

---

## 8. Пример запроса

```http
GET /api/admin/v1/equipment-types/7b4f4b4d-52d4-4a77-b6b7-f2b7d7c81111
Authorization: Bearer <token>
Content-Type: application/json
```

У метода нет request body.

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий [`result wrapper`](../../common/Result%20Wrapper.md).

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | value | object | object | — | backend aggregation |  |
| 1.1 | Идентификатор записи | id | uuid | UUID v4 | — | EquipmentTypes.id |  |
| 1.2 | Наименование | name | object | object | — | EquipmentTypes |  |
| 1.3 | Тип мобильности техники | mobilityType | string | string | — | EquipmentTypes + ref_equipment_mobility_type |  |
| 1.4 | Признак необходимости транспортировки | requiresTransport | bool | boolean | — | EquipmentTypes.requiresTransport |  |
| 1.5 | Порядок сортировки | sortOrder | int | integer | — | EquipmentTypes.sortOrder |  |
| 1.6 | Количество единиц техники | equipmentsCount | int | integer | — | backend composition from EquipmentTypes + Equipments + EquipmentTypeProperties + Properties + MeasurementUnits + PropertyEnumValues |  |
| 1.7 | Список свойств | properties | array<object> | object[] | — | backend composition from EquipmentTypes + Equipments + EquipmentTypeProperties + Properties + MeasurementUnits + PropertyEnumValues | Коллекция объектов |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | object[] | `[]` | backend |  |

### Структура `value`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | EquipmentTypes.id |  |
| 2 | Наименование | name | object | object | — | EquipmentTypes |  |
| 3 | Тип мобильности техники | mobilityType | string | string | — | EquipmentTypes + ref_equipment_mobility_type |  |
| 4 | Признак необходимости транспортировки | requiresTransport | bool | boolean | — | EquipmentTypes.requiresTransport |  |
| 5 | Порядок сортировки | sortOrder | int | integer | — | EquipmentTypes.sortOrder |  |
| 6 | Количество единиц техники | equipmentsCount | int | integer | — | backend composition from EquipmentTypes + Equipments + EquipmentTypeProperties + Properties + MeasurementUnits + PropertyEnumValues |  |
| 7 | Список свойств | properties | array<object> | object[] | — | backend composition from EquipmentTypes + Equipments + EquipmentTypeProperties + Properties + MeasurementUnits + PropertyEnumValues | Коллекция объектов |

### Структура `value.name`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string | string | — | EquipmentTypes |  |
| 2 | Значение на русском языке | Ru | string | string | — | EquipmentTypes |  |
| 3 | Значение на казахском языке | Kz | string | string | — | EquipmentTypes |  |

### Структура `value.properties[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор привязки свойства к типу техники | etpId | uuid | UUID v4 | — | backend composition from EquipmentTypes + Equipments + EquipmentTypeProperties + Properties + MeasurementUnits + PropertyEnumValues |  |
| 2 | Порядок сортировки | sortOrder | int | integer | — | EquipmentTypes.sortOrder |  |
| 3 | Признак обязательности поля | isRequired | bool | boolean | — | backend composition from EquipmentTypes + Equipments + EquipmentTypeProperties + Properties + MeasurementUnits + PropertyEnumValues |  |
| 4 | Признак доступности свойства в фильтрах | isFilterable | bool | boolean | — | backend composition from EquipmentTypes + Equipments + EquipmentTypeProperties + Properties + MeasurementUnits + PropertyEnumValues |  |
| 5 | Признак отображения в карточке | isVisibleInCard | bool | boolean | — | backend composition from EquipmentTypes + Equipments + EquipmentTypeProperties + Properties + MeasurementUnits + PropertyEnumValues |  |
| 6 | Наименование свойства | property | object | object | — | backend composition from EquipmentTypes + Equipments + EquipmentTypeProperties + Properties + MeasurementUnits + PropertyEnumValues |  |

### Структура `value.properties[].property`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | string | string | — | EquipmentTypes.id |  |
| 2 | Наименование | name | object | object | — | EquipmentTypes |  |
| 3 | Тип данных свойства | dataType | string | string | — | backend composition from EquipmentTypes + Equipments + EquipmentTypeProperties + Properties + MeasurementUnits + PropertyEnumValues |  |
| 4 | Единица измерения | unit | object | object | — | backend composition from EquipmentTypes + Equipments + EquipmentTypeProperties + Properties + MeasurementUnits + PropertyEnumValues |  |
| 5 | Список enum-значений | enumValues | array<object> | object[] | `[]` | backend composition from EquipmentTypes + Equipments + EquipmentTypeProperties + Properties + MeasurementUnits + PropertyEnumValues | Коллекция объектов |

### Структура `value.properties[].property.name`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string | string | — | EquipmentTypes |  |
| 2 | Значение на русском языке | Ru | string | string | — | EquipmentTypes |  |
| 3 | Значение на казахском языке | Kz | string | string | — | EquipmentTypes |  |

### Структура `value.properties[].property.unit`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | string | string | — | EquipmentTypes.id |  |
| 2 | Код записи | code | string | string | — | backend composition from EquipmentTypes + Equipments + EquipmentTypeProperties + Properties + MeasurementUnits + PropertyEnumValues |  |
| 3 | Отображаемое наименование | displayName | string | string | — | backend composition from EquipmentTypes + Equipments + EquipmentTypeProperties + Properties + MeasurementUnits + PropertyEnumValues |  |

## 10. Пример ответа

```json
{
  "value": {
    "id": "7b4f4b4d-52d4-4a77-b6b7-f2b7d7c81111",
    "name": {
      "En": "Compressor",
      "Ru": "Компрессор",
      "Kz": "Компрессор"
    },
    "mobilityType": "Stationary",
    "requiresTransport": true,
    "sortOrder": 1,
    "equipmentsCount": 12,
    "properties": [
      {
        "etpId": "a1b2c3d4-0001-4000-8000-000000000001",
        "sortOrder": 1,
        "isRequired": true,
        "isFilterable": true,
        "isVisibleInCard": true,
        "property": {
          "id": "p0000001-0000-4000-8000-000000000001",
          "name": {
            "En": "Receiver volume",
            "Ru": "Объём ресивера",
            "Kz": "Ресивер көлемі"
          },
          "dataType": "double",
          "unit": {
            "id": "u0000001-0000-4000-8000-000000000001",
            "code": "л",
            "displayName": "Литр"
          },
          "enumValues": []
        }
      },
      {
        "etpId": "a1b2c3d4-0001-4000-8000-000000000002",
        "sortOrder": 2,
        "isRequired": false,
        "isFilterable": true,
        "isVisibleInCard": true,
        "property": {
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
          ]
        }
      }
    ]
  },
  "isSuccess": true,
  "errors": []
}
```

---

## Замечания

1. Список `properties` включён непосредственно в ответ карточки (не запрашивается отдельно) — один HTTP round-trip для загрузки AP-02.
2. Метод применяет скрытый фильтр `isDeleted = false` ко всем участвующим таблицам: `EquipmentTypes`, `Equipments`, `EquipmentTypeProperties`, `Properties`, `PropertyEnumValues`.
3. `equipmentsCount` используется во frontend для guard-логики удаления типа техники (блокировать `DELETE` если счётчик > 0).
4. Поля `name` внутри типа техники и внутри вложенных характеристик возвращаются как локализованные объекты `{ En, Ru, Kz }`.
