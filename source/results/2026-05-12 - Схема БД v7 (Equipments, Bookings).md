# Схема БД v7: Equipments + Bookings (сводный файл)

**Created:** 2026-05-08  
**Last updated:** 2026-05-12  
**Author:** Telman Nurzhanov (SA)

---

## Что изменилось в v7

| # | Изменение | Затронутые таблицы |
|---|---|---|
| 1 | В `Fleets` добавлено поле `userId` | Fleets |
| 2 | В `EquipmentTypes` добавлены поля `equipmentClass` (HDE / HDV) и `workCenterId` | EquipmentTypes |
| 3 | Добавлена новая справочная таблица `WorkCenters` | WorkCenters |
| 4 | Схема расширена до 20 таблиц | Все таблицы |

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

### По версии v7

| Тема | Решение |
|---|---|
| Аудит-поля | Единый набор из 7 полей добавлен во все 20 таблиц. Для log/history-таблиц updatedAt/updatedBy ожидаются как NULL. createdBy / updatedBy / deletedBy — UUID без FK-ограничения |
| Fleets.userId | Добавлен владелец/ответственный пользователь флота |
| EquipmentTypes.equipmentClass | Добавлен атрибут «Класс техники» со значениями HDE / HDV |
| EquipmentTypes.workCenterId | Добавлена привязка типа техники к справочнику WorkCenters |
| WorkCenters | Добавлен новый справочник производственных центров |
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
| name | VARCHAR | Компрессор, Экскаватор, Самосвал... |
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
| name | VARCHAR | |
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
| name | VARCHAR | Наименование производственного центра |
| code | VARCHAR | Код производственного центра |
| createdAt | TIMESTAMP NOT NULL | |
| createdBy | UUID NOT NULL | Без FK-ограничения |
| updatedAt | TIMESTAMP nullable | |
| updatedBy | UUID nullable | |
| isDeleted | BOOLEAN NOT NULL DEFAULT false | |
| deletedAt | TIMESTAMP nullable | |
| deletedBy | UUID nullable | |

---

### 2б. FleetManagePermissions

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
| stateNumber | VARCHAR nullable | Госномер |
| serialNumber | VARCHAR nullable | |
| description | TEXT nullable | |
| brand | VARCHAR | |
| model | VARCHAR | |
| shareType | ENUM | Shared / SharedWithConditions / Assigned |
| isCritical | BOOLEAN | Требует охраны (service de sécurité) при транспортировке |
| yearOfManufacture | INT nullable | Год выпуска |
| serviceZoneCode | VARCHAR | Код сервисной зоны (строка из PSWS) |
| costCenter | VARCHAR | Финансовый ЦЗ (строка из JDE) |
| baseLocation | VARCHAR | Базовое местоположение (из Master Fleet) |
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
| name | VARCHAR | |
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
| name | VARCHAR | |
| contactInfo | TEXT nullable | |
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

| Поле | Тип | Описание |
|---|---|---|
| id | UUID PK | |
| externalId | VARCHAR | AAD object ID (для TCO) или ID из системы БП |
| email | VARCHAR | |
| displayName | VARCHAR | |
| departmentCode | VARCHAR nullable | |
| userType | ENUM | TCO / BP |
| createdAt | TIMESTAMP NOT NULL | |
| createdBy | UUID NOT NULL | Без FK-ограничения |
| updatedAt | TIMESTAMP nullable | |
| updatedBy | UUID nullable | |
| isDeleted | BOOLEAN NOT NULL DEFAULT false | |
| deletedAt | TIMESTAMP nullable | |
| deletedBy | UUID nullable | |

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
| 2б | FleetManagePermissions | Права управления флотом |
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

### Booking-блок

| # | Таблица | Назначение |
|---|---|---|
| 1 | BookingRequests | Заявка на бронирование |
| 2 | Bookings | Атомарная единица бронирования |
| 3 | BookingStatuses | История состояний брони |
| 4 | BookingRequestStatuses | История состояний заявки |

**Итого: 20 таблиц**

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

---

## Закрытые открытые вопросы

| # | Вопрос | Решение |
|---|---|---|
| OQ-DB-1 | История изменений shareType у техники? | Не нужна |
| OQ-DB-2 | Users.departmentCode | Отложено |
| OQ-DB-3 | Аудит-поля | Закрыт в v7: добавлены во все 20 таблиц (createdAt, createdBy, updatedAt, updatedBy, isDeleted, deletedAt, deletedBy) |
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
