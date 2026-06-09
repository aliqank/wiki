**Created:** 2026-05-12  
**Last updated:** 2026-05-22  
**Автор документов:** Telman Nurzhanov (SA)

---

# DELETE /properties/:id

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Выполнить soft delete характеристики |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Admin Panel` |
| Endpoint URL | `/api/admin/v1/properties/:id` |
| Метод запроса | `DELETE` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Удаляет характеристику через soft delete. Если она используется активными типами техники или имеет активные значения/использование, удаление блокируется.

---

## 2. Функциональные требования

Общий перечень требований: Требования к системе DMMS

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-010 | [`Admin`](../../../requirements/Roles%20and%20Access%20Model.md) configures dynamic custom characteristics per equipment type via [`Admin`](../../../requirements/Roles%20and%20Access%20Model.md) Panel tool | Confirmed | BRD v13 | Guard обязателен, т.к. характеристика участвует в EAV |
| TCO Booking Tool | FR-014 | [`Admin`](../../../requirements/Roles%20and%20Access%20Model.md) редактирует параметры техники и связанные справочные сущности | Confirmed | BRD v13 | Удаление входит в управление справочником |

---

## 3. Описание логики работы метода

1. Найти запись в `Properties` WHERE `id` = `:id` AND `isDeleted = false`. Если запись не найдена — вернуть `404 NOT_FOUND`.
2. Проверить наличие активных связей в `EquipmentTypeProperties` WHERE `propertyId` = `:id` AND `isDeleted = false`. Если count > 0 — вернуть `409 PROPERTY_IN_USE`.
3. Проверить наличие активных значений в `EquipmentProperties` WHERE `propertyId` = `:id` AND `isDeleted = false`. Если count > 0 — вернуть `409 PROPERTY_IN_USE`.
4. Выполнить soft delete: установить `isDeleted = true`, `deletedAt`, `deletedBy`.
5. Вернуть HTTP 204 No Content.

Сущности, участвующие в методе:
- читаются: `Properties`, `EquipmentTypeProperties`, `EquipmentProperties`
- изменяются: `Properties` (soft delete)
- транзакционность: не требуется

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| [`Admin`](../../../requirements/Roles%20and%20Access%20Model.md) | Доступ к административной панели и справочнику характеристик |

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
| `NOT_FOUND` | Характеристика не найдена |
| `PROPERTY_IN_USE` | Удаление невозможно: характеристика используется активными данными |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор характеристики | `id` | `uuid` | `+` | Валидный UUID v4 | — | Path param | |

---

## 8. Пример запроса

```http
DELETE /api/admin/v1/properties/p0000001-0000-4000-8000-000000000003
Authorization: Bearer <token>
```

---

## 9. Возвращаемые данные

При успешном выполнении метод возвращает **HTTP 204 No Content** без тела ответа.

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | value | null | — | `null` | — | При 204 тело отсутствует |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend | Присутствует только в error-ответах |
| 3 | Ошибки | errors | array<object> | ApiError[] | `[]` | backend | Присутствует только в error-ответах |

## 10. Пример ответа

**Успех — HTTP 204 No Content:**

```
(нет тела ответа)
```

**Ошибка — HTTP 409 Conflict (`PROPERTY_IN_USE`):**

```json
{
  "value": null,
  "isSuccess": false,
  "errors": [
    {
      "code": "PROPERTY_IN_USE",
      "message": "Нельзя удалить: характеристика используется в 4 типах техники",
      "details": { "propertyCode": "maximum_depth", "equipmentTypesCount": 4, "equipmentValuesCount": 0 }
    }
  ]
}
```

---

## Замечания

1. Guard обязателен, даже если frontend заранее блокирует кнопку удаления.
2. Записи `PropertyEnumValues` не удаляются отдельно; они становятся неактивными вместе с родительской характеристикой через фильтр по `Properties.isDeleted = false`.
3. В error-response допускается возврат `propertyCode` в `errors[].details`, чтобы frontend мог точнее показать пользователю, какая характеристика заблокирована к удалению.
