# GET /reference/ownership-types

**Created:** 2026-05-18  
**Last updated:** 2026-06-02  
**Автор документов:** OpenCode

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить справочник ownership types для фильтра формы создания заявки |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Requestor UI` |
| Endpoint URL | `/api/booking/v1/reference/ownership-types` |
| Метод запроса | `GET` |
| Связанные use cases | [`UC-REQ-02 - Создание новой заявки (Draft-first)`](../../../requirements/usecases/Requestor/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first)/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first).md), [`UC-REQ-02.3 - Поиск техники для добавления в заявку`](../../../requirements/usecases/Requestor/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first)/UC-REQ-02.3%20-%20Поиск%20техники%20для%20добавления%20в%20заявку.md) |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый reference API для UC-2. Используется для загрузки значений фильтра `ownershipType`.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-031 | Requestor can add/remove equipment items to a request | Confirmed | BRD v13 | Тип владения участвует в фильтрации техники |

---

## 3. Описание логики работы метода

1. Вернуть набор reference values для `ownershipType`, используемых в booking workflow.
2. Исключить значения, которые не участвуют в поиске Requestor, например `OnDemand`.

Сущности, участвующие в методе:
- читаются: [`ref_ownership_type`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#reference-tables-instead-of-enums)
- изменения не выполняются

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Requestor` | Создание и редактирование собственных заявок |
| `ServiceWorkProcessor` | Работа с Service Work Requests |

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
GET /api/booking/v1/reference/ownership-types
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
| 1 | Код значения | code | string | string | — | ref_ownership_type.code |  |
| 2 | Наименование значения | name | object | object | — | ref_ownership_type | Локализованное значение |

### Структура `value[].name`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string | string | — | ref_ownership_type |  |
| 2 | Значение на русском языке | Ru | string | string | — | ref_ownership_type |  |
| 3 | Значение на казахском языке | Kz | string | string | — | ref_ownership_type |  |

## 10. Пример ответа

```json
{
  "value": [
    {
      "code": "TcoOwned",
      "name": {
        "En": "TCO Owned",
        "Ru": "Собственная техника TCO",
        "Kz": "TCO меншікті техникасы"
      }
    },
    {
      "code": "LongTermRented",
      "name": {
        "En": "Long-term rented",
        "Ru": "Долгосрочно арендованная",
        "Kz": "Ұзақ мерзімге жалға алынған"
      }
    }
  ],
  "isSuccess": true,
  "errors": []
}
```
