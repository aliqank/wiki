# API спецификация v2: Admin Panel — HDV/HDE Booking Tool

**Дата:** 2026-05-12  
**Основание:** ТЗ Admin Panel v2 (2026-05-04), Схема БД v8 (2026-05-12)  
**Для кого:** Backend-разработчик, Frontend-разработчик  
**Формат:** RESTful JSON API

---

## 1. Соглашения

| Параметр | Значение |
|----------|----------|
| Base URL | `/api/admin/v1` |
| Авторизация | `Authorization: Bearer <token>` (роль: Admin) |
| Content-Type | `application/json` |
| Result wrapper | Все ответы должны использовать общий result wrapper. См. `Результаты claude/2026-05-05 - Шаблон обертки результата API.md` |
| Пагинация | Для постраничных списков использовать `PaginatedResult` внутри `value`. См. `Результаты claude/2026-05-05 - Шаблон результата пагинации API.md` |
| Query params пагинации | `?page=1&limit=20` |
| Soft delete | Удаление через `DELETE`; ответ `204 No Content` |
| Ошибки | Ошибки должны возвращаться внутри общего wrapper: `value`, `isSuccess`, `errors[]` |
| UUID | Все `id` — UUID v4 |

**Локализованные поля `name`:**

- Для ресурсов `EquipmentTypes`, `Fleets`, `WorkCenters`, `Properties`, `MaintenancePartners`, `BusinessPartners` поле `name` больше не строка, а объект:
```json
{
  "En": "Excavator",
  "Ru": "Экскаватор",
  "Kz": "Экскаватор"
}
```
- API возвращает полный объект локализации во всех affected endpoints.
- Для `POST` / `PUT` / `PATCH` поле `name` должно передаваться в том же формате.
- Поиск по `search` выполняется по всем трём значениям: `name.En`, `name.Ru`, `name.Kz`.
- Сортировка по `name` в списках выполняется по `name.Ru` как значению по умолчанию, если отдельная locale-aware сортировка не оговорена явно.

**Коды ошибок (бизнес-логика):**

| HTTP | code | Когда |
|------|------|-------|
| 409 | `EQUIPMENT_TYPE_IN_USE` | Удаление типа, у которого есть техника |
| 409 | `PROPERTY_IN_USE` | Удаление характеристики из справочника, пока привязана к типам |
| 409 | `UNIT_IN_USE` | Удаление единицы измерения, пока привязана к характеристикам |
| 409 | `PERMISSION_ALREADY_EXISTS` | Пользователь уже имеет активное право на флот |
| 422 | `VALIDATION_ERROR` | Невалидные поля формы |
| 404 | `NOT_FOUND` | Ресурс не найден |

---

## 2. Сводная таблица API — все разделы

### Раздел 1: Типы техники (AP-01, AP-02)

| Метод | Путь | Экран / Действие |
|-------|------|-----------------|
| `GET` | `/equipment-types` | AP-01: список типов |
| `POST` | `/equipment-types` | AP-02: создать тип |
| `GET` | `/equipment-types/:id` | AP-02: открыть карточку |
| `PUT` | `/equipment-types/:id` | AP-02: сохранить базовые параметры |
| `DELETE` | `/equipment-types/:id` | AP-01: удалить тип (soft delete) |
| `POST` | `/equipment-types/:id/copy` | AP-01/AP-02: скопировать тип |
| `PATCH` | `/equipment-types/reorder` | AP-01: drag & drop списка |
| `GET` | `/equipment-types/:id/properties` | AP-02, блок B: список характеристик типа |
| `POST` | `/equipment-types/:id/properties` | M-AP-01 (таб «Новая»): создать и добавить характеристику |
| `POST` | `/equipment-types/:id/properties/link` | M-AP-01 (таб «Из справочника»): привязать существующую |
| `POST` | `/equipment-types/:id/properties/import` | M-AP-02: импорт из другого типа |
| `PATCH` | `/equipment-types/:id/properties/:etpId` | AP-02: изменить флаги (isRequired / isFilterable / isVisibleInCard) |
| `DELETE` | `/equipment-types/:id/properties/:etpId` | AP-02: удалить характеристику из типа |
| `PATCH` | `/equipment-types/:id/properties/reorder` | AP-02: drag & drop характеристик |

