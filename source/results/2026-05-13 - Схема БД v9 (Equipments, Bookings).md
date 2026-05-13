# Схема БД v9: Equipments + Bookings (сводный файл)

**Created:** 2026-05-13  
**Last updated:** 2026-05-13  
**Author:** Telman Nurzhanov (SA)

---

## Что изменилось в v9

| # | Изменение | Затронутые таблицы |
|---|---|---|
| 1 | В `EquipmentTypes` добавлено поле `iconUrl` | EquipmentTypes |
| 2 | Добавлен иерархический справочник `Brand -> Model`; строковые поля `brand` / `model` заменены на FK | EquipmentBrandModels, Equipments |
| 3 | Добавлен справочник локаций; строковое поле `baseLocation` заменено на FK | Locations, Equipments |
| 4 | Добавлен справочник cost centers; строковое поле `costCenter` заменено на FK | CostCenters, Equipments |
| 5 | Добавлен справочник service zones; строковое поле `serviceZoneCode` заменено на FK | ServiceZones, Equipments |
| 6 | Добавлен иерархический справочник подразделений `Division -> Group -> Department -> Section`; в `Equipments` добавлен FK на запись уровня `Section` | OrganizationalUnits, Equipments |
| 7 | В `Equipments` добавлены поля `vin` и `e1Id` | Equipments |

Стандартный набор аудит-полей:

| Поле | Тип | Описание |
|---|---|---|
| createdAt | TIMESTAMP NOT NULL | Дата создания записи |
| createdBy | UUID NOT NULL | ID пользователя-создателя (без FK-ограничения) |
| updatedAt | TIMESTAMP nullable | Дата последнего обновления; NULL если запись не обновлялась |
| updatedBy | UUID nullable | ID пользователя, изменившего запись |
| isDeleted | BOOLEAN NOT NULL DEFAULT false | Флаг мягкого удаления |
| deletedAt | TIMESTAMP nullable | Дата soft delete |
| deletedBy | UUID nullable | ID пользователя, выполнившего soft delete |

> **Log/history-таблицы** (BookingStatuses, BookingRequestStatuses): поля `updatedAt` и `updatedBy` всегда NULL (записи append-only). `changedAt` ≡ `createdAt`, `changedBy` ≡ `createdBy` для этих таблиц по смыслу.  
> **createdBy для SystemSettings**: nullable, т.к. записи могут создаваться системой.

---

## Принятые решения

### По версии v9

| Тема | Решение |
|---|---|
| EquipmentTypes.iconUrl | Добавлено поле URL иконки типа техники для Admin Panel и каталогов выбора |
| Equipment brands / models | Производители и модели вынесены в единый self-reference справочник `EquipmentBrandModels` с уровнями `Brand / Model`; в `Equipments` хранятся `equipmentBrandId` и `equipmentModelId`, оба FK ссылаются на одну таблицу |
| Base locations | Базовые местоположения вынесены в справочник `Locations`; строка `baseLocation` в `Equipments` заменена на `baseLocationId` |
| Cost centers | Финансовые ЦЗ вынесены в справочник `CostCenters`; строка `costCenter` в `Equipments` заменена на `costCenterId` |
| Service zones | Сервисные зоны вынесены в справочник `ServiceZones`; строка `serviceZoneCode` в `Equipments` заменена на `serviceZoneId` |
| Organizational structure | Для подразделений используется единый self-reference справочник `OrganizationalUnits` с уровнями `Division / Group / Department / Section`; в `Equipments` хранится ссылка только на запись уровня `Section` |
| External equipment identifiers | `vin` и `e1Id` добавлены как отдельные nullable string-поля в `Equipments` |
| Локализация `name` | Все поля с именем `name` хранятся как `JSON` в формате `{ "En": "...", "Ru": "...", "Kz": "..." }` |
| MaintenancePartners | Поле `contactInfo` разделено на отдельные поля `phoneNumber`, `email`, `address` |
| Equipments usage targets | Добавлены поля `plannedEngineHoursPerDay` и `plannedMileagePerDay` — плановые значения использования техники в сутки, редактируются Fleet Owner |
| Аудит-поля | Единый набор из 7 полей добавлен во все 26 таблиц. Для log/history-таблиц updatedAt/updatedBy ожидаются как NULL. createdBy / updatedBy / deletedBy — UUID без FK-ограничения |
| Users | Таблица хранит 2 типа пользователей: `internal` и `external`; `internal` создаются автоматически при авторизации, `external` — вручную. Общие поля: `id`, `fullName`, `email`, `jobTitle`, `isActive` + аудит; nullable-поля используются для разделения атрибутов по типам |
| isDeleted в API | GET-методы Admin Panel по умолчанию фильтруют `WHERE isDeleted = false`; параметр не выставляется наружу |

### По встрече 30.04.2026

