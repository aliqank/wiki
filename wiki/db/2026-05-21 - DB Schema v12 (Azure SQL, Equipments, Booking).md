# DB Schema v12: Azure SQL adaptation for Equipments + Booking

**Created:** 2026-05-21  
**Last updated:** 2026-05-21  
**Автор документов:** Telman Nurzhanov (SA)

---

## Changelog v12

| # | Изменение | Комментарий |
|---|---|---|
| 1 | Версия схемы повышена до `v12` | `v12` становится новой актуальной версией схемы |
| 2 | `Fleets.userId` удалён | Владение fleet больше не хранится в записи флота |
| 3 | `ref_fleet_type` сокращён до `Internal`, `External` | Убран избыточный набор внутренних подтипов |
| 4 | `FleetManagePermissions` расширена до общей таблицы владения и делегирования | Таблица теперь хранит не только delegated access, но и ownership assignments |
| 5 | Добавлен lookup `ref_fleet_manage_permission_type` | Значения: `Owner`, `Delegated` |
| 6 | В `FleetManagePermissions` добавлен `permissionTypeId` | Позволяет различать owner vs delegated assignment |
| 7 | `JdeWorkOrders` удалена | JDE Work Order больше не хранится отдельной локальной таблицей схемы |
| 8 | `JdeWorkOrderSteps` удалена | JDE step-level сущность исключена из схемы |
| 9 | `BookingRequests.jdeWorkOrderRefId` удалён | На request-level остаётся только `workOrderNumber` как business field |
| 10 | `Bookings.jdeWorkOrderStepRefId` удалён | На item-level остаётся только `workCenterId` |
| 11 | `isActive` удалён из `ref_*` таблиц | Reference tables упрощены до статического справочного состава |
| 12 | `Users.sharedEmail` удалён | Поле исключено из user model |
| 13 | `changedBy`, `changedAt`, `isDeleted`, `deletedAt`, `deletedBy` удалены из `BookingStatuses` и `BookingRequestStatuses` | History tables упрощены |
| 14 | В `Fleets` добавлен `businessPartnerId` | Флот теперь может быть явно привязан к Business Partner |

---

## Влияние изменений v12 на БД

1. Владение и делегирование fleet теперь нормализованы через `FleetManagePermissions`; `Fleets` перестаёт содержать прямую ссылку на одного owner-пользователя.
2. Все backend-запросы, которые раньше искали владельца через `Fleets.userId`, должны перейти на выборку owner-records из `FleetManagePermissions` с `permissionTypeId = Owner`.
3. `Fleets.businessPartnerId` позволяет явно ограничивать внешний fleet контур конкретным Business Partner и упрощает DB-level проверки принадлежности.
4. Все backend-запросы, которые раньше использовали `jdeWorkOrderRefId` и `jdeWorkOrderStepRefId`, должны перейти на business-поля `workOrderNumber` и `workCenterId` без FK на локальные JDE-таблицы.
5. Схема становится менее связанной с локальным хранением JDE-объектов и переносит JDE-контекст на integration/business level, а не на level relational storage.
6. История `v11` сохраняется как предыдущая версия; `v12` фиксирует новый ownership model и удаление локальных JDE reference tables.

---

## Azure SQL conventions

### 1. Типы данных

| Было в v10 | Стало в v12 | Комментарий |
|---|---|---|
| `UUID` | `uniqueidentifier` | PK, FK и внешние идентификаторы ссылочного типа |
| `TIMESTAMP` | `datetime2(3)` | Для `createdAt`, `updatedAt`, `changedAt`, бизнес-дат времени |
| `BOOLEAN` | `bit` | `0 / 1` |
| `TEXT` | `nvarchar(max)` | Unicode-friendly текст |
| `VARCHAR` | `nvarchar(...)` | Для человекочитаемых и интеграционных строк |
| `JSON` | Отдельные колонки | Для локализованных имен и структур с фиксированной формой |
| `JSONB` | Отдельные колонки или `nvarchar(max)` | В зависимости от назначения поля |
| `SERIAL` | `int IDENTITY(1,1)` | Для `requestNumber` |

### 2. Аудит-поля

Стандартный набор аудит-полей для большинства таблиц:

| Поле | Тип |
|---|---|
| `createdAt` | `datetime2(3) not null` |
| `createdBy` | `uniqueidentifier not null` |
| `updatedAt` | `datetime2(3) null` |
| `updatedBy` | `uniqueidentifier null` |
| `isDeleted` | `bit not null default 0` |
| `deletedAt` | `datetime2(3) null` |
| `deletedBy` | `uniqueidentifier null` |

Для append-only history tables:
- `updatedAt`, `updatedBy` всегда `NULL`

### 2а. Temporal tables

