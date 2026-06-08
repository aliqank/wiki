# GET /equipment/bp-showcase

**Created:** 2026-06-08  
**Last updated:** 2026-06-08  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить постраничный список техники внешних business partners для read-only showcase |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Requestor UI` |
| Endpoint URL | `/api/booking/v1/equipment/bp-showcase` |
| Метод запроса | `GET` |
| Связанные use cases | [`UC-REQ-BP-01 - Просмотр витрины техники внешних бизнес-партнеров`](../../../requirements/usecases/Requestor/UC-REQ-BP-01%20-%20Просмотр%20витрины%20техники%20внешних%20бизнес-партнеров.md) |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется для отдельной read-only витрины `On-demand BP (Showcase)` и не участвует в booking search flow.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-NEW-25 | On-demand BP (Showcase): Requestor can view on-demand BP equipment; no booking action in Phase 1 | Confirmed | BRD v13 | Прямое покрытие showcase browse flow |
| TCO Booking Tool | FR-NEW-26 | On-demand BP (Showcase) equipment prices/rates NOT displayed | Confirmed | BRD v13 | Метод не возвращает коммерческие данные |
| TCO Booking Tool | FR-NEW-27 | BP populates own catalog cards | Confirmed | BRD v13 | Метод возвращает BP-provided showcase data |

---

## 3. Описание логики работы метода

1. Принять query params showcase search без booking period.
2. Выбрать записи из `Equipments` WHERE `isDeleted = false`.
3. Отобрать только технику `ownershipType = OnDemand`.
4. Исключить из ответа любые booking-oriented действия и availability semantics.
5. Если передан `businessPartnerId`, применить фильтр по `Fleets.businessPartnerId` или эквивалентной business-partner linkage модели.
6. Если передан `search`, выполнять поиск только по `stateNumber`.
7. Если передан `workCenterId`, применить фильтр по `EquipmentTypes.workCenterId`.
8. Если переданы `propertyFilters`, применить dynamic showcase filtering по тем же техническим правилам, что и в equipment search, но без booking-period logic.
9. Вернуть пагинированный read-only список в общем [`result wrapper`](../../common/Result%20Wrapper.md).

Сущности, участвующие в методе:
- читаются: [`Equipments`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#8-equipments), [`EquipmentTypes`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#1-equipmenttypes), [`EquipmentProperties`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#15-equipmentproperties), [`EquipmentPhotos`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#9-equipmentphotos), [`BusinessPartners`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#20-businesspartners), [`Fleets`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#2-fleets)
- изменения не выполняются
- транзакционность не требуется

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Requestor` | Просмотр BP showcase техники в read-only режиме |

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
| `FORBIDDEN` | У пользователя нет роли `Requestor` |
| `VALIDATION_ERROR` | Передан невалидный `businessPartnerId`, `workCenterId` или некорректный формат `propertyFilters` |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Business partner | `businessPartnerId` | `uuid` | `-` | Если передан, должен существовать | — | Query param | Фильтр по внешнему владельцу showcase-карточки |
| 2 | Поисковая строка | `search` | `string` | `-` | Поиск только по `stateNumber` | — | Query param | Не используется для `tcoId`, бренда или модели |
| 3 | Work Center | `workCenterId` | `uuid` | `-` | Если передан, должен существовать | — | Query param | Optional showcase filter |
| 4 | Динамические фильтры | `propertyFilters` | `array<object>` | `-` | Для `number` использовать `from` / `to`; для `enum` и `string` использовать `values`; для `bool` использовать `value` | `[]` | Query param | Optional showcase filter |
| 5 | Номер страницы | `page` | `int` | `-` | Целое число >= 1 | `1` | Query param | |
| 6 | Размер страницы | `limit` | `int` | `-` | Целое число >= 1 | `20` | Query param | |

### Структура `propertyFilters[]`

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 4.1 | Идентификатор динамического свойства | `propertyId` | `uuid` | `+` | Должен существовать | — | Query param | |
| 4.2 | Тип данных свойства | `dataType` | `string` | `+` | `string / number / enum / bool` | — | Query param | |
| 4.3 | Нижняя граница диапазона | `from` | `decimal` | `-` | Используется только для `dataType = number` | `null` | Query param | |
| 4.4 | Верхняя граница диапазона | `to` | `decimal` | `-` | Используется только для `dataType = number` | `null` | Query param | |
| 4.5 | Значение boolean-фильтра | `value` | `bool` | `-` | Используется только для `dataType = bool` | `null` | Query param | |
| 4.6 | Набор значений фильтра | `values` | `array<string>` | `-` | Используется только для `dataType = string` или `enum` | `[]` | Query param | |