| Тема | Решение |
|---|---|
| EquipmentTrackers | Исключена из схемы — не входит в зону ответственности модуля букинга |
| Статус техники (EquipmentStatuses) | Одна таблица вместо трёх (EquipmentFreezes + EquipmentRepairHistory + decommission); хранит историю всех состояний; активная запись определяет текущий статус |
| currentStatus в Equipments | Не хранимое поле — вычисляется при запросе из EquipmentStatuses |
| FleetManagePermissions | Переименована из FleetDelegations, упрощена: id, fleetId, userId, createdAt, expiresAt |
| isDefaultWorkOrder в BookingRequests | Убрано; workOrderJdeId остаётся nullable |
| isExternal в Fleets | Убрано |
| status в BookingRequests и Bookings | Хранится как денормализованный кэш; обновляется атомарно вместе с вставкой в BookingRequestStatuses / BookingStatuses. Источник правды — история |
| BookingRequestStatuses | RequestStatusHistories переименована; `oldStatus` + `newStatus` → `status` (текущий статус в момент записи) |
| isCritical в Equipments | Булевый флаг «требует охраны при транспортировке»; отображается в карточке техники |
| yearOfManufacture в Equipments | Год выпуска; nullable; рекомендуется для TCO Owned |
| actualStartDt / actualEndDt в Bookings | Фактические даты использования; вводятся при закрытии брони (FR-NEW-33) |
| EquipmentFeedbacks | Отзывы заявителей/SWP на технику (FR-022); привязаны к конкретной брони |
| assignedFleetOwnerId в Bookings | Убрано |
| BookingStatuses | BookingStatusHistories переименована; запись фиксирует текущее состояние (не переход); убран previousSnapshot |

### Ранее принятые решения (v1–v4)

| Тема | Решение |
|---|---|
| EquipmentTypes | Плоский список; requiresTransport — хранимое поле |
| EquipmentTypeProperties | Схема свойств для каждого типа техники |
| EquipmentProperties.value | JSONB: `{"Bool", "Double", "Number", "String"}` |
| ownershipType | На Equipments — единая таблица TCO + BP |
| Фото | Отдельная таблица EquipmentPhotos |
| ТО | Несколько партнёров на одну технику (через EquipmentMaintenanceContracts) |
| shareType history | Не нужна — хранится только текущее значение |
| Архитектура Request / Booking | BookingRequests (шапка заявки) + Bookings (одна единица техники); Order / OrderLine pattern |
| Статус BookingRequests | Хранится явно (денормализация); агрегируется из вложенных Bookings |
| Внешние ID | equipmentId, fleetId — UUID без FK-ограничений |
| requestNumber | SERIAL (INT) в БД; форматируется в UI как REQ-YYYY-NNNNN |
| Транспортная бронь | Self-ref FK `transportBookingId` в Bookings; статус `TransportConfirmed`; обязана принадлежать тому же requestId |

---

## Блок Equipment

### 1. EquipmentTypes

| Поле | Тип | Описание |
|---|---|---|
| id | UUID PK | |
| name | JSON | Локализованное наименование: `{ "En": "...", "Ru": "...", "Kz": "..." }` |
| iconUrl | VARCHAR nullable | URL иконки типа техники |
| mobilityType | ENUM | SelfPropelled / NonSelfPropelledMotorized / Stationary / NonMotorized |
| requiresTransport | BOOLEAN | Требуется транспортная техника для доставки |
| equipmentClass | ENUM | Класс техники: HDE / HDV |
| workCenterId | FK → WorkCenters | Производственный центр / work center |
| sortOrder | INT | Порядок отображения в UI |
| createdAt | TIMESTAMP NOT NULL | |
| createdBy | UUID NOT NULL | Без FK-ограничения |
| updatedAt | TIMESTAMP nullable | |
| updatedBy | UUID nullable | |
| isDeleted | BOOLEAN NOT NULL DEFAULT false | |
| deletedAt | TIMESTAMP nullable | |
| deletedBy | UUID nullable | |

---

### 2. Fleets

| Поле | Тип | Описание |
|---|---|---|
| id | UUID PK | |
| name | JSON | Локализованное наименование: `{ "En": "...", "Ru": "...", "Kz": "..." }` |
| type | ENUM | Maintenance / Construction / TFM / SCMLogistics / ExternalBP |
| userId | UUID | ID пользователя-владельца / ответственного за флот |
| aadGroupId | VARCHAR | ID AAD-группы Fleet Owner |
| createdAt | TIMESTAMP NOT NULL | |
| createdBy | UUID NOT NULL | Без FK-ограничения |
| updatedAt | TIMESTAMP nullable | |
| updatedBy | UUID nullable | |
| isDeleted | BOOLEAN NOT NULL DEFAULT false | |
| deletedAt | TIMESTAMP nullable | |
| deletedBy | UUID nullable | |

---

### 2а. WorkCenters

Справочник производственных центров (work centers).

| Поле | Тип | Описание |
|---|---|---|
| id | UUID PK | |
| name | JSON | Локализованное наименование производственного центра: `{ "En": "...", "Ru": "...", "Kz": "..." }` |
| code | VARCHAR | Код производственного центра |
| createdAt | TIMESTAMP NOT NULL | |
| createdBy | UUID NOT NULL | Без FK-ограничения |
| updatedAt | TIMESTAMP nullable | |
| updatedBy | UUID nullable | |
| isDeleted | BOOLEAN NOT NULL DEFAULT false | |
| deletedAt | TIMESTAMP nullable | |
| deletedBy | UUID nullable | |

