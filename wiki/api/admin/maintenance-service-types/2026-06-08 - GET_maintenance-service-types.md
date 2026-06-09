**Created:** 2026-06-08  
**Last updated:** 2026-06-08  
**Автор документов:** Telman Nurzhanov (SA)

---

# GET /maintenance-service-types

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить постраничный список видов сервисного обслуживания для справочника [`Admin`](../../../requirements/Roles and Access Model.md) Panel |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Admin Panel` |
| Endpoint URL | `/api/admin/v1/maintenance-service-types` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется для экрана управления справочником видов сервисного обслуживания, которые затем выбираются в `EquipmentMaintenanceContracts`.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-NEW-50 | [`Admin`](../../../requirements/Roles and Access Model.md) manages all reference/handbook values via [`Admin`](../../../requirements/Roles and Access Model.md) Panel | Confirmed | BRD v13 | Справочник управляется через [`Admin`](../../../requirements/Roles and Access Model.md) Panel |
| TCO Booking Tool | AFR-06 | [`Admin`](../../../requirements/Roles and Access Model.md) manages maintenance service types directory used in equipment maintenance contracts | Confirmed | wiki/brd/new FR's/Список FR по equipment.md | Новый handbook для нормализации `serviceType` |

---

## 3. Описание логики работы метода

1. Получить записи из `MaintenanceServiceTypes` WHERE `isDeleted = false`.
2. Если передан `search`, применить фильтр по `nameEn`, `nameRu`, `nameKz`.
3. Для каждой записи рассчитать `contractsCount` = COUNT(`EquipmentMaintenanceContracts` WHERE `serviceTypeId` = `MaintenanceServiceTypes.id` AND `isDeleted = false`).
4. Отсортировать результат по `sortOrder ASC`, затем по `nameRu ASC`.
5. Вернуть страницу данных в формате общего `result wrapper` с `PaginatedResult` внутри `value`.

Сущности, участвующие в методе:
- читаются: `MaintenanceServiceTypes`, `EquipmentMaintenanceContracts`
- изменения не выполняются
- транзакционность не требуется, метод read-only

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| [`Admin`](../../../requirements/Roles and Access Model.md) | Доступ к административной панели и справочнику maintenance service types |

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

HTTP-коды: `401 Unauthorized`, `403 Forbidden`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Поисковая строка по названию | `search` | `string` | `-` | Если передан, используется как фильтр по `nameEn`, `nameRu`, `nameKz` | — | Query param | |
| 2 | Номер страницы | `page` | `int` | `-` | Целое число >= 1 | `1` | Query param | |
| 3 | Размер страницы | `limit` | `int` | `-` | Целое число >= 1 | `20` | Query param | |

---

## 8. Пример запроса

```http
GET /api/admin/v1/maintenance-service-types?search=oil&page=1&limit=20
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
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | MaintenanceServiceTypes.id |  |
| 2 | Наименование | name | object | object | — | MaintenanceServiceTypes |  |
| 3 | Порядок сортировки | sortOrder | int | integer | — | MaintenanceServiceTypes.sortOrder |  |
| 4 | Количество связанных контрактов | contractsCount | int | integer | — | COUNT(EquipmentMaintenanceContracts) | Используется для guard-логики удаления |

### Структура `value.items[].name`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string | string | — | MaintenanceServiceTypes.nameEn |  |
| 2 | Значение на русском языке | Ru | string | string | — | MaintenanceServiceTypes.nameRu |  |
| 3 | Значение на казахском языке | Kz | string | string | — | MaintenanceServiceTypes.nameKz |  |

## 10. Пример ответа

```json
{
  "value": {
    "items": [
      {
        "id": "56000001-0000-4000-8000-000000000001",
        "name": {
          "En": "Oil change",
          "Ru": "Замена масла",
          "Kz": "Май ауыстыру"
        },
        "sortOrder": 10,
        "contractsCount": 12
      }
    ],
    "total": 6
  },
  "isSuccess": true,
  "errors": []
}
```

---

## Замечания

1. Справочник предназначен для выбора `serviceTypeId` в `EquipmentMaintenanceContracts`, а не для свободного текстового ввода.
2. `contractsCount` используется для блокировки удаления service type, если он уже используется активными контрактами.
