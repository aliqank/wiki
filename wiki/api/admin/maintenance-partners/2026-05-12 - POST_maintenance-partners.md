**Created:** 2026-05-12  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

# POST /maintenance-partners

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Создать нового партнера ТО |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Admin Panel` |
| Endpoint URL | `/api/admin/v1/maintenance-partners` |
| Метод запроса | `POST` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется для добавления записи в справочник партнеров технического обслуживания.

---

## 2. Функциональные требования

Общий перечень требований: Требования к системе DMMS

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-014 | [`Admin`](../../../requirements/Roles and Access Model.md) редактирует параметры техники и связанные справочные сущности | Confirmed | BRD v13 | Нужен для пополнения справочника |
| TCO Booking Tool | FR-NEW-50 | [`Admin`](../../../requirements/Roles and Access Model.md) manages all reference/handbook values via [`Admin`](../../../requirements/Roles and Access Model.md) Panel | Confirmed | BRD v13 | Справочник управляется через [`Admin`](../../../requirements/Roles and Access Model.md) Panel |

---

## 3. Описание логики работы метода

1. Принять тело запроса; провалидировать обязательное поле `name`.
2. Проверить поле `name`: должен быть передан объект `{ En, Ru, Kz }`; `name.En`, `name.Ru`, `name.Kz` обязательны.
3. Проверить уникальность `name` среди `MaintenancePartners` WHERE `isDeleted = false`. Проверка выполняется по правилу локализованной уникальности для набора `name.En`, `name.Ru`, `name.Kz`.
4. Создать запись в `MaintenancePartners`; заполнить аудит-поля `createdAt`, `createdBy`.
5. Вернуть созданный объект с `contractsCount = 0`.

Сущности, участвующие в методе:
- читаются: `MaintenancePartners`
- изменяются: `MaintenancePartners` (INSERT)
- транзакционность: не требуется

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| [`Admin`](../../../requirements/Roles and Access Model.md) | Доступ к административной панели и справочнику партнеров ТО |

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
| `VALIDATION_ERROR` | Поле `name` не передано, одно из полей `name.En`, `name.Ru`, `name.Kz` пустое или локализованное имя уже существует |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Название партнера | `name` | `object` | `+` | Объект `{ En, Ru, Kz }`; `name.En`, `name.Ru`, `name.Kz` обязательны; локализованное имя уникально среди активных записей | — | Request body | |
| 2 | Контактный телефон | `phoneNumber` | `string` | `-` | Свободный текст | `null` | Request body | |
| 3 | Контактный email | `email` | `string` | `-` | Валидный email, если передан | `null` | Request body | |
| 4 | Адрес | `address` | `string` | `-` | Свободный текст | `null` | Request body | |

---

## 8. Пример запроса

```http
POST /api/admin/v1/maintenance-partners
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "name": {
    "En": "Shell Service",
    "Ru": "Shell Service",
    "Kz": "Shell Service"
  },
  "phoneNumber": "+7 701 000 0000",
  "email": "shell@example.com",
  "address": "Atyrau, Industrial Zone 1"
}
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий [`result wrapper`](../../common/Result%20Wrapper.md).

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | value | object | object | — | backend aggregation |  |
| 1.1 | Идентификатор записи | id | string | string | — | MaintenancePartners |  |
| 1.2 | Наименование | name | object | object | — | MaintenancePartners |  |
| 1.3 | Номер телефона | phoneNumber | string | string | — | MaintenancePartners |  |
| 1.4 | Email | email | string | string | — | MaintenancePartners |  |
| 1.5 | Адрес | address | string | string | — | MaintenancePartners |  |
| 1.6 | Количество связанных контрактов | contractsCount | int | integer | — | COUNT(EquipmentMaintenanceContracts) |  |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | object[] | `[]` | backend |  |

### Структура `value`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | string | string | — | MaintenancePartners |  |
| 2 | Наименование | name | object | object | — | MaintenancePartners |  |
| 3 | Номер телефона | phoneNumber | string | string | — | MaintenancePartners |  |
| 4 | Email | email | string | string | — | MaintenancePartners |  |
| 5 | Адрес | address | string | string | — | MaintenancePartners |  |
| 6 | Количество связанных контрактов | contractsCount | int | integer | — | COUNT(EquipmentMaintenanceContracts) |  |

### Структура `value.name`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string | string | — | MaintenancePartners |  |
| 2 | Значение на русском языке | Ru | string | string | — | MaintenancePartners |  |
| 3 | Значение на казахском языке | Kz | string | string | — | MaintenancePartners |  |

## 10. Пример ответа

```json
{
  "value": {
    "id": "m0000001-0000-4000-8000-000000000001",
    "name": {
      "En": "Shell Service",
      "Ru": "Shell Service",
      "Kz": "Shell Service"
    },
    "phoneNumber": "+7 701 000 0000",
    "email": "shell@example.com",
    "address": "Atyrau, Industrial Zone 1",
    "contractsCount": 0
  },
  "isSuccess": true,
  "errors": []
}
```

---

## Замечания

1. После создания партнер ТО сразу доступен для выбора в `EquipmentMaintenanceContracts`.
2. Вместо поля `contactInfo` метод принимает и возвращает отдельные поля `phoneNumber`, `email`, `address`.
