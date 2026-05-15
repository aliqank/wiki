# GET /transport/bookings

**Created:** 2026-05-14  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить очередь заявок на транспортировку |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Transportation UI` |
| Endpoint URL | `/api/booking/v1/transport/bookings` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Наполняет очередь для Transportation Responsible.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-NEW-29 | Three-step Unwheeled flow | Confirmed | BRD v13 | Очередь транспортных задач |
| TCO Booking Tool | FR-NEW-35 | Notification to Transportation Responsible | Confirmed | BRD v13 | Получатель рассматривает очередь |

---

## 3. Описание логики работы метода

1. Проверить роль текущего пользователя `TransportationResponsible`.
2. Выбрать брони, для которых требуется транспортировка и ожидается решение транспортной роли.
3. Подтянуть данные заявки и техники.
4. Вернуть пагинированный список.

Сущности:
- читаются: `Bookings`, `BookingRequests`, `Equipments`, `EquipmentTypes`

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `TransportationResponsible` | Доступ к транспортным задачам по unwheeled технике |

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
| `FORBIDDEN` | Нет роли `TransportationResponsible` |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Поисковая строка | `search` | `string` | `-` | Поиск по заявке и технике | — | Query param | |
| 2 | Номер страницы | `page` | `int` | `-` | >= 1 | `1` | Query param | |
| 3 | Размер страницы | `limit` | `int` | `-` | >= 1 | `20` | Query param | |

---

## 8. Пример запроса

```http
GET /api/booking/v1/transport/bookings?page=1&limit=20
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
| 1.1 | Элементы текущей страницы | items | array<object> | object[] | — | backend composition from Bookings + BookingRequests + Equipments + EquipmentTypes | Коллекция объектов |
| 1.2 | Общее количество записей | total | int | integer | — | backend |  |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | object[] | `[]` | backend |  |

### Структура `value`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Элементы текущей страницы | items | array<object> | object[] | — | backend composition from Bookings + BookingRequests + Equipments + EquipmentTypes | Коллекция объектов |
| 2 | Общее количество записей | total | int | integer | — | backend |  |

### Структура `value.items[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | Bookings.id |  |
| 2 | Номер заявки | requestNumber | string | string | — | BookingRequests.requestNumber |  |
| 3 | ТШО-номер техники | tcoId | string | string | — | Equipments.tcoId |  |
| 4 | Текущий статус | status | string | string | — | Bookings + BookingStatuses |  |

## 10. Пример ответа

```json
{
  "value": {
    "items": [
      {
        "id": "aaabbbcc-dddd-4444-8888-123456780001",
        "requestNumber": "REQ-2026-00018",
        "tcoId": "TCO-200112",
        "status": "Confirmed"
      }
    ],
    "total": 1
  },
  "isSuccess": true,
  "errors": []
}
```
