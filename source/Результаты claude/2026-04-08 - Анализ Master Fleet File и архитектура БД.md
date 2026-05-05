# Анализ Master Fleet File HDV/HDE
## Структура данных, сущности и рекомендации по архитектуре БД

**Дата:** 2026-04-08  
**Документ:** Анализ исходного файла и дизайн БД  
**Источник:** FINAL Master fleet file HDV HDE.xlsx

---

## ЧАСТЬ 1 — Структура файла и содержимое

### Обзор листов

| Лист | Строк | Колонок | Назначение |
|------|-------|---------|-----------|
| **Main** | 786 | 85 | Основной каталог техники (786 единиц) |
| **Written off equipment** | 104 | 87 | Списанная техника (исторические данные) |
| **directory all** | 72 | 4 | Справочник категорий и классификаций |
| **CC directory** | 19,075 | 17 | Справочник Cost Centers из JDE E1 |
| **Trackers** | 30 | 17 | Конфигурация систем отслеживания (WIALON, IVMS, Vega) |
| **Glossary** | 77 | 5 | Глоссарий определений и аббревиатур |

### Состав Main-листа (786 единиц техники)

**По Operation Status:**
- Active: 464 (59%)
- Sent to BP (внешний флот): 273 (35%)
- Reserved (зарезервирована): 48 (6%)

**По Equipment Class:**
- HDE (Heavy Duty Equipment): 608 (77%)
- HDV (Heavy Duty Vehicle): 177 (23%)

---

## ЧАСТЬ 2 — Детальный анализ полей и сущностей

### Группа 1: GENERAL INFO (20 полей) — Базовые свойства техники

| # | Поле | Тип | Назначение | Примечание |
|---|------|-----|-----------|----------|
| 1 | **Equipment** | VARCHAR(50) | Уникальный ID техники | PK: 108019, 108020... |
| 2 | **Operation Status** | ENUM | Статус операции | Active, Reserved, Sent to BP |
| 3 | **License** | VARCHAR(50) | Лицензионный/регистрационный номер | H725152, H089106 |
| 4 | **Description** | VARCHAR(200) | Полное описание техники | "TRUCK HD RAILROAD INTERNATIONAL 7400 SBA" |
| 5 | **Service Zone** | VARCHAR(50) | Зона обслуживания | TMF1 (Tengiz Main Field 1), другие |
| 6 | **Class** | VARCHAR(50) | Класс техники | HDV / HDE |
| 7 | **Location** | VARCHAR(50) | Текущее размещение | Tengiz, другие |
| 8 | **Serial Number** | VARCHAR(50) | Серийный номер | 1HTWEAAN9BJ374649 |
| 9 | **VIN/Chassis No** | VARCHAR(50) | VIN номер | 1HTWEAAN9BJ374649 |
| 10 | **CC code** | VARCHAR(20) | Cost Center код | 20053, 13780 |
| 11 | **CC status** | VARCHAR(50) | Статус CC (derived from CC directory) | Формула VLOOKUP |
| 12 | **Division code** | VARCHAR(20) | Код отделения (derived) | Формула VLOOKUP |
| 13 | **CC name** | VARCHAR(100) | Наименование CC (derived) | Формула VLOOKUP |
| 14 | **Division name** | VARCHAR(100) | Наименование отделения (derived) | Формула VLOOKUP |
| 15 | **Group code** | VARCHAR(20) | Код группы (derived) | Формула VLOOKUP |
| 16 | **Group name** | VARCHAR(100) | Наименование группы (derived) | Формула VLOOKUP |
| 17 | **Department code** | VARCHAR(20) | Код отдела (derived) | Формула VLOOKUP |
| 18 | **Department name from WP** | VARCHAR(100) | Название отдела (derived) | Формула VLOOKUP |
| 19 | **Department name** | VARCHAR(100) | Название отдела (из рапорта) | "FIELD & EXPORT" |
| 20 | **User Department** | VARCHAR(100) | Отдел пользователя | "Rail Road Maintenance" |

**Ключевой вывод:** Поля 11-18 — это DERIVED (вычисляемые) поля через VLOOKUP из CC directory. В БД их не нужно хранить дублей; достаточно FK на Cost Center.

### Группа 2: FROM REPORT (36 полей) — Данные из отчёта (контрактно-техническая информация)