### Раздел 2: Характеристики (AP-03, M-AP-03)

| Метод | Путь | Экран / Действие |
|-------|------|-----------------|
| `GET` | `/properties` | AP-03: список справочника характеристик |
| `POST` | `/properties` | M-AP-03: создать характеристику |
| `GET` | `/properties/:id` | M-AP-03: данные для редактирования |
| `PUT` | `/properties/:id` | M-AP-03: сохранить изменения |
| `DELETE` | `/properties/:id` | AP-03: удалить характеристику |

### Раздел 3: Флоты и доступы (AP-04, AP-05, M-AP-04)

| Метод | Путь | Экран / Действие |
|-------|------|-----------------|
| `GET` | `/fleets` | AP-04: список флотов |
| `GET` | `/fleets/:id` | AP-05: карточка флота |
| `GET` | `/fleets/:id/permissions` | AP-05, блок B: список прав |
| `POST` | `/fleets/:id/permissions` | M-AP-04: добавить пользователя к флоту |
| `DELETE` | `/fleets/:id/permissions/:permId` | AP-05: отозвать право |

### Раздел 4: Справочники (AP-06, M-AP-05)

| Метод | Путь | Экран / Действие |
|-------|------|-----------------|
| `GET` | `/measurement-units` | AP-06, таб «Единицы измерения» |
| `POST` | `/measurement-units` | AP-06: добавить единицу (inline-форма) |
| `PUT` | `/measurement-units/:id` | AP-06: сохранить inline-редактирование |
| `DELETE` | `/measurement-units/:id` | AP-06: удалить единицу |
| `PATCH` | `/measurement-units/reorder` | AP-06: drag & drop |
| `GET` | `/maintenance-partners` | AP-06, таб «Партнёры ТО» |
| `POST` | `/maintenance-partners` | M-AP-05: создать партнёра |
| `PUT` | `/maintenance-partners/:id` | M-AP-05: редактировать партнёра |
| `DELETE` | `/maintenance-partners/:id` | AP-06: удалить партнёра |

### Раздел 5: Пользователи (AP-07)

| Метод | Путь | Экран / Действие |
|-------|------|-----------------|
| `GET` | `/users` | AP-07: список с поиском и фильтром |
| `GET` | `/users/:id` | AP-07: drawer с деталями |

### Раздел 6: Системные параметры (AP-08)

| Метод | Путь | Экран / Действие |
|-------|------|-----------------|
| `GET` | `/system-settings` | AP-08: все параметры |
| `PATCH` | `/system-settings` | AP-08: сохранить форму (bulk update) |

### Раздел 7: Мониторинг (AP-09)

| Метод | Путь | Экран / Действие |
|-------|------|-----------------|
| `GET` | `/monitoring/booking-requests` | AP-09, таб «Заявки» |
| `GET` | `/monitoring/booking-requests/:id` | AP-09: drawer заявки |
| `GET` | `/monitoring/bookings` | AP-09, таб «Брони» |
| `GET` | `/monitoring/bookings/:id` | AP-09: drawer брони |
| `GET` | `/monitoring/feedbacks` | AP-09, таб «Отзывы» |

---

## 3. Детальный пример: Раздел «Типы техники»

### 3.1 Use Case — таблица покрытия