Для Azure SQL рекомендуется использовать **system-versioned temporal tables** для большинства изменяемых business/master-data таблиц.

Рекомендация v12:

| Категория | Таблицы | Решение |
|---|---|---|
| Temporal tables включить | `EquipmentTypes`, `Fleets`, `WorkCenters`, `EquipmentBrands`, `EquipmentModels`, `Locations`, `CostCenters`, `ServiceZones`, `Divisions`, `Groups`, `Departments`, `Sections`, `FleetManagePermissions`, `Equipments`, `EquipmentPhotos`, `MeasurementUnits`, `Properties`, `PropertyEnumValues`, `EquipmentTypeProperties`, `EquipmentProperties`, `MaintenancePartners`, `EquipmentMaintenanceContracts`, `EquipmentFeedbacks`, `EquipmentBookingAuthorizations`, `SystemSettings`, `Users`, `BusinessPartners`, `BookingRequests`, `Bookings` | Включить temporal history |
| Temporal tables не использовать | `BookingStatuses`, `BookingRequestStatuses`, `EquipmentStatuses` | Оставить явные append-only history/event tables |

Причина исключений:
- `BookingStatuses`, `BookingRequestStatuses`, `EquipmentStatuses` уже являются доменными history/event таблицами
- для них temporal не даёт дополнительной ценности и только усложняет модель

Стандарт для temporal tables:
- добавить `ValidFrom datetime2(3) generated always as row start`
- добавить `ValidTo datetime2(3) generated always as row end`
- добавить `period for system_time (ValidFrom, ValidTo)`
- включить `system_versioning = on`

Пример:

```sql
alter table dbo.Bookings add
    ValidFrom datetime2(3) generated always as row start not null,
    ValidTo   datetime2(3) generated always as row end   not null,
    period for system_time (ValidFrom, ValidTo);

alter table dbo.Bookings
set (system_versioning = on (history_table = dbo.BookingsHistory));
```

### 3. Локализация

Во всех таблицах, где в v10 использовался `name JSON`, в v12 используются:

| Поле | Тип | Комментарий |
|---|---|---|
| `nameEn` | `nvarchar(255) not null` | Базовое обязательное отображаемое имя |
| `nameRu` | `nvarchar(255) null` | |
| `nameKz` | `nvarchar(255) null` | |

### 4. Уникальные индексы

Так как модель использует soft delete, уникальность должна задаваться как **filtered unique index**.

Примеры:

```sql
create unique index UX_Equipments_tcoId_Active
on dbo.Equipments (tcoId)
where isDeleted = 0 and tcoId is not null;

create unique index UX_EquipmentBookingAuthorizations_Active
on dbo.EquipmentBookingAuthorizations (equipmentId, userId)
where isDeleted = 0;
```

---

## Reference tables instead of enums

Вместо enum-полей вводятся lookup/reference tables:

| Таблица | Значения |
|---|---|
| `ref_fleet_type` | Internal, External |
| `ref_fleet_manage_permission_type` | Owner, Delegated |
| `ref_equipment_mobility_type` | SelfPropelled, NonSelfPropelledMotorized, Stationary, NonMotorized |
| `ref_equipment_class` | HDE, HDV |
| `ref_ownership_type` | TcoOwned, LongTermRented, OnDemand |
| `ref_share_type` | Shared, SharedWithConditions, Assigned |
| `ref_equipment_status_type` | Frozen, InRepair, Decommissioned |
| `ref_equipment_current_status` | Available, Frozen, InRepair, Decommissioned |
| `ref_equipment_status_source` | Manual, JDE |
| `ref_property_data_type` | Int, Decimal, String, Bit, Enum |
| `ref_user_type` | Internal, External |
| `ref_request_type` | Regular, ServiceWork |
| `ref_request_priority` | P1, P2, P3, P4 |
| `ref_booking_request_status` | Draft, Submitted, InProgress, Closed, Cancelled |
| `ref_booking_status` | Draft, Cancelled, Submitted, ConfirmedByFo, Confirmed, TransportConfirmed, Declined, Revoked, Terminated, InProgress, Closed, EquipmentChanged, Extended |

Минимальный шаблон reference table:

| Поле | Тип |
|---|---|
| `id` | `uniqueidentifier PK` |
| `code` | `nvarchar(100) not null` |
| `nameEn` | `nvarchar(255) not null` |
| `nameRu` | `nvarchar(255) null` |
| `nameKz` | `nvarchar(255) null` |
| `iconUrl` | `nvarchar(1000) null` |
| `sortOrder` | `int not null default 0` |

Индекс:
- unique index on `code`

---

## Блок Equipment

### 1. EquipmentTypes

Назначение: справочник типов техники. Определяет классификацию единиц техники, их mobility type, связь с Work Center и базовые поведенческие признаки типа.

