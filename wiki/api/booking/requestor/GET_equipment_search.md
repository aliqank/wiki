# GET /equipment/search

**Created:** 2026-05-14  
**Last updated:** 2026-06-05  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить постраничный список техники для выбора в форме создания заявки |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / [`Requestor`](../../../requirements/Roles and Access Model.md) UI` |
| Endpoint URL | `/api/booking/v1/equipment/search` |
| Метод запроса | `GET` |
| Связанные use cases | [`UC-REQ-02 - Создание новой заявки (Draft-first)`](../../../requirements/usecases/Requestor/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first)/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first).md), [`UC-REQ-02.3 - Поиск техники для добавления в заявку`](../../../requirements/usecases/Requestor/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first)/UC-REQ-02.3%20-%20Поиск%20техники%20для%20добавления%20в%20заявку.md), [`UC-REQ-02.5 - Редактирование брони или замена техники в draft`](../../../requirements/usecases/[`Requestor`](../../../requirements/Roles and Access Model.md)/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first)/UC-REQ-02.5%20-%20Редактирование%20брони%20или%20замена%20техники%20в%20draft.md) |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется в форме создания заявки [`Requestor`](../../../requirements/Roles and Access Model.md) для поиска техники по типу, периоду и динамическим фильтрам.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-031 | [`Requestor`](../../../requirements/Roles and Access Model.md) can add/remove equipment items to a request; availability updated | Confirmed | BRD v13 | Метод является точкой входа для выбора техники |
| TCO Booking Tool | FR-032 | System prioritizes internal fleet | Confirmed | BRD v13 | Метод отбирает технику для requestor booking flow с учетом бизнес-правила приоритета internal fleet |
| TCO Booking Tool | FR-033 | [`Requestor`](../../../requirements/Roles and Access Model.md) can add Shared equipment to request | Confirmed | BRD v13 | Shared equipment участвует в результатах поиска и дальнейшем booking flow |
| TCO Booking Tool | FR-NEW-38 | Search: TCO equipment number + model mandatory; госномер if present | Confirmed | BRD v13 | В результатах поиска должен возвращаться `stateNumber`, если он заполнен |
| TCO Booking Tool | FR-040 | System validates availability before booking | Confirmed | BRD v13 | Метод показывает [hard availability restrictions](../../../glossary/Glossary.md#hard-availability-restriction); [booking conflict context](../../../glossary/Glossary.md#booking-conflict-context) должен отображаться отдельно и не исключает технику из booking flow |
| TCO Booking Tool | FR-041 | Equipment attributes displayed in booking | Confirmed | BRD v13 | Метод возвращает атрибуты техники, используемые при выборе и последующем отображении booking item |
| TCO Booking Tool | FR-NEW-39 | Dynamic search filters by equipment type | Confirmed | BRD v13 | Метод принимает динамические фильтры |
| TCO Booking Tool | FR-NEW-50 | [`Admin`](../../../requirements/Roles and Access Model.md) manages all reference/handbook values via [`Admin`](../../../requirements/Roles and Access Model.md) Panel | Confirmed | BRD v13 | Метод использует reference values для фильтров `ownershipType` и `shareType` |
| TCO Booking Tool | FR-NEW-68 | System dynamically shows only type-specific characteristics | Confirmed | BRD v13 | Набор фильтров зависит от equipment type |

---

## 3. Описание логики работы метода

1. Принять query params поиска, включая период `plannedStartDateTime` / `plannedEndDateTime`.
2. Выбрать записи из `Equipments` WHERE `isDeleted = false`.
3. Исключить из выдачи списанную технику: записи с текущим статусом `Decommissioned` не должны возвращаться в результатах поиска.
4. Исключить технику `ownershipType = OnDemand`, так как она не участвует в booking workflow.
5. Исключить стационарную HDE из поиска [`Requestor`](../../../requirements/Roles and Access Model.md) (`mobilityType = Stationary`).
6. Применить фильтры по `equipmentTypeId`, `ownershipType`, `shareType`, `fleetId`, `workCenterId`, флагу скрытия техники в ремонте, текстовому поиску и динамическим свойствам.
   Для dynamic properties использовать правило:
   - если `dataType = number`, фильтр задается диапазоном через `from` и/или `to`;
   - если `dataType = enum`, фильтр задается через `values`;
   - если `dataType = string`, фильтр задается через `values`;
   - если `dataType = bool`, фильтр задается через `value`.
7. Если `hideInRepair = true`, исключить из выдачи технику, у которой есть активный текущий статус `InRepair`.
8. Для `shareType = Assigned` вернуть элемент в списке, но пометить его как `isBookable = false`, если у пользователя нет записи в `EquipmentBookingAuthorizations`.
9. Для выбранного периода рассчитать state context по `EquipmentStates`:
   - определить, есть ли пересечение с активными state-записями `InRepair`, `Frozen` и другими blocking / informative state-типами;
   - вернуть агрегированный объект состояния техники на выбранный период;
   - использовать этот state context как отдельную информацию для UI и backend availability calculation.
10. Для периода рассчитать пересечения с активными записями `Bookings` со статусами `Submitted`, `Confirmed`, `InProgress`.
   Эти пересечения не должны автоматически делать технику недоступной для выбора. Они используются как conflict/load information для UI.
11. Вернуть пагинированный список в общем [`result wrapper`](../../common/Result%20Wrapper.md).

Сущности, участвующие в методе:
- читаются: [`Equipments`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#8-equipments), [`EquipmentStates`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#10-equipmentstates), [`EquipmentTypes`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#1-equipmenttypes), [`EquipmentProperties`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#15-equipmentproperties), [`EquipmentBookingAuthorizations`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#17-equipmentfeedbacks--equipmentbookingauthorizations), [`Bookings`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#22-bookings), [`Fleets`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#2-fleets)
- изменения не выполняются
- транзакционность не требуется

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| [`Requestor`](../../../requirements/Roles and Access Model.md) | Создание и просмотр собственных заявок |

---

## 5. Настройки системы, используемые в методе

| Наименование | Код | Тип значения | Описание | Значение по умолчанию |
|---|---|---|---|---|
| Горизонт бронирования | `BOOKING_HORIZON_DAYS` | `int` | Ограничивает максимально допустимую дату поиска | Конфигурируется [`Admin`](../../../requirements/Roles and Access Model.md) |
| Максимальная длительность брони | `MAX_BOOKING_DURATION_DAYS` | `int` | Ограничивает диапазон поиска | Конфигурируется [`Admin`](../../../requirements/Roles and Access Model.md) |

---

## 6. Ошибки, возвращаемые методом

| Код | Описание ошибки |
|---|---|
| `UNAUTHORIZED` | Пользователь не авторизован |
| `FORBIDDEN` | У пользователя нет роли [`Requestor`](../../../requirements/Roles and Access Model.md) |
| `VALIDATION_ERROR` | Период поиска невалиден или превышает системные ограничения |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор типа техники | `equipmentTypeId` | `uuid` | `-` | Если передан, должен существовать | — | Query param | |
| 2 | Плановая дата/время начала периода | `plannedStartDateTime` | `datetime` | `+` | Должна быть меньше `plannedEndDateTime` | — | Query param | |
| 3 | Плановая дата/время окончания периода | `plannedEndDateTime` | `datetime` | `+` | Должна быть больше `plannedStartDateTime` | — | Query param | |
| 4 | Поисковая строка | `search` | `string` | `-` | Поиск по TCO-номеру, госномеру, модели, бренду | — | Query param | |
| 5 | Тип владения | `ownershipType` | `string` | `-` | `TcoOwned / LongTermRented` | — | Query param | Значения загружаются через [`GET /reference/ownership-types`](../reference/GET_reference_ownership_types.md); `OnDemand` не допускается |
| 6 | Тип доступности | `shareType` | `string` | `-` | `Shared / SharedWithConditions / Assigned` | — | Query param | Значения загружаются через [`GET /reference/share-types`](../reference/GET_reference_share_types.md) |
| 7 | Fleet | `fleetId` | `uuid` | `-` | Если передан, должен существовать и соответствовать доступному для booking workflow флоту | — | Query param | Фильтр по `Equipments.fleetId` |
| 8 | Work Center | `workCenterId` | `uuid` | `-` | Если передан, должен соответствовать work center, на который можно бронировать технику | — | Query param | Фильтр по `EquipmentTypes.workCenterId` |
| 9 | Скрыть технику на ремонте | `hideInRepair` | `bool` | `-` | `true / false` | `false` | Query param | Если `true`, техника с активным текущим статусом `InRepair` исключается из результата |
| 10 | Динамические фильтры | `propertyFilters` | `array<object>` | `-` | Для `number` использовать `from` / `to`; для `enum` и `string` использовать `values`; для `bool` использовать `value` | `[]` | Query param | Передаются сериализованно |
| 11 | Номер страницы | `page` | `int` | `-` | Целое число >= 1 | `1` | Query param | |
| 12 | Размер страницы | `limit` | `int` | `-` | Целое число >= 1 | `20` | Query param | |

### Структура `propertyFilters[]`

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 10.1 | Идентификатор динамического свойства | `propertyId` | `uuid` | `+` | Должен существовать среди свойств выбранного `equipmentTypeId` | — | Query param | |
| 10.2 | Тип данных свойства | `dataType` | `string` | `+` | `string / number / enum / bool` | — | Query param | Должен соответствовать [`GET /reference/equipment-types/{equipmentTypeId}/properties`](../reference/GET_reference_equipment_types_id_properties.md) |
| 10.3 | Нижняя граница диапазона | `from` | `decimal` | `-` | Используется только для `dataType = number` | `null` | Query param | Для numeric property |
| 10.4 | Верхняя граница диапазона | `to` | `decimal` | `-` | Используется только для `dataType = number`; если переданы оба значения, `from <= to` | `null` | Query param | Для numeric property |
| 10.5 | Значение boolean-фильтра | `value` | `bool` | `-` | Используется только для `dataType = bool`; не должно передаваться вместе с `from`, `to` или `values` | `null` | Query param | Для boolean property |
| 10.6 | Набор значений фильтра | `values` | `array<string>` | `-` | Используется только для `dataType = string` или `enum`; массив должен содержать как минимум одно значение; не должно передаваться вместе с `from`, `to` или `value` | `[]` | Query param | Для `enum` значения являются кодами справочника; для `string` значения интерпретируются как exact-match список |

Правила заполнения `propertyFilters[]`:
- для `dataType = number` frontend передает хотя бы одно из полей `from` или `to`;
- для `dataType = number` поля `value` и `values` не передаются;
- для `dataType = enum` frontend передает поле `values`;
- для `dataType = string` frontend передает поле `values`;
- для `dataType = bool` frontend передает поле `value`;
- для `dataType IN (string, enum, bool)` поля `from` и `to` не передаются.

---

## 8. Пример запроса

```http
GET /api/booking/v1/equipment/search?equipmentTypeId=7b4f4b4d-52d4-4a77-b6b7-f2b7d7c81111&plannedStartDateTime=2026-05-20T08:00:00Z&plannedEndDateTime=2026-05-22T18:00:00Z&ownershipType=TcoOwned&fleetId=f3caa8da-11f2-4108-997f-2204041a1001&workCenterId=12a8b1ce-3aaf-4f55-8ac8-f8cf5d86c222&hideInRepair=true&propertyFilters=%5B%7B%22propertyId%22%3A%22a1b2c3d4-0001-4000-8000-000000000010%22%2C%22dataType%22%3A%22number%22%2C%22from%22%3A10%2C%22to%22%3A25%7D%2C%7B%22propertyId%22%3A%22a1b2c3d4-0001-4000-8000-000000000011%22%2C%22dataType%22%3A%22enum%22%2C%22values%22%3A%5B%22STANDARD%22%2C%22HEAVY_DUTY%22%5D%7D%2C%7B%22propertyId%22%3A%22a1b2c3d4-0001-4000-8000-000000000012%22%2C%22dataType%22%3A%22string%22%2C%22values%22%3A%5B%22CAT%22%2C%22KOMATSU%22%5D%7D%5D&page=1&limit=20
Authorization: Bearer <token>
Content-Type: application/json
```

Пример декодированного `propertyFilters`:

```json
[
  {
    "propertyId": "a1b2c3d4-0001-4000-8000-000000000010",
    "dataType": "number",
    "from": 10,
    "to": 25
  },
  {
    "propertyId": "a1b2c3d4-0001-4000-8000-000000000011",
    "dataType": "enum",
    "values": ["STANDARD", "HEAVY_DUTY"]
  },
  {
    "propertyId": "a1b2c3d4-0001-4000-8000-000000000012",
    "dataType": "string",
    "values": ["CAT", "KOMATSU"]
  }
]
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий [`result wrapper`](../../common/Result%20Wrapper.md).

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | value | object | PaginatedResult | — | backend aggregation |  |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | object[] | `[]` | backend |  |