---

### 2б. EquipmentBrandModels

Иерархический справочник производителей и моделей техники.

| Поле | Тип | Описание |
|---|---|---|
| id | UUID PK | |
| parentId | FK → EquipmentBrandModels nullable | Родительский узел; `NULL` только для `Brand` |
| level | ENUM | Brand / Model |
| name | JSON | Локализованное наименование узла: `{ "En": "...", "Ru": "...", "Kz": "..." }` |
| sortOrder | INT | Порядок отображения в UI |
| createdAt | TIMESTAMP NOT NULL | |
| createdBy | UUID NOT NULL | Без FK-ограничения |
| updatedAt | TIMESTAMP nullable | |
| updatedBy | UUID nullable | |
| isDeleted | BOOLEAN NOT NULL DEFAULT false | |
| deletedAt | TIMESTAMP nullable | |
| deletedBy | UUID nullable | |

> Правила целостности:
> - запись уровня `Brand` имеет `parentId = NULL`
> - запись уровня `Model` обязана ссылаться на родителя уровня `Brand`
> - `Equipments.equipmentBrandId` должен ссылаться на запись уровня `Brand`
> - `Equipments.equipmentModelId` должен ссылаться на запись уровня `Model`

**Индексы / ограничения:** `parentId`; UNIQUE `(parentId, name)`

---

### 2в. Locations

Справочник базовых локаций техники.

| Поле | Тип | Описание |
|---|---|---|
| id | UUID PK | |
| code | VARCHAR | Код локации / внешний идентификатор |
| name | JSON | Локализованное наименование локации: `{ "En": "...", "Ru": "...", "Kz": "..." }` |
| sortOrder | INT | Порядок отображения в UI |
| createdAt | TIMESTAMP NOT NULL | |
| createdBy | UUID NOT NULL | Без FK-ограничения |
| updatedAt | TIMESTAMP nullable | |
| updatedBy | UUID nullable | |
| isDeleted | BOOLEAN NOT NULL DEFAULT false | |
| deletedAt | TIMESTAMP nullable | |
| deletedBy | UUID nullable | |

---

### 2г. CostCenters

Справочник cost centers / финансовых ЦЗ.

| Поле | Тип | Описание |
|---|---|---|
| id | UUID PK | |
| code | VARCHAR | Код cost center из JDE |
| name | JSON | Локализованное наименование cost center: `{ "En": "...", "Ru": "...", "Kz": "..." }` |
| sortOrder | INT | Порядок отображения в UI |
| createdAt | TIMESTAMP NOT NULL | |
| createdBy | UUID NOT NULL | Без FK-ограничения |
| updatedAt | TIMESTAMP nullable | |
| updatedBy | UUID nullable | |
| isDeleted | BOOLEAN NOT NULL DEFAULT false | |
| deletedAt | TIMESTAMP nullable | |
| deletedBy | UUID nullable | |

---

### 2д. ServiceZones

Справочник сервисных зон.

| Поле | Тип | Описание |
|---|---|---|
| id | UUID PK | |
| code | VARCHAR | Код сервисной зоны |
| name | JSON | Локализованное наименование сервисной зоны: `{ "En": "...", "Ru": "...", "Kz": "..." }` |
| sortOrder | INT | Порядок отображения в UI |
| createdAt | TIMESTAMP NOT NULL | |
| createdBy | UUID NOT NULL | Без FK-ограничения |
| updatedAt | TIMESTAMP nullable | |
| updatedBy | UUID nullable | |
| isDeleted | BOOLEAN NOT NULL DEFAULT false | |
| deletedAt | TIMESTAMP nullable | |
| deletedBy | UUID nullable | |

---

### 2е. OrganizationalUnits

Иерархический справочник подразделений. Одна таблица хранит уровни `Division -> Group -> Department -> Section`.

| Поле | Тип | Описание |
|---|---|---|
| id | UUID PK | |
| parentId | FK → OrganizationalUnits nullable | Родительский узел; `NULL` только для `Division` |
| level | ENUM | Division / Group / Department / Section |
| code | VARCHAR nullable | Код подразделения / внешний идентификатор |
| name | JSON | Локализованное наименование узла: `{ "En": "...", "Ru": "...", "Kz": "..." }` |
| sortOrder | INT | Порядок отображения внутри родителя |
| createdAt | TIMESTAMP NOT NULL | |
| createdBy | UUID NOT NULL | Без FK-ограничения |
| updatedAt | TIMESTAMP nullable | |
| updatedBy | UUID nullable | |
| isDeleted | BOOLEAN NOT NULL DEFAULT false | |
| deletedAt | TIMESTAMP nullable | |
| deletedBy | UUID nullable | |

> Правило целостности: `Equipments.organizationalUnitId` должен ссылаться только на запись уровня `Section`.

---

### 2ж. FleetManagePermissions

*(Переименована из FleetDelegations; упрощена структура)*

Кому из конкретных пользователей Fleet Owner предоставлены права управления флотом.

