**Created:** 2026-05-12  
**Last updated:** 2026-05-12  
**Автор документов:** Telman Nurzhanov (SA)

---

# DELETE /equipment-types/:id/properties/:etpId

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Удалить привязку характеристики к типу техники |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Admin Panel` |
| Endpoint URL | `/api/admin/v1/equipment-types/:id/properties/:etpId` |
| Метод запроса | `DELETE` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Удаляет строку характеристики из блока B карточки AP-02. Удаление реализовано как soft delete записи в `EquipmentTypeProperties`.

---

## 2. Функциональные требования

Общий перечень требований: Требования к системе DMMS

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-010 | Admin configures dynamic custom characteristics per equipment type via Admin Panel tool; characteristics appear automatically in equipment card, request form, and search filters based on equipment type | Confirmed | BRD v13 | Метод удаляет привязку характеристики из типа |
| TCO Booking Tool | FR-014 | Admin редактирует параметры техники и связанные справочные сущности | Confirmed | BRD v13 | Нужен для удаления строки характеристики из блока B |
| TCO Booking Tool | FR-NEW-68 | System dynamically shows only type-specific characteristics in request creation form and equipment search filters, based on the selected equipment type | Confirmed | BRD v13 | Удаление привязки влияет на UI и фильтры |

---

## 3. Описание логики работы метода

1. Найти запись в `EquipmentTypes` WHERE `id` = `:id` AND `isDeleted = false`. Если запись не найдена — вернуть `404 NOT_FOUND`.
2. Найти запись в `EquipmentTypeProperties` WHERE `id` = `:etpId` AND `equipmentTypeId` = `:id` AND `isDeleted = false`. Если запись не найдена — вернуть `404 NOT_FOUND`.
3. Выполнить soft delete: обновить запись `EquipmentTypeProperties` — установить `isDeleted = true`, `deletedAt` = текущее время, `deletedBy` = ID аутентифицированного пользователя.
4. Вернуть HTTP 204 No Content (без тела ответа).

Сущности, участвующие в методе:
- читаются: `EquipmentTypes`, `EquipmentTypeProperties`
- изменяются: `EquipmentTypeProperties` (soft delete)
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
| `NOT_FOUND` | Тип техники или запись привязки характеристики не найдена |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор типа техники | `id` | `uuid` | `+` | Должен быть валидным UUID v4 | — | Path param | |
| 2 | Идентификатор привязки характеристики | `etpId` | `uuid` | `+` | Должен быть валидным UUID v4 | — | Path param | |

---

## 8. Пример запроса

```http
DELETE /api/admin/v1/equipment-types/7b4f4b4d-52d4-4a77-b6b7-f2b7d7c81111/properties/a1b2c3d4-0001-4000-8000-000000000003
Authorization: Bearer <token>
```

У метода нет request body.

---

## 9. Возвращаемые данные

При успешном выполнении метод возвращает **HTTP 204 No Content** без тела ответа.

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

---

## Замечания

1. Удаляется только привязка `EquipmentTypeProperties`; сама справочная характеристика в `Properties` остаётся без изменений.
2. После soft delete данная характеристика перестаёт возвращаться в `GET /equipment-types/:id/properties` и в карточке `GET /equipment-types/:id`.
