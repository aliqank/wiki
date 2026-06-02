# GET /reference/equipment-types/{equipmentTypeId}/properties

**Created:** 2026-05-18  
**Last updated:** 2026-06-02  
**Автор документов:** OpenCode

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить динамические свойства выбранного типа техники для фильтров формы создания заявки |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Requestor UI` |
| Endpoint URL | `/api/booking/v1/reference/equipment-types/{equipmentTypeId}/properties` |
| Метод запроса | `GET` |
| Связанные use cases | [`UC-REQ-02 - Создание новой заявки (Draft-first)`](../../../requirements/usecases/Requestor/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first)/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first).md), [`UC-REQ-02.3 - Поиск техники для добавления в заявку`](../../../requirements/usecases/Requestor/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first)/UC-REQ-02.3%20-%20Поиск%20техники%20для%20добавления%20в%20заявку.md) |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый reference API для UC-2. Используется после выбора `equipmentType` для построения динамических фильтров поиска.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-NEW-39 | Dynamic search filters by equipment type | Confirmed | BRD v13 | Метод возвращает конфигурацию динамических фильтров |
| TCO Booking Tool | FR-NEW-68 | System dynamically shows only type-specific characteristics | Confirmed | BRD v13 | Набор свойств зависит от выбранного `equipmentTypeId` |

---

## 3. Описание логики работы метода

1. Проверить, что `equipmentTypeId` существует и активен.
2. Получить свойства типа техники, помеченные как доступные для поиска.
3. Для enum-свойств вернуть список допустимых значений.
4. Для numeric-свойств вернуть единицу измерения, если она настроена.

Сущности, участвующие в методе:
- читаются: [`EquipmentTypes`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#1-equipmenttypes), [`Properties`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#12-properties), [`EquipmentTypeProperties`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#14-equipmenttypeproperties), [`PropertyEnumValues`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#13-propertyenumvalues), [`MeasurementUnits`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#11-measurementunits)
- изменения не выполняются

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Requestor` | Создание и редактирование собственных заявок |
| `ServiceWorkProcessor` | Работа с Service Work Requests |

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
| `FORBIDDEN` | Нет необходимой роли |
| `NOT_FOUND` | Тип техники не найден |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор типа техники | `equipmentTypeId` | `uuid` | `+` | Должен существовать | — | Path param | |

---

## 8. Пример запроса

```http
GET /api/booking/v1/reference/equipment-types/7b4f4b4d-52d4-4a77-b6b7-f2b7d7c81111/properties
Authorization: Bearer <token>
Content-Type: application/json
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий `result wrapper`.

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | value | array<object> | object[] | — | backend aggregation |  |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | object[] | `[]` | backend |  |

### Структура `value[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор свойства | propertyId | uuid | UUID v4 | — | Properties.id |  |
| 2 | Код свойства | code | string | string | — | Properties.code |  |
| 3 | Наименование свойства | name | object | object | — | Properties | Локализованное значение |
| 4 | Тип данных свойства | dataType | string | string | — | Properties.dataType | Например: `string`, `number`, `enum`, `bool` |
| 5 | Признак множественного выбора | allowsMultiple | bool | boolean | — | EquipmentTypeProperties.allowsMultiple | Актуально для enum |
| 6 | Единица измерения | unit | object | object | — | MeasurementUnits | Для numeric-свойств; иначе `null` |
| 7 | Список допустимых enum-значений | enumValues | array<object> | object[] | `[]` | PropertyEnumValues | Для non-enum возвращается пустой массив |

### Структура `value[].name`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string | string | — | Properties |  |
| 2 | Значение на русском языке | Ru | string | string | — | Properties |  |
| 3 | Значение на казахском языке | Kz | string | string | — | Properties |  |

### Структура `value[].unit`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Код единицы измерения | code | string | string | — | MeasurementUnits.code |  |
| 2 | Наименование единицы измерения | name | string | string | — | MeasurementUnits.name |  |

### Структура `value[].enumValues[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Код значения | code | string | string | — | PropertyEnumValues.code |  |
| 2 | Наименование значения | name | object | object | — | PropertyEnumValues | Локализованное значение |

### Структура `value[].enumValues[].name`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string | string | — | PropertyEnumValues |  |
| 2 | Значение на русском языке | Ru | string | string | — | PropertyEnumValues |  |
| 3 | Значение на казахском языке | Kz | string | string | — | PropertyEnumValues |  |

## 10. Пример ответа

```json
{
  "value": [
    {
      "propertyId": "a1b2c3d4-0001-4000-8000-000000000003",
      "code": "BUCKET_TYPE",
      "name": {
        "En": "Bucket type",
        "Ru": "Тип ковша",
        "Kz": "Шөміш түрі"
      },
      "dataType": "enum",
      "allowsMultiple": false,
      "unit": null,
      "enumValues": [
        {
          "code": "STANDARD",
          "name": {
            "En": "Standard",
            "Ru": "Стандартный",
            "Kz": "Стандартты"
          }
        }
      ]
    }
  ],
  "isSuccess": true,
  "errors": []
}
```