| Поле | Тип | Описание |
|---|---|---|
| id | UUID PK | |
| fleetId | FK → Fleets | |
| userId | FK → Users | Кому предоставлены права |
| createdAt | TIMESTAMP NOT NULL | Дата предоставления |
| createdBy | UUID NOT NULL | Без FK-ограничения |
| expiresAt | TIMESTAMP nullable | NULL = бессрочно |
| updatedAt | TIMESTAMP nullable | |
| updatedBy | UUID nullable | |
| isDeleted | BOOLEAN NOT NULL DEFAULT false | |
| deletedAt | TIMESTAMP nullable | |
| deletedBy | UUID nullable | |

---

### 3. Equipments

Единица техники. Единая таблица для всей техники: TCO Owned, Long-term rented, On-demand BP.

| Поле | Тип | Описание |
|---|---|---|
| id | UUID PK | |
| equipmentTypeId | FK → EquipmentTypes | |
| fleetId | FK → Fleets | |
| ownershipType | ENUM | Owned / LongTermRented / OnDemand |
| tcoId | VARCHAR UNIQUE | ТШО-номер (обязателен для TCO) |
| e1Id | VARCHAR nullable | Идентификатор техники во внешней системе JDE E1 |
| stateNumber | VARCHAR nullable | Госномер |
| vin | VARCHAR nullable | VIN-код техники |
| serialNumber | VARCHAR nullable | |
| description | TEXT nullable | |
| equipmentBrandId | FK → EquipmentBrandModels | Производитель техники; ссылка на уровень `Brand` |
| equipmentModelId | FK → EquipmentBrandModels | Модель техники; ссылка на уровень `Model` |
| shareType | ENUM | Shared / SharedWithConditions / Assigned |
| isCritical | BOOLEAN | Требует охраны (service de sécurité) при транспортировке |
| yearOfManufacture | INT nullable | Год выпуска |
| plannedEngineHoursPerDay | DECIMAL nullable | Плановые мото-часы в сутки; задаются Fleet Owner |
| plannedMileagePerDay | DECIMAL nullable | Плановый километраж в сутки; задаётся Fleet Owner |
| serviceZoneId | FK → ServiceZones | Сервисная зона |
| costCenterId | FK → CostCenters | Финансовый ЦЗ |
| baseLocationId | FK → Locations | Базовое местоположение |
| organizationalUnitId | FK → OrganizationalUnits | Подразделение техники; ссылка только на уровень `Section` |
| createdAt | TIMESTAMP NOT NULL | |
| createdBy | UUID NOT NULL | Без FK-ограничения |
| updatedAt | TIMESTAMP nullable | |
| updatedBy | UUID nullable | |
| isDeleted | BOOLEAN NOT NULL DEFAULT false | |
| deletedAt | TIMESTAMP nullable | |
| deletedBy | UUID nullable | |

> **Актуальный статус** (не хранится, вычисляется из EquipmentStatuses при запросе):
> - **Decommissioned** → активная запись со `status = Decommissioned` (без `cancelledAt`)
> - **InRepair** → активная запись со `status = InRepair` и `actualEndsAt IS NULL`
> - **Frozen** → активная запись со `status = Frozen`, `startsAt ≤ today AND (endsAt IS NULL OR endsAt ≥ today) AND cancelledAt IS NULL`
> - **Available** → нет активных записей ни одного из перечисленных типов

---

### 4. EquipmentPhotos

| Поле | Тип | Описание |
|---|---|---|
| id | UUID PK | |
| equipmentId | FK → Equipments | |
| url | VARCHAR | |
| sortOrder | INT | |
| isPrimary | BOOLEAN | Главная фотография |
| createdAt | TIMESTAMP NOT NULL | |
| createdBy | UUID NOT NULL | Без FK-ограничения |
| updatedAt | TIMESTAMP nullable | |
| updatedBy | UUID nullable | |
| isDeleted | BOOLEAN NOT NULL DEFAULT false | |
| deletedAt | TIMESTAMP nullable | |
| deletedBy | UUID nullable | |

---

### 5. EquipmentStatuses

*(Новая таблица; объединяет EquipmentFreezes, EquipmentRepairHistory и decommission-состояние)*

Полная история состояний техники. Активная запись определяет текущий статус.

| Поле | Тип | Описание |
|---|---|---|
| id | UUID PK | |
| equipmentId | FK → Equipments | |
| status | ENUM | Frozen / InRepair / Decommissioned |
| reason | TEXT nullable | Причина / описание |
| startsAt | DATE NOT NULL | Дата начала состояния |
| endsAt | DATE nullable | Плановая/ожидаемая дата окончания; NULL = бессрочно |
| actualEndsAt | DATE nullable | Фактическая дата окончания (для InRepair) |
| cancelledAt | TIMESTAMP nullable | Дата отмены (для Frozen: досрочная отмена заморозки) |
| source | ENUM nullable | Manual / JDE (актуально для InRepair) |
| createdAt | TIMESTAMP NOT NULL | |
| createdBy | UUID NOT NULL | Без FK-ограничения |
| updatedAt | TIMESTAMP nullable | |
| updatedBy | UUID nullable | |
| isDeleted | BOOLEAN NOT NULL DEFAULT false | |
| deletedAt | TIMESTAMP nullable | |
| deletedBy | UUID nullable | |

