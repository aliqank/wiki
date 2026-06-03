# GET /equipment-types

**Created:** 2026-05-08  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить постраничный список типов техники для экрана AP-01 в Admin Panel |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Admin Panel` |
| Endpoint URL | `/api/admin/v1/equipment-types` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод публикации в `wiki` для dev-ready описания списка типов техники.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-010 | Admin конфигурирует динамические характеристики по типам техники через Admin Panel tool | Confirmed | BRD v13 | Метод нужен для списка типов техники в Admin Panel |
| TCO Booking Tool | FR-013 | Admin создаёт equipment types / equipment-related master data | Confirmed | BRD v13 | Метод чтения списка нужен для AP-01 |
| TCO Booking Tool | FR-014 | Admin редактирует параметры техники и связанные справочные сущности | Confirmed | BRD v13 | Список типов техники является входной точкой к карточке типа |

---

## 3. Описание логики работы метода

1. Получить записи из `EquipmentTypes` WHERE `isDeleted = false`.
2. Если передан `search`, применить фильтр по `EquipmentTypes.name.En`, `EquipmentTypes.name.Ru`, `EquipmentTypes.name.Kz`.
3. Подтянуть данные `WorkCenters` по `EquipmentTypes.workCenterId` для возврата названия и кода work center.
4. Отсортировать результат по `sortOrder ASC`, затем по `name.Ru ASC`.
5. Для каждой записи рассчитать:
   - `propertiesCount` = COUNT(`EquipmentTypeProperties` WHERE `equipmentTypeId` = `EquipmentTypes.id`)
   - `equipmentsCount` = COUNT(`Equipments` WHERE `equipmentTypeId` = `EquipmentTypes.id` AND `isDeleted = false`)
6. Вернуть страницу данных в формате общего `result wrapper` с `PaginatedResult` внутри `value`.

Сущности, участвующие в методе:
- читаются: `EquipmentTypes`, `WorkCenters`, `EquipmentTypeProperties`, `Equipments`
- изменения не выполняются
- транзакционность не требуется, метод read-only

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Admin` | Доступ к административной панели и справочникам типов техники |

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
| 1 | Поисковая строка по названию типа техники | `search` | `string` | `-` | Если передан, используется как фильтр по `EquipmentTypes.name.En`, `EquipmentTypes.name.Ru`, `EquipmentTypes.name.Kz` | — | Query param | |
| 2 | Номер страницы | `page` | `int` | `-` | Целое число >= 1 | `1` | Query param | |
| 3 | Размер страницы | `limit` | `int` | `-` | Целое число >= 1 | `20` | Query param | |

---

## 8. Пример запроса

```http
GET /api/admin/v1/equipment-types?search=Компрессор&page=1&limit=20
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
| 1 | Результат выполнения метода | value | object | PaginatedResult | — | backend aggregation |  |
| 1.1 | Элементы текущей страницы | items | array<object> | object[] | — | backend composition from EquipmentTypes + WorkCenters + EquipmentTypeProperties + Equipments | Коллекция объектов |
| 1.2 | Общее количество записей | total | int | integer | — | backend |  |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | object[] | `[]` | backend |  |

### Структура `value`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Элементы текущей страницы | items | array<object> | object[] | — | backend composition from EquipmentTypes + WorkCenters + EquipmentTypeProperties + Equipments | Коллекция объектов |
| 2 | Общее количество записей | total | int | integer | — | backend |  |

### Структура `value.items[]`

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
| 10 | Количество свойств | propertiesCount | int | integer | — | COUNT(EquipmentTypeProperties) |  |
| 11 | Количество единиц техники | equipmentsCount | int | integer | — | backend composition from EquipmentTypes + WorkCenters + EquipmentTypeProperties + Equipments |  |

### Структура `value.items[].name`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string | string | — | EquipmentTypes |  |
| 2 | Значение на русском языке | Ru | string | string | — | EquipmentTypes |  |
| 3 | Значение на казахском языке | Kz | string | string | — | EquipmentTypes |  |

### Структура `value.items[].workCenterName`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string | string | — | EquipmentTypes |  |
| 2 | Значение на русском языке | Ru | string | string | — | EquipmentTypes |  |
| 3 | Значение на казахском языке | Kz | string | string | — | EquipmentTypes |  |

## 10. Пример ответа

```json
{
  "value": {
    "items": [
      {
        "id": "7b4f4b4d-52d4-4a77-b6b7-f2b7d7c81111",
        "name": {
          "En": "Compressor",
          "Ru": "Компрессор",
          "Kz": "Компрессор"
        },
        "mobilityType": "Stationary",
        "requiresTransport": true,
        "equipmentClass": "HDE",
        "workCenterId": "12a8b1ce-3aaf-4f55-8ac8-f8cf5d86c222",
        "workCenterCode": "WC-100",
        "workCenterName": {
          "En": "Drilling Operations",
          "Ru": "Буровые работы",
          "Kz": "Бұрғылау жұмыстары"
        },
        "sortOrder": 1,
        "propertiesCount": 7,
        "equipmentsCount": 12
      }
    ],
    "total": 15
  },
  "isSuccess": true,
  "errors": []
}
```

---

## Замечания

1. Метод применяет скрытый фильтр `isDeleted = false` как к `EquipmentTypes`, так и к `Equipments` при расчёте `equipmentsCount`. Параметр не выставляется наружу — удалённые записи никогда не попадают в список.
2. `equipmentsCount` используется в UI для guard-логики удаления типа техники (блокировать удаление, если счётчик > 0).
3. Поля `workCenterId`, `workCenterCode`, `workCenterName` могут быть `null`, если для типа техники work center ещё не назначен.
4. Поля `name` и `workCenterName` возвращаются как локализованные объекты `{ En, Ru, Kz }`.
