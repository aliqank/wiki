# Схема БД v11: Azure SQL adaptation for Equipments + Bookings

**Created:** 2026-05-14  
**Last updated:** 2026-05-14  
**Author:** OpenCode

---

## Что изменилось в v11

| # | Изменение | Комментарий |
|---|---|---|
| 1 | Базовая СУБД зафиксирована как **Azure SQL / SQL Server** | Версия v11 является SQL Server-friendly адаптацией v10 |
| 2 | `UUID` заменён на `uniqueidentifier` | Во всех таблицах и FK |
| 3 | `TIMESTAMP` заменён на `datetime2(3)` | Единый стандарт для business/audit дат |
| 4 | `BOOLEAN` заменён на `bit` | Во всех таблицах |
| 5 | `TEXT` заменён на `nvarchar(max)` | Для описаний, комментариев, причин, отзывов |
| 6 | `SERIAL` заменён на `IDENTITY(1,1)` | Для `BookingRequests.requestNumber` |
| 7 | `ENUM` заменён на lookup/reference tables | Введены таблицы `ref_*` |
| 8 | `JSON` с локализованными `name` заменён на отдельные колонки | `nameEn`, `nameRu`, `nameKz` |
| 9 | `JSONB` в `EquipmentProperties.value` заменён на типизированные колонки | `valueString`, `valueInt`, `valueDecimal`, `valueBit`, `propertyEnumValueId` |
| 10 | `JSONB sourcePayload` в JDE-таблицах заменён на `nvarchar(max)` | JSON хранится как raw payload text |
| 11 | Добавлен `Equipments.currentStatusId` | Денормализованный кэш текущего статуса техники |
| 12 | Уникальные ограничения приведены к filtered unique indexes | С учетом soft delete (`isDeleted = 0`) |

---

## Azure SQL conventions

### 1. Типы данных

| Было в v10 | Стало в v11 | Комментарий |
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
- `changedAt` и `changedBy` остаются отдельными полями

### 3. Локализация

Во всех таблицах, где в v10 использовался `name JSON`, в v11 используются:

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
| `ref_fleet_type` | Maintenance, Construction, TFM, SCMLogistics, ExternalBP |
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
| `ref_booking_request_status` | Draft, Submitted, InProgress, Completed, Cancelled |
| `ref_booking_status` | Draft, Submitted, ConfirmedByFo, Confirmed, TransportConfirmed, Declined, Revoked, Terminated, InProgress, Closed, EquipmentChanged, Extended |

Минимальный шаблон reference table:

| Поле | Тип |
|---|---|
| `id` | `uniqueidentifier PK` |
| `code` | `nvarchar(100) not null` |
| `nameEn` | `nvarchar(255) not null` |
| `nameRu` | `nvarchar(255) null` |
| `nameKz` | `nvarchar(255) null` |
| `sortOrder` | `int not null default 0` |
| `isActive` | `bit not null default 1` |

Индекс:
- filtered unique index on `code where isActive = 1`

---

## Блок Equipment

### 1. EquipmentTypes

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

| Поле | Тип | Описание |
|---|---|---|
| `id` | `uniqueidentifier PK` | |
| `nameEn` | `nvarchar(255) not null` | |
| `nameRu` | `nvarchar(255) null` | |
| `nameKz` | `nvarchar(255) null` | |
| `fleetTypeId` | `uniqueidentifier FK -> ref_fleet_type` | |
| `userId` | `uniqueidentifier` | Ответственный пользователь / владелец флота |
| `aadGroupId` | `nvarchar(255)` | ID AAD-группы Fleet Owner |
| audit fields | см. conventions | |

### 3. WorkCenters

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

| Поле | Тип |
|---|---|
| `id` | `uniqueidentifier PK` |
| `nameEn` | `nvarchar(255) not null` |
| `nameRu` | `nvarchar(255) null` |
| `nameKz` | `nvarchar(255) null` |
| `sortOrder` | `int not null` |
| audit fields | см. conventions |

### 5. EquipmentModels

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

Для этих таблиц используется единый паттерн:

| Поле | Тип |
|---|---|
| `id` | `uniqueidentifier PK` |
| `code` | `nvarchar(100) not null` |
| `nameEn` | `nvarchar(255) not null` |
| `nameRu` | `nvarchar(255) null` |
| `nameKz` | `nvarchar(255) null` |
| `sortOrder` | `int not null default 0` |
| Parent FK | `uniqueidentifier null` | Для иерархии `Groups -> Divisions`, `Departments -> Groups`, `Sections -> Departments` |
| audit fields | см. conventions |