> **Правило активной записи:**
> - **Frozen**: `startsAt ≤ today AND (endsAt IS NULL OR endsAt ≥ today) AND cancelledAt IS NULL`
> - **InRepair**: `actualEndsAt IS NULL`
> - **Decommissioned**: `cancelledAt IS NULL`

**Индексы:** `equipmentId`, составной `(equipmentId, status)`

---

## EAV-блок: динамические свойства техники

### 6. MeasurementUnits

| Поле | Тип | Описание |
|---|---|---|
| id | UUID PK | |
| code | VARCHAR | м / кг / кВт / л / бар / м³... |
| displayName | VARCHAR | |
| sortOrder | INT | |
| createdAt | TIMESTAMP NOT NULL | |
| createdBy | UUID NOT NULL | Без FK-ограничения |
| updatedAt | TIMESTAMP nullable | |
| updatedBy | UUID nullable | |
| isDeleted | BOOLEAN NOT NULL DEFAULT false | |
| deletedAt | TIMESTAMP nullable | |
| deletedBy | UUID nullable | |

---

### 7. Properties

| Поле | Тип | Описание |
|---|---|---|
| id | UUID PK | |
| name | JSON | Локализованное наименование свойства: `{ "En": "...", "Ru": "...", "Kz": "..." }` |
| dataType | ENUM | number / double / text / boolean / enum |
| unitId | FK → MeasurementUnits nullable | Только для number/double |
| createdAt | TIMESTAMP NOT NULL | |
| createdBy | UUID NOT NULL | Без FK-ограничения |
| updatedAt | TIMESTAMP nullable | |
| updatedBy | UUID nullable | |
| isDeleted | BOOLEAN NOT NULL DEFAULT false | |
| deletedAt | TIMESTAMP nullable | |
| deletedBy | UUID nullable | |

---

### 8. PropertyEnumValues

| Поле | Тип | Описание |
|---|---|---|
| id | UUID PK | |
| propertyId | FK → Properties | |
| value | VARCHAR | |
| sortOrder | INT | |
| createdAt | TIMESTAMP NOT NULL | |
| createdBy | UUID NOT NULL | Без FK-ограничения |
| updatedAt | TIMESTAMP nullable | |
| updatedBy | UUID nullable | |
| isDeleted | BOOLEAN NOT NULL DEFAULT false | |
| deletedAt | TIMESTAMP nullable | |
| deletedBy | UUID nullable | |

---

### 9. EquipmentTypeProperties

| Поле | Тип | Описание |
|---|---|---|
| id | UUID PK | |
| equipmentTypeId | FK → EquipmentTypes | |
| propertyId | FK → Properties | |
| isRequired | BOOLEAN | |
| isFilterable | BOOLEAN | |
| isVisibleInCard | BOOLEAN | |
| sortOrder | INT | |
| UNIQUE | — | (equipmentTypeId, propertyId) |
| createdAt | TIMESTAMP NOT NULL | |
| createdBy | UUID NOT NULL | Без FK-ограничения |
| updatedAt | TIMESTAMP nullable | |
| updatedBy | UUID nullable | |
| isDeleted | BOOLEAN NOT NULL DEFAULT false | |
| deletedAt | TIMESTAMP nullable | |
| deletedBy | UUID nullable | |

---

### 10. EquipmentProperties

| Поле | Тип | Описание |
|---|---|---|
| id | UUID PK | |
| equipmentId | FK → Equipments | |
| propertyId | FK → Properties | |
| value | JSONB | `{"Bool": null, "Double": 6.7, "Number": null, "String": null}` |
| UNIQUE | — | (equipmentId, propertyId) |
| createdAt | TIMESTAMP NOT NULL | |
| createdBy | UUID NOT NULL | Без FK-ограничения |
| updatedAt | TIMESTAMP nullable | |
| updatedBy | UUID nullable | |
| isDeleted | BOOLEAN NOT NULL DEFAULT false | |
| deletedAt | TIMESTAMP nullable | |
| deletedBy | UUID nullable | |

---

### 11. MaintenancePartners

| Поле | Тип | Описание |
|---|---|---|
| id | UUID PK | |
| name | JSON | Локализованное наименование партнёра ТО: `{ "En": "...", "Ru": "...", "Kz": "..." }` |
| phoneNumber | VARCHAR nullable | Контактный телефон |
| email | VARCHAR nullable | Контактный email |
| address | VARCHAR nullable | Адрес |
| createdAt | TIMESTAMP NOT NULL | |
| createdBy | UUID NOT NULL | Без FK-ограничения |
| updatedAt | TIMESTAMP nullable | |
| updatedBy | UUID nullable | |
| isDeleted | BOOLEAN NOT NULL DEFAULT false | |
| deletedAt | TIMESTAMP nullable | |
| deletedBy | UUID nullable | |

---

### 12. EquipmentMaintenanceContracts

