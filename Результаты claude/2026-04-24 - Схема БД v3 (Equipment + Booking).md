# Схема БД v3: Equipment + Booking (сводный файл)

**Дата создания:** 2026-04-24 00:00  
**Последнее обновление:** 2026-04-24 12:00  
**Автор:** Тельман Нуржанов (SA)

---

## Что изменилось в v3

| # | Изменение |
|---|-----------|
| 1 | Закрыт OQ-DB-9: транспортировка unwheeled решена через `transportBookingId` + статус `TransportConfirmed` |
| 2 | Таблица Bookings: добавлено поле `transportBookingId` (self-referencing FK) |
| 3 | Таблица Bookings: статус `TransportConfirmed` добавлен в ENUM и жизненный цикл |
| 4 | BookingStatusHistories.previousSnapshot расширен — покрывает переход `TransportConfirmed` |
| 5 | Добавлен OQ-DB-11: скоуп заявки для транспортной брони (тот же RequestId или отдельный?) |

---

## Принятые решения

### Equipment-блок (из v1, 2026-04-23)

| Тема | Решение |
|------|---------|
| EquipmentTypes | Плоский список; requiresTransport — хранимое поле |
| EquipmentTypeProperties | Схема свойств для каждого типа техники |
| EquipmentProperties.value | JSONB: `{"Bool", "Double", "Number", "String"}` |
| ownershipType | На Equipments — единая таблица TCO + BP (BRD v8 §5.1) |
| Статус техники | Динамический: вычисляется из EquipmentFreezes и EquipmentRepairHistory; isDecommissioned в Equipments |
| Фото | Отдельная таблица EquipmentPhotos |
| Трекеры | С историей назначения/снятия |
| ТО | Несколько партнёров на одну технику (через EquipmentMaintenanceContracts) |
| Доступ Assigned | Через AAD-группу (таблица прав не нужна) |
| Аудит-поля | Добавим в конце отдельным шагом |
| shareType history (OQ-DB-1) | История изменений shareType не нужна — хранится только текущее значение |
| Users.departmentCode (OQ-DB-2) | Отложено, пропускаем в этой итерации |
| Аудит-поля глобально (OQ-DB-3) | createdAt/updatedAt/createdBy — НЕ добавляем в этой итерации |

### Booking-блок (из v1, 2026-04-24)

| Тема | Решение |
|------|---------|
| RequestStatusHistory (OQ-DB-4) | Добавляем — явная история статусов заявки нужна для аудита |
| createdBy / createdAt на BookingRequests | Бизнесовые поля (кто создал заявку и когда) — не аудит, включаем |
| Архитектура Request / Booking | BookingRequests (шапка заявки) + Bookings (одна единица техники); Order / OrderLine pattern (BRD v11 §5.2) |
| Статус BookingRequests | Хранится явно (денормализация для производительности); агрегируется из статусов вложенных Bookings |
| Статус Bookings | Полный набор: Draft / Submitted / Confirmed / Declined / Revoked / Terminated / InProgress / Closed / **EquipmentChanged** / **Extended** |
| Внешние ID | equipmentId, fleetId — UUID без FK-ограничений; внешние сервисы обращаются по API |
| Ручное закрытие | Только ручное (FR-NEW-32); фиксируем closedBy + closedAt на Bookings |
| closeComment | TEXT nullable на Bookings — заявитель может оставить комментарий при закрытии |
| justification | NOT NULL не ставим на уровне БД — проверяется в приложении (зависит от shareType техники) |
| requestNumber | SERIAL (INT) в БД; форматируется в UI как REQ-YYYY-NNNNN, где YYYY берётся из createdAt |
| isDefaultWorkOrder | BOOLEAN на BookingRequests (FR-NEW-51) — для команд, не работающих в JDE |

### Новые решения (2026-04-24, рассуждения по схеме БД)

