**Created:** 2026-05-12  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

# GET /business-partners/:id

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить карточку бизнес-партнёра |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Admin Panel` |
| Endpoint URL | `/api/admin/v1/business-partners/:id` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Возвращает полную карточку бизнес-партнёра для формы просмотра и редактирования.

---

## 2. Функциональные требования

Общий перечень требований: Требования к системе DMMS

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-014 | [`Admin`](../../../requirements/Roles and Access Model.md) редактирует параметры техники и связанные справочные сущности | Confirmed | BRD v13 | Метод нужен для загрузки карточки бизнес-партнёра |

---

## 3. Описание логики работы метода

1. Найти запись в `BusinessPartners` WHERE `id` = `:id` AND `isDeleted = false`. Если запись не найдена — вернуть `404 NOT_FOUND`.
2. Рассчитать `externalUsersCount` = COUNT(`Users` WHERE `businessPartnerId` = `:id` AND `type` = `external` AND `isDeleted = false`).
3. Вернуть объект `BusinessPartnerDetail` в формате общего `result wrapper`.

Сущности, участвующие в методе:
- читаются: `BusinessPartners`, `Users`
- изменения не выполняются
- транзакционность не требуется, метод read-only

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| [`Admin`](../../../requirements/Roles and Access Model.md) | Доступ к административной панели и справочнику бизнес-партнёров |

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
| `NOT_FOUND` | Бизнес-партнёр не найден |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор бизнес-партнёра | `id` | `uuid` | `+` | Валидный UUID v4 | — | Path param | |

---

## 8. Пример запроса

```http
GET /api/admin/v1/business-partners/bp000001-0000-4000-8000-000000000001
Authorization: Bearer <token>
Content-Type: application/json
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий [`result wrapper`](../../common/Result%20Wrapper.md).

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | value | object | object | — | backend aggregation |  |
| 1.1 | Идентификатор записи | id | string | string | — | BusinessPartners |  |
| 1.2 | Наименование | name | object | object | — | BusinessPartners |  |
| 1.3 | Описание | description | string | string | — | BusinessPartners |  |
| 1.4 | БИН контрагента | bin | string | string | — | BusinessPartners |  |
| 1.5 | Страна | country | string | string | — | BusinessPartners |  |
| 1.6 | Город | city | string | string | — | BusinessPartners |  |
| 1.7 | Адрес | address | string | string | — | BusinessPartners |  |
| 1.8 | Email | email | string | string | — | BusinessPartners |  |
| 1.9 | Номер телефона | phoneNumber | string | string | — | BusinessPartners |  |
| 1.10 | Внешний идентификатор | externalId | string | string | — | BusinessPartners |  |
| 1.11 | Признак активности | isActive | bool | boolean | — | BusinessPartners |  |
| 1.12 | Количество внешних пользователей | externalUsersCount | int | integer | — | backend composition from BusinessPartners + Users |  |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | object[] | `[]` | backend |  |

### Структура `value`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | string | string | — | BusinessPartners |  |
| 2 | Наименование | name | object | object | — | BusinessPartners |  |
| 3 | Описание | description | string | string | — | BusinessPartners |  |
| 4 | БИН контрагента | bin | string | string | — | BusinessPartners |  |
| 5 | Страна | country | string | string | — | BusinessPartners |  |
| 6 | Город | city | string | string | — | BusinessPartners |  |
| 7 | Адрес | address | string | string | — | BusinessPartners |  |
| 8 | Email | email | string | string | — | BusinessPartners |  |
| 9 | Номер телефона | phoneNumber | string | string | — | BusinessPartners |  |
| 10 | Внешний идентификатор | externalId | string | string | — | BusinessPartners |  |
| 11 | Признак активности | isActive | bool | boolean | — | BusinessPartners |  |
| 12 | Количество внешних пользователей | externalUsersCount | int | integer | — | backend composition from BusinessPartners + Users |  |

### Структура `value.name`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string | string | — | BusinessPartners |  |
| 2 | Значение на русском языке | Ru | string | string | — | BusinessPartners |  |
| 3 | Значение на казахском языке | Kz | string | string | — | BusinessPartners |  |

## 10. Пример ответа

```json
{
  "value": {
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
  },
  "isSuccess": true,
  "errors": []
}
```

---

## Замечания

1. Поле `name` возвращается как локализованный объект `{ En, Ru, Kz }`.
2. `externalUsersCount` нужен для guard-логики удаления и отображения связности с external users.