| Поле | Тип | Описание |
|---|---|---|
| `id` | `uniqueidentifier PK` | |
| `nameEn` | `nvarchar(255) not null` | |
| `nameRu` | `nvarchar(255) null` | |
| `nameKz` | `nvarchar(255) null` | |
| `iconUrl` | `nvarchar(1000) null` | URL иконки типа техники |
| `mobilityTypeId` | `uniqueidentifier FK -> ref_equipment_mobility_type` | |
| `requiresTransport` | `bit not null` | Требуется транспортировка |
| `equipmentClassId` | `uniqueidentifier FK -> ref_equipment_class` | |
| `workCenterId` | `uniqueidentifier FK -> WorkCenters` | Производственный центр |
| `sortOrder` | `int not null` | |
| audit fields | см. conventions | |

### 2. Fleets

Назначение: справочник флотов. Используется для группировки техники и маршрутизации согласования. Владение флотом определяется не через поле в `Fleets`, а через `FleetManagePermissions`.

| Поле | Тип | Описание |
|---|---|---|
| `id` | `uniqueidentifier PK` | |
| `nameEn` | `nvarchar(255) not null` | |
| `nameRu` | `nvarchar(255) null` | |
| `nameKz` | `nvarchar(255) null` | |
| `fleetTypeId` | `uniqueidentifier FK -> ref_fleet_type` | |
| `businessPartnerId` | `uniqueidentifier null FK -> BusinessPartners` | Для внешних fleet-ов задаёт принадлежность к BP-контуру |
| `aadGroupId` | `nvarchar(255)` | ID AAD-группы Fleet Owner |
| audit fields | см. conventions | |

### 3. WorkCenters

Назначение: справочник рабочих центров. Используется при сопоставлении типов техники и booking-контекста.

| Поле | Тип |
|---|---|
| `id` | `uniqueidentifier PK` |
| `code` | `nvarchar(100) not null` |
| `nameEn` | `nvarchar(255) not null` |
| `nameRu` | `nvarchar(255) null` |
| `nameKz` | `nvarchar(255) null` |
| audit fields | см. conventions |

Filtered unique indexes:
- `code where isDeleted = 0`

### 4. EquipmentBrands

Назначение: справочник брендов/производителей техники. Нужен для нормализованного хранения производителя в карточке техники.

| Поле | Тип |
|---|---|
| `id` | `uniqueidentifier PK` |
| `nameEn` | `nvarchar(255) not null` |
| `nameRu` | `nvarchar(255) null` |
| `nameKz` | `nvarchar(255) null` |
| `sortOrder` | `int not null` |
| audit fields | см. conventions |

### 5. EquipmentModels

Назначение: справочник моделей техники. Позволяет хранить модель отдельно от бренда и переиспользовать её в карточках техники.

| Поле | Тип |
|---|---|
| `id` | `uniqueidentifier PK` |
| `brandId` | `uniqueidentifier FK -> EquipmentBrands` |
| `nameEn` | `nvarchar(255) not null` |
| `nameRu` | `nvarchar(255) null` |
| `nameKz` | `nvarchar(255) null` |
| `sortOrder` | `int not null` |
| audit fields | см. conventions |

Filtered unique indexes:
- `(brandId, nameRu) where isDeleted = 0`

### 6. Locations / CostCenters / ServiceZones / Divisions / Groups / Departments / Sections

Ниже таблицы расписаны отдельно, так как они относятся к разным типам справочников:
- `Locations`, `CostCenters`, `ServiceZones` - плоские справочники без иерархии;
- `Divisions`, `Groups`, `Departments`, `Sections` - оргструктура с явными отдельными FK между уровнями.

#### 6.1 Locations

Назначение: справочник базовых локаций техники. Показывает, где техника базируется или откуда обычно предоставляется.

| Поле | Тип | Описание |
|---|---|---|
| `id` | `uniqueidentifier PK` | |
| `code` | `nvarchar(100) not null` | Код локации / внешний идентификатор |
| `nameEn` | `nvarchar(255) not null` | |
| `nameRu` | `nvarchar(255) null` | |
| `nameKz` | `nvarchar(255) null` | |
| `sortOrder` | `int not null default 0` | Порядок отображения в UI |
| audit fields | см. conventions | |

Filtered unique indexes:
- `code where isDeleted = 0`

#### 6.2 CostCenters

Назначение: справочник cost centers / финансовых ЦЗ. Используется для организационной и финансовой привязки техники.