| Тема | Решение | Обоснование |
|------|---------|-------------|
| Именование таблиц | Множественное число: `Booking` → `Bookings` | Единое правило именования |
| `fleetId` в Bookings | **Оставить** | Исторический: fleet техники может измениться после создания брони; денормализация оправдана |
| `assignedFleetOwnerId` в Bookings | **Оставить** | НЕ выводится из equipmentId — конкретный FO определяется через FleetDelegations (меняются во времени); нужен для маршрутизации уведомлений |
| BookingSnapshot (OQ-DB-5) | Отдельная таблица **не нужна** | Заменяется полем `previousSnapshot JSONB nullable` в BookingStatusHistories |
| Замена техники FO (OQ-DB-10) | Новый статус `EquipmentChanged` в Bookings.status | Фиксирует факт замены; старый equipmentId / fleetId хранятся в previousSnapshot |
| Продление брони (OQ-DB-8) | Новый статус `Extended` в Bookings.status | Фиксирует продление; старый endDt хранится в previousSnapshot |

### Новые решения (2026-04-24, рассуждения по транспортировке — OQ-DB-9)

| Тема | Решение | Обоснование |
|------|---------|-------------|
| Архитектура транспорт-брони | Расширение Bookings: поле `transportBookingId` (self-ref FK) | Отдельная таблица TransportRequests не нужна — транспортная техника полноценная единица в Bookings со своим жизненным циклом |
| Направление FK | unwheeled бронь → транспортная бронь | Один грузовик может везти несколько unwheeled; несколько грузовиков не везут один unwheeled — поэтому FK у "пассажира", а не у "носителя" |
| Новый статус | `TransportConfirmed` в Bookings.status | Отражает момент, когда FO подтвердил основную бронь И привязал транспортную технику; только для unwheeled-брони |
| Откат при отмене транспорта | Если транспортная бронь отменяется/отклоняется — unwheeled бронь возвращается в `Confirmed`, `transportBookingId` обнуляется | Обрабатывается на уровне приложения; no cascade delete на FK |

---

## Блок Equipment

### 1. EquipmentTypes

Классификатор техники: плоский список типов (Компрессор, Экскаватор, Самосвал...).
Определяет mobilityType и — через EquipmentTypeProperties — набор динамических свойств.

| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID PK | |
| name | VARCHAR | Компрессор, Экскаватор, Самосвал... |
| mobilityType | ENUM | SelfPropelled / NonSelfPropelledMotorized / Stationary / NonMotorized |
| requiresTransport | BOOLEAN | Требуется транспортная техника для доставки |
| sortOrder | INT | Порядок отображения в UI |

---

### 2. Fleets

Логическая группа оборудования под одним Fleet Owner (AAD-группа).

| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID PK | |
| name | VARCHAR | |
| type | ENUM | Maintenance / Construction / TFM / SCMLogistics / ExternalBP |
| isExternal | BOOLEAN | TRUE для БП-флотов |
| aadGroupId | VARCHAR | ID AAD-группы Fleet Owner |

---

### 2а. FleetDelegations

Кому из конкретных пользователей Fleet Owner временно делегировал свои права.

| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID PK | |
| fleetId | FK → Fleets | |
| userId | FK → Users | Кому делегировано |
| delegatedAt | TIMESTAMP | |
| expiresAt | TIMESTAMP | NULL = бессрочно |
| delegatedBy | FK → Users | Кто делегировал |

---

### 3. Equipments

Единица техники. Единая таблица для всей техники: TCO Owned, Long-term rented, On-demand BP.
Поля stateNumber и serialNumber — опциональны для BP-техники (BRD v8 §5.1).

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
| brand | VARCHAR | Отображается в поиске |
| model | VARCHAR | Отображается в поиске |
| shareType | ENUM | Shared / SharedWithConditions / Assigned |
| serviceZoneCode | VARCHAR | Код сервисной зоны (строка из PSWS) |
| costCenter | VARCHAR | Финансовый ЦЗ (строка из JDE) |
| baseLocation | VARCHAR | Базовое местоположение (из Master Fleet) |
| isDecommissioned | BOOLEAN | DEFAULT FALSE |
| decommissionedAt | DATE nullable | Дата списания |

