# GET /equipment/search

**Created:** 2026-05-14  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить постраничный список техники для выбора в форме создания заявки |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Requestor UI` |
| Endpoint URL | `/api/booking/v1/equipment/search` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется в форме создания заявки Requestor / SWP для поиска техники по типу, периоду и динамическим фильтрам.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-031 | Requestor can add/remove equipment items to a request; availability updated | Confirmed | BRD v13 | Метод является точкой входа для выбора техники |
| TCO Booking Tool | FR-NEW-38 | Search: TCO equipment number + model mandatory; госномер if present | Confirmed | BRD v13 | В результатах поиска должен возвращаться `stateNumber`, если он заполнен |
| TCO Booking Tool | FR-040 | System validates availability before booking | Confirmed | BRD v13 | Метод показывает hard-доступность техники; конфликты с активными бронями должны отображаться отдельно и не исключают технику из booking flow |
| TCO Booking Tool | FR-NEW-39 | Dynamic search filters by equipment type | Confirmed | BRD v13 | Метод принимает динамические фильтры |
| TCO Booking Tool | FR-NEW-68 | System dynamically shows only type-specific characteristics | Confirmed | BRD v13 | Набор фильтров зависит от equipment type |

---

## 3. Описание логики работы метода

1. Принять query params поиска, включая период `plannedStartDateTime` / `plannedEndDateTime`.
2. Выбрать записи из `Equipments` WHERE `isDeleted = false`.
3. Исключить из выдачи списанную технику: записи с текущим статусом `Decommissioned` не должны возвращаться в результатах поиска.
4. Исключить технику `ownershipType = OnDemand`, так как она не участвует в booking workflow.
5. Исключить стационарную HDE из поиска Requestor (`mobilityType = Stationary`).
6. Применить фильтры по `equipmentTypeId`, `ownershipType`, `shareType`, `fleetOwnerUserId`, `workCenterId`, текстовому поиску и динамическим свойствам.
7. Для `shareType = Assigned` вернуть элемент в списке, но пометить его как `isBookable = false`, если у пользователя нет записи в `EquipmentBookingAuthorizations`.
8. Для периода рассчитать пересечения с активными записями `Bookings` со статусами `Submitted`, `Confirmed`, `InProgress`.
   Эти пересечения не должны автоматически делать технику недоступной для выбора. Они используются как conflict/load information для UI.
9. Вернуть пагинированный список в общем `result wrapper`.

Сущности, участвующие в методе:
- читаются: `Equipments`, `EquipmentStatuses`, `EquipmentTypes`, `EquipmentProperties`, `EquipmentBookingAuthorizations`, `Bookings`, `Fleets`
- изменения не выполняются
- транзакционность не требуется

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Requestor` | Создание и просмотр собственных заявок |
| `ServiceWorkProcessor` | Работа с Service Work Requests |

---

## 5. Настройки системы, используемые в методе

| Наименование | Код | Тип значения | Описание | Значение по умолчанию |
|---|---|---|---|---|
| Горизонт бронирования | `BOOKING_HORIZON_DAYS` | `int` | Ограничивает максимально допустимую дату поиска | Конфигурируется Admin |
| Максимальная длительность брони | `MAX_BOOKING_DURATION_DAYS` | `int` | Ограничивает диапазон поиска | Конфигурируется Admin |

---

## 6. Ошибки, возвращаемые методом

| Код | Описание ошибки |
|---|---|
| `UNAUTHORIZED` | Пользователь не авторизован |
| `FORBIDDEN` | У пользователя нет роли `Requestor` или `ServiceWorkProcessor` |
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
| 5 | Тип владения | `ownershipType` | `enum` | `-` | `TcoOwned / LongTermRented` | — | Query param | `OnDemand` не допускается |
| 6 | Тип доступности | `shareType` | `enum` | `-` | `Shared / SharedWithConditions / Assigned` | — | Query param | |
| 7 | Fleet Owner | `fleetOwnerUserId` | `uuid` | `-` | Если передан, должен соответствовать пользователю, у которого есть owner-assignment для флота техники | — | Query param | Фильтр по `FleetManagePermissions` с `permissionType = Owner` |
| 8 | Work Center | `workCenterId` | `uuid` | `-` | Если передан, должен соответствовать work center, на который можно бронировать технику | — | Query param | Фильтр по `EquipmentTypes.workCenterId` |
| 9 | Динамические фильтры | `propertyFilters` | `array<object>` | `-` | Формат зависит от типа свойства | `[]` | Query param | Передаются сериализованно |
| 10 | Номер страницы | `page` | `int` | `-` | Целое число >= 1 | `1` | Query param | |
| 11 | Размер страницы | `limit` | `int` | `-` | Целое число >= 1 | `20` | Query param | |

---

## 8. Пример запроса

```http
GET /api/booking/v1/equipment/search?equipmentTypeId=7b4f4b4d-52d4-4a77-b6b7-f2b7d7c81111&plannedStartDateTime=2026-05-20T08:00:00Z&plannedEndDateTime=2026-05-22T18:00:00Z&ownershipType=TcoOwned&fleetOwnerUserId=4c9ad2d2-6df8-4f7b-87fe-36cefc100001&workCenterId=12a8b1ce-3aaf-4f55-8ac8-f8cf5d86c222&page=1&limit=20
Authorization: Bearer <token>
Content-Type: application/json
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий `result wrapper`.

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
| 11 | Признак доступности бронирования | isBookable | bool | boolean | — | backend availability calculation from Equipments + EquipmentStatuses + EquipmentBookingAuthorizations | Отражает только hard-ограничения доступности, не competing bookings |
| 12 | Причина недоступности бронирования | bookingUnavailableReason | null | — | `null` | backend availability calculation from Equipments + EquipmentStatuses + EquipmentBookingAuthorizations | Заполняется только для hard-ограничений доступности |
| 13 | URL превью-фотографии | previewPhotoUrl | string | string | — | EquipmentPhotos |  |
| 14 | Fleet | fleet | object | object | — | Fleets | Базовый контекст флота техники |
| 15 | Список Fleet Owners | fleetOwners | array<object> | object[] | `[]` | Fleets + FleetManagePermissions + Users | Только owner-assignment'ы для флота |
| 16 | Рабочий центр | workCenter | object | object | — | WorkCenters |  |
| 17 | Наименование базовой локации | baseLocationName | object | object | — | Locations |  |

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

### Структура `value.items[].fleetOwners[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор пользователя | userId | uuid | UUID v4 | — | Users.id | Только owner-assignment'ы из `FleetManagePermissions` |
| 2 | Полное имя пользователя | fullName | string | string | — | Users.fullName |  |
| 3 | Email | email | string | string | — | Users.email |  |

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
        "previewPhotoUrl": "https://cdn.example.com/equipment/c3b5af91-61f8-4bc0-bd88-d099d3e90001/preview.jpg",
        "fleet": {
          "id": "f0000001-0000-4000-8000-000000000001",
          "name": "Maintenance Fleet"
        },
        "fleetOwners": [
          {
            "userId": "4c9ad2d2-6df8-4f7b-87fe-36cefc100001",
            "fullName": "Nurlan Sarsenov",
            "email": "nurlan.sarsenov@tco.example"
          },
          {
            "userId": "4c9ad2d2-6df8-4f7b-87fe-36cefc100002",
            "fullName": "Aidos Beketov",
            "email": "aidos.beketov@tco.example"
          }
        ],
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