Filtered unique indexes:
- `code where isDeleted = 0`

### 7. FleetManagePermissions

| Поле | Тип |
|---|---|
| `id` | `uniqueidentifier PK` |
| `fleetId` | `uniqueidentifier FK -> Fleets` |
| `userId` | `uniqueidentifier FK -> Users` |
| `expiresAt` | `datetime2(3) null` |
| audit fields | см. conventions |

### 8. Equipments

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
| `plannedEngineHoursPerDay` | `decimal(18,4) null` | |
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

| Поле | Тип |
|---|---|
| `id` | `uniqueidentifier PK` |
| `equipmentId` | `uniqueidentifier FK -> Equipments` |
| `url` | `nvarchar(1000) not null` |
| `sortOrder` | `int not null default 0` |
| `isPrimary` | `bit not null default 0` |
| audit fields | см. conventions |

### 10. EquipmentStatuses

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

---

## EAV-блок: динамические свойства техники

### 11. MeasurementUnits

| Поле | Тип |
|---|---|
| `id` | `uniqueidentifier PK` |
| `code` | `nvarchar(50) not null` |
| `displayName` | `nvarchar(100) not null` |
| `sortOrder` | `int not null default 0` |
| audit fields | см. conventions |

### 12. Properties

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

| Поле | Тип |
|---|---|
| `id` | `uniqueidentifier PK` |
| `propertyId` | `uniqueidentifier FK -> Properties` |
| `value` | `nvarchar(255) not null` |
| `sortOrder` | `int not null default 0` |
| audit fields | см. conventions |

### 14. EquipmentTypeProperties

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

`MaintenancePartners`:
- `nameEn`, `nameRu`, `nameKz`
- `phoneNumber nvarchar(100)`
- `email nvarchar(255)`
- `address nvarchar(500)`
- audit fields

`EquipmentMaintenanceContracts`:
- `equipmentId uniqueidentifier FK`
- `partnerId uniqueidentifier FK`
- `serviceType nvarchar(255)`
- `notes nvarchar(max) null`
- audit fields

Filtered unique index:
- `(equipmentId, partnerId, serviceType) where isDeleted = 0`

### 17. EquipmentFeedbacks / EquipmentBookingAuthorizations

`EquipmentFeedbacks`:
- `feedback nvarchar(max) not null`
- `bookingId uniqueidentifier FK`
- `equipmentId uniqueidentifier FK`

`EquipmentBookingAuthorizations`:
- `equipmentId uniqueidentifier FK`
- `userId uniqueidentifier FK`
- filtered unique index `(equipmentId, userId) where isDeleted = 0`

---

## Служебные таблицы

### 18. SystemSettings

| Поле | Тип |
|---|---|
| `key` | `nvarchar(200) PK` |
| `value` | `nvarchar(max) not null` |
| audit fields | см. conventions |

### 19. Users

| Поле | Тип |
|---|---|
| `id` | `uniqueidentifier PK` |
| `badgeNumber` | `nvarchar(100) null` |
| `fullName` | `nvarchar(255) not null` |
| `email` | `nvarchar(255) not null` |
| `sharedEmail` | `nvarchar(255) null` |
| `jobTitle` | `nvarchar(255) null` |
| `departmentId` | `uniqueidentifier null FK -> Departments` |
| `isActive` | `bit not null default 1` |
| `businessPartnerId` | `uniqueidentifier null FK -> BusinessPartners` |
| `userTypeId` | `uniqueidentifier FK -> ref_user_type` |
| audit fields | см. conventions |

### 20. BusinessPartners

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

### 21. JdeWorkOrders

| Поле | Тип |
|---|---|
| `id` | `uniqueidentifier PK` |
| `jdeWorkOrderId` | `nvarchar(100) not null` |
| `workOrderName` | `nvarchar(255) null` |
| `workOrderStatus` | `nvarchar(100) null` |
| `workOrderStatusDescription` | `nvarchar(255) null` |
| `priorityId` | `uniqueidentifier null FK -> ref_request_priority` |
| `sourcePayload` | `nvarchar(max) null` |
| `lastSyncedAt` | `datetime2(3) not null` |
| `isActive` | `bit not null default 1` |
| audit fields | см. conventions |

Filtered unique indexes:
- `jdeWorkOrderId where isDeleted = 0`

### 22. JdeWorkOrderSteps