> **Динамический статус** (не хранится, вычисляется при запросе):
> - **Decommissioned** → `isDecommissioned = true`
> - **InRepair** → активная запись в EquipmentRepairHistory (`actualEndsAt IS NULL`)
> - **Frozen** → активная запись в EquipmentFreezes (`startsAt ≤ today AND (endsAt IS NULL OR endsAt ≥ today) AND cancelledAt IS NULL`)
> - **Available** → ни одно из условий выше не выполняется

---

### 4. EquipmentPhotos

Одна единица техники — несколько фотографий.

| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID PK | |
| equipmentId | FK → Equipments | |
| url | VARCHAR | |
| sortOrder | INT | |
| isPrimary | BOOLEAN | Главная фотография |

---

## EAV-блок: динамические свойства техники

Разные типы техники имеют принципиально разные атрибуты (мощность, объём, тип привода, грузоподъёмность и т.д.). EAV позволяет хранить любой набор свойств без изменения схемы. Набор свойств для каждого типа задаётся через Admin Panel.

### 5. MeasurementUnits

Справочник единиц измерения: м, кг, кВт, л, бар, м³ и т.д.

| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID PK | |
| code | VARCHAR | м / кг / кВт / л / бар / м³... |
| displayName | VARCHAR | |
| sortOrder | INT | |

---

### 6. Properties

Справочник свойств техники (что может быть измерено/описано): мощность, объём, тип привода и т.п.

| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID PK | |
| name | VARCHAR | |
| dataType | ENUM | number / double / text / boolean / enum |
| unitId | FK → MeasurementUnits nullable | Только для number/double |

---

### 7. PropertyEnumValues

Допустимые значения для enum-свойств.
Пример: свойство "Тип привода" → значения "Дизельный", "Электрический".

| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID PK | |
| propertyId | FK → Properties | |
| value | VARCHAR | |
| sortOrder | INT | |

---

### 8. EquipmentTypeProperties

Схема свойств для каждого типа техники: какие свойства показывать, какие обязательны, какие фильтруемы.
Управляется через Admin Panel.

| Поле | Тип | Описание |
|------|-----|----------|
| equipmentTypeId | FK → EquipmentTypes | |
| propertyId | FK → Properties | |
| isRequired | BOOLEAN | Обязательное при создании |
| isFilterable | BOOLEAN | Используется как фильтр в поиске |
| isVisibleInCard | BOOLEAN | Показывать в карточке техники |
| sortOrder | INT | |
| PK | — | (equipmentTypeId, propertyId) |

---

### 9. EquipmentProperties

Фактические значения динамических свойств конкретной единицы техники.
Значение хранится в JSONB; заполняется только ключ, соответствующий dataType свойства.

| Поле | Тип | Описание |
|------|-----|----------|
| equipmentId | FK → Equipments | |
| propertyId | FK → Properties | |
| value | JSONB | `{"Bool": null, "Double": 6.7, "Number": null, "String": null}` |
| PK | — | (equipmentId, propertyId) |

**Пример: компрессор (equipmentId = 42)**

| propertyId | Свойство | dataType | value |
|------------|----------|----------|-------|
| 1 | Объём ресивера | double | `{"Double": 500.0}` |
| 2 | Рабочее давление | double | `{"Double": 6.7}` |
| 3 | Количество ступеней | number | `{"Number": 2}` |
| 4 | Тип привода | enum | `{"String": "Дизельный"}` |

---

### 10. EquipmentTrackers

История привязки трекеров к технике.
Один equipment может иметь несколько активных трекеров разных типов одновременно (WIALON + IVMS).

| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID PK | |
| equipmentId | FK → Equipments | |
| trackerType | ENUM | WIALON / IVMS / Vega / Teltonika250 / Teltonika650 |
| externalId | VARCHAR | ID во внешней системе (group ID / reg number / device ID) |
| assignedAt | TIMESTAMP | |
| removedAt | TIMESTAMP nullable | NULL = трекер активен |

---

