**Created:** 2026-05-08  
**Last updated:** 2026-05-08  
**Author:** Telman Nurzhanov (SA)

---

# POST /equipment-types

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Создать новый тип техники в справочнике Admin Panel |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Admin Panel` |
| Endpoint URL | `/api/admin/v1/equipment-types` |
| Метод запроса | `POST` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Обеспечивает создание типа техники через форму AP-02 в Admin Panel. Тип техники является корневой справочной сущностью: к нему привязываются динамические характеристики (EquipmentTypeProperties) и единицы техники (Equipments).

---

## 2. Функциональные требования

Общий перечень требований: Требования к системе DMMS

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-010 | Admin configures dynamic custom characteristics per equipment type via Admin Panel tool; characteristics appear automatically in equipment card, request form, and search filters based on equipment type | Confirmed | BRD v13 | Создание типа является предусловием для настройки его характеристик |
| TCO Booking Tool | FR-NEW-50 | Admin manages all reference/handbook values via Admin Panel | Confirmed | BRD v13 | Типы техники — справочная сущность, управляемая Admin Panel |
| TCO Booking Tool | FR-NEW-68 | System dynamically shows only type-specific characteristics in request creation form and equipment search filters, based on the selected equipment type | Confirmed | BRD v13 | Требует наличия созданных типов техники в справочнике |

---

## 3. Описание логики работы метода

1. Принять тело запроса; провалидировать обязательные поля (`name`, `mobilityType`, `requiresTransport`).
2. Проверить уникальность `name` в таблице `EquipmentTypes` WHERE `isDeleted = false`. Если запись с таким именем уже существует — вернуть `422 VALIDATION_ERROR`.
3. Проверить, что `mobilityType` входит в допустимые значения ENUM (`SelfPropelled / NonSelfPropelledMotorized / Stationary / NonMotorized`). Если нет — вернуть `422 VALIDATION_ERROR`.
4. Если `sortOrder` равен `null`, вычислить значение как `MAX(sortOrder) + 1` по всем записям `EquipmentTypes` WHERE `isDeleted = false`. Если таблица пуста — присвоить `1`.
5. Создать новую запись в `EquipmentTypes`; заполнить аудит-поля `createdAt` (текущее время) и `createdBy` (ID аутентифицированного пользователя).
6. Вернуть созданный объект в формате общего `result wrapper` с HTTP 201.

Сущности, участвующие в методе:
- читаются: `EquipmentTypes` (для проверки уникальности `name` и расчёта `sortOrder`)
- изменяются: `EquipmentTypes` (INSERT)
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
| `VALIDATION_ERROR` | Поле `name` пустое или уже существует в справочнике (с учётом `isDeleted = false`) |
| `VALIDATION_ERROR` | Поле `mobilityType` содержит недопустимое значение |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Наименование типа техники | `name` | `string` | `+` | Непустая строка; уникальная среди `EquipmentTypes` WHERE `isDeleted = false` | — | Request body | |
| 2 | Тип мобильности | `mobilityType` | `enum` | `+` | Одно из: `SelfPropelled`, `NonSelfPropelledMotorized`, `Stationary`, `NonMotorized` | — | Request body | |
| 3 | Требуется ли транспортировка | `requiresTransport` | `bool` | `+` | Булево значение | — | Request body | |
| 4 | Порядок отображения | `sortOrder` | `int` | `-` | Целое число >= 1, если передано | `null` → MAX + 1 | Request body | При `null` backend вычисляет автоматически |

---

## 8. Пример запроса

```http
POST /api/admin/v1/equipment-types
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "name": "Экскаватор",
  "mobilityType": "SelfPropelled",
  "requiresTransport": false,
  "sortOrder": null
}
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий `result wrapper` (использовать во всех API).  
В обёртке, в значении `value`, возвращается созданный объект типа `EquipmentType`.

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | `value` | `object` | `EquipmentType` | — | `EquipmentTypes` | Созданная запись |
| 2 | Признак успешности | `isSuccess` | `bool` | boolean | — | backend | |
| 3 | Ошибки | `errors` | `array<object>` | `ApiError[]` | `[]` | backend | При успешном ответе — пустой массив |

### Структура `EquipmentType`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор типа техники | `id` | `uuid` | UUID v4 | — | `EquipmentTypes.id` | Генерируется backend |
| 2 | Наименование типа техники | `name` | `string` | string | — | `EquipmentTypes.name` | |
| 3 | Тип мобильности | `mobilityType` | `enum` | `SelfPropelled / NonSelfPropelledMotorized / Stationary / NonMotorized` | — | `EquipmentTypes.mobilityType` | |
| 4 | Требуется ли транспортировка | `requiresTransport` | `bool` | boolean | — | `EquipmentTypes.requiresTransport` | |
| 5 | Порядок отображения | `sortOrder` | `int` | integer | — | `EquipmentTypes.sortOrder` | Фактическое значение (вычисленное или переданное) |

---

## 10. Пример ответа

```json
{
  "value": {
    "id": "3e2a1c4d-88f1-4b3e-a9d2-c7f0e1234567",
    "name": "Экскаватор",
    "mobilityType": "SelfPropelled",
    "requiresTransport": false,
    "sortOrder": 16
  },
  "isSuccess": true,
  "errors": []
}
```

---

## Замечания

1. При `sortOrder: null` backend присваивает `MAX(sortOrder) + 1` по всем не удалённым записям `EquipmentTypes`. Если таблица пуста — присваивает `1`.
2. Уникальность `name` проверяется только среди активных записей (`isDeleted = false`); повторное использование имени удалённого типа допустимо.
