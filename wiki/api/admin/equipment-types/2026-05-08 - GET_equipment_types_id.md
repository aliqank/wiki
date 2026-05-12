**Created:** 2026-05-08  
**Last updated:** 2026-05-08  
**Author:** Telman Nurzhanov (SA)

---

# GET /equipment-types/:id

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить карточку типа техники с полным списком его характеристик для экрана AP-02 в Admin Panel |
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
| TCO Booking Tool | FR-010 | Admin configures dynamic custom characteristics per equipment type via Admin Panel tool; characteristics appear automatically in equipment card, request form, and search filters based on equipment type | Confirmed | BRD v13 | Метод загружает карточку типа вместе с привязанными характеристиками для редактирования в AP-02 |
| TCO Booking Tool | FR-NEW-50 | Admin manages all reference/handbook values via Admin Panel | Confirmed | BRD v13 | Типы техники — справочная сущность, управляемая Admin Panel |
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

Возвращаемые данные обёрнуты в общий `result wrapper` (использовать во всех API).  
В обёртке, в значении `value`, возвращается объект типа `EquipmentTypeDetail`.

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | `value` | `object` | `EquipmentTypeDetail` | — | `EquipmentTypes` + joins | Полная карточка типа |
| 2 | Признак успешности | `isSuccess` | `bool` | boolean | — | backend | |
| 3 | Ошибки | `errors` | `array<object>` | `ApiError[]` | `[]` | backend | При успешном ответе — пустой массив |

### Структура `EquipmentTypeDetail`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор типа техники | `id` | `uuid` | UUID v4 | — | `EquipmentTypes.id` | |
| 2 | Наименование типа техники | `name` | `object` | `LocalizedName` | — | `EquipmentTypes.name` | `{ En, Ru, Kz }` |
| 3 | Тип мобильности | `mobilityType` | `enum` | `SelfPropelled / NonSelfPropelledMotorized / Stationary / NonMotorized` | — | `EquipmentTypes.mobilityType` | |
| 4 | Требуется ли транспортировка | `requiresTransport` | `bool` | boolean | — | `EquipmentTypes.requiresTransport` | |
| 5 | Порядок отображения | `sortOrder` | `int` | integer | — | `EquipmentTypes.sortOrder` | |
| 6 | Количество единиц техники типа | `equipmentsCount` | `int` | integer | `0` | COUNT(`Equipments`) | Нужен для guard удаления типа |
| 7 | Список характеристик типа | `properties` | `array<object>` | `EquipmentTypePropertyItem[]` | `[]` | `EquipmentTypeProperties` + joins | Отсортирован по `sortOrder ASC` |
| 7.1 | Идентификатор записи привязки характеристики | `etpId` | `uuid` | UUID v4 | — | `EquipmentTypeProperties.id` | |
| 7.2 | Порядок отображения характеристики | `sortOrder` | `int` | integer | — | `EquipmentTypeProperties.sortOrder` | |
| 7.3 | Обязательность характеристики | `isRequired` | `bool` | boolean | — | `EquipmentTypeProperties.isRequired` | |
| 7.4 | Используется в фильтрах поиска | `isFilterable` | `bool` | boolean | — | `EquipmentTypeProperties.isFilterable` | |
| 7.5 | Отображается в карточке техники | `isVisibleInCard` | `bool` | boolean | — | `EquipmentTypeProperties.isVisibleInCard` | |
| 7.6 | Данные характеристики из справочника | `property` | `object` | `PropertyDetail` | — | `Properties` | |
| 7.6.1 | Идентификатор характеристики | `id` | `uuid` | UUID v4 | — | `Properties.id` | |
| 7.6.2 | Наименование характеристики | `name` | `object` | `LocalizedName` | — | `Properties.name` | `{ En, Ru, Kz }` |
| 7.6.3 | Тип данных | `dataType` | `enum` | `number / double / text / boolean / enum` | — | `Properties.dataType` | |
| 7.6.4 | Единица измерения (nullable) | `unit` | `object` | `MeasurementUnitRef \| null` | `null` | `MeasurementUnits` | `null` если `Properties.unitId IS NULL` |
| 7.6.4.1 | Идентификатор единицы измерения | `id` | `uuid` | UUID v4 | — | `MeasurementUnits.id` | |
| 7.6.4.2 | Код единицы измерения | `code` | `string` | string | — | `MeasurementUnits.code` | |
| 7.6.4.3 | Отображаемое название | `displayName` | `string` | string | — | `MeasurementUnits.displayName` | |
| 7.6.5 | Допустимые значения (только для enum) | `enumValues` | `array<object>` | `PropertyEnumValueRef[]` | `[]` | `PropertyEnumValues` | Пустой массив если `dataType ≠ enum` |
| 7.6.5.1 | Идентификатор значения | `id` | `uuid` | UUID v4 | — | `PropertyEnumValues.id` | |
| 7.6.5.2 | Значение | `value` | `string` | string | — | `PropertyEnumValues.value` | |
| 7.6.5.3 | Порядок отображения | `sortOrder` | `int` | integer | — | `PropertyEnumValues.sortOrder` | |

---

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
