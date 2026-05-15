**Created:** 2026-05-12  
**Last updated:** 2026-05-12  
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
2. Если передан `search`, применить фильтр по `Properties.name.En`, `Properties.name.Ru`, `Properties.name.Kz`.
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
| 1 | Поиск по названию характеристики | `search` | `string` | `-` | Если передан, используется как фильтр по `Properties.name.En`, `Properties.name.Ru`, `Properties.name.Kz` | — | Query param | |
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

В `value` возвращается `PaginatedResult<PropertyListItem>`.

### Структура `PropertyListItem`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор характеристики | `id` | `uuid` | UUID v4 | — | `Properties.id` | |
| 2 | Наименование характеристики | `name` | `object` | `LocalizedName` | — | `Properties.name` | `{ En, Ru, Kz }` |
| 3 | Тип данных | `dataType` | `enum` | `number / double / text / boolean / enum` | — | `Properties.dataType` | |
| 4 | Единица измерения | `unit` | `object` | `MeasurementUnitRef \| null` | `null` | `MeasurementUnits` | `null`, если `unitId IS NULL` |
| 5 | Количество enum-значений | `enumValuesCount` | `int` | integer | `0` | COUNT(`PropertyEnumValues`) | Актуально в основном для `dataType = enum` |
| 6 | Количество типов техники | `equipmentTypesCount` | `int` | integer | `0` | COUNT(`EquipmentTypeProperties`) | Нужен для guard удаления |

---

## 10. Пример ответа

```json
{
  "value": {
    "items": [
      {
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
