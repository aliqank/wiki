# GET /reference/fleet-owners

**Created:** 2026-05-18  
**Last updated:** 2026-06-02  
**Автор документов:** OpenCode

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить справочник fleet owners для фильтра формы создания заявки |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Requestor UI` |
| Endpoint URL | `/api/booking/v1/reference/fleet-owners` |
| Метод запроса | `GET` |
| Связанные use cases | [`UC-REQ-02 - Создание новой заявки (Draft-first)`](../../../requirements/usecases/Requestor/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first)/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first).md), [`UC-REQ-02.3 - Поиск техники для добавления в заявку`](../../../requirements/usecases/Requestor/UC-REQ-02%20-%20Создание%20новой%20заявки%20(Draft-first)/UC-REQ-02.3%20-%20Поиск%20техники%20для%20добавления%20в%20заявку.md) |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый reference API для UC-2. Используется для загрузки списка доступных fleet owners перед поиском техники.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-031 | Requestor can add/remove equipment items to a request | Confirmed | BRD v13 | Fleet owner участвует в фильтрации списка техники |

---

## 3. Описание логики работы метода

1. Выбрать активные fleet-ы, доступные для booking workflow.
2. Вернуть пользователя, ответственного за fleet, и краткие данные fleet-а.
3. Исключить удаленные и недоступные записи.

Сущности, участвующие в методе:
- читаются: [`Fleets`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#2-fleets), [`Users`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#19-users)
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
GET /api/booking/v1/reference/fleet-owners
Authorization: Bearer <token>
Content-Type: application/json
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий `result wrapper`.

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | value | array<object> | object[] | — | backend aggregation |  |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | object[] | `[]` | backend |  |

### Структура `value[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор fleet-а | fleetId | uuid | UUID v4 | — | Fleets.id |  |
| 2 | Идентификатор пользователя | userId | uuid | UUID v4 | — | Users.id | Значение используется в `GET /equipment/search` как `fleetOwnerUserId` |
| 3 | Наименование fleet-а | fleetName | string | string | — | Fleets.name |  |
| 4 | Полное имя пользователя | fullName | string | string | — | Users.fullName |  |
| 5 | Email пользователя | email | string | string | — | Users.email |  |

## 10. Пример ответа

```json
{
  "value": [
    {
      "fleetId": "f3caa8da-11f2-4108-997f-2204041a1001",
      "userId": "4c9ad2d2-6df8-4f7b-87fe-36cefc100001",
      "fleetName": "Maintenance Fleet",
      "fullName": "Nurlan Sarsenov",
      "email": "nurlan.sarsenov@tco.example"
    }
  ],
  "isSuccess": true,
  "errors": []
}
```