| Поле | Тип | Описание |
|---|---|---|
| `id` | `uniqueidentifier PK` | |
| `code` | `nvarchar(100) not null` | Код cost center из JDE |
| `nameEn` | `nvarchar(255) not null` | |
| `nameRu` | `nvarchar(255) null` | |
| `nameKz` | `nvarchar(255) null` | |
| `sortOrder` | `int not null default 0` | Порядок отображения в UI |
| audit fields | см. conventions | |

Filtered unique indexes:
- `code where isDeleted = 0`

#### 6.3 ServiceZones

Назначение: справочник сервисных зон. Нужен для логистики, обслуживания и фильтрации техники по зоне ответственности.

| Поле | Тип | Описание |
|---|---|---|
| `id` | `uniqueidentifier PK` | |
| `code` | `nvarchar(100) not null` | Код сервисной зоны |
| `nameEn` | `nvarchar(255) not null` | |
| `nameRu` | `nvarchar(255) null` | |
| `nameKz` | `nvarchar(255) null` | |
| `sortOrder` | `int not null default 0` | Порядок отображения в UI |
| audit fields | см. conventions | |

Filtered unique indexes:
- `code where isDeleted = 0`

#### 6.4 Divisions

Назначение: верхний уровень организационной иерархии, используемой в привязке техники и пользователей.

| Поле | Тип | Описание |
|---|---|---|
| `id` | `uniqueidentifier PK` | |
| `code` | `nvarchar(100) null` | Код дивизиона / внешний идентификатор |
| `nameEn` | `nvarchar(255) not null` | |
| `nameRu` | `nvarchar(255) null` | |
| `nameKz` | `nvarchar(255) null` | |
| `sortOrder` | `int not null default 0` | Порядок отображения в UI |
| audit fields | см. conventions | |

Filtered unique indexes:
- `code where isDeleted = 0 and code is not null`

#### 6.5 Groups

Назначение: промежуточный уровень организационной иерархии между `Divisions` и `Departments`.

| Поле | Тип | Описание |
|---|---|---|
| `id` | `uniqueidentifier PK` | |
| `divisionId` | `uniqueidentifier FK -> Divisions` | Родительский дивизион |
| `code` | `nvarchar(100) null` | Код группы / внешний идентификатор |
| `nameEn` | `nvarchar(255) not null` | |
| `nameRu` | `nvarchar(255) null` | |
| `nameKz` | `nvarchar(255) null` | |
| `sortOrder` | `int not null default 0` | Порядок отображения в UI |
| audit fields | см. conventions | |

Filtered unique indexes:
- `code where isDeleted = 0 and code is not null`
- при необходимости: составной индекс по `divisionId`

#### 6.6 Departments

Назначение: справочник подразделений. Используется в отчетности, оргструктуре пользователей и привязке техники через `Sections`.

| Поле | Тип | Описание |
|---|---|---|
| `id` | `uniqueidentifier PK` | |
| `groupId` | `uniqueidentifier FK -> Groups` | Родительская группа |
| `code` | `nvarchar(100) null` | Код департамента / внешний идентификатор |
| `nameEn` | `nvarchar(255) not null` | |
| `nameRu` | `nvarchar(255) null` | |
| `nameKz` | `nvarchar(255) null` | |
| `sortOrder` | `int not null default 0` | Порядок отображения в UI |
| audit fields | см. conventions | |

Filtered unique indexes:
- `code where isDeleted = 0 and code is not null`
- при необходимости: составной индекс по `groupId`

#### 6.7 Sections

Назначение: нижний уровень оргструктуры для unit-level привязки техники к конкретной части подразделения.

| Поле | Тип | Описание |
|---|---|---|
| `id` | `uniqueidentifier PK` | |
| `departmentId` | `uniqueidentifier FK -> Departments` | Родительский департамент |
| `code` | `nvarchar(100) null` | Код отдела / unit / внешний идентификатор |
| `nameEn` | `nvarchar(255) not null` | |
| `nameRu` | `nvarchar(255) null` | |
| `nameKz` | `nvarchar(255) null` | |
| `sortOrder` | `int not null default 0` | Порядок отображения в UI |
| audit fields | см. conventions | |

Filtered unique indexes:
- `code where isDeleted = 0 and code is not null`
- при необходимости: составной индекс по `departmentId`

Замечание по parent-связям:
- у `Locations`, `CostCenters`, `ServiceZones` поля `Parent` / `parentId` быть не должно;
- в оргструктуре не нужен единый абстрактный `Parent FK`, потому что связь типизирована по уровням: `Groups.divisionId`, `Departments.groupId`, `Sections.departmentId`;
- nullable FK технически допустим в SQL Server, но для обязательной иерархии в этой модели такие связи должны быть `not null`, кроме случаев, когда бизнес явно допускает "осиротевшую" запись.

### 7. FleetManagePermissions

Назначение: таблица прав управления fleet. Хранит как ownership, так и delegated access для конкретного флота и пользователя.