| Поле | Тип | Описание |
|---|---|---|
| id | UUID PK | |
| equipmentId | FK → Equipments | |
| partnerId | FK → MaintenancePartners | |
| serviceType | VARCHAR | Направление: замена масла / колёса / электрика... |
| notes | TEXT nullable | |
| UNIQUE | — | (equipmentId, partnerId, serviceType) |
| createdAt | TIMESTAMP NOT NULL | |
| createdBy | UUID NOT NULL | Без FK-ограничения |
| updatedAt | TIMESTAMP nullable | |
| updatedBy | UUID nullable | |
| isDeleted | BOOLEAN NOT NULL DEFAULT false | |
| deletedAt | TIMESTAMP nullable | |
| deletedBy | UUID nullable | |

---

### 12а. EquipmentFeedbacks

Отзывы заявителей и SWP на технику. Создаётся после того, как бронь достигла статуса Confirmed и выше; требует привязки к конкретной брони.

| Поле | Тип | Описание |
|---|---|---|
| id | UUID PK | |
| equipmentId | FK → Equipments | |
| bookingId | FK → Bookings | Бронь, в рамках которой оставлен отзыв |
| feedback | TEXT NOT NULL | Текст отзыва |
| createdBy | UUID NOT NULL | ID заявителя/SWP (без FK-ограничения) |
| createdAt | TIMESTAMP NOT NULL | |
| updatedAt | TIMESTAMP nullable | |
| updatedBy | UUID nullable | |
| isDeleted | BOOLEAN NOT NULL DEFAULT false | |
| deletedAt | TIMESTAMP nullable | |
| deletedBy | UUID nullable | |

**Индексы:** `equipmentId`, `bookingId`

---

### 13. SystemSettings

| Поле | Тип | Описание |
|---|---|---|
| key | VARCHAR PK | |
| value | VARCHAR | |
| createdAt | TIMESTAMP NOT NULL | |
| createdBy | UUID nullable | NULL для системных записей |
| updatedAt | TIMESTAMP NOT NULL | |
| updatedBy | UUID nullable | |
| isDeleted | BOOLEAN NOT NULL DEFAULT false | |
| deletedAt | TIMESTAMP nullable | |
| deletedBy | UUID nullable | |

Примеры ключей: `booking.horizonDays`, `booking.maxDurationDays`, `fo.timeoutHours24`, `fo.timeoutHours48`

---

### 14. Users

Хранит 2 типа пользователей:
- `internal` — автодобавление при авторизации
- `external` — ручное добавление

Общие поля для всех пользователей: `id`, `fullName`, `email`, `jobTitle`, `isActive` и аудит-поля.

| Поле | Тип | Описание |
|---|---|---|
| id | UUID PK | |
| badgeNumber | VARCHAR nullable | Только для internal: табельный номер / badge number пользователя |
| fullName | VARCHAR | ФИО пользователя |
| email | VARCHAR | |
| sharedEmail | VARCHAR nullable | Только для internal: общий/групповой email, если используется |
| jobTitle | VARCHAR nullable | Должность; nullable для совместимости обоих типов пользователей |
| department | VARCHAR nullable | Только для internal: подразделение |
| isActive | BOOLEAN NOT NULL DEFAULT true | Активен ли пользователь |
| businessPartnerId | FK → BusinessPartners nullable | Только для external: заполняется для внешних пользователей |
| type | ENUM | internal / external |
| createdAt | TIMESTAMP NOT NULL | |
| createdBy | UUID NOT NULL | Без FK-ограничения |
| updatedAt | TIMESTAMP nullable | |
| updatedBy | UUID nullable | |
| isDeleted | BOOLEAN NOT NULL DEFAULT false | |
| deletedAt | TIMESTAMP nullable | |
| deletedBy | UUID nullable | |

---

### 15. BusinessPartners

| Поле | Тип | Описание |
|---|---|---|
| id | UUID PK | |
| name | JSON | Локализованное наименование бизнес-партнёра: `{ "En": "...", "Ru": "...", "Kz": "..." }` |
| description | TEXT nullable | Описание / примечание |
| bin | VARCHAR | БИН / регистрационный номер |
| country | VARCHAR nullable | Страна |
| city | VARCHAR nullable | Город |
| address | VARCHAR nullable | Адрес |
| email | VARCHAR nullable | Контактный email |
| phoneNumber | VARCHAR nullable | Контактный телефон |
| externalId | VARCHAR nullable | Внешний ID из мастер-системы |
| isActive | BOOLEAN NOT NULL DEFAULT true | Активен ли бизнес-партнёр |
| createdAt | TIMESTAMP NOT NULL | |
| createdBy | UUID NOT NULL | Без FK-ограничения |
| updatedAt | TIMESTAMP nullable | |
| updatedBy | UUID nullable | |
| isDeleted | BOOLEAN NOT NULL DEFAULT false | |
| deletedAt | TIMESTAMP nullable | |
| deletedBy | UUID nullable | |

> Для `Users`: чтобы избежать конфликтов между `internal` и `external`, поля `badgeNumber`, `sharedEmail`, `jobTitle`, `department`, `businessPartnerId` допускают `NULL`.

---

## Блок Booking

### 1. BookingRequests

Заявка на бронирование. Один Work Order — одна заявка.

