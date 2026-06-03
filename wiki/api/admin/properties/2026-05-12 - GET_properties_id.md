**Created:** 2026-05-12  
**Last updated:** 2026-05-22  
**Автор документов:** Telman Nurzhanov (SA)

---

# GET /properties/:id

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить карточку характеристики |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Admin Panel` |
| Endpoint URL | `/api/admin/v1/properties/:id` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Возвращает полную карточку характеристики, включая единицу измерения и enum-значения.

---

## 2. Функциональные требования

Общий перечень требований: Требования к системе DMMS

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-010 | Admin configures dynamic custom characteristics per equipment type via Admin Panel tool | Confirmed | BRD v13 | Метод нужен для загрузки карточки характеристики |
| TCO Booking Tool | FR-014 | Admin редактирует параметры техники и связанные справочные сущности | Confirmed | BRD v13 | Нужен для редактирования |

---

## 3. Описание логики работы метода

1. Найти запись в `Properties` WHERE `id` = `:id` AND `isDeleted = false`. Если запись не найдена — вернуть `404 NOT_FOUND`.
2. Подтянуть `MeasurementUnits` по `unitId`.
3. Получить список `PropertyEnumValues` WHERE `propertyId` = `:id` AND `isDeleted = false`, отсортировать по `sortOrder ASC`.
4. Рассчитать `equipmentTypesCount` = COUNT(`EquipmentTypeProperties` WHERE `propertyId` = `:id` AND `isDeleted = false`).
5. Вернуть объект `PropertyDetail` в формате общего `result wrapper`.

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
| `NOT_FOUND` | Характеристика не найдена |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор характеристики | `id` | `uuid` | `+` | Валидный UUID v4 | — | Path param | |

---

## 8. Пример запроса

```http
GET /api/admin/v1/properties/p0000001-0000-4000-8000-000000000003
Authorization: Bearer <token>
Content-Type: application/json
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий [`result wrapper`](../../common/Result%20Wrapper.md).

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | value | object | object | — | backend aggregation |  |
| 1.1 | Идентификатор записи | id | string | string | — | Properties.id |  |
| 1.2 | Код характеристики | code | string | string | — | Properties.code |  |
| 1.3 | Наименование | name | object | object | — | Properties |  |
| 1.4 | Тип данных свойства | dataType | string | string | — | Properties + ref_property_data_type |  |
| 1.5 | Единица измерения | unit | object | object | — | MeasurementUnits |  |
| 1.6 | Список enum-значений | enumValues | array<object> | object[] | `[]` | PropertyEnumValues | Коллекция объектов |
| 1.7 | Количество типов техники | equipmentTypesCount | int | integer | — | COUNT(EquipmentTypeProperties) |  |
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
    "enumValues": [],
    "equipmentTypesCount": 4
  },
  "isSuccess": true,
  "errors": []
}
```

---

## Замечания

1. Для `dataType != enum` поле `enumValues` всегда пустое.
2. Поле `name` возвращается как локализованный объект `{ En, Ru, Kz }`.
