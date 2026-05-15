**Created:** 2026-05-12  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

# GET /business-partners

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить постраничный список бизнес-партнёров для справочника Admin Panel |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Admin Panel` |
| Endpoint URL | `/api/admin/v1/business-partners` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется для экрана управления справочником бизнес-партнёров в Admin Panel. Возвращает список записей с данными, достаточными для отображения и guard-логики удаления.

---

## 2. Функциональные требования

Общий перечень требований: Требования к системе DMMS

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-014 | Admin редактирует параметры техники и связанные справочные сущности | Confirmed | BRD v13 | Метод нужен для управления справочником бизнес-партнёров |
| TCO Booking Tool | FR-NEW-50 | Admin manages all reference/handbook values via Admin Panel | Confirmed | BRD v13 | Справочник управляется через Admin Panel |

---

## 3. Описание логики работы метода

1. Получить записи из `BusinessPartners` WHERE `isDeleted = false`.
2. Если передан `search`, применить фильтр по `BusinessPartners.name.En`, `BusinessPartners.name.Ru`, `BusinessPartners.name.Kz`, `bin`, `externalId`, `email`, `phoneNumber`, `country`, `city`.
3. Для каждой записи рассчитать `externalUsersCount` = COUNT(`Users` WHERE `businessPartnerId` = `BusinessPartners.id` AND `type` = `external` AND `isDeleted = false`).
4. Отсортировать результат по `name.Ru ASC`, затем по `bin ASC`.
5. Вернуть страницу данных в формате общего `result wrapper` с `PaginatedResult` внутри `value`.

Сущности, участвующие в методе:
- читаются: `BusinessPartners`, `Users`
- изменения не выполняются
- транзакционность не требуется, метод read-only

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Admin` | Доступ к административной панели и справочнику бизнес-партнёров |

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

HTTP-коды: `401 Unauthorized`, `403 Forbidden`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Поисковая строка | `search` | `string` | `-` | Если передан, используется как фильтр по `name.En`, `name.Ru`, `name.Kz`, `bin`, `externalId`, `email`, `phoneNumber`, `country`, `city` | — | Query param | |
| 2 | Номер страницы | `page` | `int` | `-` | Целое число >= 1 | `1` | Query param | |
| 3 | Размер страницы | `limit` | `int` | `-` | Целое число >= 1 | `20` | Query param | |

---

## 8. Пример запроса

```http
GET /api/admin/v1/business-partners?search=Chevron&page=1&limit=20
Authorization: Bearer <token>
Content-Type: application/json
```

У метода нет request body.

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий `result wrapper`.

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | value | object | PaginatedResult | — | backend aggregation |  |
| 1.1 | Элементы текущей страницы | items | array<object> | object[] | — | response DTO | Коллекция объектов |
| 1.2 | Общее количество записей | total | int | integer | — | backend |  |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | array | `[]` | backend |  |

### Структура `value`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Элементы текущей страницы | items | array<object> | object[] | — | response DTO | Коллекция объектов |
| 2 | Общее количество записей | total | int | integer | — | backend |  |

### Структура `value.items[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | string | string | — | response DTO |  |
| 2 | Наименование | name | object | object | — | response DTO |  |
| 3 | Описание | description | string | string | — | response DTO |  |
| 4 | БИН контрагента | bin | string | string | — | response DTO |  |
| 5 | Страна | country | string | string | — | response DTO |  |
| 6 | Город | city | string | string | — | response DTO |  |
| 7 | Адрес | address | string | string | — | response DTO |  |
| 8 | Email | email | string | string | — | response DTO |  |
| 9 | Номер телефона | phoneNumber | string | string | — | response DTO |  |
| 10 | Внешний идентификатор | externalId | string | string | — | response DTO |  |
| 11 | Признак активности | isActive | bool | boolean | — | response DTO |  |
| 12 | Количество внешних пользователей | externalUsersCount | int | integer | — | response DTO |  |

### Структура `value.items[].name`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string | string | — | response DTO |  |
| 2 | Значение на русском языке | Ru | string | string | — | response DTO |  |
| 3 | Значение на казахском языке | Kz | string | string | — | response DTO |  |

## 10. Пример ответа

```json
{
  "value": {
    "items": [
      {
        "id": "bp000001-0000-4000-8000-000000000001",
        "name": {
          "En": "Chevron Business Partner",
          "Ru": "Бизнес-партнёр Chevron",
          "Kz": "Chevron бизнес-серіктесі"
        },
        "description": "Primary external contractor",
        "bin": "123456789012",
        "country": "Kazakhstan",
        "city": "Atyrau",
        "address": "Industrial zone 1",
        "email": "contact@chevron.example",
        "phoneNumber": "+7 701 123 4567",
        "externalId": "BP-EXT-001",
        "isActive": true,
        "externalUsersCount": 4
      }
    ],
    "total": 12
  },
  "isSuccess": true,
  "errors": []
}
```

---

## Замечания

1. Поле `name` возвращается как локализованный объект `{ En, Ru, Kz }`.
2. `externalUsersCount` используется во frontend для guard-логики удаления бизнес-партнёра.
