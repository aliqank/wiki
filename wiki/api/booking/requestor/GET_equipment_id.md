# GET /equipment/{id}

**Created:** 2026-05-14  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить карточку единицы техники в booking-контексте |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Requestor UI` |
| Endpoint URL | `/api/booking/v1/equipment/{id}` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется перед добавлением техники в заявку и при просмотре деталей выбранной единицы.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-041 | Equipment attributes displayed in booking | Confirmed | BRD v13 | Метод возвращает карточку техники |
| TCO Booking Tool | FR-NEW-05 | FO uploads multiple photos; Requestor sees them in request form | Confirmed | BRD v13 | В ответе нужны фото |

---

## 3. Описание логики работы метода

1. Получить запись из `Equipments` WHERE `id = :id` AND `isDeleted = false`.
2. Подтянуть связанные справочники: `EquipmentTypes`, `EquipmentBrands`, `EquipmentModels`, `Locations`, `Fleets`.
3. Подтянуть динамические свойства и фотографии техники.
4. Вернуть агрегированную карточку техники в общем `result wrapper`.

Сущности, участвующие в методе:
- читаются: [`Equipments`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#8-equipments), [`EquipmentTypes`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#1-equipmenttypes), [`EquipmentBrands`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#4-equipmentbrands), [`EquipmentModels`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#5-equipmentmodels), [`EquipmentProperties`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#15-equipmentproperties), [`EquipmentPhotos`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#9-equipmentphotos), [`Locations`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#6-locations--costcenters--servicezones--divisions--groups--departments--sections), [`Fleets`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#2-fleets)
- изменения не выполняются

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Requestor` | Просмотр техники для создания заявки |
| `ServiceWorkProcessor` | Просмотр техники для Service Work Requests |

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
| `NOT_FOUND` | Техника не найдена или удалена |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор техники | `id` | `uuid` | `+` | Должен существовать | — | Path param | |

---

## 8. Пример запроса

```http
GET /api/booking/v1/equipment/c3b5af91-61f8-4bc0-bd88-d099d3e90001
Authorization: Bearer <token>
Content-Type: application/json
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий `result wrapper`.

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | value | object | object | — | backend aggregation |  |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | object[] | `[]` | backend |  |

### Структура `value`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | Equipments.id |  |
| 2 | ТШО-номер техники | tcoId | string | string | — | Equipments.tcoId |  |
| 3 | Тип техники | equipmentType | object | object | — | backend composition from Equipments + EquipmentTypes + EquipmentBrands + EquipmentModels + EquipmentProperties + EquipmentPhotos + Locations + Fleets |  |
| 4 | Бренд техники | brand | string | string | — | EquipmentBrands |  |
| 5 | Модель техники | model | string | string | — | EquipmentModels |  |
| 6 | Тип владения техникой | ownershipType | string | string | — | Equipments + ref_ownership_type |  |
| 7 | Тип доступности техники | shareType | string | string | — | Equipments + ref_share_type |  |
| 8 | Список фотографий | photos | array<object> | object[] | `[]` | EquipmentPhotos | Коллекция объектов |
| 9 | Список свойств | properties | array<object> | object[] | `[]` | backend composition from EquipmentProperties + Properties + PropertyEnumValues + MeasurementUnits | Коллекция объектов |

### Структура `value.equipmentType`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | Equipments.id |  |
| 2 | Наименование | name | object | object | — | backend composition from Equipments + EquipmentTypes + EquipmentBrands + EquipmentModels + EquipmentProperties + EquipmentPhotos + Locations + Fleets |  |

### Структура `value.equipmentType.name`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string | string | — | Equipments |  |
| 2 | Значение на русском языке | Ru | string | string | — | Equipments |  |
| 3 | Значение на казахском языке | Kz | string | string | — | Equipments |  |

## 10. Пример ответа

```json
{
  "value": {
    "id": "c3b5af91-61f8-4bc0-bd88-d099d3e90001",
    "tcoId": "TCO-100245",
    "equipmentType": {
      "id": "7b4f4b4d-52d4-4a77-b6b7-f2b7d7c81111",
      "name": {
        "En": "Excavator",
        "Ru": "Экскаватор",
        "Kz": "Экскаватор"
      }
    },
    "brand": "CAT",
    "model": "320D",
    "ownershipType": "TcoOwned",
    "shareType": "Shared",
    "photos": [],
    "properties": []
  },
  "isSuccess": true,
  "errors": []
}
```
