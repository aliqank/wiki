**Created:** 2026-05-08  
**Last updated:** 2026-05-08  
**Author:** Telman Nurzhanov (SA)

---

# POST /equipment-types/:id/copy

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Создать новый тип техники как копию существующего, вместе с привязанными характеристиками |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Admin Panel` |
| Endpoint URL | `/api/admin/v1/equipment-types/:id/copy` |
| Метод запроса | `POST` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Обрабатывает действие «Копировать тип» из списка AP-01 и из карточки AP-02. Метод создаёт новый тип техники на основе существующего: копирует базовые параметры типа и все активные привязки характеристик, чтобы пользователь мог быстро создать похожий тип без ручной повторной настройки блока B.

---

## 2. Функциональные требования

Общий перечень требований: Требования к системе DMMS

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-010 | Admin configures dynamic custom characteristics per equipment type via Admin Panel tool; characteristics appear automatically in equipment card, request form, and search filters based on equipment type | Confirmed | BRD v13 | Копирование типа ускоряет создание новой схемы динамических характеристик на основе уже настроенного типа |
| TCO Booking Tool | FR-NEW-50 | Admin manages all reference/handbook values via Admin Panel | Confirmed | BRD v13 | Типы техники — справочная сущность; копирование входит в сценарий управления справочником |
| TCO Booking Tool | FR-NEW-68 | System dynamically shows only type-specific characteristics in request creation form and equipment search filters, based on the selected equipment type | Confirmed | BRD v13 | При копировании важно сохранить конфигурацию type-specific characteristics без ручной пересборки |

---

## 3. Описание логики работы метода

1. Найти исходную запись в `EquipmentTypes` WHERE `id` = `:id` AND `isDeleted = false`. Если запись не найдена — вернуть `404 NOT_FOUND`.
2. Прочитать все активные привязки характеристик исходного типа из `EquipmentTypeProperties` WHERE `equipmentTypeId` = `:id` AND `isDeleted = false`, сохранив значения `propertyId`, `isRequired`, `isFilterable`, `isVisibleInCard`, `sortOrder`.
3. Сформировать новую запись `EquipmentTypes` как копию исходной: перенести поля `mobilityType` и `requiresTransport`; поле `name` сформировать на основе исходного имени с постфиксом `(копия)`; поле `sortOrder` вычислить как `MAX(sortOrder) + 1` среди `EquipmentTypes` WHERE `isDeleted = false`. Если таблица пуста — присвоить `1`.
4. Создать новую запись в `EquipmentTypes`; заполнить аудит-поля `createdAt` (текущее время) и `createdBy` (ID аутентифицированного пользователя).
5. Для каждой активной записи из `EquipmentTypeProperties`, прочитанной на шаге 2, создать новую запись привязки для нового `equipmentTypeId`, сохранив те же `propertyId`, `isRequired`, `isFilterable`, `isVisibleInCard`, `sortOrder`; заполнить аудит-поля `createdAt` и `createdBy`.
6. Вернуть созданный объект нового типа в формате общего `result wrapper` с HTTP 201. После этого frontend может выполнить `GET /equipment-types/:newId` для открытия карточки новой копии.

Сущности, участвующие в методе:
- читаются: `EquipmentTypes`, `EquipmentTypeProperties`
- изменяются: `EquipmentTypes` (INSERT), `EquipmentTypeProperties` (batch INSERT)
- транзакционность: required

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
| `NOT_FOUND` | Исходный тип техники с указанным `id` не найден или помечен как удалённый |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор типа техники, который нужно скопировать | `id` | `uuid` | `+` | Должен быть валидным UUID v4 | — | Path param | |

У метода нет бизнес-параметров в request body. Допускается передача пустого JSON-объекта `{}`.

---

## 8. Пример запроса

```http
POST /api/admin/v1/equipment-types/7b4f4b4d-52d4-4a77-b6b7-f2b7d7c81111/copy
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{}
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий `result wrapper` (использовать во всех API).  
В обёртке, в значении `value`, возвращается созданный объект типа `EquipmentType`.

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | `value` | `object` | `EquipmentType` | — | `EquipmentTypes` | Созданная копия типа |
| 2 | Признак успешности | `isSuccess` | `bool` | boolean | — | backend | |
| 3 | Ошибки | `errors` | `array<object>` | `ApiError[]` | `[]` | backend | При успешном ответе — пустой массив |

### Структура `EquipmentType`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор нового типа техники | `id` | `uuid` | UUID v4 | — | `EquipmentTypes.id` | Генерируется backend |
| 2 | Наименование нового типа техники | `name` | `string` | string | — | `EquipmentTypes.name` | Формируется на основе имени исходного типа с постфиксом `(копия)` |
| 3 | Тип мобильности | `mobilityType` | `enum` | `SelfPropelled / NonSelfPropelledMotorized / Stationary / NonMotorized` | — | `EquipmentTypes.mobilityType` | Скопировано из исходного типа |
| 4 | Требуется ли транспортировка | `requiresTransport` | `bool` | boolean | — | `EquipmentTypes.requiresTransport` | Скопировано из исходного типа |
| 5 | Порядок отображения | `sortOrder` | `int` | integer | — | `EquipmentTypes.sortOrder` | Всегда назначается как следующий доступный `sortOrder` |

---

## 10. Пример ответа

```json
{
  "value": {
    "id": "8c5a6d7e-63b4-4ff1-9a12-0c1d23456789",
    "name": "Компрессор воздушный (копия)",
    "mobilityType": "Stationary",
    "requiresTransport": false,
    "sortOrder": 17
  },
  "isSuccess": true,
  "errors": []
}
```

---

## Замечания

1. Метод копирует только запись `EquipmentTypes` и привязки `EquipmentTypeProperties`. Справочные записи `Properties`, `MeasurementUnits` и `PropertyEnumValues` не дублируются: новая копия ссылается на те же существующие характеристики.
2. Порядок характеристик внутри блока B сохраняется один в один: для новых записей `EquipmentTypeProperties` копируются исходные значения `sortOrder`.
3. Метод должен выполняться в одной транзакции: нельзя допустить ситуацию, когда новый тип уже создан, а часть его характеристик ещё не скопирована.
