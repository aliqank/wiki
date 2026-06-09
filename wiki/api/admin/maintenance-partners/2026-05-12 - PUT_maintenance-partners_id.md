**Created:** 2026-05-12  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

# PUT /maintenance-partners/:id

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Изменить партнера ТО |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Admin Panel` |
| Endpoint URL | `/api/admin/v1/maintenance-partners/:id` |
| Метод запроса | `PUT` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется для редактирования записи справочника партнеров ТО.

---

## 2. Функциональные требования

Общий перечень требований: Требования к системе DMMS

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-014 | [`Admin`](../../../requirements/Roles and Access Model.md) редактирует параметры техники и связанные справочные сущности | Confirmed | BRD v13 | Нужен для редактирования справочника |
| TCO Booking Tool | FR-NEW-50 | [`Admin`](../../../requirements/Roles and Access Model.md) manages all reference/handbook values via [`Admin`](../../../requirements/Roles and Access Model.md) Panel | Confirmed | BRD v13 | Справочник управляется через [`Admin`](../../../requirements/Roles and Access Model.md) Panel |

---

## 3. Описание логики работы метода

1. Найти запись в `MaintenancePartners` WHERE `id` = `:id` AND `isDeleted = false`. Если запись не найдена — вернуть `404 NOT_FOUND`.
2. Провалидировать `name`: должен быть передан объект `{ En, Ru, Kz }`; `name.En`, `name.Ru`, `name.Kz` обязательны; локализованное имя уникально среди `MaintenancePartners` WHERE `id` != `:id` AND `isDeleted = false`.
3. Обновить запись; заполнить `updatedAt`, `updatedBy`.
4. Рассчитать `contractsCount` = COUNT(`EquipmentMaintenanceContracts` WHERE `partnerId` = `:id` AND `isDeleted = false`).
5. Вернуть обновлённый объект `MaintenancePartner`.

Сущности, участвующие в методе:
- читаются: `MaintenancePartners`, `EquipmentMaintenanceContracts`
- изменяются: `MaintenancePartners` (UPDATE)
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
| `NOT_FOUND` | Партнер ТО не найден |
| `VALIDATION_ERROR` | Поле `name` не передано, одно из полей `name.En`, `name.Ru`, `name.Kz` пустое или локализованное имя уже существует |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор партнера ТО | `id` | `uuid` | `+` | Валидный UUID v4 | — | Path param | |
| 2 | Название партнера | `name` | `object` | `+` | Объект `{ En, Ru, Kz }`; `name.En`, `name.Ru`, `name.Kz` обязательны; локализованное имя уникально среди активных записей кроме текущей | — | Request body | |
| 3 | Контактный телефон | `phoneNumber` | `string` | `-` | Свободный текст | `null` | Request body | |
| 4 | Контактный email | `email` | `string` | `-` | Валидный email, если передан | `null` | Request body | |
| 5 | Адрес | `address` | `string` | `-` | Свободный текст | `null` | Request body | |

---

## 8. Пример запроса

```http
PUT /api/admin/v1/maintenance-partners/m0000001-0000-4000-8000-000000000001
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "name": {
    "En": "Shell Service KZ",
    "Ru": "Shell Service KZ",
    "Kz": "Shell Service KZ"
  },
  "phoneNumber": "+7 701 111 1111",
  "email": "service@shell.kz",
  "address": "Atyrau, North industrial area"
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
      "En": "Shell Service KZ",
      "Ru": "Shell Service KZ",
      "Kz": "Shell Service KZ"
    },
    "phoneNumber": "+7 701 111 1111",
    "email": "service@shell.kz",
    "address": "Atyrau, North industrial area",
    "contractsCount": 5
  },
  "isSuccess": true,
  "errors": []
}
```

---

## Замечания

1. Изменение данных партнера не требует обновления `EquipmentMaintenanceContracts`, так как связи идут по `partnerId`.
2. Вместо поля `contactInfo` метод принимает и возвращает отдельные поля `phoneNumber`, `email`, `address`.
