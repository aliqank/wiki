**Created:** 2026-06-08  
**Last updated:** 2026-06-08  
**Автор документов:** Telman Nurzhanov (SA)

---

# DELETE /maintenance-service-types/:id

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Выполнить soft delete вида сервисного обслуживания |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Admin Panel` |
| Endpoint URL | `/api/admin/v1/maintenance-service-types/:id` |
| Метод запроса | `DELETE` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Удаляет запись service type через soft delete. Если запись используется активными maintenance contracts, удаление запрещено.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-NEW-50 | [`Admin`](../../../requirements/Roles and Access Model.md) manages all reference/handbook values via [`Admin`](../../../requirements/Roles and Access Model.md) Panel | Confirmed | BRD v13 | Справочник управляется через [`Admin`](../../../requirements/Roles and Access Model.md) Panel |
| TCO Booking Tool | AFR-06 | [`Admin`](../../../requirements/Roles and Access Model.md) manages maintenance service types directory used in equipment maintenance contracts | Confirmed | wiki/brd/new FR's/Список FR по equipment.md | Новый handbook для нормализации `serviceType` |

---

## 3. Описание логики работы метода

1. Найти запись в `MaintenanceServiceTypes` WHERE `id` = `:id` AND `isDeleted = false`. Если запись не найдена - вернуть `404 NOT_FOUND`.
2. Проверить наличие активных контрактов: COUNT(`EquipmentMaintenanceContracts` WHERE `serviceTypeId` = `:id` AND `isDeleted = false`). Если count > 0 - вернуть `409 MAINTENANCE_SERVICE_TYPE_IN_USE`.
3. Выполнить soft delete: установить `isDeleted = true`, `deletedAt`, `deletedBy`.
4. Вернуть HTTP 204 No Content.

Сущности, участвующие в методе:
- читаются: `MaintenanceServiceTypes`, `EquipmentMaintenanceContracts`
- изменяются: `MaintenanceServiceTypes` (soft delete)
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
| `MAINTENANCE_SERVICE_TYPE_IN_USE` | Удаление невозможно: service type используется активными maintenance contracts |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор service type | `id` | `uuid` | `+` | Валидный UUID v4 | — | Path param | |

---

## 8. Пример запроса

```http
DELETE /api/admin/v1/maintenance-service-types/56000001-0000-4000-8000-000000000001
Authorization: Bearer <token>
```

---

## 9. Возвращаемые данные

При успешном выполнении метод возвращает **HTTP 204 No Content** без тела ответа.

## 10. Пример ответа

**Успех — HTTP 204 No Content:**

```
(нет тела ответа)
```

**Ошибка — HTTP 409 Conflict (`MAINTENANCE_SERVICE_TYPE_IN_USE`):**

```json
{
  "value": null,
  "isSuccess": false,
  "errors": [
    {
      "code": "MAINTENANCE_SERVICE_TYPE_IN_USE",
      "message": "Нельзя удалить: service type используется 12 активными контрактами",
      "details": { "contractsCount": 12 }
    }
  ]
}
```

---

## Замечания

1. Guard обязателен, даже если frontend заранее блокирует кнопку удаления по `contractsCount`.
