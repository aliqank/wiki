**Created:** 2026-05-12  
**Last updated:** 2026-05-12  
**Author:** Telman Nurzhanov (SA)

---

# GET /equipment-types/:id/properties

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить список характеристик, привязанных к типу техники |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Admin Panel` |
| Endpoint URL | `/api/admin/v1/equipment-types/:id/properties` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Возвращает только блок B карточки AP-02: список характеристик, привязанных к типу техники. Используется для адресного обновления блока характеристик без повторной загрузки всей карточки типа.

---

## 2. Функциональные требования

Общий перечень требований: Требования к системе DMMS

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-010 | Admin configures dynamic custom characteristics per equipment type via Admin Panel tool; characteristics appear automatically in equipment card, request form, and search filters based on equipment type | Confirmed | BRD v13 | Метод читает блок характеристик типа для AP-02 |
| TCO Booking Tool | FR-014 | Admin редактирует параметры техники и связанные справочные сущности | Confirmed | BRD v13 | Нужен для редактирования состава и порядка характеристик |
| TCO Booking Tool | FR-NEW-68 | System dynamically shows only type-specific characteristics in request creation form and equipment search filters, based on the selected equipment type | Confirmed | BRD v13 | Читает источник правды по характеристикам типа |

---

## 3. Описание логики работы метода

1. Найти запись в `EquipmentTypes` WHERE `id` = `:id` AND `isDeleted = false`. Если запись не найдена — вернуть `404 NOT_FOUND`.
2. Получить список характеристик типа: выбрать записи из `EquipmentTypeProperties` WHERE `equipmentTypeId` = `:id` AND `isDeleted = false`, JOIN `Properties` (WHERE `isDeleted = false`), LEFT JOIN `MeasurementUnits`, LEFT JOIN `PropertyEnumValues` (WHERE `isDeleted = false`).
3. Отсортировать результат по `EquipmentTypeProperties.sortOrder ASC`, затем по `EquipmentTypeProperties.createdAt ASC`.
4. Вернуть список в формате общего `result wrapper`.

Сущности, участвующие в методе:
- читаются: `EquipmentTypes`, `EquipmentTypeProperties`, `Properties`, `MeasurementUnits`, `PropertyEnumValues`
- изменения не выполняются
- транзакционность не требуется, метод read-only

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
GET /api/admin/v1/equipment-types/7b4f4b4d-52d4-4a77-b6b7-f2b7d7c81111/properties
Authorization: Bearer <token>
Content-Type: application/json
```

У метода нет request body.

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий `result wrapper` (использовать во всех API).  
В обёртке, в значении `value`, возвращается массив `EquipmentTypePropertyItem[]`.

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | `value` | `array<object>` | `EquipmentTypePropertyItem[]` | `[]` | `EquipmentTypeProperties` + joins | Список характеристик типа |
| 2 | Признак успешности | `isSuccess` | `bool` | boolean | — | backend | |
| 3 | Ошибки | `errors` | `array<object>` | `ApiError[]` | `[]` | backend | При успешном ответе — пустой массив |

### Структура `EquipmentTypePropertyItem`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи привязки характеристики | `etpId` | `uuid` | UUID v4 | — | `EquipmentTypeProperties.id` | |
| 2 | Порядок отображения характеристики | `sortOrder` | `int` | integer | — | `EquipmentTypeProperties.sortOrder` | |
| 3 | Обязательность характеристики | `isRequired` | `bool` | boolean | — | `EquipmentTypeProperties.isRequired` | |
| 4 | Используется в фильтрах поиска | `isFilterable` | `bool` | boolean | — | `EquipmentTypeProperties.isFilterable` | |
| 5 | Отображается в карточке техники | `isVisibleInCard` | `bool` | boolean | — | `EquipmentTypeProperties.isVisibleInCard` | |
| 6 | Данные характеристики из справочника | `property` | `object` | `PropertyDetail` | — | `Properties` | |
| 6.1 | Идентификатор характеристики | `id` | `uuid` | UUID v4 | — | `Properties.id` | |
| 6.2 | Наименование характеристики | `name` | `string` | string | — | `Properties.name` | |
| 6.3 | Тип данных | `dataType` | `enum` | `number / double / text / boolean / enum` | — | `Properties.dataType` | |
| 6.4 | Единица измерения | `unit` | `object` | `MeasurementUnitRef \| null` | `null` | `MeasurementUnits` | |
| 6.5 | Допустимые значения | `enumValues` | `array<object>` | `PropertyEnumValueRef[]` | `[]` | `PropertyEnumValues` | |

---

## 10. Пример ответа

```json
{
  "value": [
    {
      "etpId": "a1b2c3d4-0001-4000-8000-000000000001",
      "sortOrder": 1,
      "isRequired": true,
      "isFilterable": true,
      "isVisibleInCard": true,
      "property": {
        "id": "p0000001-0000-4000-8000-000000000001",
        "name": "Объём ресивера",
        "dataType": "double",
        "unit": {
          "id": "u0000001-0000-4000-8000-000000000001",
          "code": "л",
          "displayName": "Литр"
        },
        "enumValues": []
      }
    }
  ],
  "isSuccess": true,
  "errors": []
}
```

---

## Замечания

1. Метод возвращает только активные привязки `EquipmentTypeProperties WHERE isDeleted = false`.
2. Для загрузки полной карточки AP-02 можно использовать `GET /equipment-types/:id`; этот эндпоинт нужен для адресного обновления блока характеристик.
