# HDV/HDE Booking Tool

# Business Requirement Document — v7

**Version:** 7.0  
**Date:** 2026-04-20  
**Based on:** BRD v6 (2026-04-17) + Requirements gathered in meeting 17.04.2026  
**Scope:** Phase 1 unless noted as Phase 2  
**Prepared by:** Telman Nurzhanov (SA)

### Changelog v6 → v7

| # | Раздел | Изменение |
|---|--------|-----------|
| 1 | 3. Glossary / Utilization | Уточнена модель утилизации: три метрики (моточасы / километраж / комбайн-максимум); обновлена таблица формул; добавлено описание целевого значения (target) и логики расчёта среднего (ремонтные дни исключаются) |
| 2 | 3. Glossary / Priority | **OQ-NEW-I — ✅ CLOSED:** Priority — атрибут Work Order (Request), а не шага и не единицы техники; все Work Center'ы в рамках одного WO имеют тот же приоритет |
| 3 | 3. Glossary / Department | **OQ-NEW-J — ✅ CLOSED:** Департамент в отчётах определяется по **Equipment** (не по Fleet Owner) |
| 4 | 6.4 Request Management | Добавлен **FR-NEW-51**: Default Work Order и Default Work Center для команд, не работающих в JDE (Logistics и другие) |
| 5 | 6.11 Search & Reporting | Добавлен **FR-NEW-52**: страница «История заявок» для Requestor-а (все завершённые заявки) |
| 6 | 6.12 Dashboards / Map | Добавлены **FR-NEW-53–FR-NEW-60**: детальные требования к дашборду утилизации (3 таба, градиент, ремонт-иконки, целевое значение, среднее, месячный вид, фильтр принадлежности, данные из трекеров) |
| 7 | 6.12 Dashboards / Map | Добавлены **FR-NEW-61–FR-NEW-62**: детальные требования к GIS-карте в разделе «Оборудование» (iframe, чекбоксы, tooltip, расширение карты) |
| 8 | 8.1 JDE E1 | Добавлены поля ремонта из JDE: `commitment date`, `planned date`, `completed date` — для отображения ремонтных дней на дашборде утилизации |
| 9 | 10. Open Questions | **OQ-NEW-I** — ✅ CLOSED (Priority = атрибут WO) |
| 10 | 10. Open Questions | **OQ-NEW-J** — ✅ CLOSED (Department = по Equipment) |
| 11 | 10. Open Questions | Добавлен **OQ-NEW-K**: Route tracking — нужно ли отображать маршрут движения техники на карте по диапазону дат? Решение к 20.04 |
| 12 | 11. New Requirements | Добавлены FR-NEW-51–FR-NEW-62 в сводную таблицу |

---

## Status Legend

| Icon | Meaning |
|------|---------|
| ✅ | **Confirmed** — requirement confirmed in meetings, no conflicts |
| ⚠️ | **Partially confirmed / open question** — requirement accepted but details TBD |
| ❌ | **Obsolete / Superseded** — cancelled or replaced by another requirement |
| 🆕 | **New** — not in original BRD; identified during requirements gathering |
| ❗ | **Conflict** — contradiction between BRD and meeting decisions; requires explicit sign-off |
| 🔵 | **Phase 2** — confirmed in scope, deferred to Phase 2 |
| 📋 | **Backlog** — confirmed direction, deferred after baseline implementation |

---

## 1 Introduction

### 1.1 Purpose

This document defines the business requirements for a booking tool that allows users to create requests for HDV/HDE equipment. Requests may include multiple pieces of equipment, each of which is treated as a separate booking with its own approval workflow.

### 1.2 Scope

The tool will support bookings for company-owned equipment as well as equipment provided by business partners. It will cover request creation, approval flow, status tracking, reporting, and integration with external systems where applicable.

> **Scope decision (08.04):** Booking through the tool is available only for TCO Owned and Long-term rented equipment. On-demand BP (external fleet) is displayed as a catalog/showcase only — no booking workflow for on-demand BP in the system. This is a significant scope reduction vs. original BRD. See Section 6.7 for detail.

---

## 2 Business Context

### 2.1 Current Challenges

- Lack of centralized booking and collaboration between departments on shared resources
- Low transparency on total number of units and spend
- No standardized process or guidelines
- No Process Owner(s)
- Lack of visibility due to manual utilization monitoring

### 2.2 Business Objectives

- Standardize the request and booking process.
- Ensure automated routing of approval based on equipment fleet ownership.
- Provide transparency and accountability in the booking process.
- Enable scalability for both internal and partner-provided equipment.
- Process ownership by Logistics.

---

## 3 Glossary

**Request** – a container grouping multiple equipment bookings (one per equipment item).

**Regular Request** – a request created by Requestor in HDV/HDE booking tool.

**Service Work Request** – a request created in HDV/HDE automatically via API based on Work Order created by Requestor in JDE E1 system.

**Booking** – individual booking within a request. One booking is linked to one unique equipment item. Treated individually within a request.

**Equipment** – individual asset (HDV/HDE unit) that can be booked through the system. Each equipment item belongs to a fleet and inherits approval responsibility from the corresponding Fleet Owner.

