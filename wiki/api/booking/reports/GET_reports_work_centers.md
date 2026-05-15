# GET /reports/work-centers

**Created:** 2026-05-14  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить отчет по Work Centers |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Reporting` |
| Endpoint URL | `/api/booking/v1/reports/work-centers` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется для группировки заявок и броней по work center.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-NEW-47 | Work center reports (group by WC code) | Pending details | BRD v13 | Прямое покрытие |

---

## 3. Описание логики работы метода

1. Проверить права доступа к отчетам.
2. Выбрать `Bookings` с заполненным `workCenterId` за выбранный период.
3. Сгруппировать данные по `WorkCenters.code`.
4. Рассчитать агрегаты: count requests, count bookings, count confirmed, count closed.
5. Вернуть пагинированный список.

Сущности:
- читаются: `Bookings`, `BookingRequests`, `WorkCenters`

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Admin` | Полный отчет по work centers |
| `ServiceWorkProcessor` | Отчет по рабочим заявкам |

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
| `FORBIDDEN` | Нет прав на отчет |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Дата начала периода | `from` | `date` | `-` | — | — | Query param | |
| 2 | Дата окончания периода | `to` | `date` | `-` | — | — | Query param | |
| 3 | Идентификатор work center | `workCenterId` | `uuid` | `-` | Если передан, должен существовать | — | Query param | |
| 4 | Номер страницы | `page` | `int` | `-` | >= 1 | `1` | Query param | |
| 5 | Размер страницы | `limit` | `int` | `-` | >= 1 | `20` | Query param | |

---

## 8. Пример запроса

```http
GET /api/booking/v1/reports/work-centers?from=2026-05-01&to=2026-05-31&page=1&limit=20
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
| 1.1 | Элементы текущей страницы | items | array<object> | object[] | — | backend composition from Bookings + BookingRequests + WorkCenters | Коллекция объектов |
| 1.2 | Общее количество записей | total | int | integer | — | backend |  |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | object[] | `[]` | backend |  |

### Структура `value`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Элементы текущей страницы | items | array<object> | object[] | — | backend composition from Bookings + BookingRequests + WorkCenters | Коллекция объектов |
| 2 | Общее количество записей | total | int | integer | — | backend |  |

### Структура `value.items[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор work center | workCenterId | uuid | UUID v4 | — | WorkCenters.id |  |
| 2 | Код work center | workCenterCode | string | string | — | WorkCenters.code |  |
| 3 | Количество заявок | requestsCount | int | integer | — | COUNT(BookingRequests) |  |
| 4 | Количество броней | bookingsCount | int | integer | — | COUNT(Bookings) |  |
| 5 | Количество подтверждённых записей | confirmedCount | int | integer | — | COUNT(Bookings) |  |
| 6 | Количество закрытых записей | closedCount | int | integer | — | COUNT(Bookings) |  |

## 10. Пример ответа

```json
{
  "value": {
    "items": [
      {
        "workCenterId": "12a8b1ce-3aaf-4f55-8ac8-f8cf5d86c222",
        "workCenterCode": "WC-100",
        "requestsCount": 12,
        "bookingsCount": 26,
        "confirmedCount": 19,
        "closedCount": 14
      }
    ],
    "total": 1
  },
  "isSuccess": true,
  "errors": []
}
```
