# GET /jde/work-orders

**Created:** 2026-05-14  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить список Work Orders из JDE для выбора при создании заявки |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Requestor UI` |
| Endpoint URL | `/api/booking/v1/jde/work-orders` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется в блоке выбора WO из JDE.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-NEW-48 | WO list from JDE E1 via direct API | Confirmed | BRD v13 | Метод отдает импортированные WO |
| TCO Booking Tool | FR-NEW-12 | WO mandatory for selected teams | Confirmed | BRD v13 | Список WO нужен для заполнения заявки |

---

## 3. Описание логики работы метода

1. Получить активные записи из `JdeWorkOrders` WHERE `isDeleted = false` AND `isActive = true`.
2. Применить поиск по `jdeWorkOrderId`, `workOrderName`, `workOrderStatusDescription`.
3. Отсортировать по `lastSyncedAt DESC`, затем по `jdeWorkOrderId ASC`.
4. Вернуть пагинированный список.

Сущности, участвующие в методе:
- читаются: `JdeWorkOrders`
- изменения не выполняются

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Requestor` | Выбор WO при создании обычной заявки |
| `ServiceWorkProcessor` | Работа с WO и SWR |

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
| 1 | Поисковая строка | `search` | `string` | `-` | Поиск по номеру и названию WO | — | Query param | |
| 2 | Номер страницы | `page` | `int` | `-` | Целое число >= 1 | `1` | Query param | |
| 3 | Размер страницы | `limit` | `int` | `-` | Целое число >= 1 | `20` | Query param | |

---

## 8. Пример запроса

```http
GET /api/booking/v1/jde/work-orders?search=WO-10025&page=1&limit=20
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
| 1.1 | Элементы текущей страницы | items | array<object> | object[] | — | backend composition from JdeWorkOrders | Коллекция объектов |
| 1.2 | Общее количество записей | total | int | integer | — | backend |  |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | object[] | `[]` | backend |  |

### Структура `value`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Элементы текущей страницы | items | array<object> | object[] | — | backend composition from JdeWorkOrders | Коллекция объектов |
| 2 | Общее количество записей | total | int | integer | — | backend |  |

### Структура `value.items[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | JdeWorkOrders.id |  |
| 2 | Идентификатор Work Order в JDE | jdeWorkOrderId | string | string | — | JdeWorkOrders.jdeWorkOrderId |  |
| 3 | Наименование Work Order | workOrderName | string | string | — | JdeWorkOrders.workOrderName |  |
| 4 | Статус Work Order | workOrderStatus | string | string | — | JdeWorkOrders.workOrderStatus |  |
| 5 | Описание статуса Work Order | workOrderStatusDescription | string | string | — | JdeWorkOrders.workOrderStatusDescription |  |
| 6 | Приоритет | priority | string | string | — | JdeWorkOrders + ref_request_priority |  |
| 7 | Дата и время последней синхронизации | lastSyncedAt | datetime | ISO 8601 | — | JdeWorkOrders.lastSyncedAt |  |

## 10. Пример ответа

```json
{
  "value": {
    "items": [
      {
        "id": "9fda7b6b-aa83-4ec7-a7f7-899a4b430001",
        "jdeWorkOrderId": "WO-10025",
        "workOrderName": "Pump station maintenance",
        "workOrderStatus": "55",
        "workOrderStatusDescription": "Ready for equipment",
        "priority": "P2",
        "lastSyncedAt": "2026-05-14T08:30:00Z"
      }
    ],
    "total": 1
  },
  "isSuccess": true,
  "errors": []
}
```