### 11. EquipmentFreezes

Заморозки техники Fleet Owner-ом (резерв, CAP-ремонт, ожидание ТО).
Хранит полную историю. Активная запись = техника недоступна для бронирования.

| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID PK | |
| equipmentId | FK → Equipments | |
| reason | TEXT | |
| startsAt | DATE | |
| endsAt | DATE nullable | NULL = бессрочно |
| cancelledAt | TIMESTAMP nullable | Если отменена досрочно |

---

### 12. EquipmentRepairHistory

История ремонтов техники. Данные вводятся вручную или синхронизируются из GDA.
Активная запись (actualEndsAt IS NULL) = техника "в ремонте".

| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID PK | |
| equipmentId | FK → Equipments | |
| repairType | VARCHAR | CAP-ремонт, ТО, аварийный... |
| startsAt | DATE | |
| expectedEndsAt | DATE | Плановая дата окончания |
| actualEndsAt | DATE nullable | NULL = ещё в ремонте |
| notes | TEXT nullable | |
| source | ENUM | Manual / GDA |

---

### 13. MaintenancePartners

Справочник подрядчиков по ТО. Один equipment может обслуживаться разными партнёрами по разным направлениям.

| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID PK | |
| name | VARCHAR | Название ДП |
| contactInfo | TEXT nullable | Контактные данные |

---

### 14. EquipmentMaintenanceContracts

Связь техники и партнёра ТО с указанием направления обслуживания.

| Поле | Тип | Описание |
|------|-----|----------|
| equipmentId | FK → Equipments | |
| partnerId | FK → MaintenancePartners | |
| serviceType | VARCHAR | Направление: замена масла / колёса / электрика... |
| notes | TEXT nullable | |
| PK | — | (equipmentId, partnerId, serviceType) |

---

### 15. SystemSettings

Глобальные настройки системы, управляемые через Admin Panel. Не hardcode.

| Поле | Тип | Описание |
|------|-----|----------|
| key | VARCHAR PK | Идентификатор настройки |
| value | VARCHAR | Значение |
| updatedAt | TIMESTAMP | |

Примеры ключей: `booking.horizonDays`, `booking.maxDurationDays`, `fo.timeoutHours24`, `fo.timeoutHours48`

---

### 16. Users

Локальный справочник пользователей. TCO-пользователи — зеркало AAD. BP-пользователи — собственные группы (не AAD).

| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID PK | |
| externalId | VARCHAR | AAD object ID (для TCO) или ID из системы БП |
| email | VARCHAR | |
| displayName | VARCHAR | |
| departmentCode | VARCHAR nullable | Код подразделения |
| userType | ENUM | TCO / BP |

---

## Блок Booking

### 1. BookingRequests

Заявка на бронирование техники. Контейнер, объединяющий один или несколько объектов Bookings — по одному на каждую единицу техники. Создаётся Requestor-ом или SWP. Один Work Order — одна заявка; все брони внутри разделяют один WO, локацию и приоритет (BRD v11 §3, §5.3).

| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID PK | |
| requestNumber | SERIAL UNIQUE NOT NULL | Порядковый номер (INT); отображается в UI как REQ-YYYY-NNNNN |
| type | ENUM NOT NULL | Regular / ServiceWork |
| status | ENUM NOT NULL | Draft / Submitted / InProgress / Completed / Cancelled |
| workOrderJdeId | VARCHAR nullable | Номер WO из JDE E1; обязателен для Maintenance / Railroad / Operations |
| isDefaultWorkOrder | BOOLEAN NOT NULL DEFAULT FALSE | TRUE — подразделение не работает в JDE, номер WO не требуется (FR-NEW-51) |
| location | VARCHAR nullable | Локация; обязательна для SCM Logistics вместо WO |
| workDescription | VARCHAR NOT NULL | Описание работ (~50 символов, мягкое ограничение) |
| comments | TEXT nullable | Необязательный комментарий заявителя |
| priority | ENUM NOT NULL | P1 / P2 / P3 / P4 |
| createdBy | UUID NOT NULL | ID заявителя (внешний; Users без FK-ограничения) |
| createdAt | TIMESTAMP NOT NULL | Момент создания заявки |

