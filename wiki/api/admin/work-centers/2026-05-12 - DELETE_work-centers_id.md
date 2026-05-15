**Created:** 2026-05-12  
**Last updated:** 2026-05-12  
**Автор документов:** Telman Nurzhanov (SA)

---

# DELETE /work-centers/:id

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Выполнить soft delete work center; операция блокируется, если к нему привязаны активные типы техники |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Admin Panel` |
| Endpoint URL | `/api/admin/v1/work-centers/:id` |
| Метод запроса | `DELETE` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Обрабатывает удаление записи work center из справочника Admin Panel. Удаление реализовано как soft delete. Если work center используется хотя бы одним активным типом техники, удаление запрещено.

---

## 2. Функциональные требования

Общий перечень требований: Требования к системе DMMS

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-013 | Admin создаёт equipment types / equipment-related master data | Confirmed | BRD v13 | Work centers являются связанным справочником для типов техники |
| TCO Booking Tool | FR-014 | Admin редактирует параметры техники и связанные справочные сущности | Confirmed | BRD v13 | Удаление work center входит в управление справочником |
| TCO Booking Tool | FR-NEW-50 | Admin manages all reference/handbook values via Admin Panel | Confirmed | BRD v13 | Work centers управляются через Admin Panel |

---

## 3. Описание логики работы метода

1. Найти запись в `WorkCenters` WHERE `id` = `:id` AND `isDeleted = false`. Если запись не найдена — вернуть `404 NOT_FOUND`.
2. Проверить наличие активных типов техники: COUNT(`EquipmentTypes` WHERE `workCenterId` = `:id` AND `isDeleted = false`). Если count > 0 — вернуть `409 WORK_CENTER_IN_USE` с деталями (`equipmentTypesCount`).
3. Выполнить soft delete: обновить запись `WorkCenters` — установить `isDeleted = true`, `deletedAt` = текущее время, `deletedBy` = ID аутентифицированного пользователя.
4. Вернуть HTTP 204 No Content (без тела ответа).

Сущности, участвующие в методе:
- читаются: `WorkCenters`, `EquipmentTypes` (guard — агрегат)
- изменяются: `WorkCenters` (soft delete: UPDATE `isDeleted`, `deletedAt`, `deletedBy`)
- транзакционность: не требуется

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Admin` | Доступ к административной панели и справочнику work centers |

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
| `NOT_FOUND` | Work center с указанным `id` не найден или уже помечен как удалённый |
| `WORK_CENTER_IN_USE` | Удаление невозможно: к work center привязаны активные типы техники |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор work center | `id` | `uuid` | `+` | Должен быть валидным UUID v4 | — | Path param | |

---

## 8. Пример запроса

```http
DELETE /api/admin/v1/work-centers/12a8b1ce-3aaf-4f55-8ac8-f8cf5d86c222
Authorization: Bearer <token>
```

У метода нет request body.

---

## 9. Возвращаемые данные

При успешном выполнении метод возвращает **HTTP 204 No Content** без тела ответа.

При возникновении ошибки (`409 WORK_CENTER_IN_USE`) возвращается стандартный `result wrapper` с заполненным массивом `errors`.

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

**Ошибка — HTTP 409 Conflict (`WORK_CENTER_IN_USE`):**

```json
{
  "value": null,
  "isSuccess": false,
  "errors": [
    {
      "code": "WORK_CENTER_IN_USE",
      "message": "Нельзя удалить: к work center привязано 6 типов техники",
      "details": { "equipmentTypesCount": 6 }
    }
  ]
}
```

---

## Замечания

1. Guard на уровне API обязателен, даже если frontend заблаговременно блокирует кнопку удаления по `equipmentTypesCount`.
2. После успешного soft delete запись перестаёт возвращаться в `GET /work-centers` и недоступна для выбора в `POST /equipment-types` / `PUT /equipment-types/:id`.
