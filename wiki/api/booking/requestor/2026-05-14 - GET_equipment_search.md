# GET /equipment/search

**Created:** 2026-05-14  
**Last updated:** 2026-05-14  
**Author:** OpenCode

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
| TCO Booking Tool | FR-040 | System validates availability before booking | Confirmed | BRD v13 | В список не должны попадать заведомо недоступные для периода единицы |
| TCO Booking Tool | FR-NEW-39 | Dynamic search filters by equipment type | Confirmed | BRD v13 | Метод принимает динамические фильтры |
| TCO Booking Tool | FR-NEW-68 | System dynamically shows only type-specific characteristics | Confirmed | BRD v13 | Набор фильтров зависит от equipment type |

---

## 3. Описание логики работы метода

1. Принять query params поиска, включая период `startDt` / `endDt`.
2. Выбрать записи из `Equipments` WHERE `isDeleted = false`.
3. Исключить технику `ownershipType = OnDemand`, так как она не участвует в booking workflow.
4. Исключить стационарную HDE из поиска Requestor.
5. Применить фильтры по `equipmentTypeId`, `ownershipType`, `shareType`, `fleetOwnerUserId`, текстовому поиску и динамическим свойствам.
6. Для `shareType = Assigned` вернуть элемент в списке, но пометить его как `isBookable = false`, если у пользователя нет записи в `EquipmentBookingAuthorizations`.
7. Для периода проверить пересечения с активными записями `Bookings` со статусами, влияющими на доступность.
8. Вернуть пагинированный список в общем `result wrapper`.

Сущности, участвующие в методе:
- читаются: `Equipments`, `EquipmentTypes`, `EquipmentProperties`, `EquipmentBookingAuthorizations`, `Bookings`, `Fleets`
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
| 2 | Дата/время начала периода | `startDt` | `datetime` | `+` | Должна быть меньше `endDt` | — | Query param | |
| 3 | Дата/время окончания периода | `endDt` | `datetime` | `+` | Должна быть больше `startDt` | — | Query param | |
| 4 | Поисковая строка | `search` | `string` | `-` | Поиск по номеру, модели, бренду | — | Query param | |
| 5 | Тип владения | `ownershipType` | `enum` | `-` | `TcoOwned / LongTermRented` | — | Query param | `OnDemand` не допускается |
| 6 | Тип доступности | `shareType` | `enum` | `-` | `Shared / SharedWithConditions / Assigned` | — | Query param | |
| 7 | Fleet Owner | `fleetOwnerUserId` | `uuid` | `-` | Если передан, должен соответствовать пользователю, назначенному на флот техники | — | Query param | Фильтр по `Fleets.userId` |
| 8 | Динамические фильтры | `propertyFilters` | `array<object>` | `-` | Формат зависит от типа свойства | `[]` | Query param | Передаются сериализованно |
| 9 | Номер страницы | `page` | `int` | `-` | Целое число >= 1 | `1` | Query param | |
| 10 | Размер страницы | `limit` | `int` | `-` | Целое число >= 1 | `20` | Query param | |

---

## 8. Пример запроса

```http
GET /api/booking/v1/equipment/search?equipmentTypeId=7b4f4b4d-52d4-4a77-b6b7-f2b7d7c81111&startDt=2026-05-20T08:00:00Z&endDt=2026-05-22T18:00:00Z&ownershipType=TcoOwned&fleetOwnerUserId=4c9ad2d2-6df8-4f7b-87fe-36cefc100001&page=1&limit=20
Authorization: Bearer <token>
Content-Type: application/json
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий `result wrapper`.  
В `value` возвращается `PaginatedResult<EquipmentSearchItem>`.

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | `value` | `object` | `PaginatedResult<EquipmentSearchItem>` | — | backend aggregation | |
| 1.1 | Элементы текущей страницы | `items` | `array<object>` | `EquipmentSearchItem[]` | `[]` | `Equipments` + вычисления | |
| 1.2 | Общее количество записей | `total` | `int` | integer | `0` | backend | |
| 2 | Признак успешности | `isSuccess` | `bool` | boolean | — | backend | |
| 3 | Ошибки | `errors` | `array<object>` | `ApiError[]` | `[]` | backend | |

`EquipmentSearchItem`: `id`, `equipmentNumber`, `equipmentTypeId`, `equipmentTypeName`, `brandName`, `modelName`, `ownershipType`, `shareType`, `requiresJustification`, `isBookable`, `bookabilityReason`, `baseLocationName`.

---

## 10. Пример ответа

```json
{
  "value": {
    "items": [
      {
        "id": "c3b5af91-61f8-4bc0-bd88-d099d3e90001",
        "equipmentNumber": "TCO-100245",
        "equipmentTypeId": "7b4f4b4d-52d4-4a77-b6b7-f2b7d7c81111",
        "equipmentTypeName": {
          "En": "Excavator",
          "Ru": "Экскаватор",
          "Kz": "Экскаватор"
        },
        "brandName": "CAT",
        "modelName": "320D",
        "ownershipType": "TcoOwned",
        "shareType": "SharedWithConditions",
        "requiresJustification": true,
        "isBookable": true,
        "bookabilityReason": null,
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