| Поле | Тип | Описание |
|---|---|---|
| id | UUID PK | |
| requestNumber | SERIAL UNIQUE NOT NULL | Порядковый номер; отображается как REQ-YYYY-NNNNN |
| type | ENUM NOT NULL | Regular / ServiceWork |
| status | ENUM NOT NULL | Draft / Submitted / InProgress / Completed / Cancelled — денормализованный кэш |
| workOrderJdeId | VARCHAR nullable | Номер WO из JDE E1 |
| location | VARCHAR nullable | Локация; обязательна для SCM Logistics вместо WO |
| workDescription | VARCHAR NOT NULL | Описание работ |
| comments | TEXT nullable | |
| priority | ENUM NOT NULL | P1 / P2 / P3 / P4 |
| createdBy | UUID NOT NULL | ID заявителя (без FK-ограничения) |
| createdAt | TIMESTAMP NOT NULL | |
| updatedAt | TIMESTAMP nullable | |
| updatedBy | UUID nullable | |
| isDeleted | BOOLEAN NOT NULL DEFAULT false | |
| deletedAt | TIMESTAMP nullable | |
| deletedBy | UUID nullable | |

> **Жизненный цикл:** Draft → Submitted → InProgress → Completed; Cancelled — из Draft

**Индексы:** `createdBy`, `status`, `requestNumber`; partial на `workOrderJdeId WHERE workOrderJdeId IS NOT NULL`

---

### 2. Bookings

Атомарная единица бронирования: одна конкретная единица техники в рамках одной заявки.

| Поле | Тип | Описание |
|---|---|---|
| id | UUID PK | |
| requestId | UUID NOT NULL FK → BookingRequests | |
| equipmentId | UUID NOT NULL | ID техники (без FK-ограничения) |
| fleetId | UUID NOT NULL | ID парка (без FK-ограничения) |
| status | ENUM NOT NULL | Draft / Submitted / Confirmed / TransportConfirmed / Declined / Revoked / Terminated / InProgress / Closed / EquipmentChanged / Extended — денормализованный кэш |
| startDt | TIMESTAMP NOT NULL | Начало бронирования |
| endDt | TIMESTAMP NOT NULL | Окончание бронирования |
| actualStartDt | TIMESTAMP nullable | Фактическая дата начала; вводится при закрытии брони |
| actualEndDt | TIMESTAMP nullable | Фактическая дата окончания; вводится при закрытии брони |
| justification | TEXT nullable | Обоснование; обязательно для Assigned и SharedWithConditions |
| transportBookingId | UUID nullable FK → Bookings | Self-ref: ID транспортной брони; только для unwheeled |
| declineReason | TEXT nullable | Причина отклонения |
| terminateReason | TEXT nullable | Причина досрочного завершения |
| createdAt | TIMESTAMP NOT NULL | |
| createdBy | UUID NOT NULL | Без FK-ограничения |
| updatedAt | TIMESTAMP nullable | |
| updatedBy | UUID nullable | |
| isDeleted | BOOLEAN NOT NULL DEFAULT false | |
| deletedAt | TIMESTAMP nullable | |
| deletedBy | UUID nullable | |

> **Жизненный цикл:**
> - Обычная техника: `Draft → Submitted → Confirmed → InProgress → Closed`
> - Unwheeled: `Draft → Submitted → Confirmed → TransportConfirmed → InProgress → Closed`
> - Общие ветки: Declined, Revoked, Terminated, EquipmentChanged, Extended

**Индексы:** `requestId`, `equipmentId`, `fleetId`, `status`, `transportBookingId`; составной `(equipmentId, startDt, endDt)`

---

### 3. BookingStatuses

*(Переименована из BookingStatusHistories)*

Фиксирует каждое состояние брони. Запись создаётся при каждом изменении `Bookings.status`. Таблица **append-only**: записи не обновляются и не удаляются в штатном режиме.

| Поле | Тип | Описание |
|---|---|---|
| id | UUID PK | |
| bookingId | UUID NOT NULL FK → Bookings | |
| status | ENUM NOT NULL | Текущий статус брони в момент записи |
| changedBy | UUID NOT NULL | ID пользователя-инициатора (без FK-ограничения) |
| changedAt | TIMESTAMP NOT NULL | Момент перехода |
| comment | TEXT nullable | Причина или контекст |
| createdAt | TIMESTAMP NOT NULL | Технически ≡ changedAt |
| createdBy | UUID NOT NULL | Технически ≡ changedBy |
| updatedAt | TIMESTAMP nullable | NULL — таблица append-only |
| updatedBy | UUID nullable | NULL — таблица append-only |
| isDeleted | BOOLEAN NOT NULL DEFAULT false | |
| deletedAt | TIMESTAMP nullable | |
| deletedBy | UUID nullable | |

**Индексы:** `bookingId`, `changedAt`

---

### 4. BookingRequestStatuses

*(Переименована из RequestStatusHistories)*

Фиксирует каждое состояние заявки. Таблица **append-only**.

