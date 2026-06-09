**Created:** 2026-05-08  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

# PUT /equipment-types/:id

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Сохранить базовые параметры типа техники (блок A) со страницы AP-02 в Admin Panel |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Admin Panel` |
| Endpoint URL | `/api/admin/v1/equipment-types/:id` |
| Метод запроса | `PUT` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Обрабатывает нажатие кнопки «Сохранить» в блоке A карточки AP-02. Обновляет только базовые параметры типа техники (`name`, `mobilityType`, `requiresTransport`, `sortOrder`). Характеристики (блок B) управляются отдельными эндпоинтами.

---

## 2. Функциональные требования

Общий перечень требований: Требования к системе DMMS

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-010 | Admin configures dynamic custom characteristics per equipment type via Admin Panel tool; characteristics appear automatically in equipment card, request form, and search filters based on equipment type | Confirmed | BRD v13 | Корректные базовые параметры типа (в т.ч. mobilityType) влияют на поведение характеристик и фильтров |
| TCO Booking Tool | FR-NEW-50 | Admin manages all reference/handbook values via Admin Panel | Confirmed | BRD v13 | Типы техники — справочная сущность; их редактирование входит в скоуп управления справочниками |
| TCO Booking Tool | FR-NEW-68 | System dynamically shows only type-specific characteristics in request creation form and equipment search filters, based on the selected equipment type | Confirmed | BRD v13 | Изменение параметров типа влияет на отображение характеристик и фильтров в форме заявки |

---

## 3. Описание логики работы метода

1. Найти запись в `EquipmentTypes` WHERE `id` = `:id` AND `isDeleted = false`. Если запись не найдена — вернуть `404 NOT_FOUND`.
2. Провалидировать поле `name`: должен быть передан объект `{ En, Ru, Kz }`; `name.En`, `name.Ru`, `name.Kz` обязательны; локализованное имя уникально в `EquipmentTypes` WHERE `id` ≠ `:id` AND `isDeleted = false`. Если нарушено — вернуть `422 VALIDATION_ERROR`.
3. Провалидировать поле `mobilityType`: должно входить в допустимые значения ENUM (`SelfPropelled / NonSelfPropelledMotorized / Stationary / NonMotorized`). Если нарушено — вернуть `422 VALIDATION_ERROR`.
4. Обновить запись в `EquipmentTypes`; заполнить аудит-поля `updatedAt` (текущее время) и `updatedBy` (ID аутентифицированного пользователя).
5. Рассчитать `equipmentsCount` = COUNT(`Equipments` WHERE `equipmentTypeId` = `:id` AND `isDeleted = false`).
6. Вернуть обновлённый объект `EquipmentType` (без поля `properties`) в формате общего `result wrapper` с HTTP 200.

Сущности, участвующие в методе:
- читаются: `EquipmentTypes` (для проверки уникальности `name`), `Equipments` (агрегат для `equipmentsCount`)
- изменяются: `EquipmentTypes` (UPDATE)
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
| `NOT_FOUND` | Тип техники с указанным `id` не найден или помечен как удалённый |
| `VALIDATION_ERROR` | Поле `name` не передано, одно из полей `name.En`, `name.Ru`, `name.Kz` пустое или локализованное имя уже существует в справочнике |
| `VALIDATION_ERROR` | Поле `mobilityType` содержит недопустимое значение |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор типа техники | `id` | `uuid` | `+` | Должен быть валидным UUID v4 | — | Path param | |
| 2 | Наименование типа техники | `name` | `object` | `+` | Объект `{ En, Ru, Kz }`; `name.En`, `name.Ru`, `name.Kz` обязательны; локализованное имя уникально среди `EquipmentTypes` WHERE `id` ≠ `:id` AND `isDeleted = false` | — | Request body | |
| 3 | Тип мобильности | `mobilityType` | `enum` | `+` | Одно из: `SelfPropelled`, `NonSelfPropelledMotorized`, `Stationary`, `NonMotorized` | — | Request body | |
| 4 | Требуется ли транспортировка | `requiresTransport` | `bool` | `+` | Булево значение | — | Request body | |
| 5 | Порядок отображения | `sortOrder` | `int` | `+` | Целое число >= 1 | — | Request body | В отличие от POST, при PUT `sortOrder` обязателен |

---

## 8. Пример запроса

```http
PUT /api/admin/v1/equipment-types/7b4f4b4d-52d4-4a77-b6b7-f2b7d7c81111
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "name": {
    "En": "Air compressor",
    "Ru": "Компрессор воздушный",
    "Kz": "Ауа компрессоры"
  },
  "mobilityType": "Stationary",
  "requiresTransport": false,
  "sortOrder": 1
}
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий [`result wrapper`](../../common/Result%20Wrapper.md).

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | value | object | object | — | backend aggregation |  |
| 1.1 | Идентификатор записи | id | uuid | UUID v4 | — | EquipmentTypes.id |  |
| 1.2 | Наименование | name | object | object | — | EquipmentTypes |  |
| 1.3 | Тип мобильности техники | mobilityType | string | string | — | EquipmentTypes + ref_equipment_mobility_type |  |
| 1.4 | Признак необходимости транспортировки | requiresTransport | bool | boolean | — | EquipmentTypes.requiresTransport |  |
| 1.5 | Порядок сортировки | sortOrder | int | integer | — | EquipmentTypes.sortOrder |  |
| 1.6 | Количество единиц техники | equipmentsCount | int | integer | — | backend composition from EquipmentTypes + Equipments |  |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | object[] | `[]` | backend |  |

### Структура `value`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | EquipmentTypes.id |  |
| 2 | Наименование | name | object | object | — | EquipmentTypes |  |
| 3 | Тип мобильности техники | mobilityType | string | string | — | EquipmentTypes + ref_equipment_mobility_type |  |
| 4 | Признак необходимости транспортировки | requiresTransport | bool | boolean | — | EquipmentTypes.requiresTransport |  |
| 5 | Порядок сортировки | sortOrder | int | integer | — | EquipmentTypes.sortOrder |  |
| 6 | Количество единиц техники | equipmentsCount | int | integer | — | backend composition from EquipmentTypes + Equipments |  |

### Структура `value.name`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Значение на английском языке | En | string | string | — | EquipmentTypes |  |
| 2 | Значение на русском языке | Ru | string | string | — | EquipmentTypes |  |
| 3 | Значение на казахском языке | Kz | string | string | — | EquipmentTypes |  |

## 10. Пример ответа

```json
{
  "value": {
    "id": "7b4f4b4d-52d4-4a77-b6b7-f2b7d7c81111",
    "name": {
      "En": "Air compressor",
      "Ru": "Компрессор воздушный",
      "Kz": "Ауа компрессоры"
    },
    "mobilityType": "Stationary",
    "requiresTransport": false,
    "sortOrder": 1,
    "equipmentsCount": 12
  },
  "isSuccess": true,
  "errors": []
}
```

---

## Замечания

1. Метод обновляет только базовые параметры типа (блок A). Характеристики типа (блок B) управляются через отдельные эндпоинты: `POST /equipment-types/:id/properties`, `PATCH /equipment-types/:id/properties/:etpId`, `DELETE /equipment-types/:id/properties/:etpId`.
2. Уникальность `name` при обновлении проверяется с исключением самой редактируемой записи (`id ≠ :id`), чтобы допустить сохранение без изменения локализованного имени.
3. В ответе возвращается `equipmentsCount` (без перечня `properties`) — достаточно для обновления guard-состояния кнопки удаления на frontend без повторного вызова `GET /:id`.