---

## 8. Пример запроса

```http
GET /api/booking/v1/equipment/bp-showcase?businessPartnerId=4a9a0001-0000-4000-8000-000000000001&search=KZ123ABC02&workCenterId=12a8b1ce-3aaf-4f55-8ac8-f8cf5d86c222&page=1&limit=20
Authorization: Bearer <token>
Content-Type: application/json
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

### Структура `value.items[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | Equipments.id |  |
| 2 | Государственный регистрационный номер | stateNumber | string | string | — | Equipments.stateNumber | Основное поле поиска |
| 3 | Идентификатор типа техники | equipmentTypeId | uuid | UUID v4 | — | Equipments.equipmentTypeId |  |
| 4 | Наименование типа техники | equipmentTypeName | object | object | — | EquipmentTypes |  |
| 5 | Бренд техники | brand | string | string | — | EquipmentBrands |  |
| 6 | Модель техники | model | string | string | — | EquipmentModels |  |
| 7 | URL превью-фотографии | previewPhotoUrl | string | string | — | EquipmentPhotos |  |
| 8 | Информация о business partner | businessPartner | object | object | — | backend composition from BusinessPartners + Fleets | Заменяет `fleet` |
| 9 | Рабочий центр | workCenter | object | object | — | WorkCenters | Optional showcase context |
| 10 | Наименование базовой локации | baseLocationName | object | object | — | Locations |  |

### Структура `value.items[].equipmentTypeName`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string | string | — | EquipmentTypes |  |
| 2 | Значение на русском языке | Ru | string | string | — | EquipmentTypes |  |
| 3 | Значение на казахском языке | Kz | string | string | — | EquipmentTypes |  |

### Структура `value.items[].businessPartner`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор business partner | id | uuid | UUID v4 | — | BusinessPartners.id |  |
| 2 | Наименование business partner | name | object | object | — | BusinessPartners |  |

### Структура `value.items[].businessPartner.name`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string | string | — | BusinessPartners |  |
| 2 | Значение на русском языке | Ru | string | string | — | BusinessPartners |  |
| 3 | Значение на казахском языке | Kz | string | string | — | BusinessPartners |  |

### Структура `value.items[].workCenter`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | WorkCenters.id |  |
| 2 | Код записи | code | string | string | — | WorkCenters.code |  |
| 3 | Наименование | name | object | object | — | WorkCenters |  |

### Структура `value.items[].workCenter.name`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string | string | — | WorkCenters |  |
| 2 | Значение на русском языке | Ru | string | string | — | WorkCenters |  |
| 3 | Значение на казахском языке | Kz | string | string | — | WorkCenters |  |

### Структура `value.items[].baseLocationName`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string | string | — | Locations |  |
| 2 | Значение на русском языке | Ru | string | string | — | Locations |  |
| 3 | Значение на казахском языке | Kz | string | string | — | Locations |  |

## 10. Пример ответа

```json
{
  "value": {
    "items": [
      {
        "id": "c3b5af91-61f8-4bc0-bd88-d099d3e90001",
        "stateNumber": "KZ 123 ABC 02",
        "equipmentTypeId": "7b4f4b4d-52d4-4a77-b6b7-f2b7d7c81111",
        "equipmentTypeName": {
          "En": "Excavator",
          "Ru": "Экскаватор",
          "Kz": "Экскаватор"
        },
        "brand": "CAT",
        "model": "320D",
        "previewPhotoUrl": "https://cdn.example.com/equipment/c3b5af91-61f8-4bc0-bd88-d099d3e90001/preview.jpg",
        "businessPartner": {
          "id": "4a9a0001-0000-4000-8000-000000000001",
          "name": {
            "En": "Balkindge",
            "Ru": "Balkindge",
            "Kz": "Balkindge"
          }
        },
        "workCenter": {
          "id": "12a8b1ce-3aaf-4f55-8ac8-f8cf5d86c222",
          "code": "CDECK",
          "name": {
            "En": "Cargo deck",
            "Ru": "Транспортная платформа",
            "Kz": "Жүк платформасы"
          }
        },
        "baseLocationName": {
          "En": "Tengiz Base",
          "Ru": "База Тенгиз",
          "Kz": "Теңіз базасы"
        }
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

1. Метод принципиально не возвращает `tcoId`, `fleet`, `isBookable`, `requiresJustification`, `hasBookingConflict`, `bookingUnavailableReason`, `equipmentStateOnPeriod`.
2. Витрина является read-only и не должна использоваться как альтернативный booking search flow.
3. При необходимости `FR-NEW-28` должен быть покрыт отдельным endpoint / CTA flow, а не данным методом.