| Поле | Тип |
|---|---|
| `id` | `uniqueidentifier PK` |
| `jdeWorkOrderRefId` | `uniqueidentifier FK -> JdeWorkOrders` |
| `workCenterId` | `uniqueidentifier FK -> WorkCenters` |
| `stepName` | `nvarchar(255) null` |
| `stepVolume` | `int null` |
| `startDt` | `datetime2(3) null` |
| `endDt` | `datetime2(3) null` |
| `sourcePayload` | `nvarchar(max) null` |
| `lastSyncedAt` | `datetime2(3) not null` |
| `isActive` | `bit not null default 1` |
| audit fields | см. conventions |

Filtered unique indexes:
- `(jdeWorkOrderRefId, workCenterId) where isDeleted = 0`

### 23. BookingRequests

| Поле | Тип | Описание |
|---|---|---|
| `id` | `uniqueidentifier PK` | |
| `requestNumber` | `int IDENTITY(1,1) unique not null` | Номер заявки; форматируется как `REQ-YYYY-NNNNN` |
| `requestTypeId` | `uniqueidentifier FK -> ref_request_type` | |
| `jdeWorkOrderRefId` | `uniqueidentifier null FK -> JdeWorkOrders` | |
| `statusId` | `uniqueidentifier FK -> ref_booking_request_status` | Денормализованный текущий статус |
| `workOrderJdeId` | `nvarchar(100) null` | |
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
- filtered index on `workOrderJdeId where isDeleted = 0 and workOrderJdeId is not null`

### 24. Bookings

| Поле | Тип | Описание |
|---|---|---|
| `id` | `uniqueidentifier PK` | |
| `requestId` | `uniqueidentifier FK -> BookingRequests` | |
| `equipmentId` | `uniqueidentifier not null` | Внешняя ссылка на технику |
| `fleetId` | `uniqueidentifier not null` | Внешняя ссылка на парк |
| `workCenterId` | `uniqueidentifier null FK -> WorkCenters` | |
| `jdeWorkOrderStepRefId` | `uniqueidentifier null FK -> JdeWorkOrderSteps` | |
| `statusId` | `uniqueidentifier FK -> ref_booking_status` | Денормализованный текущий статус |
| `startDt` | `datetime2(3) not null` | |
| `endDt` | `datetime2(3) not null` | |
| `actualStartDt` | `datetime2(3) null` | |
| `actualEndDt` | `datetime2(3) null` | |
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
- при наличии `jdeWorkOrderStepRefId`, `workCenterId` должен совпадать с шагом WO

Индексы:
- `requestId`, `equipmentId`, `fleetId`, `statusId`, `transportBookingId`
- составной `(equipmentId, startDt, endDt)`

### 25. BookingStatuses

| Поле | Тип |
|---|---|
| `id` | `uniqueidentifier PK` |
| `bookingId` | `uniqueidentifier FK -> Bookings` |
| `statusId` | `uniqueidentifier FK -> ref_booking_status` |
| `changedBy` | `uniqueidentifier not null` |
| `changedAt` | `datetime2(3) not null` |
| `comment` | `nvarchar(max) null` |
| `createdAt` | `datetime2(3) not null` |
| `createdBy` | `uniqueidentifier not null` |
| `updatedAt` | `datetime2(3) null` |
| `updatedBy` | `uniqueidentifier null` |
| `isDeleted` | `bit not null default 0` |
| `deletedAt` | `datetime2(3) null` |
| `deletedBy` | `uniqueidentifier null` |

### 26. BookingRequestStatuses

| Поле | Тип |
|---|---|
| `id` | `uniqueidentifier PK` |
| `requestId` | `uniqueidentifier FK -> BookingRequests` |
| `statusId` | `uniqueidentifier FK -> ref_booking_request_status` |
| `changedBy` | `uniqueidentifier not null` |
| `changedAt` | `datetime2(3) not null` |
| `comment` | `nvarchar(max) null` |
| `createdAt` | `datetime2(3) not null` |
| `createdBy` | `uniqueidentifier not null` |
| `updatedAt` | `datetime2(3) null` |
| `updatedBy` | `uniqueidentifier null` |
| `isDeleted` | `bit not null default 0` |
| `deletedAt` | `datetime2(3) null` |
| `deletedBy` | `uniqueidentifier null` |

---

## Итоговая позиция по замечаниям

| Замечание | Решение в v11 |
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

---

## Открытые вопросы для следующей итерации

1. Нужно ли переводить все `createdBy / updatedBy / deletedBy` на полноценные FK к `Users`, или оставить soft-reference без ограничений.
2. Нужен ли `SEQUENCE` вместо `IDENTITY` для `requestNumber`, если появятся дополнительные типы документов с общей нумерацией.
3. Требуется ли отдельная материализованная таблица/индекс для overlap-проверок по `Bookings`, если ожидается высокий объем конкурентных бронирований.
