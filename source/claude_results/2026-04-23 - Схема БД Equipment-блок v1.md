# Схема БД: Equipment-блок v1

**Дата создания:** 2026-04-23  
**Последнее обновление:** 2026-04-23  
**Автор:** Тельман Нуржанов (SA)

---

## Принятые решения

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

---

## Таблицы

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

## Вспомогательные таблицы (добавляем в этой итерации)

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
| userType | ENUM | TCO / BP | |

---

## Итоговый список таблиц

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

---

## Открытые вопросы

| # | Вопрос | Impact |
|---|--------|--------|
| OQ-DB-1 | Нужно ли хранить историю изменений shareType у техники (Shared → Assigned)? Или только текущее значение? | Audit trail, отчёты |
| OQ-DB-2 | Users.departmentCode — это Cost Center из JDE или отдельный код подразделения? | Нормализация данных |
| OQ-DB-3 | Аудит-поля (createdAt/updatedAt/createdBy) — добавить на все таблицы или только на Equipments? | Стандарт разработки |

---

## Следующая итерация

Блок **Bookings / Requests**: таблицы Request, Booking, BookingSnapshot, BookingStatusHistory, TransportSubrequest и пр.
