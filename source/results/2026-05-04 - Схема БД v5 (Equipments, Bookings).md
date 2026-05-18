# Схема БД v5: Equipments + Bookings (сводный файл)

**Дата создания:** 2026-05-04  
**Последнее обновление:** 2026-05-04  
**Автор:** Тельман Нуржанов (SA)

---

## Что изменилось в v5

| # | Изменение | Затронутые таблицы |
|---|-----------|--------------------|
| 1 | EquipmentTrackers — исключена из схемы (вне зоны ответственности букинга) | ~~EquipmentTrackers~~ |
| 2 | EquipmentFreezes + EquipmentRepairHistory + decommission-поля → объединены в одну таблицу EquipmentStatuses | ~~EquipmentFreezes~~, ~~EquipmentRepairHistory~~ → **EquipmentStatuses** |
| 3 | Equipments: убраны `isDecommissioned` и `decommissionedAt`; актуальный статус теперь вычисляется из EquipmentStatuses | Equipments |
| 4 | FleetDelegations → **FleetManagePermissions** (упрощена структура: убраны `delegatedAt`, `delegatedBy`; добавлен `createdAt`) | FleetDelegations → **FleetManagePermissions** |
| 5 | BookingRequests: убрано поле `isDefaultWorkOrder` | BookingRequests |
| 6 | Bookings: убраны `assignedFleetOwnerId`, `mobilizationStartedAt`, `closedAt`, `closedBy`, `closeComment` | Bookings |
| 7 | BookingStatusHistories → **BookingStatuses**: убран `previousSnapshot`; `oldStatus` + `newStatus` → `status` (текущее состояние на момент записи) | BookingStatusHistories → **BookingStatuses** |
| 8 | Fleets: исключено поле `isExternal` | Fleets |
| 9 | BookingRequests: `status` хранится как денормализованный кэш; обновляется атомарно с BookingRequestStatuses | BookingRequests |
| 10 | Bookings: `status` хранится как денормализованный кэш; обновляется атомарно с BookingStatuses | Bookings |
| 11 | RequestStatusHistories → **BookingRequestStatuses**: `oldStatus` + `newStatus` → `status` (аналогично BookingStatuses) | RequestStatusHistories → **BookingRequestStatuses** |
| 12 | Equipments: добавлены `isCritical` и `yearOfManufacture` (FR-022, BRD §3 Criticality) | Equipments |
| 13 | Bookings: добавлены `actualStartDt` и `actualEndDt` — фактические даты использования, вводятся при закрытии брони (FR-NEW-33) | Bookings |
| 14 | Добавлена таблица **EquipmentFeedbacks** — отзывы на технику, привязанные к брони (FR-022) | **EquipmentFeedbacks** |

**Итого: 19 таблиц** (было 20 в v4: удалены EquipmentTrackers и EquipmentFreezes/EquipmentRepairHistory схлопнуты в одну)

---

## Принятые решения

### По встрече 30.04.2026

| Тема | Решение |
|------|---------|
| EquipmentTrackers | Исключена из схемы — не входит в зону ответственности модуля букинга |
| Статус техники (EquipmentStatuses) | Одна таблица вместо трёх (EquipmentFreezes + EquipmentRepairHistory + decommission); хранит историю всех состояний; активная запись определяет текущий статус |
| currentStatus в Equipments | Не хранимое поле — вычисляется при запросе из EquipmentStatuses (логика вычисления описана в разделе Equipments) |
| FleetManagePermissions | Переименована из FleetDelegations, упрощена: id, fleetId, userId, createdAt, expiresAt |
| isDefaultWorkOrder в BookingRequests | Убрано; workOrderNumber остаётся nullable |
| isExternal в Fleets | Убрано |
| status в BookingRequests и Bookings | Хранится как денормализованный кэш; обновляется атомарно (в одной транзакции) вместе с вставкой в BookingRequestStatuses / BookingStatuses. Источник правды — история. |
| BookingRequestStatuses (переименование) | RequestStatusHistories переименована в BookingRequestStatuses; семантика изменена: `oldStatus` + `newStatus` → `status` (текущий статус заявки в момент записи; аналогично BookingStatuses) |
| isCritical в Equipments | Булевый флаг «требует охраны (service de sécurité) при транспортировке»; отображается в карточке техники |
| yearOfManufacture в Equipments | Год выпуска; nullable; рекомендуется для TCO Owned, опционален для Long-term rented |
| actualStartDt / actualEndDt в Bookings | Фактические даты использования; вводятся пользователем при закрытии брони (FR-NEW-33); основа для расчёта Usage Rate |
| EquipmentFeedbacks | Отзывы заявителей/SWP на технику (FR-022); привязаны к конкретной брони; поля: id, equipmentId, bookingId, feedback, createdBy, createdAt |
| assignedFleetOwnerId в Bookings | Убрано |
| Статусные datetime в Bookings | mobilizationStartedAt, closedAt, closedBy, closeComment — убраны; эти данные хранятся в BookingStatuses (changedAt, changedBy, comment) |
| BookingStatuses (переименование) | BookingStatusHistories переименована в BookingStatuses; семантика изменена: запись фиксирует текущее состояние (не переход); убран previousSnapshot; поле status = текущий статус брони в момент записи |

