**Created:** 2026-05-08  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

# POST /equipment-types

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Создать новый тип техники в справочнике [`Admin`](../../../requirements/Roles and Access Model.md) Panel |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Admin Panel` |
| Endpoint URL | `/api/admin/v1/equipment-types` |
| Метод запроса | `POST` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Обеспечивает создание типа техники через форму AP-02 в [`Admin`](../../../requirements/Roles and Access Model.md) Panel. Тип техники является корневой справочной сущностью: к нему привязываются динамические характеристики (EquipmentTypeProperties) и единицы техники (Equipments). Метод поддерживает передачу стартового списка характеристик типа в одном запросе.

---

## 2. Функциональные требования

Общий перечень требований: Требования к системе DMMS

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-010 | [`Admin`](../../../requirements/Roles and Access Model.md) configures dynamic custom characteristics per equipment type via [`Admin`](../../../requirements/Roles and Access Model.md) Panel tool; characteristics appear automatically in equipment card, request form, and search filters based on equipment type | Confirmed | BRD v13 | Создание типа является предусловием для настройки его характеристик |
| TCO Booking Tool | FR-NEW-50 | [`Admin`](../../../requirements/Roles and Access Model.md) manages all reference/handbook values via [`Admin`](../../../requirements/Roles and Access Model.md) Panel | Confirmed | BRD v13 | Типы техники — справочная сущность, управляемая [`Admin`](../../../requirements/Roles and Access Model.md) Panel |
| TCO Booking Tool | FR-NEW-68 | System dynamically shows only type-specific characteristics in request creation form and equipment search filters, based on the selected equipment type | Confirmed | BRD v13 | Требует наличия созданных типов техники в справочнике |

---

## 3. Описание логики работы метода

1. Принять тело запроса; провалидировать обязательные поля (`name`, `mobilityType`, `requiresTransport`, `equipmentClass`).
2. Проверить объект `name`: должны быть переданы локализованные поля `En`, `Ru`, `Kz`; `name.En`, `name.Ru`, `name.Kz` обязательны.
3. Проверить уникальность `name` в таблице `EquipmentTypes` WHERE `isDeleted = false`. Проверка выполняется по правилу локализованной уникальности для набора `name.En`, `name.Ru`, `name.Kz`. Если запись с таким локализованным именем уже существует — вернуть `422 VALIDATION_ERROR`.
4. Проверить, что `mobilityType` входит в допустимые значения ENUM (`SelfPropelled / NonSelfPropelledMotorized / Stationary / NonMotorized`). Если нет — вернуть `422 VALIDATION_ERROR`.
5. Проверить, что `equipmentClass` входит в допустимые значения ENUM (`HDE / HDV`). Если нет — вернуть `422 VALIDATION_ERROR`.
6. Если передан `workCenterId`, проверить существование записи в `WorkCenters` WHERE `id` = `workCenterId` AND `isDeleted = false`. Если запись не найдена — вернуть `422 VALIDATION_ERROR`.
7. Если передан массив `properties`, провалидировать каждый элемент: `propertyId` должен ссылаться на активную запись `Properties`, `sortOrder` должен быть целым числом >= 1, `propertyId` не должен дублироваться внутри одного запроса. Если условие нарушено — вернуть `422 VALIDATION_ERROR`.
8. Если `sortOrder` равен `null`, вычислить значение как `MAX(sortOrder) + 1` по всем записям `EquipmentTypes` WHERE `isDeleted = false`. Если таблица пуста — присвоить `1`.
9. Создать новую запись в `EquipmentTypes`; заполнить аудит-поля `createdAt` (текущее время) и `createdBy` (ID аутентифицированного пользователя).
10. Для каждого элемента `properties[]` создать запись в `EquipmentTypeProperties`, сохранив `propertyId`, `isRequired`, `isFilterable`, `isVisibleInCard`, `sortOrder`; заполнить аудит-поля `createdAt` и `createdBy`.
11. Подтянуть `WorkCenters` по `workCenterId` для возврата полей `workCenterCode` и `workCenterName`.
12. Вернуть созданный объект в формате общего `result wrapper` с HTTP 201.

Сущности, участвующие в методе:
- читаются: `EquipmentTypes` (для проверки уникальности `name` и расчёта `sortOrder`), `WorkCenters` (валидация `workCenterId`, возврат `workCenterCode` / `workCenterName`), `Properties` (валидация `properties[].propertyId`)
- изменяются: `EquipmentTypes` (INSERT), `EquipmentTypeProperties` (batch INSERT)
- транзакционность: требуется единая транзакция, чтобы тип техники и его стартовые характеристики создавались атомарно

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
| `VALIDATION_ERROR` | Поле `name` не передано, одно из полей `name.En`, `name.Ru`, `name.Kz` пустое или локализованное имя уже существует в справочнике |
| `VALIDATION_ERROR` | Поле `mobilityType` содержит недопустимое значение |
| `VALIDATION_ERROR` | Поле `equipmentClass` содержит недопустимое значение |
| `VALIDATION_ERROR` | Передан несуществующий или удалённый `workCenterId` |
| `VALIDATION_ERROR` | В `properties[]` передан несуществующий или удалённый `propertyId` |
| `VALIDATION_ERROR` | В `properties[]` обнаружены дубли `propertyId` |
| `VALIDATION_ERROR` | В `properties[]` поле `sortOrder` имеет недопустимое значение |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Наименование типа техники | `name` | `object` | `+` | Объект `{ En, Ru, Kz }`; `name.En`, `name.Ru`, `name.Kz` обязательны; локализованное имя уникально среди `EquipmentTypes` WHERE `isDeleted = false` | — | Request body | |
| 2 | Тип мобильности | `mobilityType` | `enum` | `+` | Одно из: `SelfPropelled`, `NonSelfPropelledMotorized`, `Stationary`, `NonMotorized` | — | Request body | |
| 3 | Требуется ли транспортировка | `requiresTransport` | `bool` | `+` | Булево значение | — | Request body | |
| 4 | Класс техники | `equipmentClass` | `enum` | `+` | Одно из: `HDE`, `HDV` | — | Request body | |
| 5 | Идентификатор work center | `workCenterId` | `uuid` | `-` | Должен быть валидным UUID v4 и ссылаться на активный `WorkCenters.id`, если передан | `null` | Request body | Nullable, если тип техники не привязан к work center |
| 6 | Список стартовых характеристик типа | `properties` | `array<object>` | `-` | Массив без дубликатов `propertyId` | `[]` | Request body | Можно передать пустой массив |
| 6.1 | Идентификатор характеристики | `propertyId` | `uuid` | `+` | Должен ссылаться на активный `Properties.id` | — | Request body / properties[] | |
| 6.2 | Обязательность характеристики | `isRequired` | `bool` | `+` | Булево значение | — | Request body / properties[] | |
| 6.3 | Используется в фильтрах поиска | `isFilterable` | `bool` | `+` | Булево значение | — | Request body / properties[] | |
| 6.4 | Отображается в карточке техники | `isVisibleInCard` | `bool` | `+` | Булево значение | — | Request body / properties[] | |
| 6.5 | Порядок отображения характеристики | `sortOrder` | `int` | `+` | Целое число >= 1 | — | Request body / properties[] | |
| 7 | Порядок отображения типа техники | `sortOrder` | `int` | `-` | Целое число >= 1, если передано | `null` → MAX + 1 | Request body | При `null` backend вычисляет автоматически |

---

## 8. Пример запроса

```http
POST /api/admin/v1/equipment-types
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "name": {
    "En": "Excavator",
    "Ru": "Экскаватор",
    "Kz": "Экскаватор"
  },
  "mobilityType": "SelfPropelled",
  "requiresTransport": false,
  "equipmentClass": "HDV",
  "workCenterId": "12a8b1ce-3aaf-4f55-8ac8-f8cf5d86c222",
  "properties": [
    {
      "propertyId": "p0000001-0000-4000-8000-000000000001",
      "isRequired": true,
      "isFilterable": true,
      "isVisibleInCard": true,
      "sortOrder": 1
    },
    {
      "propertyId": "p0000001-0000-4000-8000-000000000002",
      "isRequired": false,
      "isFilterable": true,
      "isVisibleInCard": true,
      "sortOrder": 2
    }
  ],
  "sortOrder": null
}
```

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
| 1.5 | Класс техники | equipmentClass | string | string | — | EquipmentTypes + ref_equipment_class |  |
| 1.6 | Идентификатор work center | workCenterId | uuid | UUID v4 | — | WorkCenters.id |  |
| 1.7 | Код work center | workCenterCode | string | string | — | WorkCenters.code |  |
| 1.8 | Наименование work center | workCenterName | object | object | — | WorkCenters |  |
| 1.9 | Порядок сортировки | sortOrder | int | integer | — | EquipmentTypes.sortOrder |  |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | object[] | `[]` | backend |  |

### Структура `value`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | EquipmentTypes.id |  |
| 2 | Наименование | name | object | object | — | EquipmentTypes |  |
| 3 | Тип мобильности техники | mobilityType | string | string | — | EquipmentTypes + ref_equipment_mobility_type |  |
| 4 | Признак необходимости транспортировки | requiresTransport | bool | boolean | — | EquipmentTypes.requiresTransport |  |
| 5 | Класс техники | equipmentClass | string | string | — | EquipmentTypes + ref_equipment_class |  |
| 6 | Идентификатор work center | workCenterId | uuid | UUID v4 | — | WorkCenters.id |  |
| 7 | Код work center | workCenterCode | string | string | — | WorkCenters.code |  |
| 8 | Наименование work center | workCenterName | object | object | — | WorkCenters |  |
| 9 | Порядок сортировки | sortOrder | int | integer | — | EquipmentTypes.sortOrder |  |

### Структура `value.name`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string | string | — | EquipmentTypes |  |
| 2 | Значение на русском языке | Ru | string | string | — | EquipmentTypes |  |
| 3 | Значение на казахском языке | Kz | string | string | — | EquipmentTypes |  |

### Структура `value.workCenterName`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string | string | — | EquipmentTypes |  |
| 2 | Значение на русском языке | Ru | string | string | — | EquipmentTypes |  |
| 3 | Значение на казахском языке | Kz | string | string | — | EquipmentTypes |  |

## 10. Пример ответа

```json
{
  "value": {
    "id": "3e2a1c4d-88f1-4b3e-a9d2-c7f0e1234567",
    "name": {
      "En": "Excavator",
      "Ru": "Экскаватор",
      "Kz": "Экскаватор"
    },
    "mobilityType": "SelfPropelled",
    "requiresTransport": false,
    "equipmentClass": "HDV",
    "workCenterId": "12a8b1ce-3aaf-4f55-8ac8-f8cf5d86c222",
    "workCenterCode": "WC-100",
    "workCenterName": {
      "En": "Drilling Operations",
      "Ru": "Буровые работы",
      "Kz": "Бұрғылау жұмыстары"
    },
    "sortOrder": 16
  },
  "isSuccess": true,
  "errors": []
}
```

---

## Замечания

1. При `sortOrder: null` backend присваивает `MAX(sortOrder) + 1` по всем не удалённым записям `EquipmentTypes`. Если таблица пуста — присваивает `1`.
2. Уникальность `name` проверяется только среди активных записей (`isDeleted = false`); повторное использование локализованного имени удалённого типа допустимо.
3. Поля `workCenterId`, `workCenterCode`, `workCenterName` могут быть `null`, если тип техники не привязан к work center.
4. Если `properties[]` не передан, тип техники создаётся без стартовых характеристик; их можно добавить позднее отдельными эндпоинтами.
