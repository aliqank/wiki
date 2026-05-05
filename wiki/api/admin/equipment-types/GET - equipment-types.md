# GET /equipment-types

**Модуль:** Admin Panel  
**Base URL:** `/api/admin/v1`  
**Endpoint URL:** `/equipment-types`  
**Метод:** `GET`  
**Авторизация:** `Authorization: Bearer <token>`  
**Роль:** `Admin`

---

## Назначение

Метод возвращает постраничный список типов техники для экрана AP-01 в Admin Panel.

Use case:
- `UC-ET-01` — просмотр списка типов техники с поиском.

Источник:
- `Результаты claude/2026-05-05 - API спецификация Admin Panel v1.md`
- `Результаты claude/2026-05-04 - Схема БД v5 (Equipments, Bookings).md`

---

## Query Params

| Параметр | Тип | Обязательный | По умолчанию | Описание |
|----------|-----|--------------|--------------|----------|
| `search` | `string` | нет | — | Фильтр по `EquipmentTypes.name` |
| `page` | `int` | нет | `1` | Номер страницы |
| `limit` | `int` | нет | `20` | Размер страницы |

---

## Логика метода

1. Получить записи из `EquipmentTypes`.
2. Если передан `search`, применить фильтр по `name`.
3. Отсортировать результат по `sortOrder ASC`, затем по `name ASC`.
4. Для каждой записи рассчитать:
   - `propertiesCount` = COUNT(`EquipmentTypeProperties` WHERE `equipmentTypeId` = `EquipmentTypes.id`)
   - `equipmentsCount` = COUNT(`Equipments` WHERE `equipmentTypeId` = `EquipmentTypes.id`)
5. Вернуть страницу данных в формате общего `result wrapper` с `PaginatedResult` внутри `value`.

---

## Источники данных

| Поле ответа | Источник |
|-------------|----------|
| `id` | `EquipmentTypes.id` |
| `name` | `EquipmentTypes.name` |
| `mobilityType` | `EquipmentTypes.mobilityType` |
| `requiresTransport` | `EquipmentTypes.requiresTransport` |
| `sortOrder` | `EquipmentTypes.sortOrder` |
| `propertiesCount` | агрегат по `EquipmentTypeProperties` |
| `equipmentsCount` | агрегат по `Equipments` |

---

## Структура ответа

Метод использует:
- общий `result wrapper`
- `PaginatedResult` внутри `value`

Структура ответа:

```json
{
  "value": {
    "items": [
      {
        "id": "uuid",
        "name": "Компрессор",
        "mobilityType": "Stationary",
        "requiresTransport": true,
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

## Поля Response Item

| Поле | Тип | Комментарий |
|------|-----|-------------|
| `id` | `uuid` | UUID типа техники |
| `name` | `string` | Наименование типа |
| `mobilityType` | `enum` | `SelfPropelled / NonSelfPropelledMotorized / Stationary / NonMotorized` |
| `requiresTransport` | `bool` | Требуется ли отдельная транспортная техника |
| `sortOrder` | `int` | Порядок отображения в UI |
| `propertiesCount` | `int` | Количество характеристик типа |
| `equipmentsCount` | `int` | Количество единиц техники этого типа |

---

## Ошибки

| HTTP | code | Когда |
|------|------|-------|
| `401` | `UNAUTHORIZED` | Пользователь не авторизован |
| `403` | `FORBIDDEN` | У пользователя нет роли `Admin` |

Ошибки должны возвращаться внутри общего wrapper:

```json
{
  "value": null,
  "isSuccess": false,
  "errors": [
    {
      "message": "Forbidden",
      "code": "FORBIDDEN",
      "property": null,
      "tags": {}
    }
  ]
}
```

---

## Замечания

1. В схеме БД v5 для `EquipmentTypes` и `Equipments` не зафиксировано поле `deletedAt`.
2. Поэтому метод не должен возвращать `deletedAt` и не должен ссылаться на фильтрацию по нему.
3. `equipmentsCount` используется в UI для guard-логики удаления типа техники.