Бизнес-правила делегирования:
- внутренним владельцам / пользователям можно делегировать только внутреннюю технику и внутренние fleet-сущности;
- внешним владельцам / пользователям можно делегировать только технику и fleet-сущности их собственного Business Partner;
- делегирование между разными Business Partner запрещено;
- проверка этих ограничений должна выполняться на уровне backend-валидации при создании записей в `FleetManagePermissions`.

| Поле | Тип |
|---|---|
| `id` | `uniqueidentifier PK` |
| `fleetId` | `uniqueidentifier FK -> Fleets` |
| `userId` | `uniqueidentifier FK -> Users` |
| `permissionTypeId` | `uniqueidentifier FK -> ref_fleet_manage_permission_type` |
| `expiresAt` | `datetime2(3) null` |
| audit fields | см. conventions |

Замечание:
- `permissionTypeId = Owner` означает, что запись задаёт владельца флота;
- `permissionTypeId = Delegated` означает временный или постоянный delegated access;
- для owner-records `expiresAt` должен быть `NULL`, если бизнес не вводит ограниченный срок владения;
- для внешнего сценария backend использует `Users.businessPartnerId` как обязательный признак принадлежности пользователя к BP-контуру;
- `Fleets.businessPartnerId` рекомендуется использовать для явной связи внешнего флота с конкретным `BusinessPartners`.

### 8. Equipments

Назначение: основная таблица карточек техники. Хранит идентификационные данные, принадлежность, тип, статус, бренд/модель и организационную привязку каждой единицы техники.

| Поле | Тип | Описание |
|---|---|---|
| `id` | `uniqueidentifier PK` | |
| `equipmentTypeId` | `uniqueidentifier FK -> EquipmentTypes` | |
| `fleetId` | `uniqueidentifier FK -> Fleets` | |
| `ownershipTypeId` | `uniqueidentifier FK -> ref_ownership_type` | |
| `currentStatusId` | `uniqueidentifier FK -> ref_equipment_current_status` | Денормализованный текущий статус |
| `tcoId` | `nvarchar(100) null` | ТШО-номер; обязателен для TCO-owned по бизнес-правилу |
| `jdeId` | `nvarchar(100) null` | Внешний ID в JDE E1 |
| `stateNumber` | `nvarchar(100) null` | Госномер |
| `vin` | `nvarchar(100) null` | |
| `serialNumber` | `nvarchar(100) null` | |
| `description` | `nvarchar(max) null` | |
| `brandId` | `uniqueidentifier FK -> EquipmentBrands` | |
| `modelId` | `uniqueidentifier FK -> EquipmentModels` | |
| `shareTypeId` | `uniqueidentifier FK -> ref_share_type` | |
| `isCritical` | `bit not null` | Требует охраны при транспортировке |
| `yearOfManufacture` | `int null` | |
| `plannedMotohourPerDay` | `decimal(18,4) null` | Плановое количество моточасов в день |
| `plannedMileagePerDay` | `decimal(18,4) null` | |
| `serviceZoneId` | `uniqueidentifier FK -> ServiceZones` | |
| `costCenterId` | `uniqueidentifier FK -> CostCenters` | |
| `baseLocationId` | `uniqueidentifier FK -> Locations` | |
| `sectionId` | `uniqueidentifier FK -> Sections` | |
| audit fields | см. conventions | |

Комментарии:
- `currentStatusId` обновляется атомарно при вставке/закрытии записей в `EquipmentStatuses`
- history остаётся source of truth, `currentStatusId` — denormalized cache
- `OnDemand` техника не может участвовать в `Bookings`

Filtered unique indexes:
- `tcoId where isDeleted = 0 and tcoId is not null`
- `jdeId where isDeleted = 0 and jdeId is not null`

### 9. EquipmentPhotos

Назначение: таблица фотографий техники. Позволяет хранить несколько изображений на одну единицу техники с сортировкой и признаком главного фото.

| Поле | Тип |
|---|---|
| `id` | `uniqueidentifier PK` |
| `equipmentId` | `uniqueidentifier FK -> Equipments` |
| `url` | `nvarchar(1000) not null` |
| `sortOrder` | `int not null default 0` |
| `isPrimary` | `bit not null default 0` |
| audit fields | см. conventions |

### 10. EquipmentStatuses

Назначение: история статусов техники. Хранит интервалы заморозки, ремонта, вывода из эксплуатации и других состояний, влияющих на доступность техники.