### Ранее принятые решения (v1–v4)

| Тема | Решение |
|------|---------|
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
| Транспортная бронь (OQ-DB-9, OQ-DB-11) | Self-ref FK `transportBookingId` в Bookings; статус `TransportConfirmed`; обязана принадлежать тому же requestId |

---

## Блок Equipment

### 1. EquipmentTypes

| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID PK | |
| name | VARCHAR | Компрессор, Экскаватор, Самосвал... |
| mobilityType | ENUM | SelfPropelled / NonSelfPropelledMotorized / Stationary / NonMotorized |
| requiresTransport | BOOLEAN | Требуется транспортная техника для доставки |
| sortOrder | INT | Порядок отображения в UI |

---

### 2. Fleets

| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID PK | |
| name | VARCHAR | |
| type | ENUM | Maintenance / Construction / TFM / SCMLogistics / ExternalBP |
| aadGroupId | VARCHAR | ID AAD-группы Fleet Owner |

---

### 2а. FleetManagePermissions

*(Переименована из FleetDelegations; упрощена структура)*

Кому из конкретных пользователей Fleet Owner предоставлены права управления флотом.

| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID PK | |
| fleetId | FK → Fleets | |
| userId | FK → Users | Кому предоставлены права |
| createdAt | TIMESTAMP | Дата предоставления |
| expiresAt | TIMESTAMP nullable | NULL = бессрочно |

---

### 3. Equipments

Единица техники. Единая таблица для всей техники: TCO Owned, Long-term rented, On-demand BP.

| Поле | Тип | Описание |
|------|-----|----------|
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

> **Актуальный статус** (не хранится, вычисляется из EquipmentStatuses при запросе):
> - **Decommissioned** → активная запись со `status = Decommissioned` (без `cancelledAt`)
> - **InRepair** → активная запись со `status = InRepair` и `actualEndsAt IS NULL`
> - **Frozen** → активная запись со `status = Frozen`, `startsAt ≤ today AND (endsAt IS NULL OR endsAt ≥ today) AND cancelledAt IS NULL`
> - **Available** → нет активных записей ни одного из перечисленных типов

---

### 4. EquipmentPhotos

| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID PK | |
| equipmentId | FK → Equipments | |
| url | VARCHAR | |
| sortOrder | INT | |
| isPrimary | BOOLEAN | Главная фотография |

---

### 5. EquipmentStatuses

*(Новая таблица; объединяет EquipmentFreezes, EquipmentRepairHistory и decommission-состояние)*

Полная история состояний техники. Активная запись определяет текущий статус.

| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID PK | |
| equipmentId | FK → Equipments | |
| status | ENUM | Frozen / InRepair / Decommissioned |
| reason | TEXT nullable | Причина / описание |
| startsAt | DATE NOT NULL | Дата начала состояния |
| endsAt | DATE nullable | Плановая/ожидаемая дата окончания; NULL = бессрочно |
| actualEndsAt | DATE nullable | Фактическая дата окончания (для InRepair) |
| cancelledAt | TIMESTAMP nullable | Дата отмены (для Frozen: досрочная отмена заморозки) |
| source | ENUM nullable | Manual / JDE (актуально для InRepair) |