| # | Поле | Тип | Назначение |
|---|------|-----|-----------|
| 21 | Title | VARCHAR(100) | Должность пользователя |
| 22 | Fleet | VARCHAR(50) | Парк/флот принадлежности |
| 23 | Allocation | VARCHAR(100) | Распределение/назначение |
| 24 | Writeoff Status | VARCHAR(50) | Статус списания |
| 25 | Fleet Active Date | DATE | Дата активации в парке |
| 26 | **Criticality** | VARCHAR(50) | Критичность техники (Low/High/Critical) |
| 27 | Manufacturer | VARCHAR(100) | Производитель |
| 28 | Model | VARCHAR(100) | Модель |
| 29 | Currency | CHAR(3) | Валюта (USD, EUR и т.д.) |
| 30 | Capital Cost | DECIMAL | Стоимость капитала |
| 31 | Year | YEAR | Год выпуска |
| 32 | Age of Fleet (Yrs) | INT | Возраст техники в годах |
| 33 | Body No | VARCHAR(50) | Номер кузова |
| 34 | Engine No | VARCHAR(50) | Номер двигателя |
| 35 | Date of Registr. In GAI | DATE | Дата регистрации в органах |
| 36 | Technical Passport No | VARCHAR(50) | № технического паспорта |
| 37 | Date Acquired by user | DATE | Дата приобретения |
| 38 | Life Cycle 2 (years) | INT | Жизненный цикл в годах |
| 39 | Commission Date | DATE | Дата ввода в эксплуатацию |
| 40 | Withdrawal date | DATE | Дата вывода из эксплуатации |
| 41 | Total Counter Reading | DECIMAL | Общее показание счётчика |
| 42 | Last Reading | DECIMAL | Последнее показание (км) |
| 43 | Last Reading Motor Hour | DECIMAL | Последнее показание (мо-часы) |
| 44 | FrontTireSize | VARCHAR(50) | Размер переднего колеса |
| 45 | RearTireSize | VARCHAR(50) | Размер заднего колеса |
| 46 | FleetCode | VARCHAR(20) | Код парка (внутренний) |
| 47 | PO | VARCHAR(50) | P.O. (Posting Order) |
| 48 | Replace in Year | YEAR | Год замены (плановый) |
| 49 | TCO Requested Replace in Year | YEAR | Год замены (запрос TCO) |
| 50 | Replaced by | VARCHAR(50) | Заменена на (ID техники) |
| 51 | TCO Charge | VARCHAR(50) | Начисление TCO |
| 52 | ID | VARCHAR(50) | Дополнительный ID |
| 53 | Eng Model | VARCHAR(100) | Модель двигателя |
| 54 | Transmission model | VARCHAR(100) | Модель коробки передач |
| 55 | NOTES | TEXT | Примечания |

**Ключевой вывод:** Многие из этих полей относятся к технико-хозяйственной части, которая НЕ требуется для Booking Tool Phase 1. Это WRITE-ONCE данные (т.е. не изменяются часто).

### Группа 3: TRACKER INFO (7 полей) — Системы отслеживания

| # | Поле | Тип | Назначение |
|---|------|-----|-----------|
| 56 | Tracker existence | BOOLEAN | Есть ли трекер на технике |
| 57 | WIALON Qty | INT | Кол-во WIALON устройств |
| 58 | WIALON "group" | VARCHAR(100) | Группа в WIALON |
| 59 | IVMS Qty | INT | Кол-во IVMS устройств |
| 60 | IVMS "Registration number" | VARCHAR(50) | Номер в IVMS |
| 61 | Vega Qty | INT | Кол-во Vega устройств |
| 62 | Vega "deviceId" | VARCHAR(50) | Device ID в Vega |

**Ключевой вывод:** Эти данные связаны с внешними системами мониторинга. Нужна отдельная сущность **Equipment_Tracker** для связей.

### Группа 4: ADDITIONAL INFO (13 полей) — Дополнительные характеристики и классификация

