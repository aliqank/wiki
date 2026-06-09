**Created:** 2026-06-08  
**Last updated:** 2026-06-08  
**Автор документов:** Telman Nurzhanov (SA)

---

# PUT /maintenance-service-types/:id

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Сохранить изменения вида сервисного обслуживания в справочнике [`Admin`](../../../requirements/Roles and Access Model.md) Panel |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Admin Panel` |
| Endpoint URL | `/api/admin/v1/maintenance-service-types/:id` |
| Метод запроса | `PUT` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Обрабатывает редактирование строки maintenance service type в справочнике [`Admin`](../../../requirements/Roles and Access Model.md) Panel.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-NEW-50 | [`Admin`](../../../requirements/Roles and Access Model.md) manages all reference/handbook values via [`Admin`](../../../requirements/Roles and Access Model.md) Panel | Confirmed | BRD v13 | Справочник управляется через [`Admin`](../../../requirements/Roles and Access Model.md) Panel |
| TCO Booking Tool | AFR-06 | [`Admin`](../../../requirements/Roles and Access Model.md) manages maintenance service types directory used in equipment maintenance contracts | Confirmed | wiki/brd/new FR's/Список FR по equipment.md | Новый handbook для нормализации `serviceType` |

---

## 3. Описание логики работы метода

1. Найти запись в `MaintenanceServiceTypes` WHERE `id` = `:id` AND `isDeleted = false`. Если запись не найдена - вернуть `404 NOT_FOUND`.
2. Провалидировать поле `name`: должен быть передан объект `{ En, Ru, Kz }`; `name.En`, `name.Ru`, `name.Kz` обязательны; `name.Ru` уникален среди `MaintenanceServiceTypes` WHERE `id` != `:id` AND `isDeleted = false`.
3. Если передан `sortOrder`, провалидировать, что это целое число >= 0.
4. Обновить запись в `MaintenanceServiceTypes`; заполнить аудит-поля `updatedAt`, `updatedBy`.
5. Рассчитать `contractsCount` = COUNT(`EquipmentMaintenanceContracts` WHERE `serviceTypeId` = `:id` AND `isDeleted = false`).
6. Вернуть обновлённый объект в формате общего `result wrapper`.

Сущности, участвующие в методе:
- читаются: `MaintenanceServiceTypes`, `EquipmentMaintenanceContracts`
- изменяются: `MaintenanceServiceTypes` (UPDATE)
- транзакционность: не требуется

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
| `NOT_FOUND` | Maintenance service type не найден |
| `VALIDATION_ERROR` | Поле `name` не передано, одно из полей `name.En`, `name.Ru`, `name.Kz` пустое или значение `name.Ru` уже существует в справочнике |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор service type | `id` | `uuid` | `+` | Должен быть валидным UUID v4 | — | Path param | |
| 2 | Наименование service type | `name` | `object` | `+` | Объект `{ En, Ru, Kz }`; `name.En`, `name.Ru`, `name.Kz` обязательны; `name.Ru` уникален среди активных записей | — | Request body | |
| 3 | Порядок сортировки | `sortOrder` | `int` | `-` | Если передан, целое число >= 0 | Текущее значение | Request body | |

---

## 8. Пример запроса

```http
PUT /api/admin/v1/maintenance-service-types/56000001-0000-4000-8000-000000000001
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "name": {
    "En": "Oil and fluid service",
    "Ru": "Замена масла и жидкостей",
    "Kz": "Май мен сұйықтықтарды ауыстыру"
  },
  "sortOrder": 10
}
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий [`result wrapper`](../../common/Result%20Wrapper.md).

### Структура `value`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | MaintenanceServiceTypes.id |  |
| 2 | Наименование | name | object | object | — | MaintenanceServiceTypes |  |
| 3 | Порядок сортировки | sortOrder | int | integer | — | MaintenanceServiceTypes.sortOrder |  |
| 4 | Количество связанных контрактов | contractsCount | int | integer | — | COUNT(EquipmentMaintenanceContracts) |  |

### Структура `value.name`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string | string | — | MaintenanceServiceTypes.nameEn |  |
| 2 | Значение на русском языке | Ru | string | string | — | MaintenanceServiceTypes.nameRu |  |
| 3 | Значение на казахском языке | Kz | string | string | — | MaintenanceServiceTypes.nameKz |  |

## 10. Пример ответа

```json
{
  "value": {
    "id": "56000001-0000-4000-8000-000000000001",
    "name": {
      "En": "Oil and fluid service",
      "Ru": "Замена масла и жидкостей",
      "Kz": "Май мен сұйықтықтарды ауыстыру"
    },
    "sortOrder": 10,
    "contractsCount": 12
  },
  "isSuccess": true,
  "errors": []
}
```

---

## Замечания

1. Изменение service type не требует обновления `EquipmentMaintenanceContracts`, так как связи идут по `serviceTypeId`.
