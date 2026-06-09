**Created:** 2026-05-12  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

# GET /measurement-units

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить постраничный список единиц измерения |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Admin Panel` |
| Endpoint URL | `/api/admin/v1/measurement-units` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется для экрана управления справочником единиц измерения в [`Admin`](../../../requirements/Roles%20and%20Access%20Model.md) Panel.

---

## 2. Функциональные требования

Общий перечень требований: Требования к системе DMMS

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-010 | [`Admin`](../../../requirements/Roles%20and%20Access%20Model.md) configures dynamic custom characteristics per equipment type via [`Admin`](../../../requirements/Roles%20and%20Access%20Model.md) Panel tool | Confirmed | BRD v13 | Единицы измерения используются в характеристиках |
| TCO Booking Tool | FR-014 | [`Admin`](../../../requirements/Roles%20and%20Access%20Model.md) редактирует параметры техники и связанные справочные сущности | Confirmed | BRD v13 | Метод нужен для управления справочником |
| TCO Booking Tool | FR-NEW-50 | [`Admin`](../../../requirements/Roles%20and%20Access%20Model.md) manages all reference/handbook values via [`Admin`](../../../requirements/Roles%20and%20Access%20Model.md) Panel | Confirmed | BRD v13 | Справочник управляется через [`Admin`](../../../requirements/Roles%20and%20Access%20Model.md) Panel |

---

## 3. Описание логики работы метода

1. Получить записи из `MeasurementUnits` WHERE `isDeleted = false`.
2. Если передан `search`, применить фильтр по `MeasurementUnits.code` и `MeasurementUnits.displayName`.
3. Для каждой записи рассчитать `propertiesCount` = COUNT(`Properties` WHERE `unitId` = `MeasurementUnits.id` AND `isDeleted = false`).
4. Отсортировать по `sortOrder ASC`, затем по `displayName ASC`.
5. Вернуть страницу данных в формате общего `result wrapper` с `PaginatedResult` внутри `value`.

Сущности, участвующие в методе:
- читаются: `MeasurementUnits`, `Properties`
- изменения не выполняются
- транзакционность не требуется, метод read-only

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| [`Admin`](../../../requirements/Roles%20and%20Access%20Model.md) | Доступ к административной панели и справочнику единиц измерения |

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
| `FORBIDDEN` | У пользователя нет роли [`Admin`](../../../requirements/Roles%20and%20Access%20Model.md) |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Поиск по коду или названию | `search` | `string` | `-` | Если передан, используется как фильтр по `code` и `displayName` | — | Query param | |
| 2 | Номер страницы | `page` | `int` | `-` | Целое число >= 1 | `1` | Query param | |
| 3 | Размер страницы | `limit` | `int` | `-` | Целое число >= 1 | `20` | Query param | |

---

## 8. Пример запроса

```http
GET /api/admin/v1/measurement-units?search=л&page=1&limit=20
Authorization: Bearer <token>
Content-Type: application/json
```

У метода нет request body.

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий [`result wrapper`](../../common/Result%20Wrapper.md).

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | value | object | PaginatedResult | — | backend aggregation |  |
| 1.1 | Элементы текущей страницы | items | array<object> | object[] | — | backend composition from MeasurementUnits + Properties | Коллекция объектов |
| 1.2 | Общее количество записей | total | int | integer | — | backend |  |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | object[] | `[]` | backend |  |

### Структура `value`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Элементы текущей страницы | items | array<object> | object[] | — | backend composition from MeasurementUnits + Properties | Коллекция объектов |
| 2 | Общее количество записей | total | int | integer | — | backend |  |

### Структура `value.items[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | string | string | — | MeasurementUnits |  |
| 2 | Код записи | code | string | string | — | MeasurementUnits |  |
| 3 | Отображаемое наименование | displayName | string | string | — | MeasurementUnits |  |
| 4 | Порядок сортировки | sortOrder | int | integer | — | MeasurementUnits |  |
| 5 | Количество свойств | propertiesCount | int | integer | — | backend composition from MeasurementUnits + Properties |  |

## 10. Пример ответа

```json
{
  "value": {
    "items": [
      {
        "id": "u0000001-0000-4000-8000-000000000001",
        "code": "л",
        "displayName": "Литр",
        "sortOrder": 1,
        "propertiesCount": 3
      }
    ],
    "total": 12
  },
  "isSuccess": true,
  "errors": []
}
```

---

## Замечания

1. Метод применяет скрытый фильтр `isDeleted = false` как к `MeasurementUnits`, так и к `Properties` при расчёте `propertiesCount`.
2. `propertiesCount` используется для guard-логики удаления.
