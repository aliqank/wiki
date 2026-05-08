**Created:** 2026-05-08  
**Last updated:** 2026-05-08  
**Author:** Telman Nurzhanov (SA)

---

# DELETE /equipment-types/:id

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Выполнить soft delete типа техники; операция блокируется, если к типу привязаны активные единицы техники |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Admin Panel` |
| Endpoint URL | `/api/admin/v1/equipment-types/:id` |
| Метод запроса | `DELETE` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Обрабатывает нажатие иконки «Удалить» в строке списка AP-01. Удаление реализовано как soft delete (установка флага `isDeleted = true`). Обязательный guard на уровне API: удаление невозможно, пока к типу привязана хотя бы одна не удалённая единица техники.

---

## 2. Функциональные требования

Общий перечень требований: Требования к системе DMMS

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-010 | Admin configures dynamic custom characteristics per equipment type via Admin Panel tool; characteristics appear automatically in equipment card, request form, and search filters based on equipment type | Confirmed | BRD v13 | Управление жизненным циклом типов техники (включая удаление) входит в скоуп Admin Panel tool |
| TCO Booking Tool | FR-NEW-50 | Admin manages all reference/handbook values via Admin Panel | Confirmed | BRD v13 | Типы техники — справочная сущность; их удаление входит в управление справочниками |

---

## 3. Описание логики работы метода

1. Найти запись в `EquipmentTypes` WHERE `id` = `:id` AND `isDeleted = false`. Если запись не найдена — вернуть `404 NOT_FOUND`.
2. Проверить наличие активных единиц техники: COUNT(`Equipments` WHERE `equipmentTypeId` = `:id` AND `isDeleted = false`). Если count > 0 — вернуть `409 EQUIPMENT_TYPE_IN_USE` с деталями (`equipmentsCount`).
3. Выполнить soft delete: обновить запись `EquipmentTypes` — установить `isDeleted = true`, `deletedAt` = текущее время, `deletedBy` = ID аутентифицированного пользователя.
4. Вернуть HTTP 204 No Content (без тела ответа).

Сущности, участвующие в методе:
- читаются: `EquipmentTypes` (проверка существования), `Equipments` (guard — агрегат)
- изменяются: `EquipmentTypes` (soft delete: UPDATE `isDeleted`, `deletedAt`, `deletedBy`)
- транзакционность: не требуется

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Admin` | Доступ к административной панели и управлению справочниками типов техники |

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
| `NOT_FOUND` | Тип техники с указанным `id` не найден или уже помечен как удалённый |
| `EQUIPMENT_TYPE_IN_USE` | Удаление невозможно: к типу привязаны активные единицы техники |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор типа техники | `id` | `uuid` | `+` | Должен быть валидным UUID v4 | — | Path param | |

---

## 8. Пример запроса

```http
DELETE /api/admin/v1/equipment-types/7b4f4b4d-52d4-4a77-b6b7-f2b7d7c81111
Authorization: Bearer <token>
```

У метода нет request body.

---

## 9. Возвращаемые данные

При успешном выполнении метод возвращает **HTTP 204 No Content** без тела ответа.

При возникновении ошибки (`409 EQUIPMENT_TYPE_IN_USE`) возвращается стандартный `result wrapper` с заполненным массивом `errors`.

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | `value` | `null` | — | `null` | — | При 204 тело отсутствует |
| 2 | Признак успешности | `isSuccess` | `bool` | boolean | — | backend | Присутствует только в error-ответах |
| 3 | Ошибки | `errors` | `array<object>` | `ApiError[]` | `[]` | backend | Присутствует только в error-ответах |

---

## 10. Пример ответа

**Успех — HTTP 204 No Content:**

```
(нет тела ответа)
```

**Ошибка — HTTP 409 Conflict (`EQUIPMENT_TYPE_IN_USE`):**

```json
{
  "value": null,
  "isSuccess": false,
  "errors": [
    {
      "code": "EQUIPMENT_TYPE_IN_USE",
      "message": "Нельзя удалить: к типу привязано 12 единиц техники",
      "details": { "equipmentsCount": 12 }
    }
  ]
}
```

---

## Замечания

1. Guard на уровне API обязателен, даже если frontend заблаговременно блокирует кнопку «Удалить» при `equipmentsCount > 0` (значение берётся из `GET /equipment-types`). Это защищает от race condition и прямых API-вызовов.
2. Метод выполняет soft delete только записи `EquipmentTypes`. Связанные записи `EquipmentTypeProperties` остаются в базе с `isDeleted = false` — они становятся неактивными вместе с типом, т.к. все GET-методы Admin Panel фильтруют `EquipmentTypes WHERE isDeleted = false`.
3. После успешного soft delete тип техники перестаёт появляться во всех GET-методах (список AP-01, карточка AP-02, поиск для Requestor). Отменить удаление через API невозможно — только прямым вмешательством в БД.