### Структура `value`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Элементы текущей страницы | items | array<object> | object[] | — | backend composition from Equipments + EquipmentTypes + EquipmentProperties + EquipmentBookingAuthorizations + Bookings + Fleets | Коллекция объектов |
| 2 | Общее количество записей | total | int | integer | — | backend |  |

### Структура `value.items[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | Equipments.id |  |
| 2 | ТШО-номер техники | tcoId | string | string | — | Equipments.tcoId |  |
| 3 | Государственный регистрационный номер | stateNumber | string | string | — | Equipments.stateNumber |  |
| 4 | Идентификатор типа техники | equipmentTypeId | uuid | UUID v4 | — | Equipments.equipmentTypeId |  |
| 5 | Наименование типа техники | equipmentTypeName | object | object | — | EquipmentTypes |  |
| 6 | Бренд техники | brand | string | string | — | EquipmentBrands |  |
| 7 | Модель техники | model | string | string | — | EquipmentModels |  |
| 8 | Тип владения техникой | ownershipType | string | string | — | Equipments + ref_ownership_type |  |
| 9 | Тип доступности техники | shareType | string | string | — | Equipments + ref_share_type |  |
| 10 | Признак обязательности обоснования | requiresJustification | bool | boolean | — | backend business rule from Equipments + ref_share_type + ref_ownership_type | `true`, если `ownershipType = LongTermRented` или `shareType IN (Assigned, SharedWithConditions)` |
| 11 | Признак доступности бронирования | isBookable | bool | boolean | — | backend availability calculation from Equipments + EquipmentStates + EquipmentBookingAuthorizations | Отражает только [hard availability restrictions](../../../glossary/Glossary.md#hard-availability-restriction), не competing bookings |
| 12 | Причина недоступности бронирования | bookingUnavailableReason | null | — | `null` | backend availability calculation from Equipments + EquipmentStates + EquipmentBookingAuthorizations | Заполняется только для [hard availability restrictions](../../../glossary/Glossary.md#hard-availability-restriction) |
| 13 | Контекст состояний техники на выбранный период | equipmentStateOnPeriod | object | object | — | backend composition from EquipmentStates | Агрегированная информация по состояниям техники (`InRepair`, `Frozen` и др.) на период поиска |
| 14 | Признак конфликта с другими бронями на выбранный период | hasBookingConflict | bool | boolean | `false` | backend overlap check against active bookings | `true`, если по этой технике есть пересечение с другой бронью в статусе `Submitted`, `Confirmed` или `InProgress` |
| 15 | URL превью-фотографии | previewPhotoUrl | string | string | — | EquipmentPhotos |  |
| 16 | Fleet | fleet | object | object | — | Fleets | Базовый контекст флота техники |
| 17 | Рабочий центр | workCenter | object | object | — | WorkCenters |  |
| 18 | Наименование базовой локации | baseLocationName | object | object | — | Locations |  |

### Структура `value.items[].equipmentStateOnPeriod`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Наличие хотя бы одного state-пересечения на выбранный период | hasStatesOnPeriod | bool | boolean | `false` | backend overlap check against EquipmentStates | `true`, если на период поиска есть хотя бы один state interval |
| 2 | Наличие ремонта на период | isInRepairOnPeriod | bool | boolean | `false` | backend overlap check against EquipmentStates | `true`, если есть пересечение с state type `InRepair` |
| 3 | Наличие заморозки на период | isFrozenOnPeriod | bool | boolean | `false` | backend overlap check against EquipmentStates | `true`, если есть пересечение с state type `Frozen` |
| 4 | Список пересекающихся state-записей | items | array<object> | object[] | `[]` | backend composition from EquipmentStates + ref_equipment_status_type + ref_equipment_status_source | Коллекция state intervals, попавших в период поиска |

### Структура `value.items[].equipmentStateOnPeriod.items[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор state-записи | id | uuid | UUID v4 | — | EquipmentStates.id |  |
| 2 | Код типа состояния | stateType | string | string | — | ref_equipment_status_type | Например: `InRepair`, `Frozen`, `Decommissioned` |
| 3 | Подпись типа состояния | stateTypeLabel | string | string | — | ref_equipment_status_type | UI-readable caption |
| 4 | Источник состояния | source | string | null | `null` | ref_equipment_status_source | Например: `JDE`, `Manual` |
| 5 | Дата и время начала состояния | startDateTime | datetime | ISO 8601 | — | EquipmentStates.startDateTime / equivalent period field |  |
| 6 | Дата и время окончания состояния | endDateTime | datetime | ISO 8601 | `null` | EquipmentStates.endDateTime / equivalent period field | `null` для открытого интервала |
| 7 | Комментарий | comment | string | null | `null` | EquipmentStates.comment |  |

### Структура `value.items[].baseLocationName`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string | string | — | Locations |  |
| 2 | Значение на русском языке | Ru | string | string | — | Locations |  |
| 3 | Значение на казахском языке | Kz | string | string | — | Locations |  |

### Структура `value.items[].equipmentTypeName`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string | string | — | EquipmentTypes |  |
| 2 | Значение на русском языке | Ru | string | string | — | EquipmentTypes |  |
| 3 | Значение на казахском языке | Kz | string | string | — | EquipmentTypes |  |

### Структура `value.items[].fleet`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор флота | id | uuid | UUID v4 | — | Fleets.id |  |
| 2 | Наименование флота | name | string | string | — | Fleets.nameEn / localized projection |  |

### Структура `value.items[].workCenter`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | Equipments.id |  |
| 2 | Код записи | code | string | string | — | backend composition from Equipments + EquipmentTypes + EquipmentProperties + EquipmentBookingAuthorizations + Bookings + Fleets |  |
| 3 | Наименование | name | object | object | — | backend composition from Equipments + EquipmentTypes + EquipmentProperties + EquipmentBookingAuthorizations + Bookings + Fleets |  |

### Структура `value.items[].workCenter.name`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string | string | — | Equipments |  |
| 2 | Значение на русском языке | Ru | string | string | — | Equipments |  |
| 3 | Значение на казахском языке | Kz | string | string | — | Equipments |  |

## 10. Пример ответа

```json
{
  "value": {
    "items": [
      {
        "id": "c3b5af91-61f8-4bc0-bd88-d099d3e90001",
        "tcoId": "TCO-100245",
        "stateNumber": "KZ 123 ABC 02",
        "equipmentTypeId": "7b4f4b4d-52d4-4a77-b6b7-f2b7d7c81111",
        "equipmentTypeName": {
          "En": "Excavator",
          "Ru": "Экскаватор",
          "Kz": "Экскаватор"
        },
        "brand": "CAT",
        "model": "320D",
        "ownershipType": "TcoOwned",
        "shareType": "SharedWithConditions",
        "requiresJustification": true,
        "isBookable": true,
        "bookingUnavailableReason": null,
        "equipmentStateOnPeriod": {
          "hasStatesOnPeriod": true,
          "isInRepairOnPeriod": true,
          "isFrozenOnPeriod": false,
          "items": [
            {
              "id": "70000001-0000-4000-8000-000000000001",
              "stateType": "InRepair",
              "stateTypeLabel": "In repair",
              "source": "JDE",
              "startDateTime": "2026-05-19T00:00:00Z",
              "endDateTime": "2026-05-23T23:59:59Z",
              "comment": "Open maintenance work order"
            }
          ]
        },
        "hasBookingConflict": true,
        "previewPhotoUrl": "https://cdn.example.com/equipment/c3b5af91-61f8-4bc0-bd88-d099d3e90001/preview.jpg",
        "fleet": {
          "id": "f0000001-0000-4000-8000-000000000001",
          "name": "Maintenance Fleet"
        },
        "workCenter": {
          "id": "12a8b1ce-3aaf-4f55-8ac8-f8cf5d86c222",
          "code": "WC-100",
          "name": {
            "En": "Drilling Operations",
            "Ru": "Буровые работы",
            "Kz": "Бұрғылау жұмыстары"
          }
        },
        "baseLocationName": {
          "En": "Base A",
          "Ru": "База А",
          "Kz": "А базасы"
        }
      }
    ],
    "total": 1
  },
  "isSuccess": true,
  "errors": []
}
```
