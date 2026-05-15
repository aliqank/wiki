# GET /booking-requests/my

**Created:** 2026-05-14  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить список собственных заявок текущего пользователя |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Requestor UI` |
| Endpoint URL | `/api/booking/v1/booking-requests/my` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется для страницы «Мои заявки» как summary API для списка заявок Requestor / SWP.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-025 | Requestor can view request details and status | Confirmed | BRD v13 | Частичное покрытие на уровне summary-list |
| TCO Booking Tool | FR-091 | Requestor/SWP can search and filter own requests | Confirmed | BRD v13 | Прямое покрытие |

---

## 3. Описание логики работы метода

1. Выбрать `BookingRequests` по `createdBy = currentUserId`.
2. Применить фильтры по `status`, `type`, `priority`, `search`, `createdFrom`, `createdTo`.
3. Отсортировать по `createdAt DESC`.
4. Для каждой заявки собрать summary-данные по связанным `Bookings`.
5. Для каждой брони вернуть только краткий `bookingSummary`, достаточный для таблицы списка заявок.
6. Вернуть пагинированный список.

Метод не возвращает полные детали заявки или полные карточки броней. Для этого используется `GET /booking-requests/{id}`.

Сущности, участвующие в методе:
- читаются: `BookingRequests`, `Bookings`, `Equipments`, `EquipmentTypes`, `EquipmentBrands`, `EquipmentModels`, `WorkCenters`, `Fleets`, `Users`
- изменения не выполняются

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Requestor` | Просмотр своих заявок |
| `ServiceWorkProcessor` | Просмотр своих SWR |

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

HTTP-коды: `401 Unauthorized`, `403 Forbidden`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Фильтр по статусу | `status` | `enum` | `-` | `Draft / Submitted / InProgress / Completed / Cancelled` | — | Query param | |
| 2 | Фильтр по типу заявки | `type` | `enum` | `-` | `Regular / ServiceWork` | — | Query param | |
| 3 | Фильтр по приоритету | `priority` | `enum` | `-` | `P1 / P2 / P3 / P4` | — | Query param | |
| 4 | Поисковая строка | `search` | `string` | `-` | Поиск по `requestNumber`, `workOrderJdeId`, `workDescription` | — | Query param | |
| 5 | Дата создания заявки: начало диапазона | `createdFrom` | `date` | `-` | `<= createdTo`, формат `YYYY-MM-DD` | — | Query param | Фильтр по `BookingRequests.createdAt` |
| 6 | Дата создания заявки: конец диапазона | `createdTo` | `date` | `-` | `>= createdFrom`, формат `YYYY-MM-DD` | — | Query param | Фильтр по `BookingRequests.createdAt` |
| 7 | Номер страницы | `page` | `int` | `-` | >= 1 | `1` | Query param | |
| 8 | Размер страницы | `limit` | `int` | `-` | >= 1 | `20` | Query param | |

---

## 8. Пример запроса

```http
GET /api/booking/v1/booking-requests/my?status=Submitted&createdFrom=2026-05-01&createdTo=2026-05-31&page=1&limit=20
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
| 1.1 | Элементы текущей страницы | items | array<object> | object[] | — | response DTO | Коллекция объектов |
| 1.2 | Общее количество записей | total | int | integer | — | backend |  |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | array | `[]` | backend |  |

### Структура `value`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Элементы текущей страницы | items | array<object> | object[] | — | response DTO | Коллекция объектов |
| 2 | Общее количество записей | total | int | integer | — | backend |  |

### Структура `value.items[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | response DTO |  |
| 2 | Номер заявки | requestNumber | string | string | — | response DTO |  |
| 3 | Тип сущности / заявки | type | string | string | — | response DTO |  |
| 4 | Текущий статус | status | string | string | — | response DTO |  |
| 5 | Приоритет | priority | string | string | — | response DTO |  |
| 6 | Номер Work Order из JDE | workOrderJdeId | string | string | — | response DTO |  |
| 7 | Дата и время создания | createdAt | datetime | ISO 8601 | — | response DTO |  |
| 8 | Данные реквестора | requestor | object | object | — | response DTO |  |
| 9 | Количество броней | bookingsCount | int | integer | — | response DTO |  |
| 10 | Сводный список броней | bookingSummaries | array<object> | object[] | — | response DTO | Коллекция объектов |

### Структура `value.items[].requestor`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | response DTO |  |
| 2 | Полное имя пользователя | fullName | string | string | — | response DTO |  |
| 3 | Email | email | string | string | — | response DTO |  |

### Структура `value.items[].bookingSummaries[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | response DTO |  |
| 2 | Тип техники | equipmentType | string | string | — | response DTO |  |
| 3 | Марка и модель техники | brandModel | string | string | — | response DTO |  |
| 4 | ТШО-номер техники | tcoId | string | string | — | response DTO |  |
| 5 | Государственный регистрационный номер | stateNumber | string | string | — | response DTO |  |
| 6 | Рабочий центр | workCenter | string | string | — | response DTO |  |
| 7 | Владелец / ответственный fleet | fleetOwner | string | string | — | response DTO |  |
| 8 | Плановая дата и время начала | plannedStartDt | datetime | ISO 8601 | — | response DTO |  |
| 9 | Плановая дата и время окончания | plannedEndDt | datetime | ISO 8601 | — | response DTO |  |
| 10 | Фактическая дата и время начала | actualStartDt | null | — | `null` | response DTO |  |
| 11 | Фактическая дата и время окончания | actualEndDt | null | — | `null` | response DTO |  |
| 12 | Текущий статус | status | string | string | — | response DTO |  |

## 10. Пример ответа

```json
{
  "value": {
    "items": [
      {
        "id": "c777f75f-029d-4d8f-8c69-e74a1d280001",
        "requestNumber": "REQ-2026-00015",
        "type": "Regular",
        "status": "Submitted",
        "priority": "P2",
        "workOrderJdeId": "WO-10025",
        "createdAt": "2026-05-14T09:15:00Z",
        "requestor": {
          "id": "11111111-1111-1111-1111-111111111111",
          "fullName": "Telman Nurzhanov",
          "email": "telman.nurzhanov@example.com"
        },
        "bookingsCount": 2,
        "bookingSummaries": [
          {
            "id": "b777f75f-029d-4d8f-8c69-e74a1d280001",
            "equipmentType": "Excavator",
            "brandModel": "CAT 320",
            "tcoId": "TCO-12345",
            "stateNumber": "A123BC",
            "workCenter": "BHOE",
            "fleetOwner": "Maintenance Fleet",
            "plannedStartDt": "2026-05-20T08:00:00Z",
            "plannedEndDt": "2026-05-22T20:00:00Z",
            "actualStartDt": null,
            "actualEndDt": null,
            "status": "Submitted"
          },
          {
            "id": "b777f75f-029d-4d8f-8c69-e74a1d280002",
            "equipmentType": "Water Truck",
            "brandModel": "HOWO 6x4",
            "tcoId": "TCO-98765",
            "stateNumber": "B456CD",
            "workCenter": "HYDR",
            "fleetOwner": "Operations Fleet",
            "plannedStartDt": "2026-05-21T08:00:00Z",
            "plannedEndDt": "2026-05-21T18:00:00Z",
            "actualStartDt": null,
            "actualEndDt": null,
            "status": "Submitted"
          }
        ]
      }
    ],
    "total": 1
  },
  "isSuccess": true,
  "errors": []
}
```