| # | Поле | Тип | Назначение | Связь |
|---|------|-----|-----------|------|
| 63 | **Assigned location** | VARCHAR(100) | Назначенное место (если Assigned status) | Ссылка на Location справочник |
| 64 | **HDV/HDE classification** | VARCHAR(100) | Классификация: Self-propelled motorized / Not self-propelled / Stationary / Non-motorized | **СПРАВОЧНИК** |
| 65 | **Expected metrics from trackers** | VARCHAR(100) | Ожидаемые метрики: Mileage, engine hours, GPS | ENUM |
| 66 | **Category** | VARCHAR(100) | Категория техники: Air compressor, Backhoe loader, Boom truck и т.д. | **СПРАВОЧНИК** из directory all |
| 67 | **Equipment Usage Status** | VARCHAR(100) | Статус использования: Assigned / Shared without conditions / Shared with conditions | **СПРАВОЧНИК** |
| 68 | Justification if "Assigned" | TEXT | Обоснование (если техника в статусе Assigned) | Conditional field |
| 69 | Weight-based, kg | DECIMAL | Вес техники в кг | Technical spec |
| 70 | Volume-based, m³ | DECIMAL | Объём в м³ | Technical spec |
| 71 | System Voltage, V | INT | Системное напряжение, В | Technical spec |
| 72 | Battery Capacity, Ah | INT | Ёмкость батареи, А·ч | Technical spec |
| 73 | Current, A | INT | Ток, А | Technical spec |
| 74 | Alternator Output, kvA | DECIMAL | Выходная мощность, кВА | Technical spec |
| 75 | Pneumatic Pressure, ft³ (CFM) | INT | Пневматическое давление, CFM | Technical spec |
| 76 | Work output, m³/h | DECIMAL | Производительность, м³/ч | Technical spec |
| 77 | Maintenance BP | VARCHAR(50) | BP для обслуживания | FK на Business Partner |
| 78 | Contact number | VARCHAR(20) | Контактный номер | Phone |
| 79 | Contact e-mail | VARCHAR(100) | Email контакта | Email |
| 80 | if required | VARCHAR(50) | Флаг "если требуется" | Boolean |
| 81 | km/day | INT | Плановый пробег в км/день | Usage metric |
| 82 | mh/day | INT | Плановые мото-часы в день | Usage metric |
| 83 | comments | TEXT | Комментарии | Free text |
| 84 | General comments | TEXT | Общие примечания | Free text |
| 85 | Assignee | VARCHAR(100) | Ответственное лицо | FK на Employee |

---

## ЧАСТЬ 3 — Справочники (Directories)

### Справочник 1: directory all — Классификация и статусы

**Состав (72 строки):**

| Колонка | Содержание | Пример |
|---------|-----------|---------|
| **Category** | Типы техники | Air compressor electric, Backhoe loader, Boom truck, Bulldozer, Chemical truck, Cleaning tools, Concrete mixer truck, Crane, Dump truck, Excavator, Generator, Grader, Mixer, Pump, Roller, Tractor, Trailer, Water truck и т.д. |
| **Equipment Usage Status** | Статусы использования (3 значения) | Assigned, Shared without conditions, Shared with conditions |
| **HDV/HDE classification** | Классификация по мобильности (4 значения) | Self-propelled motorized (HDV/HDE) - Wheeled, Self-propelled motorized (HDV/HDE) - Tracked, Not self-propelled motorized HDE, Stationary HDE, Non-motorized HDE |
| **Expected metrics from trackers** | Ожидаемые метрики для каждого типа | Mileage + engine hours + GPS, Engine hours + GPS, GPS only и т.д. |

**Назначение:** Это справочник, который используется для валидации и заполнения выпадающих списков в Main листе.

### Справочник 2: CC directory — Cost Centers (19,075 записей)

**Из JDE E1, импортируется периодически**

| # | Колонка | Тип | Назначение |
|---|---------|-----|-----------|
| 1 | **CostCenterCode** | VARCHAR(20) | PK: код cost center (20053, 13780) |
| 2 | CostCenterStatus | VARCHAR(50) | ACTIVE / INACTIVE |
| 3 | DivisionName | VARCHAR(100) | Название отделения |
| 4 | DivisionCd | VARCHAR(20) | Код отделения |
| 5 | GroupName | VARCHAR(100) | Название группы |
| 6 | CostCenterGrpCode | VARCHAR(20) | Код группы |
| 7 | DepartmentName | VARCHAR(100) | Название отдела |
| 8 | CostCenterDepartmentCode | VARCHAR(20) | Код отдела |
| 9 | **CostCenterName** | VARCHAR(200) | Полное название CC |
| 10 | CostCenterParentCode | VARCHAR(20) | FK на родительский CC |
| 11 | (пусто) | — | — |
| 12 | CostCenterGroupCode | VARCHAR(20) | Альтернативный код группы |
| 13 | PrimaryOwnerBadge | VARCHAR(20) | Badge ID владельца |
| 14 | SecondaryOwnerBadge | VARCHAR(20) | Badge ID вторичного владельца |
| 15 | CostCenterDescription1 | VARCHAR(200) | Описание 1 |
| 16 | CostCenterDescription2 | VARCHAR(200) | Описание 2 |
| 17 | CategoryCodeBusinessUnit17 | VARCHAR(20) | Категория BU |