**Fleet** – logical group of equipment owned by a Fleet Owner (for internal: Maintenance, Construction, TFM, etc.; for external: business partner's name). Each fleet has a unique name assigned at creation.

**Fleet Owner (FO)** – person responsible for approvals within a fleet. One fleet can have 2+ Fleet Owners (e.g. shift workers). A fleet has a shared team email used for system notifications.

**Approver** – Fleet Owner and/or Cost Center (CC) owner, who confirms/declines bookings.

**🆕 Mobilization** – the period of preparation/movement of equipment before it arrives at the work site. Booking period starts from mobilization start, not from arrival. *(Мобилизация — период подготовки/перемещения техники до прибытия на объект)*

**🆕 Equipment categories (Usage Status):**
- **Assigned** – equipment assigned to a specific department; visible to all users, bookable only by authorized users with FO role. *(Закреплённая техника)*
- **Shared without Conditions** – equipment available for general booking without any preconditions. *(Общедоступная без условий)*
- **Shared with Conditions** – equipment available for booking, but the Requestor must provide a **Justification** field. Used for specialized equipment where FO needs context (e.g. specific compressors, chemical tanks). Fleet Owner can recall any booking at any time regardless of this status — recall is a separate mechanism. *(Общедоступная с условиями: требуется обоснование. [Updated 13.04 — OQ-NEW-D CLOSED])*

**🆕 Equipment Ownership Types:**
- **TCO Owned** – internally owned by TCO.
- **Long-term rented** – rented long-term; treated as internal fleet for booking purposes. Managed by internal TCO team (not by BP).
- **On-demand rented (BP Showcase)** – rented on demand; displayed as catalog/showcase only in Phase 1.

**🆕 Utilization** *(Утилизация)* – daily efficiency of equipment usage; reflects how effectively the company manages its equipment.
- Tracked by **mileage** (for wheeled equipment) or **engine hours** (for other equipment types), or **both** depending on equipment category.
- Each Fleet Owner independently defines the **100% utilization threshold** for each equipment type under their fleet (maximum moто-hours or km per 24 hours = 100%).
- **Maximum Utilization Capacity per Shift** — the maximum possible utilization of a given equipment unit within a working shift; serves as the 100% reference point for utilization calculations.
- Utilization percentage is calculated per day, month, or any selected period.
- **🆕 [Updated 13.04]:** Some equipment types have **both** mileage and engine hours parameters. Planned values are entered by FO; actual values are sourced from trackers only (not manually editable).
- **🆕 [Updated 17.04]:** Utilization is sourced from **trackers only**, independent of whether a booking exists in the system. This allows detecting equipment usage that is not linked to any system request.

**🆕 [Updated 17.04] Utilization Calculation Formulas:**

| Metric type | Formula |
|-------------|---------|
| Engine hours only | `utilization = engine_time / target_mh_per_day` |
| Mileage only | `utilization = mileage / target_km_per_day` |
| Both (hours + km) | `utilization = max(engine_time / target_mh_per_day, mileage / target_km_per_day)` |
| Cap rule | If `utilization > 1` → display as **1 (100%)** |

> **Business goal (14.04):** Eliminate equipment downtime. Primary metric — how effectively equipment is used.

**🆕 [New 17.04] Utilization Dashboard tabs:**
- **Моточасы** — engine hours utilization only. Applicable to HDE equipment.
- **Километраж** — mileage utilization only. Applicable to HDV equipment.
- **Комбайн** — maximum of the two metrics (Combined). For equipment with both engine hours and mileage.

**🆕 [New 17.04] Utilization average calculation:**
- Average is calculated over the **selected date range**.
- **Repair days are excluded** from the calculation denominator (days when equipment had an open JDE maintenance Work Order).
- Days when equipment stood idle (not on repair) **are included** in the calculation.

**🆕 [New 17.04] Priority — OQ-NEW-I CLOSED:**
> Priority is an **attribute of the Work Order (Request)** as a whole, not of an individual Work Center step and not of a specific equipment item within the request. All Work Center steps within the same Work Order share the same priority as the parent WO.

**🆕 [New 17.04] Department in reports — OQ-NEW-J CLOSED:**
> When generating department-level reports, the department is determined by the **Equipment's** department assignment — not by the Fleet Owner's department. Fleet Owner and Equipment may belong to different departments.

**🆕 [Updated 13.04] Criticality** *(Критичность)* — equipment marked as critical requires **security escort (service de sécurité)** during transportation. Not a maintenance priority indicator. Displayed in equipment card.

**🆕 [New 16.04] Work Center** *(Рабочий центр / Шаг)* — объём работ, которые нужно выполнить в рамках Work Order; называется «шагом», так как работы выполняются последовательно. Является атрибутом Work Order Step, а не атрибутом единицы техники.

> **Ключевые ограничения:**
> - Один тип Equipment работает только по **одному** Work Center.
> - Один Work Center может обслуживаться **несколькими** типами Equipment.

**Примеры кодов Work Center (предварительный список — подлежит корректировке Асылбеком):**

| Код    | Описание                          |
|--------|-----------------------------------|
| LRG    | Группа ГПМ                        |
| MGT    | Механики по дизелям               |
| BHOE   | Экскаватор, трактор ОТО           |
| DTRK   | Самосвал ОТО                      |
| FTRK   | Грузовик ОТО                      |
| GDRV   | Грузовик с краном ОТО             |
| HYDR   | Водоструйная очистка ОТО          |
| OILT   | Oil Tanker                        |
| VTRK   | Вакуумная машина ОТО              |
| CDECK  | Транспортная платформа ОТО        |
| CRANE  | Кран ОТО                          |
| FORKL  | Вилочный погрузчик ОТО            |
| GMLOG  | Механик передвижного оборудования |
| MLIFT  | Подъёмник ОТО                     |
| MSINS  | ППУ                               |

> **Note:** Logistics subdivision does not operate Work Centers in JDE. WO and WC fields are optional for Logistics. *(OQ-NEW-G CLOSED 16.04)*

---

## 4 Stakeholder & Roles

### 4.1 Requestor

All users in TCO network will by default have Requestor role to book equipments of the internal fleet (TCO owned + Long-term rented HDV/HDE). On-demand external (BP) equipment is displayed as a showcase/catalog only — booking of external equipment is done in DCT, not in this tool.

Requestors are allowed to:

1. Create a Regular Request — ✅
2. Submit the Regular Request — ✅
3. Prolong booking — ✅
4. Submit feedback on the booked equipment — ✅
5. Monitor utilization of the booked equipment for the booking period, if the equipment has tracker(s) — ✅
6. Terminate booking (only external fleet) — ❌ (external fleet booking is not available in the system)
7. 🆕 Close booking by pressing "Close" button (manual close only) — ✅
8. 🆕 Book Assigned equipment only if authorized by Administrator — ✅
9. 🆕 **[New 17.04]** View own request history (completed requests) — ✅

### 4.2 Service Work Processor

> **⚠️ PARTIALLY CLARIFIED (13.04):** Based on meeting discussion, the Service Work Processor role is likely **graphists / schedulers** who work with Work Orders in JDE and process drafts once a Work Order step reaches a confirmed status (≈55 in JDE). They push Service Work Requests from Draft to Submitted for Fleet Owner processing. Formal confirmation from TCO required (OQ-NEW-C).

~~Access provision is managed by Azure Active Directory (AAD) group. Similar to Requestor, however, Service Work Processor works with Service Work Requests, created automatically in HDV/HDE booking tool.~~

### 4.3 Fleet Owner

For internal Fleet Owners access provision is managed by AAD group. There can be 2+ AAD groups to manage internal Fleet Owners (separate group for each fleet created by Administrator).

**🆕 [Updated 13.04]:** One fleet can have **two or more Fleet Owners** (e.g. shift workers / сменщики). Both owners are displayed in the equipment card. Each fleet has a **shared team email** address used as a single notification target (alongside personal emails via AD group). When a Fleet Owner changes role, Administrator reassigns the fleet without rebuilding the fleet structure.

Fleet Owners are allowed to:

1. 🆕 **[Updated 10.04]** Create new equipment under own fleet — ✅ for internal; 🔵 Phase 2 for external fleet onboarding workflow
2. Update equipments (only parameters that are allowed for editing by Administrator) — ✅
3. Delete equipment (soft delete) — ✅
4. Confirm / decline requests to book equipments under own fleet — ✅
5. 🆕 **[Updated 10.04]** Update booking period before or after confirmation; Requestor is notified automatically; no re-confirmation required — ✅
6. Change equipment in booking — ✅
7. Terminate already confirmed bookings (only internal fleet) — ✅
8. Freeze / unfreeze own equipment from booking for a period or without end date — ✅
9. Monitor utilization of own equipment via utilization dashboard — ✅
10. 🆕 Press "Mobilization started" button to fix mobilization start time — ✅
11. 🆕 Delegate FO privileges to any TCO employee for a specified period — ⚠️ Pending cybersecurity sign-off (OQ-NEW-E)
12. 🆕 One user can be an owner of several fleets — ✅
13. 🆕 **[Updated 13.04]** One fleet can have multiple Fleet Owners; both displayed with shared team email — ✅

### 4.4 CC Owner (incl. DOA)

> **[Confirmed 10.04]:** CC Owner confirmation is NOT required for Long-term rented equipment. CC Owner is applicable only to on-demand external fleet booking, which is in DCT system.

CC owners are allowed to:

1. Confirm / decline external fleet equipment booking request — ❌ **Obsolete**

### 4.5 Administrator

Administrator role will be provided to process owners – Logistics. Access provision is managed by Azure Active Directory (group). Users with an Administrator role are allowed to:

1. Create/edit internal fleet and link with AAD group — ✅
2. Create/edit external fleet and manage external fleet owners — 🔵 Phase 2
3. Delete fleet — ✅
4. Create internal fleet equipment — ✅
5. Update equipment's parameters (extended edit) — ✅
6. Delete equipment (soft delete) — ✅
7. Manage 'Request' template — ⚠️ Baseline first; custom attributes → 📋 Backlog (OQ-NEW-B)
8. Manage 'Equipment' template — ⚠️ Baseline first; custom attributes → 📋 Backlog (OQ-NEW-B)
9. Manage 'Booking' template — ⚠️ TBD (OQ-NEW-B)
10. Access to custom reportings — ✅
11. Access to utilization dashboard(s), if the equipment has tracker(s) — ✅
12. 🆕 Manage all configurable system parameters via Admin Panel — ✅
13. 🆕 Authorize specific users to book Assigned equipment — ✅
14. 🆕 Manage delegation settings — ⚠️ Pending cybersecurity sign-off (OQ-NEW-E)
15. 🆕 **[Updated 13.04]** Manage fleet shared team email — ✅
16. 🆕 **[Updated 16.04]** Manage reference/handbook values (equipment classes, categories, subcategories, manufacturers, models, companies, locations, work centers, divisions, groups, departments, units) via Admin Panel — ✅

### 4.6 🆕 Transportation Responsible (Мобилизация/Транспортировка)

A dedicated system role for the team responsible for transporting non-motorized (Unwheeled) equipment (Heavy Ops / Maintenance team).

This role is allowed to:

1. 🆕 Receive notification about transportation request for Unwheeled equipment — ✅
2. 🆕 Confirm or decline the transportation request — ✅

---

## 5 Entities

### 5.1 Equipment

All equipments must have a mandatory attribute – fleet. The attribute will affect on which template the equipment will have, who can create/edit/delete equipments, approval and processing flow. — ✅

**🆕 Fleet naming (10.04):** Each fleet must have a unique **name** assigned at creation. Equipment assignment is linked to a specific employee; when a Fleet Owner changes, the Administrator manually re-assigns the fleet.

**🆕 Fleet Owners display (13.04):** Equipment card displays **up to 2+ Fleet Owners** alongside their **shared team email**.

**🆕 [New 16.04] Work Center assignment:** Each equipment type is assigned to **exactly one Work Center**. Managed via Admin Panel reference.

**🆕 [New 16.04] Reference/Handbook policy:** The following are handbook-managed via Admin Panel:
- Equipment classes (HDE, HDV)
- Categories and subcategories
- Manufacturer, Model
- Companies (TCO White Pages)
- Locations
- Work Centers
- Divisions, Groups, Departments, Units

Internal fleet equipments have the next parameters:

1. Basic fixed info (TCO ID, Description, Service Zone, Class/Category, License Plate, Serial number, etc.) — ✅ ⚠️
2. Dynamic parameters: Equipment Usage Status, **Cost Center**, CC/Division/Group/Department/Unit, Assigned location, Maintenance BP, Contact number / email, etc. — ✅
   > **🆕 [Updated 16.04]:** «Операционный статус» (Operational Status) **removed** from equipment card parameters.
3. **🆕 [New 16.04] Ownership field** — «Принадлежность»: **TCO Owned** / **Long term rented** — ✅
4. **🆕 [New 16.04] Freeze fields**:
   - **Заморозка** (freeze flag)
   - **Причина заморозки** (freeze reason)
   - **Дата завершения заморозки** (freeze end date; empty = indefinite)
   > Visible and editable by FO and Admin. (FR-NEW-49)
5. Equipment specifications (picture, Chassis type, Drive type, Loading capacity, etc.) — ✅
6. Trackers' IDs (WIALON, IVMS, Vega, etc.) — ✅ **🆕 Multiple trackers per unit supported**
7. **🆕** Planned engine hours (per shift/day) — editable by FO — ✅
8. **🆕** Planned daily mileage (km/day) — editable by FO — ✅
9. **🆕** Actual engine hours / mileage — sourced from tracker only; not manually editable — ✅

**🆕 Equipment card UI (13.04–17.04):**
- **Booking calendar**: visual calendar showing booked periods ✅
- **Criticality flag**: displayed in card ✅
- **Work schedule display**: split into day shift / night shift blocks ✅
- **Weekend coverage**: default shift 06:00–06:00; Long Term — 07:00–07:00 ✅
- **Multiple photos**: supports uploading multiple photos per unit ✅
- **Sidebar**: Comments block in sidebar ✅
- **Tracker block**: add new trackers from equipment card UI ✅
- **🆕 [New 16.04] Schedule slider**: slider control for selecting time ranges in work schedule block ✅

**🆕 [New 17.04] GIS map integration in equipment list:**
- Equipment list (FO view) includes **checkboxes** to select units for display on the GIS map
- Map is rendered by GIS team (MAPH/Atlas) as an **iframe**; system passes equipment identifiers as parameters
- See FR-NEW-61, FR-NEW-62

**🆕 License plate and TCO number rules (13.04):**
- **License plate**: present on most wheeled equipment; may be absent on stationary units.
- **TCO number**: assigned at purchase; long-term rented equipment may not have one. BP may have neither.

**🆕 Equipment Classification (three dimensions):**

| Dimension | Values |
|-----------|--------|
| Mobility | Self-propelled / Not self-propelled (Unwheeled) / Stationary |
| Ownership | TCO Owned / Long-term rented / On-demand rented (BP Showcase) |
| Usage Status | Assigned / Shared without Conditions / Shared with Conditions |

> **Note (08.04):** Mobility category NOT displayed in Requestor UI. ✅  
> **Note (08.04):** Stationary HDE excluded from Requestor search. ✅  
> **Note (13.04):** Shared with Conditions differs only in presence of mandatory Justification field. ✅

**🆕 Equipment availability display rules:**
- Equipment under maintenance shown grayed out; label "Under maintenance until [Planned date from JDE]" ✅
- Repair status sourced from JDE → DataLake integration ⚠️

---

### 🆕 5.1.1 Equipment Type-Specific Parameters (13.04)

> Parameters define specific attributes and search filters for each equipment type. Preliminary — subject to final sign-off from TCO Fleet Owners.

#### Loader / Погрузчик

| Parameter | Values / Format | Used in filter |
|-----------|----------------|----------------|
| Load capacity | Ranges: up to 2 t / 2–4.5 t / 4.5–8 t | ✅ |
| Type | Fork / Bucket | ✅ |
| Engine type | Electric / Diesel | ✅ |

#### Crane-Manipulator / Кранманипулятор

| Parameter | Values / Format | Used in filter |
|-----------|----------------|----------------|
| Vehicle load capacity | numeric (tons) | ✅ |
| Crane load capacity | numeric (tons) | ✅ |
| Boom length | numeric (meters) | ✅ |
| Body length | numeric (meters) | ✅ |

#### Mobile Generator / Генератор передвижной

| Parameter | Values / Format | Used in filter |
|-----------|----------------|----------------|
| Power | Dropdown: 30 / 60 / 80 / 100 / 120 / 130 / 200 / 250 kW (TBD) | ✅ |
| Voltage | — | ⚠️ TBD |
| Current | — | ⚠️ TBD |
| Battery capacity | — | ⚠️ TBD |
| Mobility type | Wheeled / Non-wheeled / Stationary | ✅ |

#### Motor Compressor / Компрессор моторный

| Parameter | Values / Format | Used in filter |
|-----------|----------------|----------------|
| Pressure | numeric (atm) | ✅ |
| Productivity | numeric (m³/h) | ✅ |

#### Dump Truck / Самосвал

| Parameter | Values / Format | Used in filter |
|-----------|----------------|----------------|
| Load capacity | Dropdown or range (TBD) | ✅ |
| Body type | Open / Closed | Via photo / optional |

#### Frac Tank / Фрактанк

| Parameter | Values / Format | Notes |
|-----------|----------------|-------|
| No type-specific filters | All frac tanks have the same volume | Any available unit |
| Dedicated transport | Kenworth Winch Truck (1 unit) | TBD if occupied |

#### Cargo Trailer / Прицеп грузовой трейлер карго

| Parameter | Values / Format | Used in filter |
|-----------|----------------|----------------|
| Type | Low-frame / Standard | ✅ |
| Load capacity | numeric (tons) | ✅ |
| Suspension system | Pneumatic / Standard | ✅ |

#### Semi-truck / Тягач HD

> Tractor head determined by trailer type. FO selects appropriate tractor. No Requestor-facing filter required.

---

### 5.2 Booking

Booking is linked to a single equipment and have booking range (datetime). Several bookings form a single request. — ✅

> **🆕 Architecture decision (09.04):** One Request → multiple RequestItems (one per equipment); each independently processed by FO. ✅

#### 5.2.1 Internal Booking

Booking of TCO owned (including Long-term rented) equipment. Requires only FO confirmation.

> **🆕 Mobilization (09.04):** Booking period starts from mobilization start. FO presses "Mobilization started" to fix actual start. ✅

> **🆕 [Updated 10.04]:** FO can adjust booking period before and after confirmation. Requestor notified automatically. ✅

#### 5.2.2 External Booking

**❗ PHASE CHANGE vs. original BRD:** On-demand BP equipment is NOT bookable in Phase 1. Displayed as catalog/showcase only.

- On-demand rented (BP Showcase) — catalog view only. ✅
- Long-term rented — treated as internal fleet. ✅
- Full external booking workflow (CC Owner → External FO) — ❌ **Obsolete**

#### 5.2.3 🆕 Booking Snapshot

At booking confirmation, system saves a snapshot of key equipment attributes. — ⚠️ Preliminary (OQ-37)

### 5.3 Request

#### 5.3.1 Regular Request

Requestor can add **1 or more** equipments from any fleet; each is treated as a separate booking. — ✅

**🆕 Request fields (08.04):**

- **Work Order (JDE)** — mandatory for Maintenance / Railroad / Operations. One WO per request. ✅
- **Work Center (WC)** — mandatory for JDE-based fleets; **optional for Logistics**. ✅
- **Location** (text input) — for SCM Logistics replaces WO. ✅
- **Work Description** — mandatory, ~50 chars. ✅
- **Comments** — optional. ✅
- **Priority** (P1–P4) — attribute of the Request / Work Order. **OQ-NEW-I CLOSED.** ✅

> **🆕 [New 16.04] WO pre-fill flow:** Block **"Выбрать Work Order из JDE"** on Submit Request page. See FR-NEW-48.

> **🆕 [New 17.04] Default WO / WC:** For teams not using JDE (Logistics and others) — **"Default Work Order"** checkbox available; no JDE number required. Analogously **"Default Work Center"** for teams without WC. See FR-NEW-51.

**🆕 Request constraints (03.04):**
- Booking horizon: 1 month ahead (configurable). ✅
- Max booking duration: 1 week (configurable). ✅

#### 5.3.2 Service Work Request (within Work Order in JDE E1)

> **⚠️ PARTIALLY CLARIFIED (13.04):** Auto-created as Draft when WO step reaches status 55 in JDE. SWP reviews and submits to FO. Formal confirmation required (OQ-NEW-C).

**🆕 [Updated 16.04] Required WO data fields from DataLake:**

| Field | Description / Notes |
|-------|---------------------|
| Work Order number | Unique identifier in JDE E1 |
| Work Order name | Examples needed from TCO |
| Status | Clarify list of valid statuses |
| **Status description** | **🆕 [New 16.04]** Text description |
| Priority | Priority value in JDE E1 — attribute of the WO |
| Work Center (Step code) | Type of work; e.g. "CRANE", "DTRK" |
| Step name | Text name of the step |
| **Step volumes** | **🆕 [New 16.04]** Required quantity of equipment units per step |
| **Date range** | **🆕 [New 16.04]** Start date / End date of the WO step |

---

## 6 Functional Requirements

### 6.1 User & Role Management

| ID | Requirement | Status |
|----|-------------|--------|
| FR-001 | System supports role-based access control | ✅ |
| FR-002 | All TCO network users have Requestor access | ❌ **Obsolete** *(superseded)* |
| FR-003 | Administrators, Service Work Processors defined by AAD groups | ⚠️ SWP pending (OQ-NEW-C) |
| FR-004 | CC Owner / DOA privileges via PSWS/DOA integration | ❌ **Obsolete** |
| 🆕 FR-NEW-01 | All time-based parameters configurable via Admin Panel (not hardcoded) | ✅ |
| 🆕 FR-NEW-02 | FO can delegate privileges to any TCO employee for a date range; auto-revoke upon expiry | ⚠️ Pending cybersec sign-off (OQ-NEW-E) |

### 6.2 Admin Panel

| ID | Requirement | Status |
|----|-------------|--------|
| FR-005 | Admin creates fleets with unique names | ✅ |
| FR-006 | Admin links internal fleet with AAD group | ✅ |
| FR-007 | Admin manages external fleet owners | 🔵 Phase 2 |
| FR-008 | Admin updates existing fleets | ✅ |
| FR-009 | Admin deletes fleets | ✅ |
| FR-010 | Admin manages equipment template (baseline first; custom attributes → backlog) | ⚠️ 📋 |
| FR-011 | Admin manages request template | 📋 Backlog |
| FR-012 | Admin manages booking template | ⚠️ TBD |
| FR-013 | Admin creates equipment (internal fleet) | ✅ |
| FR-014 | Admin edits equipment (extended edit) | ✅ |
| FR-015 | Admin deletes equipment | ✅ |
| FR-016 | Admin accesses all dashboards and reports | ✅ |
| 🆕 FR-NEW-03 | Admin configures: FO timeout (24h/48h), booking horizon, max duration | ✅ |
| 🆕 FR-NEW-04 | Admin authorizes specific users to book Assigned equipment | ✅ |
| 🆕 FR-NEW-42 | Admin sets and updates fleet shared team email | ✅ |
| 🆕 FR-NEW-50 | Admin manages all reference/handbook values via Admin Panel | ✅ |

### 6.3 Equipment Management

| ID | Requirement | Status |
|----|-------------|--------|
| FR-017 | Both internal and external FO can create equipment under own fleet | ✅ |
| FR-018 | FO must indicate fleet of equipment; only own fleets selectable | ✅ |
| FR-019 | FO can edit allowed equipment parameters; one user can own several fleets | ✅ |
| FR-020 | FO can delete equipment (soft delete) | ✅ |
| FR-021 | FO can freeze/unfreeze equipment for a period or indefinitely | ✅ |
| FR-022 | Requestor / SWP can submit feedback on equipment with confirmed booking | ✅ |
| 🆕 FR-NEW-05 | FO uploads multiple photos; Requestor sees them in request form | ✅ |
| 🆕 FR-NEW-06 | Repair status from JDE → DataLake displayed in search and equipment list | ⚠️ Pending |
| 🆕 FR-NEW-07 | Stationary HDE excluded from Requestor search | ✅ |
| 🆕 FR-NEW-08 | Assigned equipment: visible to all, bookable by authorized only | ✅ |
| 🆕 FR-NEW-09 | Shared with Conditions has visual color marker | ✅ |
| 🆕 FR-NEW-40 | Equipment card (FO view) has booking calendar visual | ✅ |
| 🆕 FR-NEW-41 | Multiple trackers per unit; FO can add trackers from equipment card UI | ✅ |
| 🆕 FR-NEW-49 | Freeze fields on equipment card: Заморозка, Причина заморозки, Дата завершения заморозки | ✅ |

### 6.4 Request Management

| ID | Requirement | Status |
|----|-------------|--------|
| FR-023 | Requestor can create a Request | ✅ |
| FR-024 | Request has unique ID and metadata | ✅ |
| FR-025 | Requestor can view request details and status | ✅ |
| FR-026 | Request statuses: Draft, Submitted, In Progress, Completed | ⚠️ Aggregated TBD (OQ-36) |
| FR-027 | Requestor can edit/cancel draft before submission | ✅ |
| FR-028 | Service Work Requests auto-created with 'Draft' status | ⚠️ Pending (OQ-NEW-C) |
| FR-029 | Only SWP can manage (submit) Service Work Requests | ⚠️ Pending (OQ-NEW-C) |
| FR-030 | System supports draft saving | ✅ |
| 🆕 FR-NEW-10 | On-demand booking only; weekly schedule management out of scope | ✅ |
| 🆕 FR-NEW-11 | Requestor selects priority P1–P4 at request creation (priority = WO-level attribute) | ✅ |
| 🆕 FR-NEW-12 | WO mandatory for Maintenance/Railroad/Operations; optional for Logistics; Location replaces WO for SCM Logistics | ✅ |
| 🆕 FR-NEW-13 | "Work Description" mandatory (~50 chars) | ✅ |
| 🆕 FR-NEW-14 | "Comments" optional | ✅ |
| 🆕 FR-NEW-15 | Single form → auto-split into individual bookings per equipment item | ✅ |
| 🆕 FR-NEW-16 | Partial confirmation: confirmed items proceed independently from declined | ✅ |
| 🆕 FR-NEW-17 | Aggregated Request status auto-calculated from RequestItem statuses | ⚠️ TBD (OQ-36) |
| 🆕 FR-NEW-48 | "Выбрать WO из JDE" block on Submit Request page; WO list from DataLake; auto-generate draft | ✅ |
| 🆕 **FR-NEW-51** | **[New 17.04]** **Default Work Order** option for teams not operating in JDE (Logistics and others): checkbox on request form; when selected — system does not require a JDE WO number. Analogously **Default Work Center** for teams without WC. Applies to all requestors from non-JDE subdivisions. | ✅ |

### 6.5 Booking Management

| ID | Requirement | Status |
|----|-------------|--------|
| FR-031 | Requestor can add/remove equipment items to a request; availability updated | ✅ |
| FR-032 | System prioritizes internal fleet (business rule) | ✅ |
| FR-033 | Requestor can add Shared equipment to request | ✅ |
| FR-034 | System suggests external fleet based on decision matrix | ❌ **Obsolete** |
| FR-035 | Requestor adds external fleet equipment if RWA initiator | ❌ **Obsolete** |
| FR-036 | Justification required when adding external fleet equipment | ❌ **Obsolete** |
| FR-037 | Cost Center required for external fleet equipment | ❌ **Obsolete** |
| FR-038 | Each equipment item in request = separate booking | ✅ |
| FR-039 | Each booking has unique ID, start/end datetimes | ✅ |
| FR-040 | System validates availability before booking | ✅ |
| FR-041 | Equipment attributes displayed in booking | ✅ |
| FR-042 | Booking lifecycle: Draft, Submitted, Confirmed, Declined, Revoked, Terminated, Completed | ✅ |
| 🆕 FR-NEW-18 | Booking start = start of mobilization | ✅ |
| 🆕 FR-NEW-19 | No mandatory buffer between bookings | ✅ |
| 🆕 FR-NEW-20 | Unwheeled equipment: notification displayed to Requestor | 🟡 Preliminary |

### 6.6 Approval Flow (Internal Fleet)

| ID | Requirement | Status |
|----|-------------|--------|
| FR-043 | Approver assigned automatically based on fleet ownership | ✅ |
| FR-044 | Internal fleet booking confirmed/declined by FO | ✅ |
| FR-045 | FO can confirm incoming bookings | ✅ |
| FR-046 | FO can replace equipment before/after confirmation if booking not yet started | ✅ |
| FR-047 | FO cannot replace if booking revoked, terminated, or past end date | ✅ |
| FR-048 | **[Updated 10.04]** FO can adjust booking date range before or after confirmation; Requestor notified automatically; no re-confirmation required | ✅ |
| FR-049 | FO can decline booking; availability updated | ✅ |
| FR-050 | FO provides reason/comment when declining | ✅ |
| 🆕 FR-NEW-21 | FO must respond within timeout. Default: 24h reminder; 48h escalation | ✅ |
| 🆕 FR-NEW-22 | If FO has free equipment and declines — mandatory reason required | ✅ |
| 🆕 FR-NEW-23 | Reminder sent only to FO of the specific selected equipment | ✅ |
| 🆕 FR-NEW-24 | FO presses "Mobilization started" to record actual start time | ✅ |

### 6.7 Approval Flow (External Fleet)

> **❗ MAJOR SCOPE CHANGE (08.04):** On-demand BP not bookable. All FR-051..057 are Obsolete.

| ID | Requirement | Status |
|----|-------------|--------|
| FR-051..057 | External fleet approval chain (CC Owner, External FO, PSWS, DOA) | ❌ **Obsolete** |
| 🆕 FR-NEW-25 | BP showcase: Requestor can view on-demand BP equipment; no booking action | ✅ Phase 1 |
| 🆕 FR-NEW-26 | BP equipment prices/rates NOT displayed | ✅ |
| 🆕 FR-NEW-27 | BP populates own catalog cards | ✅ |
| 🆕 FR-NEW-28 | "Go to BP catalog" button after FO timeout | ✅ |

### 6.8 Booking Lifecycle Management

| ID | Requirement | Status |
|----|-------------|--------|
| FR-058 | Requestor/SWP can revoke booking if not yet processed by FO | ✅ |
| FR-059 | FO can terminate confirmed booking; reason required; availability updated | ✅ |
| FR-060 | Requestor/SWP can terminate confirmed booking (internal only) | ✅ |
| FR-061 | Requestor/SWP can extend confirmed booking | ✅ |
| 🆕 FR-NEW-29 | Three-step Unwheeled flow: Requestor → FO → Transportation Responsible → FO final | ✅ |
| 🆕 FR-NEW-30 | If transport unavailable for Unwheeled — FO proposes nearest available date | ⚠️ |
| 🆕 FR-NEW-31 | Booking can be closed early via "Close early" button | ⚠️ |

### 6.9 Booking/Request Status Management

> **CONFLICT-02 — ✅ RESOLVED (14.04):** Auto status transition by datetime cancelled. All status changes are **manual only**.

| ID | Requirement | Status |
|----|-------------|--------|
| FR-062 | Booking → 'Submitted' once Request submitted | ✅ |
| FR-063 | Booking → 'Confirmed' once confirmed by FO | ✅ |
| FR-064 | Booking → 'Declined' once declined by FO | ✅ |
| FR-065 | External booking → 'Partially Confirmed' by CC Owner | ❌ **Obsolete** |
| FR-066 | External booking → 'Declined' by CC Owner | ❌ **Obsolete** |
| FR-067 | Booking → 'Submitted' once extended by Requestor/SWP | ✅ |
| FR-068 | Booking → 'Revoked' once revoked | ✅ |
| FR-069 | Booking → 'Terminated' once terminated | ✅ |
| FR-070 | ~~Auto-transition to 'In Progress' by datetime~~ | ❌ **Obsolete** *(CONFLICT-02)* |
| FR-071 | ~~Auto-transition to 'Completed' by datetime~~ | ❌ **Obsolete** *(CONFLICT-02)* |
| FR-072 | Request → 'Draft' when saved as draft | ✅ |
| FR-073 | SWR → 'Draft' when imported from JDE | ⚠️ Pending (OQ-NEW-C) |
| FR-074 | Request → 'In Progress' when first booking (excl. Declined) goes In Progress | ✅ |
| FR-075 | ~~Request → 'Completed' by datetime~~ | ❌ **Obsolete** *(see FR-NEW-44)* |
| 🆕 FR-NEW-32 | Booking closure is manual only — "Close" button by Requestor or FO | ✅ |
| 🆕 FR-NEW-33 | At close: Requestor inputs actual start/end time for utilization analytics | ✅ |
| 🆕 FR-NEW-44 | Request → 'Completed' when last active (non-Declined) booking closed | ✅ |

### 6.10 Notifications

| ID | Requirement | Status |
|----|-------------|--------|
| FR-076 | System notifies SWP when SWR auto-created | ⚠️ Pending (OQ-NEW-C) |
| FR-077 | System notifies FO when request submitted | ✅ |
| FR-078 | System notifies FO when booking revoked | ✅ |
| FR-079 | System notifies Requestor/SWP of FO decision | ✅ |
| FR-079a | System notifies Requestor when FO updates booking period | ✅ |
| FR-080 | System notifies Requestor/SWP when FO terminates booking | ✅ |
| FR-081 | System notifies FO when Requestor extends booking | ✅ |
| FR-082..090 | External fleet notification chain | ❌ **Obsolete** |
| 🆕 FR-NEW-34 | 24h reminder to FO; 48h escalation to manager | ✅ |
| 🆕 FR-NEW-35 | Notification to Transportation Responsible for Unwheeled booking | ✅ |
| 🆕 FR-NEW-36 | FO notified when Transportation Responsible decides | ✅ |
| 🆕 FR-NEW-43 | Notifications delivered to fleet shared team email (and/or AD group personal emails) | ✅ |

### 6.11 Search & Reporting

| ID | Requirement | Status |
|----|-------------|--------|
| FR-091 | Requestor/SWP can search and filter own requests | ✅ |
| FR-092 | Approver (FO) can view all pending and completed approvals | ✅ |
| FR-093 | Admin can view all requests | ✅ |
| FR-094 | System generates reports on Requests/Bookings/Equipment with filters | ✅ |
| FR-095 | Admin can view/download all reports | ✅ |
| FR-096 | FO can view/download own approval reports | ✅ |
| FR-097 | FO can view/download own equipment reports | ✅ |
| FR-098 | SWP (and FOs) can view SWR grouped by Work Orders | ⚠️ Pending (OQ-NEW-C) |
| 🆕 FR-NEW-37 | Audit trail and history on demand | ✅ |
| 🆕 FR-NEW-38 | Search: TCO equipment number + model mandatory; госномер if present | ✅ |
| 🆕 FR-NEW-39 | Dynamic search filters by equipment type (type → filter matrix in 5.1.1) | ⚠️ Partial |
| 🆕 FR-NEW-45 | Dedicated "Completed Requests" page for Requestor, SWP, FO, Admin | ✅ |
| 🆕 FR-NEW-46 | Utilization reports: by day / department / equipment unit | ⚠️ Details pending |
| 🆕 FR-NEW-47 | Work center reports (group by WC code) | ⚠️ Details pending |
| 🆕 **FR-NEW-52** | **[New 17.04]** Requestor has access to **«История заявок»** (Request History) — a dedicated page listing all completed requests by this Requestor. Minimum columns: request number, equipment type, booking period, status, Work Order number. | ✅ |

### 6.12 Dashboards / Map

| ID | Requirement | Status |
|----|-------------|--------|
| FR-099 | System displays utilization and coordinates from DataLake/PI (tracker data) | ✅ |
| FR-100 | System displays digitized location of Work Orders from DataLake (JDE E1) | ✅ |
| 🆕 **FR-NEW-53** | **[New 17.04]** Utilization dashboard has **three tabs**: «Моточасы» (engine hours) / «Километраж» (mileage) / «Комбайн» (max of both). Active tabs depend on equipment type: HDE → Моточасы only; HDV → Километраж only; mixed equipment → all three. | ✅ |
| 🆕 **FR-NEW-54** | **[New 17.04]** Utilization cells use a **color gradient**: 0% — red → 50% — yellow → 100% — green. Smooth gradient, not discrete steps. | ✅ |
| 🆕 **FR-NEW-55** | **[New 17.04]** Repair days are marked in the dashboard with a **wrench icon** (not a color) to distinguish from low-utilization days. Repair data sourced from JDE (see Section 8.1): `commitment date` = start of repair, `planned date` = expected end, `completed date` = actual end. Repair days are **excluded from average utilization calculation**. Manual repair input for non-JDE equipment — Phase 2. | ✅ / 🔵 Phase 2 (manual) |
| 🆕 **FR-NEW-56** | **[New 17.04]** Utilization dashboard includes a **Target value column** per equipment: maximum daily utilization threshold set by FO (e.g. 2 000 engine hours / 1 500 km per day). Displayed first after equipment name/type columns. | ✅ |
| 🆕 **FR-NEW-57** | **[New 17.04]** Utilization dashboard includes an **Average utilization column** for the selected period (repair days excluded). Table is **sortable by this column** (descending / ascending). | ✅ |
| 🆕 **FR-NEW-58** | **[New 17.04]** Utilization dashboard supports **two time granularity views**: «По дням» (daily breakdown, current default) / «По месяцам» (monthly average per cell). User toggles via a control above the table. | ✅ |
| 🆕 **FR-NEW-59** | **[New 17.04]** Utilization dashboard left sidebar includes an **Ownership filter**: TCO Owned / Long-term rented. | ✅ |
| 🆕 **FR-NEW-60** | **[New 17.04]** Utilization data is sourced from **tracker telemetry only**, independent of bookings in the system. This allows detecting equipment usage not linked to any booking (anomaly detection). Future Phase 2 feature: overlay with booking count per day for correlation. | ✅ |
| 🆕 **FR-NEW-61** | **[New 17.04]** In the FO Equipment section, equipment list includes **checkboxes** to select units for display on the GIS map. The GIS map is rendered as an **iframe** by the GIS team (MAPH/Atlas); system passes selected equipment identifiers as URL parameters. Map supports **expand** button for full-screen view. Search in equipment list by **ТШО-номер**. | ✅ *(GIS team involvement required)* |
| 🆕 **FR-NEW-62** | **[New 17.04]** GIS map **tooltip** on equipment marker: displays (1) current Work Order number (or "No booking") and (2) today's utilization percentage. Tooltip content is passed to GIS team as parameters. | ⚠️ *(To be aligned with GIS team)* |

### 6.13 Audit & Compliance

| ID | Requirement | Status |
|----|-------------|--------|
| FR-101 | System logs all request creation, modifications, approvals, rejections | ✅ |
| FR-102 | Audit trail accessible for compliance purposes | ✅ |
| FR-103 | Exportable logs for legal/regulatory audits | ✅ |

---

## 7 Non-Functional Requirements

| Area | Requirement | Status |
|------|-------------|--------|
| Availability | 99.5% uptime | ✅ |
| Performance | High volume of requests per hour without degradation | ✅ |
| Security | RBAC; Azure AD SSO | ✅ |
| Compliance | Full audit history of all approvals / rejections / revokes / terminations | ✅ |
| Scalability | Support internal and external partner use cases | ✅ |
| Usability | Intuitive UI; UX reference: Booking.com | ✅ |
| 🆕 Users | ~200 users (per ACR assessment 07.04); No SPD data stored | ✅ |
| 🆕 Security tier | ACR assessment result: Low Risk (07.04) | ✅ |
| 🆕 Authentication | Azure SSO for internal users; separate account for external BP (Phase 2) | ✅ |

---

## 8 Integrations

### 8.1 JDE E1

Work Orders created in JDE E1 are imported to HDV/HDE as Service Work Requests once a certain status is assigned. Each Step becomes a separate SWR. Only steps with relevant types (BHOE, CDECK, DTRK, FORKL, FTRK, GDRV, GMLOG, MLIFT, VTRK, etc.) are imported.

**🆕 [Updated 13.04] Integration path:** JDE E1 → **DataLake** (sync every ~30 min) → HDV/HDE Booking Tool. No direct JDE API.

**🆕 [Updated 13.04] JDE WO flow:** Steps: status ~50 (scheduler review) → status **55** (released to operations). Steps at status 55 → auto-import as SWR Drafts.

**🆕 [Updated 16.04] Required Work Order fields from DataLake:**

| Field | Description / Notes |
|-------|---------------------|
| WO Number | Unique identifier in JDE E1 |
| WO Name | Examples needed from TCO |
| WO Status | Exact list of statuses TBD |
| **WO Status Description** | **🆕 [New 16.04]** Text description |
| WO Priority | Priority = WO-level attribute *(OQ-NEW-I CLOSED 17.04)* |
| Work Center (Step code) | Defines required equipment type |
| Step Name | Text name of the step |
| **Step Volumes** | **🆕 [New 16.04]** Required quantity per step |
| **Date Range** | **🆕 [New 16.04]** Start / End date of the WO step |

**🆕 [New 17.04] Repair/Maintenance data fields from JDE (via DataLake):**

| Field | Description |
|-------|-------------|
| **Commitment date** | Date when the maintenance WO was committed / accepted |
| **Planned date** | Expected completion date of maintenance work |
| **Completed date** | Actual completion date (set when maintenance WO is closed) |

> These fields are used to identify **repair days** in the utilization dashboard (FR-NEW-55). Repair days are excluded from average utilization calculation. For equipment not maintained via JDE, manual repair input is deferred to Phase 2.

**🆕 Integration contact (13.04):** **Аяжан** — Maintenance Plan Consultant, group TFM 2835. Action: verify exact field names in DataLake (Аян, Тельман).

**Status:** ⚠️ Partially confirmed — JDE trigger status TBD; DataLake path confirmed

🆕 **Repair status (08.04):** Repair status (maintenance in progress / planned completion date) displayed in Requestor's equipment search from DataLake. — ⚠️ Details TBD

### 8.2 DCT (Digital Contractor Timesheet)

#### 8.2.1 RWA / Change Order Creation
**Status:** ❌ **Obsolete**

#### 8.2.2 Tracker's data receival
**Status:** ❌ **Obsolete**

### 8.3 PSWS (TCO White Page)

**Status:** ❌ **Obsolete** — CC Owner role not applicable

> **Note (13.04):** White Pages may still be relevant for auto-populating user contact data. Requires verification.

### 8.4 DOA (Delegation of Authority)

**Status:** ❌ **Obsolete**

### 8.5 DataLake / PI

From DataLake: utilization information from IVMS and WIALON trackers. Equipment "Tracker ID" allows merging records with DataLake utilization and location data.

**Status:** ✅ Confirmed

> **Note (07.04):** WIALON integration blocked pending cyber assessment. IVMS and Vega in scope. — ⚠️

🆕 **GIS / Map integration (07.04, 17.04):** Integration with MAPH/Atlas GIS tool via **iframe + URL parameters**; equipment ID sync; Azure SSO pass-through. Equipment list checkboxes pass selected IDs to GIS iframe. — ⚠️ Pending cyber assessment approval (Светлана)

> **🆕 [New 17.04] Route tracking (OQ-NEW-K):** Capability to display equipment movement route on map for a selected date range — data would be sourced from trackers via DataLake. Decision pending from fleet owners (due 20.04).

---

## 9 Open Conflicts Summary

| # | Conflict | BRD Position | Meeting Decision | Status |
|---|----------|-------------|-----------------|--------|
| ~~CONFLICT-01~~ | ~~FO editing booking period after confirmation~~ | ~~FR-048: editing before confirmation only~~ | ~~OQ-40: open~~ | ✅ **RESOLVED (10.04):** FO can edit before AND after confirmation. |
| ~~CONFLICT-02~~ | ~~Booking status auto-transition by datetime~~ | ~~FR-070/071/075: automatic~~ | ~~R-75 (09.04): manual close~~ | ✅ **RESOLVED (14.04):** Manual close accepted. FR-070, FR-071, FR-075 → Obsolete. |
| ~~CONFLICT-03~~ | ~~External fleet booking workflow~~ | ~~FR-051..057: CC Owner → External FO~~ | ~~08.04: on-demand BP = showcase only~~ | ✅ **RESOLVED** |

---

## 10 Open Questions

| OQ ID | Description | Owner | Status |
|-------|-------------|-------|--------|
| OQ-29 | Dynamic filter matrix: equipment type → filter set | TCO | ⚠️ Partially defined (see 5.1.1); remaining types → OQ-NEW-F |
| ~~OQ-30 / R-48~~ | ~~One fleet → multiple FOs~~ | — | ✅ **PARTIALLY CLOSED (13.04):** Fleet has 2+ FOs with shared email. Conflict resolution rules TBD |
| OQ-36 | Aggregated Request status state machine (exact mapping) | Team | ⚠️ Open |
| OQ-37 | Booking Snapshot — exact field list | Team | ⚠️ Open |
| OQ-41 | UI details for Unwheeled flow / manual close fields | Team | ⚠️ Open |
| **OQ-NEW-A** | Concrete field list for equipment cards per equipment type | TCO Fleet Owners | ⚠️ Partially closed (13.04); remaining types — Action Асылбек |
| **OQ-NEW-B** | Request / Equipment Template scope — baseline first; custom attrs → backlog | TCO | ⚠️ Open |
| **OQ-NEW-C** | Service Work Processor role definition — graphist/scheduler? JDE trigger status? | TCO (Аяжан, Dias / SA) | ⚠️ Open |
| ~~OQ-NEW-D~~ | ~~Shared with Conditions vs. without~~ | — | ✅ **CLOSED (13.04)** |
| **OQ-NEW-E** | FO delegation — pending TCO Cybersecurity sign-off | TCO Cybersecurity | ⚠️ Open |
| **OQ-NEW-F** | Equipment type parameters not yet covered: Vacuum Truck and others | TCO (Асылбек) | ⚠️ Open |
| ~~**OQ-NEW-G**~~ | ~~Work Center — attribute of equipment or type of work?~~ | — | ✅ **CLOSED (16.04):** WC = тип работ (шаг), not equipment attribute. |
| **OQ-NEW-H** | Utilization booking threshold — can booking be submitted for equipment near 100% utilization? | TCO | ⚠️ Open |
| ~~**OQ-NEW-I**~~ | ~~Priority level attribution — Request vs. WC step vs. equipment item?~~ | — | ✅ **CLOSED (17.04):** Priority = attribute of Work Order (Request). All WC steps share the same WO priority. |
| ~~**OQ-NEW-J**~~ | ~~Department in reports — Fleet Owner's department or Equipment's department?~~ | — | ✅ **CLOSED (17.04):** Reports use **Equipment's department**. |
| **OQ-NEW-K** | **🆕 [New 17.04] Route tracking** — should the system display equipment movement route on GIS map for a selected date range? Data from trackers via DataLake. | Флитоунеры (Асылбек, Ирлан) | ⚠️ Open — decision due **20.04** |

---

## 11 Requirements Not in Original BRD (New from Meetings)

| New ID | Summary | Source | Status |
|--------|---------|--------|--------|
| FR-NEW-01 | All time parameters configurable via Admin Panel | 03.04 | ✅ |
| FR-NEW-02 | FO delegation to any TCO employee with auto-revoke | 08.04 | ⚠️ Cybersec pending |
| FR-NEW-03 | Admin configures FO timeout thresholds | 03.04 | ✅ |
| FR-NEW-04 | Admin authorizes users to book Assigned equipment | 08.04 | ✅ |
| FR-NEW-05 | FO uploads multiple photos; Requestor sees them | 08.04, 14.04 | ✅ |
| FR-NEW-06 | Repair status from JDE/DataLake in search and list | 08.04, 14.04 | ⚠️ |
| FR-NEW-07 | Stationary HDE excluded from Requestor search | 08.04 | ✅ |
| FR-NEW-08 | Assigned equipment: visible to all, bookable by authorized only | 08.04 | ✅ |
| FR-NEW-09 | Shared with Conditions has visual color marker | 03.04 | ✅ |
| FR-NEW-10 | On-demand only; no weekly schedules | 03.04 | ✅ |
| FR-NEW-11 | Priority P1–P4 selected by Requestor; WO-level attribute | 03.04, **17.04** | ✅ |
| FR-NEW-12 | WO mandatory for Maint/Railroad/Ops; optional for Logistics; Location for SCM | 08.04, 16.04 | ✅ |
| FR-NEW-13 | "Work Description" mandatory (~50 chars) | 08.04 | ✅ |
| FR-NEW-14 | "Comments" optional | 08.04 | ✅ |
| FR-NEW-15 | Single form → auto-split into atomic bookings | 09.04 | ✅ |
| FR-NEW-16 | Partial confirmation per RequestItem | 03.04 | ✅ |
| FR-NEW-17 | Aggregated Request status auto-calculated | 09.04 arch | ⚠️ |
| FR-NEW-18 | Booking start = mobilization start | 03.04 | ✅ |
| FR-NEW-19 | No mandatory buffer between bookings | 09.04 | ✅ |
| FR-NEW-20 | Unwheeled equipment warning in UI | 07.04 | 🟡 |
| FR-NEW-21 | FO timeout: 24h reminder / 48h escalation | 03.04, 06.04 | ✅ |
| FR-NEW-22 | FO must state reason for decline if equipment available | 06.04 | ✅ |
| FR-NEW-23 | Reminder only to FO of specific equipment | 07.04 | ✅ |
| FR-NEW-24 | FO "Mobilization started" button | 09.04 | ✅ |
| FR-NEW-25 | BP showcase: view-only catalog | 08.04 | ✅ |
| FR-NEW-26 | BP prices not displayed | 08.04 | ✅ |
| FR-NEW-27 | BP self-populates catalog cards | 08.04 | ✅ |
| FR-NEW-28 | "Go to BP catalog" button after FO timeout | 08.04 | ✅ |
| FR-NEW-29 | Three-step Unwheeled flow with Transportation role | 09.04 | ✅ |
| FR-NEW-30 | FO proposes nearest date if transport unavailable | 03.04 | ⚠️ |
| FR-NEW-31 | Early close button for Requestor/FO | 07.04 | ⚠️ |
| FR-NEW-32 | Manual close only | 09.04 | ✅ |
| FR-NEW-33 | Actual start/end time at close for utilization | 09.04 | ✅ |
| FR-NEW-34 | FO reminder 24h / escalation 48h | 03.04, 06.04 | ✅ |
| FR-NEW-35 | Notification to Transportation Responsible | 09.04 | ✅ |
| FR-NEW-36 | FO notified of Transportation Responsible decision | 09.04 | ✅ |
| FR-NEW-37 | Audit trail on demand | 07.04 | ✅ |
| FR-NEW-38 | Search: TCO number + model mandatory; госномер conditional | 08.04, 13.04 | ✅ |
| FR-NEW-39 | Dynamic search filters by equipment type | 08.04 | ⚠️ Partial |
| FR-NEW-40 | Booking calendar in equipment card (FO view) | 13.04 | ✅ |
| FR-NEW-41 | Multiple trackers per unit; add from UI | 13.04, 14.04 | ✅ |
| FR-NEW-42 | Shared team email per fleet (Admin-managed) | 13.04 | ✅ |
| FR-NEW-43 | Notifications to fleet shared team email | 13.04 | ✅ |
| FR-NEW-44 | Request → Completed when last active booking closed | 14.04 | ✅ |
| FR-NEW-45 | "Completed Requests" page | 14.04 | ✅ |
| FR-NEW-46 | Utilization reports by day / department / equipment | 14.04 | ⚠️ |
| FR-NEW-47 | Work center reports | 14.04, 16.04 | ⚠️ |
| FR-NEW-48 | "Выбрать WO из JDE" block; auto-generate draft | 16.04 | ✅ |
| FR-NEW-49 | Freeze fields in equipment card | 16.04 | ✅ |
| FR-NEW-50 | Admin manages all reference/handbook values | 16.04 | ✅ |
| **FR-NEW-51** | **Default WO / Default WC for non-JDE users (Logistics and others)** | **17.04** | ✅ |
| **FR-NEW-52** | **"История заявок" page for Requestor (completed requests history)** | **17.04** | ✅ |
| **FR-NEW-53** | **Utilization dashboard — 3 tabs: Моточасы / Километраж / Комбайн** | **17.04** | ✅ |
| **FR-NEW-54** | **Utilization cells — color gradient: 0% red → 50% yellow → 100% green** | **17.04** | ✅ |
| **FR-NEW-55** | **Repair days — wrench icon; JDE fields (commitment/planned/completed date); excluded from average** | **17.04** | ✅ / 🔵 Phase 2 (manual) |
| **FR-NEW-56** | **Utilization dashboard — Target value column (FO-defined max per equipment)** | **17.04** | ✅ |
| **FR-NEW-57** | **Utilization dashboard — Average column; sortable by average** | **17.04** | ✅ |
| **FR-NEW-58** | **Utilization dashboard — Day/Month view toggle** | **17.04** | ✅ |
| **FR-NEW-59** | **Utilization dashboard — Ownership filter (TCO Owned / Long-term rented)** | **17.04** | ✅ |
| **FR-NEW-60** | **Utilization from trackers only; independent of bookings** | **17.04** | ✅ |
| **FR-NEW-61** | **GIS map — iframe; checkboxes in equipment list; ТШО search; expand button** | **17.04** | ✅ *(GIS team)* |
| **FR-NEW-62** | **GIS map tooltip — current WO + today's utilization %** | **17.04** | ⚠️ *(GIS team)* |