| Поле | Тип | Описание |
|---|---|---|
| `id` | `uniqueidentifier PK` | |
| `equipmentId` | `uniqueidentifier FK -> Equipments` | |
| `statusTypeId` | `uniqueidentifier FK -> ref_equipment_status_type` | Frozen / InRepair / Decommissioned |
| `reason` | `nvarchar(max) null` | |
| `startsAt` | `date not null` | |
| `endsAt` | `date null` | |
| `actualEndsAt` | `date null` | |
| `cancelledAt` | `datetime2(3) null` | |
| `sourceId` | `uniqueidentifier null FK -> ref_equipment_status_source` | Manual / JDE |
| audit fields | см. conventions | |

Индексы:
- `(equipmentId, statusTypeId)`
- filtered index on active statuses per business rules

Замечание:
- иконка статуса должна храниться в `ref_equipment_status_type.iconUrl`, так как это атрибут типа статуса, а не конкретной исторической записи `EquipmentStatuses`

---

## EAV-блок: динамические свойства техники

### 11. MeasurementUnits

Назначение: справочник единиц измерения для dynamic characteristics. Используется там, где свойство техники имеет числовое значение с единицей измерения.

| Поле | Тип |
|---|---|
| `id` | `uniqueidentifier PK` |
| `code` | `nvarchar(50) not null` |
| `displayName` | `nvarchar(100) not null` |
| `sortOrder` | `int not null default 0` |
| audit fields | см. conventions |

### 12. Properties

Назначение: справочник определений dynamic characteristics. Описывает имя свойства, его тип данных и при необходимости единицу измерения.

| Поле | Тип |
|---|---|
| `id` | `uniqueidentifier PK` |
| `nameEn` | `nvarchar(255) not null` |
| `nameRu` | `nvarchar(255) null` |
| `nameKz` | `nvarchar(255) null` |
| `dataTypeId` | `uniqueidentifier FK -> ref_property_data_type` |
| `unitId` | `uniqueidentifier null FK -> MeasurementUnits` |
| audit fields | см. conventions |

### 13. PropertyEnumValues

Назначение: справочник возможных enum-значений для properties с типом enum.

| Поле | Тип |
|---|---|
| `id` | `uniqueidentifier PK` |
| `propertyId` | `uniqueidentifier FK -> Properties` |
| `value` | `nvarchar(255) not null` |
| `sortOrder` | `int not null default 0` |
| audit fields | см. conventions |

### 14. EquipmentTypeProperties

Назначение: таблица настройки свойств по типу техники. Определяет, какие dynamic characteristics доступны для конкретного типа техники и как они ведут себя в UI.

| Поле | Тип |
|---|---|
| `id` | `uniqueidentifier PK` |
| `equipmentTypeId` | `uniqueidentifier FK -> EquipmentTypes` |
| `propertyId` | `uniqueidentifier FK -> Properties` |
| `isRequired` | `bit not null` |
| `isFilterable` | `bit not null` |
| `isVisibleInCard` | `bit not null` |
| `sortOrder` | `int not null default 0` |
| audit fields | см. conventions |

Filtered unique indexes:
- `(equipmentTypeId, propertyId) where isDeleted = 0`

### 15. EquipmentProperties

`JSONB value` из v10 заменён на типизированные колонки.

Назначение: значения dynamic characteristics для конкретной единицы техники. Таблица хранит фактические значения свойств, заданных через `EquipmentTypeProperties`.

| Поле | Тип | Комментарий |
|---|---|---|
| `id` | `uniqueidentifier PK` | |
| `equipmentId` | `uniqueidentifier FK -> Equipments` | |
| `propertyId` | `uniqueidentifier FK -> Properties` | |
| `valueString` | `nvarchar(max) null` | Для string |
| `valueInt` | `bigint null` | Для integer-like number |
| `valueDecimal` | `decimal(18,4) null` | Для decimal/double |
| `valueBit` | `bit null` | Для boolean |
| `propertyEnumValueId` | `uniqueidentifier null FK -> PropertyEnumValues` | Для enum |
| audit fields | см. conventions | |

Правило:
- заполнено должно быть ровно одно значение из `valueString`, `valueInt`, `valueDecimal`, `valueBit`, `propertyEnumValueId`

Filtered unique indexes:
- `(equipmentId, propertyId) where isDeleted = 0`

### 16. MaintenancePartners / EquipmentMaintenanceContracts

#### 16.1 MaintenancePartners

Назначение: справочник ремонтных и сервисных партнеров, которые обслуживают технику.

`MaintenancePartners`:
- `nameEn`, `nameRu`, `nameKz`
- `phoneNumber nvarchar(100)`
- `email nvarchar(255)`
- `address nvarchar(500)`
- audit fields

#### 16.2 EquipmentMaintenanceContracts

Назначение: связующая таблица между техникой и сервисным партнером. Позволяет хранить, кто и по какому виду сервиса обслуживает конкретную единицу техники.