**Примечание:** Это REFERENCE DATA из JDE, которое должно быть синхронизировано в Booking Tool для: а) валидации CC code, б) получения иерархии Division → Group → Department, в) поиска владельца CC для согласования.

### Справочник 3: Trackers — Конфигурация систем отслеживания

**Задача:** Настройка, какие метрики требуются для каждого типа техники

Например:
- Self-propelled motorized: Mileage + engine hours + GPS
- Not self-propelled motorized: Engine hours + GPS
- Stationary: GPS only
- Non-motorized: GPS only

---

## ЧАСТЬ 4 — Потребности для Booking Tool Phase 1

### Минимальный набор полей (для бронирования)

**ОБЯЗАТЕЛЬНО ИМЕТЬ:**

```sql
Equipment_Master
├── equipment_id (PK)          # Equipment
├── equipment_status          # Operation Status
├── license_plate             # License
├── description               # Description
├── service_zone              # Service Zone
├── equipment_class           # Class (HDV/HDE)
├── location                  # Location
├── cost_center_id (FK)       # CC code → CC directory
├── hdv_hde_classification    # HDV/HDE classification
├── equipment_category        # Category (из directory all)
├── equipment_usage_status    # Assigned / Shared without conditions / Shared with conditions
├── assigned_location         # Assigned location (если Assigned)
├── justification             # Justification if Assigned
├── criticality              # Criticality (Low/High/Critical)
├── fleet_id (FK)            # Fleet
├── writeoff_status          # Writeoff Status
├── fleet_active_date        # Fleet Active Date
└── last_updated             # Дата последнего обновления
```

**ОПЦИОНАЛЬНО (для Phase 1.5+):**
- Manufacturer, Model, Engine No, Technical specs (вес, напряжение и т.д.)
- Tracker info (для интеграции с системами мониторинга)
- Contacts и Maintenance BP info

### Иерархия собственности техники

```
Cost Center (из JDE)
├── Division
├── Group
├── Department
└── Equipment (один CC может иметь несколько Equipment)
```

**Проблема в Master File:** Из него можно получить equipment → CC code, но нужно ИМЕТЬ CC directory в Booking Tool для получения полной иерархии и Owner информации.

### Маппинг Fleet Owners

**В текущем Master File нет явной информации о том, какой Fleet Owner отвечает за каждую единицу техники.**

**Решение:**
1. Fleet Owner = CC Owner (Cost Center Owner) — из CC directory (PrimaryOwnerBadge, SecondaryOwnerBadge)
2. Или нужна отдельная таблица Fleet_Owner_Assignment, которая маппирует:
   - Fleet ID → Fleet Owner Badge
   - Equipment ID → Fleet Owner Badge

**Вывод:** Нужно уточнить с заказчиком:
- Совпадают ли Fleet Owners и CC Owners?
- Или один Fleet Owner может управлять техникой из разных CC?

---

## ЧАСТЬ 5 — Рекомендации по архитектуре БД

### 5.1 Основные сущности