| UC ID | Описание | Эндпоинты |
|-------|----------|-----------|
| UC-ET-01 | Просмотр списка типов техники с поиском | `GET /equipment-types` |
| UC-ET-02 | Изменение порядка типов (drag & drop) | `PATCH /equipment-types/reorder` |
| UC-ET-03 | Открыть карточку типа | `GET /equipment-types/:id` |
| UC-ET-04 | Создать новый тип | `POST /equipment-types` |
| UC-ET-05 | Сохранить изменения базовых параметров типа | `PUT /equipment-types/:id` |
| UC-ET-06 | Скопировать тип | `POST /equipment-types/:id/copy` |
| UC-ET-07 | Удалить тип (guard: нет привязанной техники) | `DELETE /equipment-types/:id` |
| UC-ET-08 | Просмотр характеристик типа (блок B) | `GET /equipment-types/:id/properties` |
| UC-ET-09 | Создать новую характеристику и добавить к типу | `POST /equipment-types/:id/properties` |
| UC-ET-10 | Добавить характеристику из справочника | `POST /equipment-types/:id/properties/link` |
| UC-ET-11 | Импортировать характеристики из другого типа | `POST /equipment-types/:id/properties/import` |
| UC-ET-12 | Изменить флаги характеристики в типе (inline) | `PATCH /equipment-types/:id/properties/:etpId` |
| UC-ET-13 | Изменить порядок характеристик (drag & drop) | `PATCH /equipment-types/:id/properties/reorder` |
| UC-ET-14 | Удалить характеристику из типа | `DELETE /equipment-types/:id/properties/:etpId` |
| UC-ET-15 | Поиск по справочнику характеристик (для M-AP-01) | `GET /properties?search=&excludeTypeId=` |

---

### 3.2 Эндпоинты — подробные спецификации

---

#### `GET /equipment-types`

**Use case:** UC-ET-01 — отобразить AP-01 (список типов техники)

**Соглашение по response:**
- используется общий result wrapper: `Результаты claude/2026-05-05 - Шаблон обертки результата API.md`
- внутри `value` используется `PaginatedResult`: `Результаты claude/2026-05-05 - Шаблон результата пагинации API.md`

**Источник данных по БД v8:**
- `EquipmentTypes` — базовые поля типа техники
- `EquipmentTypeProperties` — для расчёта `propertiesCount`
- `Equipments` — для расчёта `equipmentsCount`

**Query params:**
```
search        string   — фильтр по `name.En` / `name.Ru` / `name.Kz`
page          int      — default 1
limit         int      — default 20
```

**Логика метода:**
1. Получить список записей из `EquipmentTypes`.
2. Применить фильтр `search` по полям `name.En`, `name.Ru`, `name.Kz`, если он передан.
3. Отсортировать результат по `sortOrder ASC`, затем по `name.Ru ASC` как вторичный стабильный порядок по умолчанию.
4. Для каждой записи рассчитать:
   - `propertiesCount` = COUNT(`EquipmentTypeProperties` WHERE `equipmentTypeId` = `EquipmentTypes.id`)
   - `equipmentsCount` = COUNT(`Equipments` WHERE `equipmentTypeId` = `EquipmentTypes.id`)
5. Вернуть страницу данных в формате общего result wrapper + `PaginatedResult`.

**Response 200:**
```json
{
  "value": {
    "items": [
      {
        "id": "uuid",
        "name": {
          "En": "Compressor",
          "Ru": "Компрессор",
          "Kz": "Компрессор"
        },
        "mobilityType": "Stationary",
        "requiresTransport": true,
        "sortOrder": 1,
        "propertiesCount": 7,
        "equipmentsCount": 12
      }
    ],
    "total": 15
  },
  "isSuccess": true,
  "errors": []
}
```

**Поля response item:**

| Поле | Источник | Комментарий |
|------|----------|-------------|
| `id` | `EquipmentTypes.id` | UUID типа техники |
| `name` | `EquipmentTypes.name` | Локализованное наименование типа: `{ En, Ru, Kz }` |
| `mobilityType` | `EquipmentTypes.mobilityType` | `SelfPropelled / NonSelfPropelledMotorized / Stationary / NonMotorized` |
| `requiresTransport` | `EquipmentTypes.requiresTransport` | Требуется ли отдельная транспортная техника |
| `sortOrder` | `EquipmentTypes.sortOrder` | Порядок отображения в UI |
| `propertiesCount` | агрегат по `EquipmentTypeProperties` | Нужен для колонки списка |
| `equipmentsCount` | агрегат по `Equipments` | Нужен для колонки списка и guard удаления |