> **Правило активной записи:**
> - **Frozen**: `startsAt ≤ today AND (endsAt IS NULL OR endsAt ≥ today) AND cancelledAt IS NULL`
> - **InRepair**: `actualEndsAt IS NULL`
> - **Decommissioned**: `cancelledAt IS NULL` (списание — необратимо; cancelledAt не используется, но поле оставлено для единообразия)

**Индексы:** `equipmentId`, составной `(equipmentId, status)` для быстрого определения текущего состояния

---

## EAV-блок: динамические свойства техники

### 6. MeasurementUnits

| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID PK | |
| code | VARCHAR | м / кг / кВт / л / бар / м³... |
| displayName | VARCHAR | |
| sortOrder | INT | |

---

### 7. Properties

| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID PK | |
| name | VARCHAR | |
| dataType | ENUM | number / double / text / boolean / enum |
| unitId | FK → MeasurementUnits nullable | Только для number/double |

---

### 8. PropertyEnumValues

| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID PK | |
| propertyId | FK → Properties | |
| value | VARCHAR | |
| sortOrder | INT | |

---

### 9. EquipmentTypeProperties

| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID PK | |
| equipmentTypeId | FK → EquipmentTypes | |
| propertyId | FK → Properties | |
| isRequired | BOOLEAN | |
| isFilterable | BOOLEAN | |
| isVisibleInCard | BOOLEAN | |
| sortOrder | INT | |
| UNIQUE | — | (equipmentTypeId, propertyId) |

---

### 10. EquipmentProperties

| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID PK | |
| equipmentId | FK → Equipments | |
| propertyId | FK → Properties | |
| value | JSONB | `{"Bool": null, "Double": 6.7, "Number": null, "String": null}` |
| UNIQUE | — | (equipmentId, propertyId) |

---

### 11. MaintenancePartners

| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID PK | |
| name | VARCHAR | |
| contactInfo | TEXT nullable | |

---

### 12. EquipmentMaintenanceContracts

| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID PK | |
| equipmentId | FK → Equipments | |
| partnerId | FK → MaintenancePartners | |
| serviceType | VARCHAR | Направление: замена масла / колёса / электрика... |
| notes | TEXT nullable | |
| UNIQUE | — | (equipmentId, partnerId, serviceType) |

---

### 12а. EquipmentFeedbacks

Отзывы заявителей и SWP на технику. Создаётся после того, как бронь достигла статуса Confirmed и выше; требует привязки к конкретной брони.

| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID PK | |
| equipmentId | FK → Equipments | |
| bookingId | FK → Bookings | Бронь, в рамках которой оставлен отзыв |
| feedback | TEXT NOT NULL | Текст отзыва |
| createdBy | UUID NOT NULL | ID заявителя/SWP (внешний; без FK-ограничения) |
| createdAt | TIMESTAMP NOT NULL | |

**Индексы:** `equipmentId`, `bookingId`

---

### 13. SystemSettings

| Поле | Тип | Описание |
|------|-----|----------|
| key | VARCHAR PK | |
| value | VARCHAR | |
| updatedAt | TIMESTAMP | |

Примеры ключей: `booking.horizonDays`, `booking.maxDurationDays`, `fo.timeoutHours24`, `fo.timeoutHours48`

---

### 14. Users

| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID PK | |
| externalId | VARCHAR | AAD object ID (для TCO) или ID из системы БП |
| email | VARCHAR | |
| displayName | VARCHAR | |
| departmentCode | VARCHAR nullable | |
| userType | ENUM | TCO / BP |

---

## Блок Booking

### 1. BookingRequests

Заявка на бронирование. Один Work Order — одна заявка; все брони разделяют один WO, локацию и приоритет. Для unwheeled-техники: транспортная бронь создаётся FO в рамках той же заявки.

| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID PK | |
| requestNumber | SERIAL UNIQUE NOT NULL | Порядковый номер; отображается как REQ-YYYY-NNNNN |
| type | ENUM NOT NULL | Regular / ServiceWork |
| status | ENUM NOT NULL | Draft / Submitted / InProgress / Completed / Cancelled — денормализованный кэш; синхронизируется с BookingRequestStatuses в одной транзакции |
| workOrderNumber | VARCHAR nullable | Номер WO из JDE E1 |
| location | VARCHAR nullable | Локация; обязательна для SCM Logistics вместо WO |
| workDescription | VARCHAR NOT NULL | Описание работ |
| comments | TEXT nullable | |
| priority | ENUM NOT NULL | P1 / P2 / P3 / P4 |
| createdBy | UUID NOT NULL | ID заявителя (внешний; без FK-ограничения) |
| createdAt | TIMESTAMP NOT NULL | |