```
┌─────────────────────────────────────────────────────────────┐
│                    EQUIPMENT DOMAIN                          │
└─────────────────────────────────────────────────────────────┘

├─ Equipment (786 записей из Main)
│   ├─ equipment_id (PK)
│   ├─ basic_info (Equipment, License, Description, Serial No, VIN)
│   ├─ classification (Equipment Class, Category, HDV/HDE class)
│   ├─ location_info (Location, Service Zone, Assigned Location)
│   ├─ ownership (Cost Center FK, Fleet FK, Department, Division)
│   ├─ usage_status (Operation Status, Equipment Usage Status, Criticality)
│   ├─ technical_specs (Weight, Volume, Voltage, Battery, etc.)
│   └─ metadata (created_at, updated_at, source)
│
├─ Cost_Center (19,075 записей из CC directory)
│   ├─ cc_code (PK)
│   ├─ cc_name
│   ├─ cc_status
│   ├─ division (FK)
│   ├─ group (FK)
│   ├─ department (FK)
│   ├─ primary_owner_badge
│   ├─ secondary_owner_badge
│   └─ parent_cc_code (FK self)
│
├─ Equipment_Category (справочник)
│   ├─ category_id (PK)
│   ├─ category_name
│   ├─ hdv_hde_classification (FK)
│   └─ expected_tracker_metrics
│
├─ Equipment_Classification (справочник)
│   ├─ classification_id (PK)
│   ├─ classification_name (Self-propelled motorized / Not self-propelled / Stationary / Non-motorized)
│   └─ type_code
│
├─ Equipment_Usage_Status (справочник)
│   ├─ status_id (PK)
│   ├─ status_name (Assigned / Shared without conditions / Shared with conditions)
│   └─ visibility_rules
│
├─ Equipment_Tracker (N:N с Equipment)
│   ├─ tracker_id (PK)
│   ├─ equipment_id (FK)
│   ├─ tracker_type (WIALON / IVMS / Vega / другой)
│   ├─ device_id
│   ├─ device_group
│   ├─ is_active
│   └─ sync_status
│
├─ Equipment_Fleet_Owner (маппинг)
│   ├─ assignment_id (PK)
│   ├─ equipment_id (FK)
│   ├─ fleet_owner_badge (FK на Cost Center primary owner)
│   ├─ from_date
│   ├─ to_date
│   └─ ownership_type (PRIMARY / SECONDARY)
│
└─ Equipment_History (аудит)
    ├─ history_id (PK)
    ├─ equipment_id (FK)
    ├─ field_changed
    ├─ old_value
    ├─ new_value
    ├─ changed_at
    └─ changed_by

```

### 5.2 Связь с Booking Domain

```
┌─────────────────────────────────────────────────────────────┐
│                   BOOKING DOMAIN                            │
└─────────────────────────────────────────────────────────────┘

Booking_Request
├─ request_id (PK)
├─ equipment_id (FK → Equipment)    # Какую технику запросили
├─ cost_center_id (FK)              # Для какого CC
├─ fleet_owner_id (FK)              # Какой FO отвечает (из Equipment_Fleet_Owner)
├─ status (PENDING / APPROVED / REJECTED / CONFIRMED)
├─ requested_from
├─ requested_to
└─ ...

Booking_Confirmation
├─ confirmation_id (PK)
├─ request_id (FK)
├─ fleet_owner_id (FK)
├─ confirmed_equipment_id (FK → Equipment)
├─ actual_from
├─ actual_to
└─ ...
```

### 5.3 Рекомендуемая структура таблиц

