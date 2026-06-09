**Created:** 2026-05-12  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

# POST /measurement-units

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Создать новую единицу измерения |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Admin Panel` |
| Endpoint URL | `/api/admin/v1/measurement-units` |
| Метод запроса | `POST` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется для создания новой единицы измерения в справочнике [`Admin`](../../../requirements/Roles and Access Model.md) Panel.

---

## 2. Функциональные требования

Общий перечень требований: Требования к системе DMMS

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-010 | [`Admin`](../../../requirements/Roles and Access Model.md) configures dynamic custom characteristics per equipment type via [`Admin`](../../../requirements/Roles and Access Model.md) Panel tool | Confirmed | BRD v13 | Единицы измерения используются в характеристиках |
| TCO Booking Tool | FR-014 | [`Admin`](../../../requirements/Roles and Access Model.md) редактирует параметры техники и связанные справочные сущности | Confirmed | BRD v13 | Нужен для пополнения справочника |
| TCO Booking Tool | FR-NEW-50 | [`Admin`](../../../requirements/Roles and Access Model.md) manages all reference/handbook values via [`Admin`](../../../requirements/Roles and Access Model.md) Panel | Confirmed | BRD v13 | Справочник управляется через [`Admin`](../../../requirements/Roles and Access Model.md) Panel |

---

## 3. Описание логики работы метода

1. Принять тело запроса; провалидировать обязательные поля `code`, `displayName`.
2. Проверить уникальность `code` среди `MeasurementUnits` WHERE `isDeleted = false`.
3. Если `sortOrder` = `null`, вычислить `MAX(sortOrder) + 1` по активным записям; если таблица пуста, присвоить `1`.
4. Создать запись в `MeasurementUnits`; заполнить аудит-поля `createdAt`, `createdBy`.
5. Вернуть созданный объект с `propertiesCount = 0`.

Сущности, участвующие в методе:
- читаются: `MeasurementUnits`
- изменяются: `MeasurementUnits` (INSERT)
- транзакционность: не требуется

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| [`Admin`](../../../requirements/Roles and Access Model.md) | Доступ к административной панели и справочнику единиц измерения |

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
| `FORBIDDEN` | У пользователя нет роли [`Admin`](../../../requirements/Roles and Access Model.md) |
| `VALIDATION_ERROR` | Поле `code` пустое или уже существует |
| `VALIDATION_ERROR` | Поле `sortOrder` имеет недопустимое значение |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Код единицы измерения | `code` | `string` | `+` | Непустая строка; уникальная среди активных записей | — | Request body | |
| 2 | Название для UI | `displayName` | `string` | `+` | Непустая строка | — | Request body | |
| 3 | Порядок отображения | `sortOrder` | `int` | `-` | Целое число >= 1, если передано | `null` → MAX + 1 | Request body | |

---

## 8. Пример запроса

```http
POST /api/admin/v1/measurement-units
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "code": "л",
  "displayName": "Литр",
  "sortOrder": null
}
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий [`result wrapper`](../../common/Result%20Wrapper.md).

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | value | object | object | — | backend aggregation |  |
| 1.1 | Идентификатор записи | id | string | string | — | MeasurementUnits |  |
| 1.2 | Код записи | code | string | string | — | MeasurementUnits |  |
| 1.3 | Отображаемое наименование | displayName | string | string | — | MeasurementUnits |  |
| 1.4 | Порядок сортировки | sortOrder | int | integer | — | MeasurementUnits |  |
| 1.5 | Количество свойств | propertiesCount | int | integer | — | backend composition from MeasurementUnits |  |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | object[] | `[]` | backend |  |

### Структура `value`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | string | string | — | MeasurementUnits |  |
| 2 | Код записи | code | string | string | — | MeasurementUnits |  |
| 3 | Отображаемое наименование | displayName | string | string | — | MeasurementUnits |  |
| 4 | Порядок сортировки | sortOrder | int | integer | — | MeasurementUnits |  |
| 5 | Количество свойств | propertiesCount | int | integer | — | backend composition from MeasurementUnits |  |

## 10. Пример ответа

```json
{
  "value": {
    "id": "u0000001-0000-4000-8000-000000000001",
    "code": "л",
    "displayName": "Литр",
    "sortOrder": 1,
    "propertiesCount": 0
  },
  "isSuccess": true,
  "errors": []
}
```

---

## Замечания

1. Уникальность `code` проверяется только среди активных записей.
2. После создания единица измерения сразу доступна для выбора в справочнике `Properties`.