`EquipmentMaintenanceContracts`:
- `equipmentId uniqueidentifier FK`
- `partnerId uniqueidentifier FK`
- `serviceType nvarchar(255)`
- `notes nvarchar(max) null`
- audit fields

Filtered unique index:
- `(equipmentId, partnerId, serviceType) where isDeleted = 0`

### 17. EquipmentFeedbacks / EquipmentBookingAuthorizations

#### 17.1 EquipmentFeedbacks

Назначение: отзывы по технике, оставленные в контексте конкретной брони. Используется для накопления обратной связи по качеству и удобству эксплуатации техники.

`EquipmentFeedbacks`:
- `feedback nvarchar(max) not null`
- `bookingId uniqueidentifier FK`
- `equipmentId uniqueidentifier FK`

#### 17.2 EquipmentBookingAuthorizations

Назначение: таблица авторизаций пользователей на бронирование Assigned техники. Отделяет общую видимость техники от права её забронировать.

`EquipmentBookingAuthorizations`:
- `equipmentId uniqueidentifier FK`
- `userId uniqueidentifier FK`
- filtered unique index `(equipmentId, userId) where isDeleted = 0`

---

## Служебные таблицы

### 18. SystemSettings

Назначение: системные настройки вида key-value. Используется для параметров, которые должны настраиваться через Admin Panel, а не быть захардкоженными.

| Поле | Тип |
|---|---|
| `key` | `nvarchar(200) PK` |
| `value` | `nvarchar(max) not null` |
| audit fields | см. conventions |

### 19. Users

Назначение: справочник пользователей системы. Содержит идентификационные и организационные данные, необходимые для ролей, прав, согласования и привязки к BP.

| Поле | Тип |
|---|---|
| `id` | `uniqueidentifier PK` |
| `badgeNumber` | `nvarchar(100) null` |
| `fullName` | `nvarchar(255) not null` |
| `email` | `nvarchar(255) not null` |
| `jobTitle` | `nvarchar(255) null` |
| `departmentId` | `uniqueidentifier null FK -> Departments` |
| `isActive` | `bit not null default 1` |
| `businessPartnerId` | `uniqueidentifier null FK -> BusinessPartners` |
| `userTypeId` | `uniqueidentifier FK -> ref_user_type` |
| audit fields | см. conventions |

### 20. BusinessPartners

Назначение: справочник внешних компаний и контрагентов. Используется в сценариях с BP, внешними пользователями и связанными reference-данными.

| Поле | Тип |
|---|---|
| `id` | `uniqueidentifier PK` |
| `nameEn` | `nvarchar(255) not null` |
| `nameRu` | `nvarchar(255) null` |
| `nameKz` | `nvarchar(255) null` |
| `description` | `nvarchar(max) null` |
| `bin` | `nvarchar(100) not null` |
| `country` | `nvarchar(100) null` |
| `city` | `nvarchar(100) null` |
| `address` | `nvarchar(500) null` |
| `email` | `nvarchar(255) null` |
| `phoneNumber` | `nvarchar(100) null` |
| `externalId` | `nvarchar(100) null` |
| `isActive` | `bit not null default 1` |
| audit fields | см. conventions |

---

## Блок Booking

### 21. BookingRequests

Назначение: заголовок заявки на бронирование. Хранит request-level данные: тип заявки, номер, приоритет, инициатора, business-поля заявки и текущий агрегированный статус.

| Поле | Тип | Описание |
|---|---|---|
| `id` | `uniqueidentifier PK` | |
| `requestNumber` | `int IDENTITY(1,1) unique not null` | Номер заявки; форматируется как `REQ-YYYY-NNNNN` |
| `requestTypeId` | `uniqueidentifier FK -> ref_request_type` | |
| `statusId` | `uniqueidentifier FK -> ref_booking_request_status` | Денормализованный текущий статус |
| `workOrderNumber` | `nvarchar(100) null` | |
| `location` | `nvarchar(255) null` | |
| `workDescription` | `nvarchar(1000) not null` | |
| `comments` | `nvarchar(max) null` | |
| `priorityId` | `uniqueidentifier FK -> ref_request_priority` | |
| `createdBy` | `uniqueidentifier not null` | ID заявителя |
| `createdAt` | `datetime2(3) not null` | |
| `updatedAt` | `datetime2(3) null` | |
| `updatedBy` | `uniqueidentifier null` | |
| `isDeleted` | `bit not null default 0` | |
| `deletedAt` | `datetime2(3) null` | |
| `deletedBy` | `uniqueidentifier null` | |

Индексы:
- `createdBy`, `statusId`, `requestNumber`
- filtered index on `workOrderNumber where isDeleted = 0 and workOrderNumber is not null`

### 22. Bookings