```sql
-- Основная таблица техники (денормализованная для быстрого доступа)
CREATE TABLE equipment (
    equipment_id VARCHAR(50) PRIMARY KEY,
    license_plate VARCHAR(50) UNIQUE,
    description VARCHAR(200) NOT NULL,
    equipment_class VARCHAR(50),      -- HDV / HDE
    equipment_category_id FK,         -- Air compressor, Backhoe, etc.
    hdv_hde_classification_id FK,     -- Self-propelled / Not self-propelled / Stationary / Non-motorized
    equipment_usage_status_id FK,     -- Assigned / Shared without / Shared with conditions
    
    -- Location & Ownership
    location VARCHAR(100),
    service_zone VARCHAR(50),
    cost_center_code VARCHAR(20) FK,
    fleet_id VARCHAR(50) FK,
    assigned_location VARCHAR(100),
    
    -- Status
    operation_status VARCHAR(50),     -- Active / Reserved / Sent to BP
    writeoff_status VARCHAR(50),
    criticality VARCHAR(50),          -- Low / High / Critical
    
    -- Technical specs
    serial_number VARCHAR(50),
    vin_chassis_no VARCHAR(50),
    weight_kg DECIMAL,
    volume_m3 DECIMAL,
    manufacturer VARCHAR(100),
    model VARCHAR(100),
    year_manufactured INT,
    
    -- Metadata
    fleet_active_date DATE,
    last_updated TIMESTAMP,
    source VARCHAR(50),               -- 'MASTER_FILE', 'API', 'MANUAL'
    
    -- For Assigned equipment
    justification TEXT,
    
    CONSTRAINT fk_category FOREIGN KEY (equipment_category_id) 
        REFERENCES equipment_category(category_id),
    CONSTRAINT fk_classification FOREIGN KEY (hdv_hde_classification_id)
        REFERENCES equipment_classification(classification_id),
    CONSTRAINT fk_usage_status FOREIGN KEY (equipment_usage_status_id)
        REFERENCES equipment_usage_status(status_id),
    CONSTRAINT fk_cost_center FOREIGN KEY (cost_center_code)
        REFERENCES cost_center(cc_code)
);

-- Справочники (читаемые из Master File)
CREATE TABLE equipment_category (
    category_id INT PRIMARY KEY AUTO_INCREMENT,
    category_name VARCHAR(100) UNIQUE NOT NULL,
    hdv_hde_classification_id FK,
    expected_tracker_metrics VARCHAR(200),
    source VARCHAR(50) DEFAULT 'DIRECTORY_ALL'
);

CREATE TABLE equipment_classification (
    classification_id INT PRIMARY KEY AUTO_INCREMENT,
    classification_name VARCHAR(100) UNIQUE NOT NULL,  -- Self-propelled motorized / Not self-propelled / Stationary / Non-motorized
    type_code VARCHAR(20),
    source VARCHAR(50) DEFAULT 'DIRECTORY_ALL'
);

CREATE TABLE equipment_usage_status (
    status_id INT PRIMARY KEY AUTO_INCREMENT,
    status_name VARCHAR(50) UNIQUE NOT NULL,  -- Assigned / Shared without / Shared with conditions
    visibility_rules JSON
);

-- Маппинг Fleet Owner
CREATE TABLE equipment_fleet_owner (
    assignment_id INT PRIMARY KEY AUTO_INCREMENT,
    equipment_id VARCHAR(50) FK NOT NULL,
    fleet_owner_badge VARCHAR(20) NOT NULL,  -- из CC.PrimaryOwnerBadge
    from_date DATE,
    to_date DATE,
    ownership_type VARCHAR(50),  -- PRIMARY / SECONDARY
    UNIQUE KEY (equipment_id, from_date, to_date),
    CONSTRAINT fk_equipment FOREIGN KEY (equipment_id)
        REFERENCES equipment(equipment_id),
    -- fleet_owner_badge ссылается на Employee или пользователя AAD
);

-- Трекеры
CREATE TABLE equipment_tracker (
    tracker_id INT PRIMARY KEY AUTO_INCREMENT,
    equipment_id VARCHAR(50) FK NOT NULL,
    tracker_type VARCHAR(50) NOT NULL,  -- WIALON / IVMS / Vega / другой
    device_id VARCHAR(100),
    device_group VARCHAR(100),
    is_active BOOLEAN DEFAULT TRUE,
    sync_status VARCHAR(50),
    last_sync TIMESTAMP,
    CONSTRAINT fk_equipment FOREIGN KEY (equipment_id)
        REFERENCES equipment(equipment_id)
);

-- Cost Centers (синхронизировать из JDE)
CREATE TABLE cost_center (
    cc_code VARCHAR(20) PRIMARY KEY,
    cc_name VARCHAR(200) NOT NULL,
    cc_status VARCHAR(50),  -- ACTIVE / INACTIVE
    division_code VARCHAR(20) FK,
    group_code VARCHAR(20) FK,
    department_code VARCHAR(20) FK,
    primary_owner_badge VARCHAR(20),
    secondary_owner_badge VARCHAR(20),
    parent_cc_code VARCHAR(20) FK,
    source VARCHAR(50) DEFAULT 'JDE_CC_DIRECTORY',
    last_synced TIMESTAMP,
    CONSTRAINT fk_division FOREIGN KEY (division_code)
        REFERENCES division(division_code),
    CONSTRAINT fk_group FOREIGN KEY (group_code)
        REFERENCES equipment_group(group_code),
    CONSTRAINT fk_department FOREIGN KEY (department_code)
        REFERENCES department(dept_code),
    CONSTRAINT fk_parent FOREIGN KEY (parent_cc_code)
        REFERENCES cost_center(cc_code)
);

-- Иерархия организации
CREATE TABLE division (
    division_code VARCHAR(20) PRIMARY KEY,
    division_name VARCHAR(100) NOT NULL
);

CREATE TABLE equipment_group (
    group_code VARCHAR(20) PRIMARY KEY,
    group_name VARCHAR(100) NOT NULL,
    division_code VARCHAR(20) FK
);

CREATE TABLE department (
    dept_code VARCHAR(20) PRIMARY KEY,
    dept_name VARCHAR(100) NOT NULL,
    group_code VARCHAR(20) FK
);

-- Аудит (очень важно для отслеживания изменений)
CREATE TABLE equipment_audit (
    audit_id INT PRIMARY KEY AUTO_INCREMENT,
    equipment_id VARCHAR(50) FK NOT NULL,
    field_name VARCHAR(100),
    old_value VARCHAR(500),
    new_value VARCHAR(500),
    changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    changed_by VARCHAR(100),
    change_source VARCHAR(50),  -- 'MANUAL', 'API', 'SYNC', 'IMPORT'
    CONSTRAINT fk_equipment FOREIGN KEY (equipment_id)
        REFERENCES equipment(equipment_id)
);
```

