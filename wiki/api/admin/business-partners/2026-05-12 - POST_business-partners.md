**Created:** 2026-05-12  
**Last updated:** 2026-05-12  
**Автор документов:** Telman Nurzhanov (SA)

---

# POST /business-partners

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Создать нового бизнес-партнёра |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Admin Panel` |
| Endpoint URL | `/api/admin/v1/business-partners` |
| Метод запроса | `POST` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется для добавления записи в справочник бизнес-партнёров.

---

## 2. Функциональные требования

Общий перечень требований: Требования к системе DMMS

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-014 | Admin редактирует параметры техники и связанные справочные сущности | Confirmed | BRD v13 | Нужен для пополнения справочника |
| TCO Booking Tool | FR-NEW-50 | Admin manages all reference/handbook values via Admin Panel | Confirmed | BRD v13 | Справочник управляется через Admin Panel |

---

## 3. Описание логики работы метода

1. Принять тело запроса; провалидировать обязательные поля `name`, `bin`, `isActive`.
2. Проверить поле `name`: должен быть передан объект `{ En, Ru, Kz }`; `name.Ru` обязателен.
3. Проверить уникальность `name` среди `BusinessPartners` WHERE `isDeleted = false` по правилу локализованной уникальности для набора `name.En`, `name.Ru`, `name.Kz`.
4. Проверить уникальность `bin` среди `BusinessPartners` WHERE `isDeleted = false`.
5. Если передан `externalId`, проверить его уникальность среди `BusinessPartners` WHERE `isDeleted = false`.
6. Создать запись в `BusinessPartners`; заполнить аудит-поля `createdAt`, `createdBy`.
7. Вернуть созданный объект `BusinessPartnerDetail` с `externalUsersCount = 0`.

Сущности, участвующие в методе:
- читаются: `BusinessPartners`
- изменяются: `BusinessPartners` (INSERT)
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
| `VALIDATION_ERROR` | Поле `name` не передано, `name.Ru` пустое или локализованное имя уже существует |
| `VALIDATION_ERROR` | Поле `bin` пустое или уже существует |
| `VALIDATION_ERROR` | Поле `externalId` уже существует |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Наименование | `name` | `object` | `+` | Объект `{ En, Ru, Kz }`; `name.Ru` обязателен; локализованное имя уникально среди активных записей | — | Request body | |
| 2 | Описание | `description` | `string` | `-` | Свободный текст | `null` | Request body | |
| 3 | БИН | `bin` | `string` | `+` | Непустая строка; уникальна среди активных записей | — | Request body | |
| 4 | Страна | `country` | `string` | `-` | Свободный текст | `null` | Request body | |
| 5 | Город | `city` | `string` | `-` | Свободный текст | `null` | Request body | |
| 6 | Адрес | `address` | `string` | `-` | Свободный текст | `null` | Request body | |
| 7 | Email | `email` | `string` | `-` | Валидный email, если передан | `null` | Request body | |
| 8 | Телефон | `phoneNumber` | `string` | `-` | Свободный текст | `null` | Request body | |
| 9 | Внешний ID | `externalId` | `string` | `-` | Уникален среди активных записей, если передан | `null` | Request body | |
| 10 | Активен ли партнёр | `isActive` | `bool` | `+` | Булево значение | `true` | Request body | |

---

## 8. Пример запроса

```http
POST /api/admin/v1/business-partners
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
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
  "isActive": true
}
```

---

## 9. Возвращаемые данные

В `value` возвращается созданный объект `BusinessPartnerDetail`.

---

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
    "externalUsersCount": 0
  },
  "isSuccess": true,
  "errors": []
}
```

---

## Замечания

1. После создания бизнес-партнёр сразу доступен для выбора/связки с external users.