Назначение: отдельные booking items внутри заявки. Каждая запись соответствует одной единице техники и проходит собственный жизненный цикл согласования и исполнения.

| Поле | Тип | Описание |
|---|---|---|
| `id` | `uniqueidentifier PK` | |
| `requestId` | `uniqueidentifier FK -> BookingRequests` | |
| `equipmentId` | `uniqueidentifier not null` | Внешняя ссылка на технику |
| `fleetId` | `uniqueidentifier not null` | Внешняя ссылка на парк |
| `workCenterId` | `uniqueidentifier null FK -> WorkCenters` | |
| `statusId` | `uniqueidentifier FK -> ref_booking_status` | Денормализованный текущий статус |
| `plannedStartDateTime` | `datetime2(3) not null` | Плановая дата и время начала брони |
| `plannedEndDateTime` | `datetime2(3) not null` | Плановая дата и время окончания брони |
| `actualStartDateTime` | `datetime2(3) null` | |
| `actualEndDateTime` | `datetime2(3) null` | |
| `justification` | `nvarchar(max) null` | |
| `requiresSupervisorApproval` | `bit not null default 0` | |
| `supervisorApprovedBy` | `uniqueidentifier null` | |
| `supervisorApprovedAt` | `datetime2(3) null` | |
| `supervisorComment` | `nvarchar(max) null` | |
| `transportBookingId` | `uniqueidentifier null FK -> Bookings` | |
| `declineReason` | `nvarchar(max) null` | |
| `terminateReason` | `nvarchar(max) null` | |
| audit fields | см. conventions | |

Правила:
- `ConfirmedByFo` допустим только для `LongTermRented`
- при `LongTermRented` поле `requiresSupervisorApproval = 1`
- `OnDemand` техника не допускается в `Bookings`

Индексы:
- `requestId`, `equipmentId`, `fleetId`, `statusId`, `transportBookingId`
- составной `(equipmentId, plannedStartDateTime, plannedEndDateTime)`

### 23. BookingStatuses

Назначение: история смены статусов individual booking. Нужна как source of truth для аудита жизненного цикла брони.

| Поле | Тип |
|---|---|
| `id` | `uniqueidentifier PK` |
| `bookingId` | `uniqueidentifier FK -> Bookings` |
| `statusId` | `uniqueidentifier FK -> ref_booking_status` |
| `comment` | `nvarchar(max) null` |
| `createdAt` | `datetime2(3) not null` |
| `createdBy` | `uniqueidentifier not null` |
| `updatedAt` | `datetime2(3) null` |
| `updatedBy` | `uniqueidentifier null` |

### 24. BookingRequestStatuses

Назначение: история смены статусов request-level сущности. Используется для аудита агрегированного жизненного цикла заявки.

| Поле | Тип |
|---|---|
| `id` | `uniqueidentifier PK` |
| `requestId` | `uniqueidentifier FK -> BookingRequests` |
| `statusId` | `uniqueidentifier FK -> ref_booking_request_status` |
| `comment` | `nvarchar(max) null` |
| `createdAt` | `datetime2(3) not null` |
| `createdBy` | `uniqueidentifier not null` |
| `updatedAt` | `datetime2(3) null` |
| `updatedBy` | `uniqueidentifier null` |

---

## Итоговая позиция по замечаниям

| Замечание | Решение в v12 |
|---|---|
| Azure SQL не имеет enum/jsonb | Исправлено через `ref_*` и отдельные колонки / `nvarchar(max)` |
| UUID -> uniqueidentifier | Исправлено |
| Timestamp -> datetime2 | Исправлено: `datetime2(3)` как стандарт |
| Boolean -> bit | Исправлено |
| text -> nvarchar(max) | Исправлено |
| enum -> reference tables | Исправлено |
| jsonb -> отдельные колонки | Исправлено для локализации и EAV; для raw payload используется `nvarchar(max)` |
| filtered unique indexes | Добавлены как правило моделирования |
| currentStatus для техники | Добавлен `Equipments.currentStatusId` |
| SERIAL -> IDENTITY / SEQUENCE | Исправлено через `IDENTITY(1,1)` |
| ownership model для fleet | Переведён на `FleetManagePermissions` + `ref_fleet_manage_permission_type` |
| локальные JDE reference tables | Исключены из схемы |

---

## Открытые вопросы для следующей итерации

1. Нужно ли переводить все `createdBy / updatedBy / deletedBy` на полноценные FK к `Users`, или оставить soft-reference без ограничений.
2. Нужен ли `SEQUENCE` вместо `IDENTITY` для `requestNumber`, если появятся дополнительные типы документов с общей нумерацией.
3. Требуется ли отдельная материализованная таблица/индекс для overlap-проверок по `Bookings`, если ожидается высокий объем конкурентных бронирований.