> **Жизненный цикл:** Draft → Submitted → InProgress → Completed; Cancelled — из Draft

**Индексы:** `createdBy`, `status`, `requestNumber`; partial на `workOrderNumber WHERE workOrderNumber IS NOT NULL`

---

### 2. Bookings

Атомарная единица бронирования: одна конкретная единица техники в рамках одной заявки.

| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID PK | |
| requestId | UUID NOT NULL FK → BookingRequests | |
| equipmentId | UUID NOT NULL | ID техники (внешний; без FK-ограничения) |
| fleetId | UUID NOT NULL | ID парка (внешний; без FK-ограничения) |
| status | ENUM NOT NULL | Draft / Submitted / Confirmed / TransportConfirmed / Declined / Revoked / Terminated / InProgress / Closed / EquipmentChanged / Extended — денормализованный кэш; синхронизируется с BookingStatuses в одной транзакции |
| startDt | TIMESTAMP NOT NULL | Начало бронирования / плановый старт мобилизации |
| endDt | TIMESTAMP NOT NULL | Окончание бронирования |
| actualStartDt | TIMESTAMP nullable | Фактическая дата начала использования; вводится при закрытии брони |
| actualEndDt | TIMESTAMP nullable | Фактическая дата окончания использования; вводится при закрытии брони |
| justification | TEXT nullable | Обоснование; обязательно для Assigned и SharedWithConditions |
| transportBookingId | UUID nullable FK → Bookings | Self-ref: ID транспортной брони; только для unwheeled; обязана принадлежать тому же requestId |
| declineReason | TEXT nullable | Причина отклонения; обязательна при переходе в Declined |
| terminateReason | TEXT nullable | Причина досрочного завершения; обязательна при переходе в Terminated |

> **Статусные даты (mobilizationStartedAt, closedAt) и автор закрытия (closedBy) хранятся в BookingStatuses** — записи со статусами InProgress и Closed соответственно.

> **Жизненный цикл:**
> - Обычная техника: `Draft → Submitted → Confirmed → InProgress → Closed`
> - Unwheeled: `Draft → Submitted → Confirmed → TransportConfirmed → InProgress → Closed`
> - Общие ветки: Declined, Revoked, Terminated, EquipmentChanged, Extended

**Индексы:** `requestId`, `equipmentId`, `fleetId`, `status`, `transportBookingId`; составной `(equipmentId, startDt, endDt)`

---

### 3. BookingStatuses

*(Переименована из BookingStatusHistories; семантика изменена)*

Фиксирует каждое состояние брони, включая начальное. Запись создаётся при каждом изменении `Bookings.status`. Поле `status` отражает текущее состояние в момент записи (не переход «было → стало»).

| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID PK | |
| bookingId | UUID NOT NULL FK → Bookings | |
| status | ENUM NOT NULL | Текущий статус брони в момент записи |
| changedBy | UUID NOT NULL | ID пользователя-инициатора (внешний; без FK-ограничения) |
| changedAt | TIMESTAMP NOT NULL | Момент перехода |
| comment | TEXT nullable | Причина или контекст (например, текст declineReason, terminateReason или closeComment) |

> Первая запись создаётся при создании брони (status = Draft). Каждый последующий переход — новая запись.

**Индексы:** `bookingId`, `changedAt`

---

### 4. BookingRequestStatuses

*(Переименована из RequestStatusHistories; семантика изменена)*

Фиксирует каждое состояние заявки, включая начальное. Запись создаётся при каждом изменении статуса заявки. Поле `status` отражает текущее состояние в момент записи (не переход «было → стало»).

| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID PK | |
| requestId | UUID NOT NULL FK → BookingRequests | |
| status | ENUM NOT NULL | Текущий статус заявки в момент записи: Draft / Submitted / InProgress / Completed / Cancelled |
| changedBy | UUID NOT NULL | Внешний UUID; без FK-ограничения |
| changedAt | TIMESTAMP NOT NULL | |
| comment | TEXT nullable | |