### 5.4 Индексы (для производительности)

```sql
-- Поиск техники для бронирования
CREATE INDEX idx_equipment_status ON equipment(operation_status);
CREATE INDEX idx_equipment_category ON equipment(equipment_category_id);
CREATE INDEX idx_equipment_classification ON equipment(hdv_hde_classification_id);
CREATE INDEX idx_equipment_usage_status ON equipment(equipment_usage_status_id);
CREATE INDEX idx_equipment_cost_center ON equipment(cost_center_code);
CREATE INDEX idx_equipment_location ON equipment(location);

-- Поиск Fleet Owner
CREATE INDEX idx_fleet_owner_equipment ON equipment_fleet_owner(equipment_id);
CREATE INDEX idx_fleet_owner_badge ON equipment_fleet_owner(fleet_owner_badge, to_date);

-- Cost Center
CREATE INDEX idx_cc_division ON cost_center(division_code);
CREATE INDEX idx_cc_status ON cost_center(cc_status);

-- Трекеры
CREATE INDEX idx_tracker_type ON equipment_tracker(tracker_type);
CREATE INDEX idx_tracker_active ON equipment_tracker(is_active);
```

---

## ЧАСТЬ 6 — Процесс загрузки и синхронизации

### 6.1 Начальная загрузка (Initial Load)

1. **Импорт Main листа** → таблица `equipment`
   - 786 записей
   - Маппинг Category → equipment_category справочник
   - Маппинг HDV/HDE classification → equipment_classification справочник
   - Маппинг Equipment Usage Status → equipment_usage_status справочник

2. **Импорт CC directory** → таблица `cost_center`
   - 19,075 записей
   - Построение иерархии Division → Group → Department

3. **Импорт directory all** → справочники
   - Equipment Category (72 записи)
   - HDV/HDE Classification (4 типа)
   - Equipment Usage Status (3 типа)
   - Expected Tracker Metrics (по типам)

4. **Построение маппинга Fleet Owner**
   - equipment_id → fleet_owner_badge (из CC.PrimaryOwnerBadge)
   - Таблица `equipment_fleet_owner`

### 6.2 Плановая синхронизация

**Периодичность:** Еженедельно (по согласованию с TCO)

```python
# Pseudo-code для синхронизации
def sync_equipment_from_master_file():
    # 1. Скачать свежий Master File из источника
    master_file = download_master_file(source_url)
    
    # 2. Прочитать Main лист
    df = read_excel(master_file, sheet='Main')
    
    # 3. Для каждого equipment
    for idx, row in df.iterrows():
        equipment_id = row['Equipment']
        
        # 4. Проверить, существует ли запись
        existing = db.query(Equipment).filter_by(equipment_id=equipment_id).first()
        
        if existing:
            # Обновить поля (с аудитом)
            for field in ['operation_status', 'location', 'cost_center_code', ...]:
                old_value = getattr(existing, field)
                new_value = row[field]
                
                if old_value != new_value:
                    # Логировать изменение
                    log_audit(equipment_id, field, old_value, new_value)
                    setattr(existing, field, new_value)
        else:
            # Создать новую запись
            new_equipment = Equipment(
                equipment_id=equipment_id,
                license_plate=row['License'],
                ...
            )
            db.add(new_equipment)
    
    db.commit()
```