**Замечание:** в схеме БД v8 `GET /equipment-types` не должен возвращать `deletedAt` и не должен ссылаться на фильтрацию по нему в описании метода.

---

#### `POST /equipment-types`

**Use case:** UC-ET-04 — создать тип через форму AP-02

**Request body:**
```json
{
  "name": {
    "En": "Excavator",
    "Ru": "Экскаватор",
    "Kz": "Экскаватор"
  },
  "mobilityType": "SelfPropelled",
  "requiresTransport": false,
  "sortOrder": null
}
```

> `sortOrder: null` → backend назначает следующий свободный (MAX + 1)

**Response 201:**
```json
{
  "id": "uuid",
  "name": {
    "En": "Excavator",
    "Ru": "Экскаватор",
    "Kz": "Экскаватор"
  },
  "mobilityType": "SelfPropelled",
  "requiresTransport": false,
  "sortOrder": 16
}
```

**Errors:**
- `422 VALIDATION_ERROR` — `name.Ru` пустое или объект `name` не передан
- `422 VALIDATION_ERROR` — сочетание `name.En` / `name.Ru` / `name.Kz` не проходит правило уникальности
- `422 VALIDATION_ERROR` — `mobilityType` не из допустимых значений

---

#### `GET /equipment-types/:id`

**Use case:** UC-ET-03 — открыть карточку AP-02

**Response 200:**
```json
{
  "id": "uuid",
  "name": {
    "En": "Compressor",
    "Ru": "Компрессор",
    "Kz": "Компрессор"
  },
  "mobilityType": "Stationary",
  "requiresTransport": true,
  "sortOrder": 1,
  "equipmentsCount": 12,
  "properties": [
    {
      "etpId": "uuid",
      "sortOrder": 1,
      "isRequired": true,
      "isFilterable": true,
      "isVisibleInCard": true,
      "property": {
        "id": "uuid",
        "name": {
          "En": "Receiver volume",
          "Ru": "Объём ресивера",
          "Kz": "Ресивер көлемі"
        },
        "dataType": "double",
        "unit": {
          "id": "uuid",
          "code": "л",
          "displayName": "Литр"
        },
        "enumValues": []
      }
    },
    {
      "etpId": "uuid",
      "sortOrder": 2,
      "isRequired": false,
      "isFilterable": false,
      "isVisibleInCard": true,
      "property": {
        "id": "uuid",
        "name": {
          "En": "Drive type",
          "Ru": "Тип привода",
          "Kz": "Жетек түрі"
        },
        "dataType": "enum",
        "unit": null,
        "enumValues": [
          { "id": "uuid", "value": "Дизельный", "sortOrder": 1 },
          { "id": "uuid", "value": "Электрический", "sortOrder": 2 }
        ]
      }
    }
  ]
}
```

> `properties` включены в ответ карточки (не отдельным запросом) — один HTTP round-trip для загрузки AP-02.

---

#### `PUT /equipment-types/:id`

**Use case:** UC-ET-05 — кнопка «Сохранить» в AP-02

**Request body:** (только базовые параметры; характеристики — через отдельные эндпоинты)
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

**Response 200:** полный объект (как в GET /:id, без properties)

---

#### `DELETE /equipment-types/:id`

**Use case:** UC-ET-07 — иконка «Удалить» в AP-01

**Response 204** — успешное soft delete

**Error:**
```json
HTTP 409
{
  "error": {
    "code": "EQUIPMENT_TYPE_IN_USE",
    "message": "Нельзя удалить: к типу привязано 12 единиц техники",
    "details": { "equipmentsCount": 12 }
  }
}
```

> Frontend блокирует кнопку заранее (по `equipmentsCount` из списка), но guard на уровне API обязателен.

---

#### `POST /equipment-types/:id/copy`

**Use case:** UC-ET-06 — «Копировать тип» из AP-01 или AP-02