> **Жизненный цикл статуса:**
> - **Draft** → создана, не отправлена
> - **Submitted** → отправлена; хотя бы одна бронь ожидает решения FO
> - **InProgress** → хотя бы одна бронь перешла в InProgress
> - **Completed** → все активные брони закрыты
> - **Cancelled** → отменена заявителем до отправки

**Индексы:** `createdBy`, `status`, `requestNumber`; partial на `workOrderJdeId WHERE workOrderJdeId IS NOT NULL`

---

### 2. Bookings

Атомарная единица бронирования: одна конкретная единица техники в рамках одной заявки. Обрабатывается Fleet Owner-ом независимо от остальных броней в заявке (частичное подтверждение, FR-NEW-16). FO может редактировать бронь до и после подтверждения без повторного согласования (решение 10.04).

Для техники с `EquipmentTypes.requiresTransport = TRUE` — после подтверждения брони FO обязан привязать транспортную бронь через `transportBookingId`. Статус переходит в `TransportConfirmed` как только транспорт выбран и привязан.

| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID PK | |
| requestId | UUID NOT NULL FK → BookingRequests | Родительская заявка |
| equipmentId | UUID NOT NULL | ID единицы техники (внешний; Equipments без FK-ограничения) |
| fleetId | UUID NOT NULL | ID парка техники (внешний; Fleets без FK-ограничения) |
| assignedFleetOwnerId | UUID nullable | Текущий ответственный FO (внешний; меняется при делегировании) |
| status | ENUM NOT NULL | Draft / Submitted / Confirmed / **TransportConfirmed** / Declined / Revoked / Terminated / InProgress / Closed / EquipmentChanged / Extended |
| startDt | TIMESTAMP NOT NULL | Начало бронирования = запланированное начало мобилизации (FR-NEW-18) |
| endDt | TIMESTAMP NOT NULL | Окончание бронирования |
| mobilizationStartedAt | TIMESTAMP nullable | Фактический старт мобилизации — проставляется кнопкой FO «Mobilization started» (FR-NEW-24) |
| justification | TEXT nullable | Обоснование; обязательно по бизнес-логике для Assigned и SharedWithConditions |
| **transportBookingId** | **UUID nullable FK → Bookings** | **Self-ref: ID брони транспортной техники, которая везёт данную единицу. Только для unwheeled-техники (requiresTransport = TRUE). NULL до момента привязки транспорта FO-ом.** |
| declineReason | TEXT nullable | Причина отклонения; обязательна при переходе в Declined (FR-050) |
| terminateReason | TEXT nullable | Причина досрочного завершения; обязательна при переходе в Terminated (FR-059) |
| closedBy | UUID nullable | Кто закрыл бронь (внешний; Users без FK-ограничения) |
| closedAt | TIMESTAMP nullable | Момент закрытия (FR-NEW-32) |
| closeComment | TEXT nullable | Комментарий заявителя при закрытии брони |

> **Жизненный цикл статуса:**
>
> **Обычная техника (requiresTransport = FALSE):**
> `Draft → Submitted → Confirmed → InProgress → Closed`
>
> **Unwheeled-техника (requiresTransport = TRUE):**
> `Draft → Submitted → Confirmed → TransportConfirmed → InProgress → Closed`
>
> Прочие переходы — общие для обоих путей:
> - **Draft** → создана вместе с заявкой
> - **Submitted** → заявка отправлена; ожидает решения FO
> - **Confirmed** → FO подтвердил бронь; для unwheeled — ожидает привязки транспорта
> - **TransportConfirmed** → FO привязал транспортную бронь (`transportBookingId` заполнен); только для unwheeled
> - **Declined** → FO отклонил (причина обязательна)
> - **Revoked** → заявитель отозвал до обработки FO
> - **Terminated** → завершена досрочно (FO или заявитель)
> - **InProgress** → FO нажал «Mobilization started»
> - **Closed** → ручное закрытие с фиксацией closedBy + closedAt
> - **EquipmentChanged** → FO предложил замену техники (FR-046); детали замены в BookingStatusHistories.previousSnapshot
> - **Extended** → бронь продлена (FR-061); старый endDt в BookingStatusHistories.previousSnapshot

