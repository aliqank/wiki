**Created:** 2026-06-08  
**Last updated:** 2026-06-08  
**Автор документов:** Telman Nurzhanov (SA)

---

# POST /maintenance-service-types

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Создать новый вид сервисного обслуживания в справочнике Admin Panel |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Admin Panel` |
| Endpoint URL | `/api/admin/v1/maintenance-service-types` |
| Метод запроса | `POST` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Обеспечивает создание записи maintenance service type через форму или inline-create на экране справочника Admin Panel.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-NEW-50 | Admin manages all reference/handbook values via Admin Panel | Confirmed | BRD v13 | Справочник управляется через Admin Panel |
| TCO Booking Tool | AFR-06 | Admin manages maintenance service types directory used in equipment maintenance contracts | Confirmed | wiki/brd/new FR's/Список FR по equipment.md | Новый handbook для нормализации `serviceType` |

---

## 3. Описание логики работы метода

1. Принять тело запроса; провалидировать обязательные поля `name`.
2. Проверить поле `name`: должен быть передан объект `{ En, Ru, Kz }`; `name.Ru` обязателен.
3. Проверить уникальность `name.Ru` в таблице `MaintenanceServiceTypes` WHERE `isDeleted = false`. Если запись уже существует - вернуть `422 VALIDATION_ERROR`.
4. Рассчитать `sortOrder` как `MAX(sortOrder) + 10`, если значение явно не передано.
5. Создать новую запись в `MaintenanceServiceTypes`; заполнить аудит-поля `createdAt`, `createdBy`.
6. Вернуть созданный объект в общем `result wrapper` с HTTP 201.

Сущности, участвующие в методе:
- читаются: `MaintenanceServiceTypes`
- изменяются: `MaintenanceServiceTypes` (INSERT)
- транзакционность: не требуется

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Admin` | Доступ к административной панели и справочнику maintenance service types |

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
| `FORBIDDEN` | У пользователя нет роли `Admin` |
| `VALIDATION_ERROR` | Поле `name` не передано, `name.Ru` пустое или значение уже существует в справочнике |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Наименование service type | `name` | `object` | `+` | Объект `{ En, Ru, Kz }`; `name.Ru` обязателен; `name.Ru` уникален среди активных записей | — | Request body | |
| 2 | Порядок сортировки | `sortOrder` | `int` | `-` | Если передан, целое число >= 0 | `MAX + 10` | Request body | |

---

## 8. Пример запроса

```http
POST /api/admin/v1/maintenance-service-types
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "name": {
    "En": "Oil change",
    "Ru": "Замена масла",
    "Kz": "Май ауыстыру"
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
| 4 | Количество связанных контрактов | contractsCount | int | integer | `0` | backend | Для новой записи всегда `0` |

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
      "En": "Oil change",
      "Ru": "Замена масла",
      "Kz": "Май ауыстыру"
    },
    "sortOrder": 10,
    "contractsCount": 0
  },
  "isSuccess": true,
  "errors": []
}
```

---

## Замечания

1. После создания запись сразу доступна для выбора в CRUD `EquipmentMaintenanceContracts`.