**Request body:** пустой `{}`

**Response 201:**
```json
{
  "id": "uuid-новый",
  "name": {
    "En": "Air compressor (copy)",
    "Ru": "Компрессор воздушный (копия)",
    "Kz": "Ауа компрессоры (көшірме)"
  },
  "mobilityType": "Stationary",
  "requiresTransport": false,
  "sortOrder": 17
}
```

> Backend: копирует запись `EquipmentTypes` вместе со всем объектом `name` и всеми записями `EquipmentTypeProperties` (включая ссылки на те же `Properties`). `sortOrder = MAX + 1`. Frontend сразу редиректит на `GET /equipment-types/:newId` (карточку новой копии).

---

#### `PATCH /equipment-types/reorder`

**Use case:** UC-ET-02 — drag & drop строк в AP-01

**Request body:**
```json
{
  "order": [
    { "id": "uuid-1", "sortOrder": 1 },
    { "id": "uuid-2", "sortOrder": 2 },
    { "id": "uuid-3", "sortOrder": 3 }
  ]
}
```

**Response 204**

> Принимает полный новый порядок (не дельту). Backend обновляет все sortOrder в одной транзакции.

---

#### `POST /equipment-types/:id/properties`

**Use case:** UC-ET-09 — M-AP-01, таб «Новая характеристика»

Создаёт запись в `Properties` + запись в `EquipmentTypeProperties` в одной транзакции.

**Request body:**
```json
{
  "property": {
    "name": {
      "En": "Working pressure",
      "Ru": "Рабочее давление",
      "Kz": "Жұмыс қысымы"
    },
    "dataType": "double",
    "unitId": "uuid-бар",
    "enumValues": []
  },
  "config": {
    "isRequired": true,
    "isFilterable": true,
    "isVisibleInCard": true
  }
}
```

Для типа `enum` — `enumValues` обязателен (минимум 2 значения):
```json
{
  "property": {
    "name": {
      "En": "Drive type",
      "Ru": "Тип привода",
      "Kz": "Жетек түрі"
    },
    "dataType": "enum",
    "unitId": null,
    "enumValues": [
      { "value": "Дизельный", "sortOrder": 1 },
      { "value": "Электрический", "sortOrder": 2 }
    ]
  },
  "config": {
    "isRequired": false,
    "isFilterable": true,
    "isVisibleInCard": true
  }
}
```

**Response 201:**
```json
{
  "etpId": "uuid",
  "sortOrder": 3,
  "isRequired": true,
  "isFilterable": true,
  "isVisibleInCard": true,
  "property": {
    "id": "uuid",
    "name": {
      "En": "Working pressure",
      "Ru": "Рабочее давление",
      "Kz": "Жұмыс қысымы"
    },
    "dataType": "double",
    "unit": { "id": "uuid", "code": "бар", "displayName": "Бар" },
    "enumValues": []
  }
}
```

**Errors:**
- `422` — `enumValues` < 2 при `dataType: "enum"`
- `422` — объект `name` невалиден или не передан
- `422` — локализованное `name` не уникально в `Properties`

---

#### `POST /equipment-types/:id/properties/link`

**Use case:** UC-ET-10 — M-AP-01, таб «Из справочника»

Привязывает существующую характеристику к типу (только запись в EquipmentTypeProperties).

**Request body:**
```json
{
  "propertyId": "uuid",
  "isRequired": false,
  "isFilterable": true,
  "isVisibleInCard": true
}
```

**Response 201:** (аналогично POST /properties — полный объект etpId + property)

**Errors:**
- `409` — характеристика уже привязана к этому типу (`PROPERTY_ALREADY_LINKED`)
- `404` — `propertyId` не существует

---

#### `POST /equipment-types/:id/properties/import`

**Use case:** UC-ET-11 — M-AP-02: импорт из другого типа

**Request body:**
```json
{
  "sourceTypeId": "uuid-источник",
  "propertyIds": ["uuid-prop-1", "uuid-prop-2"]
}
```