> **Транспортный откат:** если транспортная бронь (на которую ссылается `transportBookingId`) отменяется (Declined / Terminated / Revoked) — приложение возвращает unwheeled-бронь в `Confirmed` и обнуляет `transportBookingId`. Каскад на уровне FK не используется.

**Индексы:** `requestId`, `equipmentId`, `fleetId`, `status`, `transportBookingId`; составной `(equipmentId, startDt, endDt)` — для проверки доступности при создании брони

---

### 3. BookingStatusHistories

Полная история переходов статусов для каждой брони. Запись создаётся при каждом изменении `Bookings.status`. Все переходы — пользовательские (системных автоматических переходов нет; BRD v11).

| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID PK | |
| bookingId | UUID NOT NULL FK → Bookings | Бронь, для которой фиксируется переход |
| oldStatus | ENUM nullable | Предыдущий статус; NULL при первой записи (создание брони) |
| newStatus | ENUM NOT NULL | Новый статус после перехода |
| changedBy | UUID NOT NULL | ID пользователя-инициатора (внешний; Users без FK-ограничения) |
| changedAt | TIMESTAMP NOT NULL | Момент перехода |
| comment | TEXT nullable | Причина или контекст (например, текст declineReason или terminateReason) |
| previousSnapshot | JSONB nullable | Снимок состояния Bookings до перехода. Заполняется при:<br>• **EquipmentChanged** → `{"equipmentId": "...", "fleetId": "...", "assignedFleetOwnerId": "..."}`<br>• **Extended** → `{"endDt": "..."}`<br>• **TransportConfirmed** → `{"transportBookingId": null}` (фиксирует факт первичной привязки транспорта)<br>• **Confirmed** (откат от TransportConfirmed) → `{"transportBookingId": "..."}` (фиксирует обнулённый ID при транспортном откате)<br>При остальных переходах — NULL. |

**Индексы:** `bookingId`, `changedAt`

---

### 4. RequestStatusHistories

История переходов статуса заявки. Аналог BookingStatusHistories, но для агрегированного статуса BookingRequests. Позволяет восстановить хронологию заявки без агрегации по дочерним бронями.

| Поле | Тип | Описание |
|------|-----|----------|
| id | UUID PK | |
| requestId | UUID NOT NULL FK → BookingRequests | Заявка, для которой фиксируется переход |
| oldStatus | ENUM nullable | Предыдущий статус; NULL при создании заявки |
| newStatus | ENUM NOT NULL | Новый статус (Draft / Submitted / InProgress / Completed / Cancelled) |
| changedBy | UUID NOT NULL | ID пользователя-инициатора (внешний; Users без FK-ограничения) |
| changedAt | TIMESTAMP NOT NULL | Момент перехода |
| comment | TEXT nullable | |

**Индексы:** `requestId`, `changedAt`

---

## Сводный список таблиц

### Equipment-блок

| # | Таблица | Назначение |
|---|---------|-----------|
| 1 | EquipmentTypes | Классификатор типов техники |
| 2 | Fleets | Парки техники |
| 2а | FleetDelegations | Делегирование прав FO |
| 3 | Equipments | Единица техники (TCO + BP unified) |
| 4 | EquipmentPhotos | Фотографии техники |
| 5 | MeasurementUnits | Справочник единиц измерения |
| 6 | Properties | Справочник свойств техники |
| 7 | PropertyEnumValues | Допустимые значения enum-свойств |
| 8 | EquipmentTypeProperties | Схема свойств по типу (Admin Panel) |
| 9 | EquipmentProperties | Значения свойств конкретной техники |
| 10 | EquipmentTrackers | История привязки трекеров |
| 11 | EquipmentFreezes | Заморозки техники (история) |
| 12 | EquipmentRepairHistory | История ремонтов |
| 13 | MaintenancePartners | Справочник ДП по ТО |
| 14 | EquipmentMaintenanceContracts | Связь техники и партнёра ТО |
| 15 | SystemSettings | Глобальные настройки Admin Panel |
| 16 | Users | Справочник пользователей (TCO + BP) |