| Поле | Тип | Описание |
|---|---|---|
| id | UUID PK | |
| requestId | UUID NOT NULL FK → BookingRequests | |
| status | ENUM NOT NULL | Текущий статус заявки в момент записи |
| changedBy | UUID NOT NULL | Без FK-ограничения |
| changedAt | TIMESTAMP NOT NULL | |
| comment | TEXT nullable | |
| createdAt | TIMESTAMP NOT NULL | Технически ≡ changedAt |
| createdBy | UUID NOT NULL | Технически ≡ changedBy |
| updatedAt | TIMESTAMP nullable | NULL — таблица append-only |
| updatedBy | UUID nullable | NULL — таблица append-only |
| isDeleted | BOOLEAN NOT NULL DEFAULT false | |
| deletedAt | TIMESTAMP nullable | |
| deletedBy | UUID nullable | |

**Индексы:** `requestId`, `changedAt`

---

## Сводный список таблиц

### Equipment-блок

| # | Таблица | Назначение |
|---|---|---|
| 1 | EquipmentTypes | Классификатор типов техники |
| 2 | Fleets | Парки техники |
| 2а | WorkCenters | Справочник производственных центров |
| 2б | EquipmentBrandModels | Иерархический справочник производителей и моделей техники |
| 2в | Locations | Справочник базовых локаций техники |
| 2г | CostCenters | Справочник финансовых ЦЗ |
| 2д | ServiceZones | Справочник сервисных зон |
| 2е | OrganizationalUnits | Иерархический справочник подразделений |
| 2ж | FleetManagePermissions | Права управления флотом |
| 3 | Equipments | Единица техники (TCO + BP unified) |
| 4 | EquipmentPhotos | Фотографии техники |
| 5 | EquipmentStatuses | История состояний: Frozen / InRepair / Decommissioned |
| 6 | MeasurementUnits | Справочник единиц измерения |
| 7 | Properties | Справочник свойств техники |
| 8 | PropertyEnumValues | Допустимые значения enum-свойств |
| 9 | EquipmentTypeProperties | Схема свойств по типу (Admin Panel) |
| 10 | EquipmentProperties | Значения свойств конкретной техники |
| 11 | MaintenancePartners | Справочник ДП по ТО |
| 12 | EquipmentMaintenanceContracts | Связь техники и партнёра ТО |
| 12а | EquipmentFeedbacks | Отзывы заявителей/SWP (FR-022) |
| 13 | SystemSettings | Глобальные настройки Admin Panel |
| 14 | Users | Справочник пользователей (TCO + BP) |
| 15 | BusinessPartners | Справочник бизнес-партнёров |

### Booking-блок

| # | Таблица | Назначение |
|---|---|---|
| 1 | BookingRequests | Заявка на бронирование |
| 2 | Bookings | Атомарная единица бронирования |
| 3 | BookingStatuses | История состояний брони |
| 4 | BookingRequestStatuses | История состояний заявки |

**Итого: 26 таблиц**

---

## Связи между блоками

| Поле Booking-блока | Ссылается на | Тип связи |
|---|---|---|
| BookingRequests.createdBy | Users.id | Внешний UUID, без FK-ограничения |
| Bookings.equipmentId | Equipments.id | Внешний UUID, без FK-ограничения |
| Bookings.fleetId | Fleets.id | Внешний UUID, без FK-ограничения |
| Bookings.transportBookingId | Bookings.id | Self-referencing FK; nullable |
| BookingStatuses.changedBy | Users.id | Внешний UUID, без FK-ограничения |
| BookingRequestStatuses.changedBy | Users.id | Внешний UUID, без FK-ограничения |
| EquipmentFeedbacks.bookingId | Bookings.id | FK с ограничением |
| EquipmentFeedbacks.createdBy | Users.id | Внешний UUID, без FK-ограничения |
| Users.businessPartnerId | BusinessPartners.id | FK; nullable, только для external users |

---

## Закрытые открытые вопросы

| # | Вопрос | Решение |
|---|---|---|
| OQ-DB-1 | История изменений shareType у техники? | Не нужна |
| OQ-DB-2 | Атрибут подразделения пользователя | Закрыт в v7: используется поле `Users.department` |
| OQ-DB-3 | Аудит-поля | Закрыт в v8; актуализировано в v9: аудит-поля используются во всех 26 таблицах (createdAt, createdBy, updatedAt, updatedBy, isDeleted, deletedAt, deletedBy) |
| OQ-DB-4 | RequestStatusHistory нужна? | Да — добавлена |
| OQ-DB-5 | BookingSnapshot | CLOSED: previousSnapshot убран в v5 |
| OQ-DB-8 | Продление брони | CLOSED: статус Extended |
| OQ-DB-9 | Unwheeled-флоу | CLOSED: transportBookingId + статус TransportConfirmed |
| OQ-DB-10 | Замена техники FO | CLOSED: статус EquipmentChanged |
| OQ-DB-11 | Скоуп транспортной брони | CLOSED: тот же requestId |

## Открытые вопросы

| # | Вопрос | Impact |
|---|---|---|
| OQ-DB-12 | **Атрибуция FO при запросе транспорта у коллег**: нужно ли поле `transportRequestedFleetOwnerId` в транспортной брони? | Модель уведомлений / маршрутизация |