> Backend: для каждого `propertyId` создаёт запись в EquipmentTypeProperties, наследуя `isRequired`, `isFilterable`, `isVisibleInCard` из источника. Уже привязанные — пропускаются (не ошибка).

**Response 200:**
```json
{
  "imported": 2,
  "skipped": 0,
  "properties": [ /* массив etpId + property, как в GET /:id */ ]
}
```

---

#### `PATCH /equipment-types/:id/properties/:etpId`

**Use case:** UC-ET-12 — изменение чекбоксов isRequired / isFilterable / isVisibleInCard прямо в таблице AP-02

**Request body:** (partial update — только изменяемые флаги)
```json
{
  "isRequired": true,
  "isFilterable": false
}
```

**Response 200:**
```json
{
  "etpId": "uuid",
  "isRequired": true,
  "isFilterable": false,
  "isVisibleInCard": true,
  "sortOrder": 2
}
```

---

#### `DELETE /equipment-types/:id/properties/:etpId`

**Use case:** UC-ET-14 — иконка «Удалить» у характеристики в AP-02

**Response 200:**
```json
{
  "deletedFromType": true,
  "usedInOtherTypes": ["Экскаватор", "Бульдозер"]
}
```

> `usedInOtherTypes` — список названий других типов, где эта же характеристика ещё используется. Если пустой массив — характеристика удалена нигде больше не используется. Frontend показывает диалог согласно п. 16.3 ТЗ только если список непустой (информационно, не блокирующий guard).

**Errors:**
- `404` — etpId не найден или не принадлежит данному типу

---

#### `PATCH /equipment-types/:id/properties/reorder`

**Use case:** UC-ET-13 — drag & drop характеристик в блоке B (AP-02)

**Request body:**
```json
{
  "order": [
    { "etpId": "uuid-1", "sortOrder": 1 },
    { "etpId": "uuid-2", "sortOrder": 2 }
  ]
}
```

**Response 204**

---

### 3.3 Вспомогательные эндпоинты для раздела «Типы техники»

#### `GET /properties?search=&excludeTypeId=&page=&limit=`

**Use case:** UC-ET-15 — поиск по справочнику в M-AP-01 (таб «Из справочника»)

| Query param | Описание |
|-------------|----------|
| `search` | Фильтр по `Properties.name.En` / `Properties.name.Ru` / `Properties.name.Kz` |
| `excludeTypeId` | UUID типа — исключить уже привязанные к нему характеристики |

**Response 200:**
```json
{
  "data": [
    {
      "id": "uuid",
      "name": {
        "En": "Receiver volume",
        "Ru": "Объём ресивера",
        "Kz": "Ресивер көлемі"
      },
      "dataType": "double",
      "unit": { "id": "uuid", "code": "л", "displayName": "Литр" },
      "enumValues": [],
      "usedInTypesCount": 3
    }
  ],
  "total": 42,
  "page": 1,
  "limit": 20
}
```

---

### 3.4 Схема задач разработки (Раздел «Типы техники»)

| # | Задача | Эндпоинты | Трудоёмкость |
|---|--------|-----------|-------------|
| T-ET-01 | **BE:** CRUD EquipmentTypes (GET list, GET by id, POST, PUT, DELETE soft) | GET `/equipment-types`, GET `/equipment-types/:id`, POST, PUT, DELETE | M |
| T-ET-02 | **BE:** Copy и Reorder типов техники | POST `/:id/copy`, PATCH `/reorder` | S |
| T-ET-03 | **BE:** CRUD характеристик в типе (новая + из справочника) | POST `/properties`, POST `/properties/link`, PATCH `/:etpId`, DELETE `/:etpId` | M |
| T-ET-04 | **BE:** Import характеристик из другого типа | POST `/properties/import` | S |
| T-ET-05 | **BE:** Reorder характеристик в типе | PATCH `/properties/reorder` | XS |
| T-ET-06 | **FE:** Экран AP-01 (список, поиск, drag & drop, действия) | GET `/equipment-types` | M |
| T-ET-07 | **FE:** Экран AP-02 (карточка: блок A + блок B, Save/Cancel) | GET `/:id`, PUT `/:id`, POST | L |
| T-ET-08 | **FE:** Модалка M-AP-01 (два таба: Новая / Из справочника) | POST `/properties`, POST `/properties/link`, GET `/properties` | M |
| T-ET-09 | **FE:** Модалка M-AP-02 (импорт из другого типа) | POST `/properties/import`, GET `/equipment-types` | M |

