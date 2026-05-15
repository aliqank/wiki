# GET /jde/work-orders/{id}/steps

**Created:** 2026-05-14  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить шаги выбранного Work Order |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Requestor UI` |
| Endpoint URL | `/api/booking/v1/jde/work-orders/{id}/steps` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется для выбора шага WO / work center на уровне booking item.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-NEW-48 | WO list from JDE E1 via direct API; auto-generate draft | Confirmed | BRD v13 | Шаги WO нужны для детализации заявки |
| TCO Booking Tool | FR-NEW-12 | WO mandatory for selected teams | Confirmed | BRD v13 | Шаг WO связан с work center |

---

## 3. Описание логики работы метода

1. Проверить существование WO в `JdeWorkOrders`.
2. Выбрать активные записи из `JdeWorkOrderSteps` по `jdeWorkOrderRefId`.
3. Подтянуть `WorkCenters` для возврата кода и наименования шага.
4. Отсортировать шаги по `startDt ASC`, затем по `workCenterId ASC`.
5. Вернуть коллекцию шагов в общем `result wrapper`.

Сущности, участвующие в методе:
- читаются: `JdeWorkOrders`, `JdeWorkOrderSteps`, `WorkCenters`
- изменения не выполняются

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Requestor` | Выбор WO и шага WO |
| `ServiceWorkProcessor` | Работа с SWR |

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
| `NOT_FOUND` | WO не найден |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор WO | `id` | `uuid` | `+` | Должен существовать | — | Path param | Идентификатор записи `JdeWorkOrders.id` |

---

## 8. Пример запроса

```http
GET /api/booking/v1/jde/work-orders/9fda7b6b-aa83-4ec7-a7f7-899a4b430001/steps
Authorization: Bearer <token>
Content-Type: application/json
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий `result wrapper`.

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | value | array<object> | object[] | — | backend aggregation |  |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | object[] | `[]` | backend |  |

### Структура `value[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | JdeWorkOrderSteps.id |  |
| 2 | Идентификатор work center | workCenterId | uuid | UUID v4 | — | WorkCenters.id |  |
| 3 | Код work center | workCenterCode | string | string | — | WorkCenters.code |  |
| 4 | Наименование work center | workCenterName | object | object | — | WorkCenters |  |
| 5 | Наименование шага | stepName | string | string | — | JdeWorkOrderSteps.stepName |  |
| 6 | Объём шага | stepVolume | int | integer | — | JdeWorkOrderSteps.stepVolume |  |
| 7 | Дата и время начала | startDt | datetime | ISO 8601 | — | JdeWorkOrderSteps.startDt |  |
| 8 | Дата и время окончания | endDt | datetime | ISO 8601 | — | JdeWorkOrderSteps.endDt |  |

### Структура `value[].workCenterName`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string | string | — | WorkCenters |  |
| 2 | Значение на русском языке | Ru | string | string | — | WorkCenters |  |
| 3 | Значение на казахском языке | Kz | string | string | — | WorkCenters |  |

## 10. Пример ответа

```json
{
  "value": [
    {
      "id": "d8975a3d-a1a0-4f78-878c-e854ff560001",
      "workCenterId": "12a8b1ce-3aaf-4f55-8ac8-f8cf5d86c222",
      "workCenterCode": "WC-100",
      "workCenterName": {
        "En": "Drilling Operations",
        "Ru": "Буровые работы",
        "Kz": "Бұрғылау жұмыстары"
      },
      "stepName": "Excavation",
      "stepVolume": 2,
      "startDt": "2026-05-20T08:00:00Z",
      "endDt": "2026-05-22T18:00:00Z"
    }
  ],
  "isSuccess": true,
  "errors": []
}
```
