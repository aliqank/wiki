**Created:** 2026-05-12  
**Last updated:** 2026-05-12  
**Author:** Telman Nurzhanov (SA)

---

# PUT /business-partners/:id

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Изменить бизнес-партнёра |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Admin Panel` |
| Endpoint URL | `/api/admin/v1/business-partners/:id` |
| Метод запроса | `PUT` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется для редактирования записи справочника бизнес-партнёров.

---

## 2. Функциональные требования

Общий перечень требований: Требования к системе DMMS

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-014 | Admin редактирует параметры техники и связанные справочные сущности | Confirmed | BRD v13 | Нужен для редактирования справочника |
| TCO Booking Tool | FR-NEW-50 | Admin manages all reference/handbook values via Admin Panel | Confirmed | BRD v13 | Справочник управляется через Admin Panel |

---

## 3. Описание логики работы метода

1. Найти запись в `BusinessPartners` WHERE `id` = `:id` AND `isDeleted = false`. Если запись не найдена — вернуть `404 NOT_FOUND`.
2. Провалидировать `name`: должен быть передан объект `{ En, Ru, Kz }`; `name.Ru` обязателен; локализованное имя уникально среди `BusinessPartners` WHERE `id` != `:id` AND `isDeleted = false`.
3. Проверить уникальность `bin` среди `BusinessPartners` WHERE `id` != `:id` AND `isDeleted = false`.
4. Если передан `externalId`, проверить его уникальность среди `BusinessPartners` WHERE `id` != `:id` AND `isDeleted = false`.
5. Обновить запись; заполнить `updatedAt`, `updatedBy`.
6. Рассчитать `externalUsersCount` = COUNT(`Users` WHERE `businessPartnerId` = `:id` AND `type` = `external` AND `isDeleted = false`).
7. Вернуть обновлённый объект `BusinessPartnerDetail`.

Сущности, участвующие в методе:
- читаются: `BusinessPartners`, `Users`
- изменяются: `BusinessPartners` (UPDATE)
- транзакционность: не требуется

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
| `NOT_FOUND` | Бизнес-партнёр не найден |
| `VALIDATION_ERROR` | Поле `name` не передано, `name.Ru` пустое или локализованное имя уже существует |
| `VALIDATION_ERROR` | Поле `bin` пустое или уже существует |
| `VALIDATION_ERROR` | Поле `externalId` уже существует |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор бизнес-партнёра | `id` | `uuid` | `+` | Валидный UUID v4 | — | Path param | |
| 2 | Наименование | `name` | `object` | `+` | Объект `{ En, Ru, Kz }`; `name.Ru` обязателен; локализованное имя уникально среди активных записей кроме текущей | — | Request body | |
| 3 | Описание | `description` | `string` | `-` | Свободный текст | `null` | Request body | |
| 4 | БИН | `bin` | `string` | `+` | Непустая строка; уникальна среди активных записей кроме текущей | — | Request body | |
| 5 | Страна | `country` | `string` | `-` | Свободный текст | `null` | Request body | |
| 6 | Город | `city` | `string` | `-` | Свободный текст | `null` | Request body | |
| 7 | Адрес | `address` | `string` | `-` | Свободный текст | `null` | Request body | |
| 8 | Email | `email` | `string` | `-` | Валидный email, если передан | `null` | Request body | |
| 9 | Телефон | `phoneNumber` | `string` | `-` | Свободный текст | `null` | Request body | |
| 10 | Внешний ID | `externalId` | `string` | `-` | Уникален среди активных записей, если передан | `null` | Request body | |
| 11 | Активен ли партнёр | `isActive` | `bool` | `+` | Булево значение | `true` | Request body | |

---

## 8. Пример запроса

```http
PUT /api/admin/v1/business-partners/bp000001-0000-4000-8000-000000000001
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "name": {
    "En": "Chevron Partner KZ",
    "Ru": "Партнёр Chevron KZ",
    "Kz": "Chevron KZ серіктесі"
  },
  "description": "Updated contractor profile",
  "bin": "123456789012",
  "country": "Kazakhstan",
  "city": "Atyrau",
  "address": "Industrial zone 2",
  "email": "updated@chevron.example",
  "phoneNumber": "+7 701 765 4321",
  "externalId": "BP-EXT-001",
  "isActive": true
}
```

---

## 9. Возвращаемые данные

В `value` возвращается обновлённый объект `BusinessPartnerDetail`.

---

## 10. Пример ответа

```json
{
  "value": {
    "id": "bp000001-0000-4000-8000-000000000001",
    "name": {
      "En": "Chevron Partner KZ",
      "Ru": "Партнёр Chevron KZ",
      "Kz": "Chevron KZ серіктесі"
    },
    "description": "Updated contractor profile",
    "bin": "123456789012",
    "country": "Kazakhstan",
    "city": "Atyrau",
    "address": "Industrial zone 2",
    "email": "updated@chevron.example",
    "phoneNumber": "+7 701 765 4321",
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

1. Изменение данных бизнес-партнёра не требует обновления `Users`, так как связи идут по `businessPartnerId`.