---

## 4. Краткое описание остальных разделов (для задач разработки)

### Характеристики (AP-03, M-AP-03)

**Ключевые особенности API:**
- `GET /properties` поддерживает фильтр `?dataType=enum` для колонки «Тип значения»
- `name` в `GET /properties`, `POST /properties`, `PUT /properties/:id` передаётся как локализованный JSON-объект `{ En, Ru, Kz }`
- `PUT /properties/:id` — поле `dataType` нельзя изменить, если `usedInTypesCount > 0` → backend возвращает 422 с `code: "DATA_TYPE_LOCKED"`
- При удалении enum-значений из `PropertyEnumValues` (в рамках редактирования) — отдельная операция внутри `PUT /properties/:id`

### Флоты и доступы (AP-04, AP-05, M-AP-04)

**Ключевые особенности API:**
- `GET /fleets` — данные из внешней системы, read-only; создание/удаление через API недоступно
- поле `name` в `GET /fleets` и `GET /fleets/:id` возвращается как локализованный JSON-объект `{ En, Ru, Kz }`
- `GET /fleets/:id/permissions` возвращает `status: "active" | "expired"` (вычисляется: `expiresAt < today → expired`)
- `POST /fleets/:id/permissions` — поиск пользователя: `GET /users?search=Иванов` (от 2 символов)
- Проверка дубликата: если `userId` уже имеет активное право на этом флоте → `409 PERMISSION_ALREADY_EXISTS`

### Справочники (AP-06)

**Ключевые особенности API:**
- `PUT /measurement-units/:id` — inline-редактирование; меняет только `code` и `displayName`
- `GET/POST/PUT /maintenance-partners` используют `name` как локализованный JSON-объект `{ En, Ru, Kz }`
- `DELETE /maintenance-partners/:id` — guard мягкий: если есть договоры ТО, возвращает `200` с `{ warning: "contractsCount": 3, confirm: true }` → Frontend показывает предупреждающий диалог (не блокирующий)

### Пользователи (AP-07)

**Ключевые особенности API:**
- `GET /users?search=&type=internal|external` — только чтение; создание/удаление недоступно
- `GET /users/:id` — для drawer; возвращает `type`, `badgeNumber`, `sharedEmail`, `department`, `businessPartnerId` и т.д.
- Этот же эндпоинт используется для поиска в M-AP-04 (добавить пользователя к флоту)

### Системные параметры (AP-08)

**Ключевые особенности API:**
- `GET /system-settings` возвращает типизированный объект (не raw key-value):
```json
{
  "booking": { "horizonDays": 90, "maxDurationDays": 30 },
  "fo": { "timeoutHours24": 24, "timeoutHours48": 48 },
  "lastUpdatedAt": "2026-05-01T10:00:00Z"
}
```
- `PATCH /system-settings` принимает тот же формат (partial update); backend обновляет SystemSettings.key-value атомарно

### Мониторинг (AP-09)

**Ключевые особенности API:**
- Все три эндпоинта — только чтение (GET)
- `GET /monitoring/booking-requests/:id` включает историю статусов из BookingRequestStatuses
- `GET /monitoring/bookings/:id` включает историю из BookingStatuses + `transportBooking` (если есть)
- Фильтр периода для броней: по пересечению периода `[startDt, endDt]` с выбранным диапазоном (`startDt <= filterEnd AND endDt >= filterStart`)

---

*Документ подготовлен на основе ТЗ Admin Panel v2 и Схемы БД v8. Версия API: v2.*