### 6.3 Обновление справочников

```python
def sync_directories():
    # Обновить category справочник
    dir_all = read_excel(master_file, sheet='directory all')
    
    for idx, row in dir_all.iterrows():
        category = EquipmentCategory.query.filter_by(
            category_name=row['Category']
        ).first()
        
        if not category:
            category = EquipmentCategory(
                category_name=row['Category'],
                hdv_hde_classification=row['HDV/HDE classification'],
                expected_tracker_metrics=row['Expected metrics from trackers']
            )
            db.add(category)
    
    db.commit()
```

---

## ЧАСТЬ 7 — Открытые вопросы для TCO

| # | Вопрос | Приоритет |
|---|--------|-----------|
| Q-1 | Являются ли Fleet Owners = Cost Center Owners? Или это разные сущности? | 🔴 Высокий |
| Q-2 | Может ли один Fleet Owner управлять техникой из разных Cost Centers? | 🔴 Высокий |
| Q-3 | Может ли один Cost Center содержать технику нескольких Fleet Owners? | 🔴 Высокий |
| Q-4 | Какой источник для Fleet Owner информации: CC.PrimaryOwnerBadge или отдельная таблица? | 🔴 Высокий |
| Q-5 | Как часто должна синхронизироваться информация из Master File? | 🟡 Средний |
| Q-6 | Какой формат для интеграции: ежемесячный XLSX export или API? | 🟡 Средний |
| Q-7 | Нужно ли сохранять историю всех изменений техники (для аудита)? | 🟡 Средний |
| Q-8 | Какие технические specs используются для поиска при бронировании? | 🟡 Средний |
| Q-9 | Как связывать Trackers (WIALON, IVMS, Vega) с Equipment в Booking Tool? | 🟡 Средний |
| Q-10 | Какой минимальный набор полей достаточен для Phase 1 бронирования? | 🔴 Высокий |

---

## ЧАСТЬ 8 — Рекомендации по дизайну

### ✅ ЧТО ДЕЛАТЬ

1. **Создать единую таблицу Equipment** с денормализацией для быстрого доступа
2. **Использовать справочники** из Master File для валидации и выпадающих списков
3. **Хранить CC directory** в Booking Tool для маппинга иерархии и владельцев
4. **Логировать все изменения** техники для аудита (критично для compliance)
5. **Использовать FK** на Cost Center, Fleet, Category, Classification для referential integrity
6. **Импортировать только необходимые поля** для Phase 1; остальное — в Phase 2
7. **Синхронизировать еженедельно** из обновленного Master File

### ❌ НЕ ДЕЛАТЬ

1. **Не дублировать данные** из CC directory в Equipment (использовать FK)
2. **Не хранить вычисляемые VLOOKUP поля** в основной таблице (Division name, Department name, CC Status и т.д.)
3. **Не писать hardcode значения** для справочников (Assigned, Shared, Active и т.д.)
4. **Не удалять исторические данные** при обновлении (использовать soft-delete или archive)
5. **Не смешивать техническую информацию** (вес, объём, напряжение) с бизнес-информацией в одной таблице (при необходимости — отдельная таблица EquipmentSpecs)

---

## Резюме

**Master Fleet File содержит:**
- 786 единиц техники (основной каталог)
- 104 списанных единиц (история)
- 19,075 Cost Centers (иерархия из JDE)
- Справочники классификаций, категорий, статусов
- Информацию о системах отслеживания (WIALON, IVMS, Vega)

**Для Booking Tool нужна архитектура с:**
- Основной таблицей Equipment (денормализованная для быстроты)
- Справочниками для валидации и UI
- Маппингом Fleet Owner через Cost Center
- Таблицей аудита для отслеживания изменений
- Еженедельной синхронизацией из Master File

**Критичные вопросы для TCO:**
- Уточнить маппинг Fleet Owner → Cost Center
- Определить минимальный набор полей для Phase 1
- Согласовать периодичность и формат синхронизации
- Уточнить требования к аудиту и истории изменений