> Первая запись создаётся при создании заявки (status = Draft). Каждый последующий переход — новая запись.

**Индексы:** `requestId`, `changedAt`

---

## Сводный список таблиц

### Equipment-блок

| # | Таблица | Назначение |
|---|---------|-----------|
| 1 | EquipmentTypes | Классификатор типов техники |
| 2 | Fleets | Парки техники |
| 2а | FleetManagePermissions | Права управления флотом (переим. из FleetDelegations) |
| 3 | Equipments | Единица техники (TCO + BP unified) |
| 4 | EquipmentPhotos | Фотографии техники |
| 5 | EquipmentStatuses | История состояний техники: Frozen / InRepair / Decommissioned |
| 6 | MeasurementUnits | Справочник единиц измерения |
| 7 | Properties | Справочник свойств техники |
| 8 | PropertyEnumValues | Допустимые значения enum-свойств |
| 9 | EquipmentTypeProperties | Схема свойств по типу (Admin Panel) |
| 10 | EquipmentProperties | Значения свойств конкретной техники |
| 11 | MaintenancePartners | Справочник ДП по ТО |
| 12 | EquipmentMaintenanceContracts | Связь техники и партнёра ТО |
| 12а | EquipmentFeedbacks | Отзывы заявителей/SWP на технику (FR-022) |
| 13 | SystemSettings | Глобальные настройки Admin Panel |
| 14 | Users | Справочник пользователей (TCO + BP) |

### Booking-блок

| # | Таблица | Назначение |
|---|---------|-----------|
| 1 | BookingRequests | Заявка на бронирование |
| 2 | Bookings | Атомарная единица бронирования (self-ref FK для unwheeled) |
| 3 | BookingStatuses | История состояний брони (переим. из BookingStatusHistories) |
| 4 | BookingRequestStatuses | История состояний заявки (переим. из RequestStatusHistories) |

**Итого: 19 таблиц**

---

## Связи между блоками

| Поле Booking-блока | Ссылается на | Тип связи |
|--------------------|--------------|-----------|
| BookingRequests.createdBy | Users.id | Внешний UUID, без FK-ограничения |
| Bookings.equipmentId | Equipments.id | Внешний UUID, без FK-ограничения |
| Bookings.fleetId | Fleets.id | Внешний UUID, без FK-ограничения |
| Bookings.transportBookingId | Bookings.id | Self-referencing FK (с FK-ограничением); nullable |
| BookingStatuses.changedBy | Users.id | Внешний UUID, без FK-ограничения |
| BookingRequestStatuses.changedBy | Users.id | Внешний UUID, без FK-ограничения |
| EquipmentFeedbacks.bookingId | Bookings.id | FK с ограничением |
| EquipmentFeedbacks.createdBy | Users.id | Внешний UUID, без FK-ограничения |

---

## Закрытые открытые вопросы

| # | Вопрос | Решение |
|---|--------|---------|
| OQ-DB-1 | История изменений shareType у техники? | Не нужна |
| OQ-DB-2 | Users.departmentCode | Отложено |
| OQ-DB-3 | Аудит-поля | Добавить в следующей итерации |
| OQ-DB-4 | RequestStatusHistory нужна? | Да — добавлена |
| OQ-DB-5 | BookingSnapshot | CLOSED: previousSnapshot был в BookingStatusHistories; в v5 — убран |
| OQ-DB-8 | Продление брони | CLOSED: статус Extended; детали продления фиксируются через comment в BookingStatuses |
| OQ-DB-9 | Unwheeled-флоу | CLOSED: transportBookingId (self-ref FK) + статус TransportConfirmed |
| OQ-DB-10 | Замена техники FO | CLOSED: статус EquipmentChanged |
| OQ-DB-11 | Скоуп транспортной брони | CLOSED: тот же requestId |

## Открытые вопросы

| # | Вопрос | Impact |
|---|--------|--------|
| OQ-DB-12 | **Атрибуция FO при запросе транспорта у коллег**: нужно ли поле `transportRequestedFleetOwnerId` в транспортной брони? | Модель уведомлений / маршрутизация |
