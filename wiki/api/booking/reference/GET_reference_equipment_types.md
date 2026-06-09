# GET /reference/equipment-types

**Created:** 2026-05-18  
**Last updated:** 2026-06-02  
**Автор документов:** OpenCode

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить справочник типов техники для фильтров формы создания заявки |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / [`Requestor`](../../../requirements/Roles and Access Model.md) UI` |
| Endpoint URL | `/api/booking/v1/reference/equipment-types` |
| Метод запроса | `GET` |
| Связанные use cases | [`UC-REQ-02 - Создание новой заявки (Draft-first)`](../../../requirements/usecases/Requestor/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first)/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first).md), [`UC-REQ-02.3 - Поиск техники для добавления в заявку`](../../../requirements/usecases/[`Requestor`](../../../requirements/Roles and Access Model.md)/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first)/UC-REQ-02.3%20-%20Поиск%20техники%20для%20добавления%20в%20заявку.md) |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый reference API для UC-2. Используется при открытии модалки выбора техники до выполнения [`GET /equipment/search`](../requestor/GET_equipment_search.md).

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-NEW-39 | Dynamic search filters by equipment type | Confirmed | BRD v13 | Тип техники выбирается до выполнения поиска |
| TCO Booking Tool | FR-NEW-68 | System dynamically shows only type-specific characteristics | Confirmed | BRD v13 | Метод отдает список доступных типов техники |

---

## 3. Описание логики работы метода

1. Выбрать активные записи из `EquipmentTypes`.
2. Вернуть компактный справочник для фильтра формы поиска техники.
3. Сортировать записи по отображаемому имени.

Сущности, участвующие в методе:
- читаются: [`EquipmentTypes`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#1-equipmenttypes), [`WorkCenters`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#3-workcenters)
- изменения не выполняются

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| [`Requestor`](../../../requirements/Roles and Access Model.md) | Создание и редактирование собственных заявок |

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
| `FORBIDDEN` | Нет необходимой роли |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`

---

## 7. Параметры метода

Метод не принимает параметров.

---

## 8. Пример запроса

```http
GET /api/booking/v1/reference/equipment-types
Authorization: Bearer <token>
Content-Type: application/json
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий [`result wrapper`](../../common/Result%20Wrapper.md).

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | value | array<object> | object[] | — | backend aggregation |  |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | object[] | `[]` | backend |  |

### Структура `value[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор типа техники | id | uuid | UUID v4 | — | EquipmentTypes.id |  |
| 2 | Код типа техники | code | string | string | — | EquipmentTypes.code |  |
| 3 | Наименование типа техники | name | object | object | — | EquipmentTypes | Локализованное значение |
| 4 | Идентификатор work center по умолчанию | workCenterId | uuid | UUID v4 | — | EquipmentTypes.workCenterId | Может использоваться для клиентских подсказок |

### Структура `value[].name`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string | string | — | EquipmentTypes |  |
| 2 | Значение на русском языке | Ru | string | string | — | EquipmentTypes |  |
| 3 | Значение на казахском языке | Kz | string | string | — | EquipmentTypes |  |

## 10. Пример ответа

```json
{
  "value": [
    {
      "id": "7b4f4b4d-52d4-4a77-b6b7-f2b7d7c81111",
      "code": "EXCAVATOR",
      "name": {
        "En": "Excavator",
        "Ru": "Экскаватор",
        "Kz": "Экскаватор"
      },
      "workCenterId": "12a8b1ce-3aaf-4f55-8ac8-f8cf5d86c222"
    }
  ],
  "isSuccess": true,
  "errors": []
}
```