### Booking-блок

| # | Таблица | Назначение |
|---|---------|-----------|
| 1 | BookingRequests | Заявка на бронирование (контейнер; один WO — одна заявка) |
| 2 | Bookings | Атомарная единица бронирования (одна техника); self-ref FK `transportBookingId` для unwheeled |
| 3 | BookingStatusHistories | История переходов статуса брони (+ previousSnapshot) |
| 4 | RequestStatusHistories | История переходов статуса заявки |

**Итого: 20 таблиц** (количество не изменилось — транспорт добавлен через поле, а не новую таблицу)

---

## Связи между блоками

| Поле Booking-блока | Ссылается на | Тип связи |
|--------------------|--------------|-----------|
| BookingRequests.createdBy | Users.id | Внешний UUID, без FK-ограничения |
| Bookings.equipmentId | Equipments.id | Внешний UUID, без FK-ограничения |
| Bookings.fleetId | Fleets.id | Внешний UUID, без FK-ограничения |
| Bookings.assignedFleetOwnerId | Users.id | Внешний UUID, без FK-ограничения |
| Bookings.closedBy | Users.id | Внешний UUID, без FK-ограничения |
| **Bookings.transportBookingId** | **Bookings.id** | **Self-referencing FK (WITH FK-ограничением); nullable** |
| BookingStatusHistories.changedBy | Users.id | Внешний UUID, без FK-ограничения |
| RequestStatusHistories.changedBy | Users.id | Внешний UUID, без FK-ограничения |

Отсутствие FK-ограничений — намеренное архитектурное решение: сервисы Users, Equipments и Fleets рассматриваются как внешние по отношению к Booking-блоку. Исключение: `transportBookingId` — self-referencing внутри той же таблицы, FK-ограничение оправдано.

---

## Закрытые открытые вопросы

| # | Вопрос | Решение |
|---|--------|---------|
| OQ-DB-1 | История изменений shareType у техники? | Не нужна — хранится только текущее значение |
| OQ-DB-2 | Users.departmentCode — Cost Center или код подразделения? | Отложено |
| OQ-DB-3 | Аудит-поля (createdAt/updatedAt/createdBy) — на все таблицы? | Добавить в следующей итерации |
| OQ-DB-4 | RequestStatusHistory нужна? | Да — добавлена |
| OQ-DB-5 | BookingSnapshot — отдельная таблица или JSONB в Booking? | **CLOSED**: отдельная таблица не нужна; previousSnapshot JSONB в BookingStatusHistories |
| OQ-DB-8 | Продление брони — отдельная таблица BookingExtensions? | **CLOSED**: статус `Extended` + previousSnapshot.endDt в BookingStatusHistories |
| OQ-DB-9 | **Unwheeled-флоу** (FR-NEW-29): TransportRequests или расширение Bookings? | **CLOSED**: поле `transportBookingId` (self-ref FK) + статус `TransportConfirmed` в Bookings; TransportRequests не нужна |
| OQ-DB-10 | Замена техники FO — originalEquipmentId или BookingEquipmentHistory? | **CLOSED**: статус `EquipmentChanged` + previousSnapshot в BookingStatusHistories |

---

## Открытые вопросы

| # | Вопрос | Impact |
|---|--------|--------|
| OQ-DB-11 | **Скоуп транспортной брони**: транспортная бронь (на которую ссылается `transportBookingId`) создаётся в рамках того же `requestId`, что и unwheeled-бронь — или в отдельном BookingRequest? Вариант «тот же request» проще, но требует, чтобы FO мог добавлять строки в уже существующую заявку Requestor-а. | Архитектура процесса, v4 |
