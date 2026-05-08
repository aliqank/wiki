# HDV/HDE Booking Tool

# Business Requirement Document — v9

**Version:** 9.0  
**Date:** 2026-04-22
**Based on:** BRD v8 (2026-04-22) + Open Questions added 22.04.2026  
**Prepared by:** Telman Nurzhanov (SA)

---

## Changelog v8 → v9

| # | Section | Change |
|---|---------|--------|
| 1 | 10. Open Questions | OQ-NEW-P added: data update frequency for JDE E1 Work Orders |
| 2 | 10. Open Questions | OQ-NEW-Q added: data update frequency for utilization metrics (tracker telemetry) |

---

## Summary of Changes vs. Original BRD

The table below reflects the most significant scope and design decisions made during requirements gathering sessions (April 2026) relative to the original BRD delivered by the client.

| Area | Original BRD | Current Version (v9) |
|------|-------------|----------------------|
| Booking scope | Full booking workflow for both internal (TCO) and external (BP) equipment | Internal and long-term rented equipment only. On-demand BP equipment is displayed as a catalog/showcase — no booking workflow for on-demand BP in Phase 1 |
| External approval chain | CC Owner → External Fleet Owner required for BP equipment bookings | CC Owner role is obsolete. All external booking FRs (FR-051–057) removed from scope |
| DCT integration | Required: RWA/Change Order creation; tracker data forwarding | Obsolete — external fleet booking removed from scope |
| PSWS integration | Required: CC Owner identification; user email from White Pages | CC Owner role removed; PSWS may be revisited for contact data only |
| DOA integration | Required: DOA processing of CC Owner bookings | Obsolete |
| Booking status transitions | Automatic by system datetime (FR-070, FR-071, FR-075) | Manual close only — status changes are triggered by user action, not datetime |
| FO date editing | Before confirmation only (FR-048 original) | Before and after confirmation; Requestor notified automatically; no re-confirmation required |
| Equipment classification | Internal fleet / External fleet | Three dimensions: Mobility (self-propelled / unwheeled / stationary) × Ownership (TCO Owned / Long-term rented / BP Showcase) × Usage Status (Assigned / Shared without Conditions / Shared with Conditions) |
| Equipment card — BP fields | Not described | Unified database for TCO and BP equipment; subset of fields optional for BP |
| Utilization | Basic monitoring via trackers (FR-099) | Full utilization dashboard: 3 tabs, color gradient, repair-day markers, target values, average, Work Center filter, ownership filter |
| GIS map | Basic coordinates display (FR-100) | Full GIS iframe integration (MAPH/Atlas team); speed/direction; historical route; tooltip with sensor data |
| New roles | — | Transportation Responsible (for unwheeled equipment transport) |
| New concepts | — | Mobilization period; Default Work Order / Work Center; equipment freeze mechanism; Booking snapshot |
| Fleet structure | One Fleet Owner per fleet | Fleet can have 2+ Fleet Owners with shared team email |

---

## Status Reference

| Status | Description |
|--------|-------------|
| Confirmed | Requirement confirmed in meetings; no conflicts |
| Pending | Requirement accepted; details TBD or pending sign-off |
| Obsolete | Cancelled or replaced by another requirement |
| Backlog | Confirmed direction; deferred after baseline implementation |
| Conflict | Contradiction between original BRD and meeting decisions; requires explicit sign-off |

---

## 1 Introduction

### 1.1 Purpose

This document defines the business requirements for a booking tool that allows users to create requests for HDV/HDE equipment. Requests may include multiple pieces of equipment, each of which is treated as a separate booking with its own approval workflow.

### 1.2 Scope

The tool will support bookings for company-owned equipment as well as long-term rented equipment. It will cover request creation, approval flow, status tracking, reporting, and integration with external systems where applicable.

> **Scope decision (08.04):** Booking through the tool is available only for TCO Owned and Long-term rented equipment. On-demand BP (external fleet) is displayed as a catalog/showcase only — no booking workflow for on-demand BP in the system. This is a significant scope reduction vs. the original BRD. See Section 6.7 for detail.

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

**Service Work Request** – a request created in HDV/HDE automatically via API based on a Work Order created in JDE E1, once the Work Order step reaches a certain status. *(Pending confirmation — see OQ-NEW-C)*

**Booking** – individual booking within a request. One booking is linked to one unique equipment item. Treated individually within a request.

**Equipment** – individual asset (HDV/HDE unit) that can be booked through the system. Each equipment item belongs to a fleet and inherits approval responsibility from the corresponding Fleet Owner.

**Fleet** – logical group of equipment owned by a Fleet Owner (for internal: Maintenance, Construction, TFM, etc.; for external: business partner's name). Each fleet has a unique name assigned at creation.

**Fleet Owner (FO)** – person responsible for approvals within a fleet. One fleet can have 2+ Fleet Owners (e.g. shift workers). A fleet has a shared team email used for system notifications.

**Approver** – Fleet Owner and/or Cost Center (CC) owner, who confirms/declines bookings.

**Mobilization** *(Мобилизация)* – the period of preparation/movement of equipment before it arrives at the work site. Booking period starts from mobilization start, not from arrival. Added during requirements gathering.

**Equipment categories — Usage Status:**
- **Assigned** – equipment assigned to a specific department; visible to all users, bookable only by authorized users with FO role. *(Закреплённая техника)*
- **Shared without Conditions** – equipment available for general booking without any preconditions. *(Общедоступная без условий)*
- **Shared with Conditions** – equipment available for booking, but the Requestor must provide a **Justification** field. Used for specialized equipment where FO needs context (e.g. specific compressors, chemical tanks). Fleet Owner can recall any booking at any time regardless of this status — recall is a separate mechanism. *(Общедоступная с условиями; обоснование обязательно. Решение закрыто 13.04)*

**Equipment Ownership Types:**
- **TCO Owned** – internally owned by TCO.
- **Long-term rented** – rented long-term; treated as internal fleet for booking purposes. Managed by internal TCO team (not by BP).
- **On-demand rented (BP Showcase)** – rented on demand; displayed as catalog/showcase only in Phase 1.

**Utilization** *(Утилизация)* – daily efficiency of equipment usage; reflects how effectively the company manages its equipment.
- Tracked by **mileage** (wheeled equipment) or **engine hours** (other equipment types), or **both** depending on equipment category.
- Each Fleet Owner independently defines the **100% utilization threshold** for each equipment type (maximum moto-hours or km per 24 hours = 100%).
- **Maximum Utilization Capacity per Shift** — maximum possible utilization within a working shift; serves as the 100% reference point.
- Utilization percentage is calculated per day, month, or any selected period.
- Some equipment types have **both** mileage and engine hours parameters. Planned values are entered by FO; actual values are sourced from trackers only (not manually editable). *(Updated 13.04)*
- Utilization is sourced from **trackers only**, independent of whether a booking exists in the system. This allows detecting equipment usage not linked to any system request. *(Updated 17.04)*

**Utilization Calculation Formulas** *(Updated 17.04):*

| Metric type | Formula |
|-------------|---------|
| Engine hours only | `utilization = engine_time / target_mh_per_day` |
| Mileage only | `utilization = mileage / target_km_per_day` |
| Both (hours + km) | `utilization = max(engine_time / target_mh_per_day, mileage / target_km_per_day)` |
| Cap rule | If `utilization > 1` → display as 1 (100%) |

> **Business goal (14.04):** Eliminate equipment downtime. Primary metric — how effectively equipment is used.

**Utilization Dashboard tabs** *(Added 17.04):*
- **Engine hours** — engine hours utilization only. Applicable to HDE equipment.
- **Mileage** — mileage utilization only. Applicable to HDV equipment.
- **Combined** — maximum of the two metrics (Combined). For equipment with both engine hours and mileage.

**Utilization average calculation** *(Added 17.04):*
- Average is calculated over the **selected date range**.
- **Repair days are excluded** from the calculation denominator (days when equipment had an open JDE maintenance Work Order).
- Days when equipment stood idle (not on repair) **are included** in the calculation.

**Priority** *(Updated 17.04):*
> Priority is an **attribute of the Work Order (Request)** as a whole, not of an individual Work Center step and not of a specific equipment item. All Work Center steps within the same Work Order share the same priority as the parent WO. *(OQ-NEW-I — CLOSED)*

**Department in reports** *(Updated 17.04):*
> When generating department-level reports, the department is determined by the **Equipment's** department assignment — not by the Fleet Owner's department. Fleet Owner and Equipment may belong to different departments. *(OQ-NEW-J — CLOSED)*

**Criticality** *(Критичность)* – equipment marked as critical requires **security escort (service de sécurité)** during transportation. Not a maintenance priority indicator. Displayed in equipment card. *(Updated 13.04)*

**Work Center** *(Рабочий центр / Шаг)* — scope of work to be performed within a Work Order; called a "step" because work is performed sequentially. Attribute of the Work Order Step, not of the equipment unit. *(Added 16.04)*

> **Key constraints:**
> - One equipment type maps to exactly **one** Work Center.
> - One Work Center can be served by **multiple** equipment types.

**Preliminary Work Center codes** (subject to update by Asylbek):

| Code | Description |
|------|-------------|
| LRG | Группа ГПМ |
| MGT | Механики по дизелям |
| BHOE | Экскаватор, трактор ОТО |
| DTRK | Самосвал ОТО |
| FTRK | Грузовик ОТО |
| GDRV | Грузовик с краном ОТО |
| HYDR | Водоструйная очистка ОТО |
| OILT | Oil Tanker |
| VTRK | Вакуумная машина ОТО |
| CDECK | Транспортная платформа ОТО |
| CRANE | Кран ОТО |
| FORKL | Вилочный погрузчик ОТО |
| GMLOG | Механик передвижного оборудования |
| MLIFT | Подъёмник ОТО |
| MSINS | ППУ |

> **Note:** Logistics subdivision does not operate Work Centers in JDE. WO and WC fields are optional for Logistics. *(OQ-NEW-G — CLOSED 16.04)*

**Historical route** *(Маршрут движения)* – display of equipment movement on the GIS map for a selected date range; data sourced from trackers via DataLake. Displayed for up to **7 days**. *(OQ-NEW-K — CLOSED 20.04)*

---

## 4 Stakeholders & Roles

### 4.1 Requestor

All users in TCO network will by default have Requestor role to book equipment of the internal fleet (TCO Owned + Long-term rented HDV/HDE). On-demand external (BP) equipment is displayed as a showcase/catalog only — booking of external equipment is handled in DCT, not in this tool.

Requestors are allowed to:

1. Create a Regular Request — Confirmed
2. Submit the Regular Request — Confirmed
3. Prolong booking — Confirmed
4. Submit feedback on the booked equipment — Confirmed
5. Monitor utilization of the booked equipment for the booking period, if equipment has tracker(s) — Confirmed
6. ~~Terminate booking (only external fleet)~~ — Obsolete (external fleet booking removed from scope)
7. Close booking by pressing "Close" button (manual close only) — Confirmed *(Added)*
8. Book Assigned equipment only if authorized by Administrator — Confirmed *(Added)*
9. View own request history (completed requests) — Confirmed *(Added 17.04)*

### 4.2 Service Work Processor

> **Pending clarification (13.04):** Based on meeting discussion, the Service Work Processor role is likely **graphists / schedulers** who work with Work Orders in JDE and process drafts once a Work Order step reaches confirmed status (≈55 in JDE). They push Service Work Requests from Draft to Submitted for Fleet Owner processing. Formal confirmation from TCO required (OQ-NEW-C).

~~Access provision is managed by Azure Active Directory (AAD) group. Similar to Requestor, however, Service Work Processor works with Service Work Requests, created automatically in HDV/HDE booking tool.~~

### 4.3 Fleet Owner

For internal Fleet Owners, access provision is managed by AAD group. There can be 2+ AAD groups to manage internal Fleet Owners (separate group for each fleet created by Administrator).

One fleet can have **two or more Fleet Owners** (e.g. shift workers). Both owners are displayed in the equipment card. Each fleet has a **shared team email** address used as a single notification target (alongside personal emails via AD group). When a Fleet Owner changes role, Administrator reassigns the fleet without rebuilding the fleet structure. *(Updated 13.04)*

Fleet Owners are allowed to:

1. Create new equipment under own fleet — Confirmed
2. Update equipment (only parameters allowed for editing by Administrator) — Confirmed
3. Delete equipment (soft delete) — Confirmed
4. Confirm / decline requests to book equipment under own fleet — Confirmed
5. Update booking period before or after confirmation; Requestor is notified automatically; no re-confirmation required — Confirmed *(Updated 10.04)*
6. Change equipment in booking — Confirmed
7. Terminate already confirmed bookings (only internal fleet) — Confirmed
8. Freeze / unfreeze own equipment from booking for a period or without end date — Confirmed
9. Monitor utilization of own equipment via utilization dashboard — Confirmed
10. Press "Mobilization started" button to fix mobilization start time — Confirmed *(Added)*
11. Delegate FO privileges to any TCO employee for a specified period — Pending cybersecurity sign-off (OQ-NEW-E) *(Added)*
12. One user can be an owner of several fleets — Confirmed *(Added)*
13. One fleet can have multiple Fleet Owners; both displayed with shared team email — Confirmed *(Updated 13.04)*

### 4.4 CC Owner (incl. DOA)

> **Obsolete (10.04):** CC Owner confirmation is NOT required for Long-term rented equipment. CC Owner is applicable only to on-demand external fleet booking, which is handled in DCT system.

CC owners are allowed to:

1. ~~Confirm / decline external fleet equipment booking request~~ — Obsolete

### 4.5 Administrator

Administrator role will be provided to process owners – Logistics. Access provision is managed by Azure Active Directory (group). Users with an Administrator role are allowed to:

1. Create/edit internal fleet and link with AAD group — Confirmed
2. Create/edit external fleet and manage external fleet owners — Confirmed
3. Delete fleet — Confirmed
4. Create internal fleet equipment — Confirmed
5. Update equipment's parameters (extended edit) — Confirmed
6. Delete equipment (soft delete) — Confirmed
7. Manage 'Request' template — Pending (baseline first; custom attributes → Backlog, OQ-NEW-B)
8. Manage 'Equipment' template — Pending (baseline first; custom attributes → Backlog, OQ-NEW-B)
9. Manage 'Booking' template — Pending (OQ-NEW-B)
10. Access to custom reportings — Confirmed
11. Access to utilization dashboard(s), if equipment has tracker(s) — Confirmed
12. Manage all configurable system parameters via Admin Panel — Confirmed *(Added)*
13. Authorize specific users to book Assigned equipment — Confirmed *(Added)*
14. Manage delegation settings — Pending cybersecurity sign-off (OQ-NEW-E) *(Added)*
15. Manage fleet shared team email — Confirmed *(Updated 13.04)*
16. Manage reference/handbook values (equipment classes, categories, subcategories, manufacturers, models, companies, locations, work centers, divisions, groups, departments, units) via Admin Panel — Confirmed *(Added 16.04)*

### 4.6 Transportation Responsible *(Мобилизация/Транспортировка)*

A dedicated system role for the team responsible for transporting non-motorized (Unwheeled) equipment (Heavy Ops / Maintenance team). Added during requirements gathering.

This role is allowed to:

1. Receive notification about transportation request for Unwheeled equipment — Confirmed
2. Confirm or decline the transportation request — Confirmed

---

## 5 Entities

### 5.1 Equipment

All equipment must have a mandatory attribute — fleet. The attribute determines the equipment template, who can create/edit/delete it, and the approval and processing flow.

**Fleet naming (10.04):** Each fleet must have a unique **name** assigned at creation. Equipment assignment is linked to a specific employee; when a Fleet Owner changes, the Administrator manually re-assigns the fleet.

**Fleet Owners display (13.04):** Equipment card displays **up to 2+ Fleet Owners** alongside their **shared team email**.

**Work Center assignment (16.04):** Each equipment type is assigned to **exactly one Work Center**. Managed via Admin Panel reference.

**Reference/Handbook policy (16.04):** The following are handbook-managed via Admin Panel:
- Equipment classes (HDE, HDV)
- Categories and subcategories
- Manufacturer, Model
- Companies (TCO White Pages)
- Locations
- Work Centers
- Divisions, Groups, Departments, Units

**Unified database (confirmed 20.04):** TCO-owned and BP (long-term rented) equipment share a single database. For BP equipment, a subset of fields is optional (null allowed):

| Field | TCO Owned | BP (Long-term rented) |
|-------|-----------|----------------------|
| Category / Subcategory | Mandatory | Mandatory |
| Manufacturer / Model | Mandatory | Mandatory |
| Ownership | Mandatory | Mandatory |
| TCO (ТШО) number | Mandatory | Optional |
| License plate | Optional | Optional |
| VIN / Serial number | Optional | Optional |
| Year of manufacture | Recommended | Optional |
| Service zone | Optional | Optional |
| Repair company | Recommended | Recommended |

> **Note on repair company (20.04):** It is recommended to store the repair company for all equipment. A single unit may have different contractors for different work types (e.g. hydraulics — Energstim; extraction — DAS).

**Operational Status removed:** «Операционный статус» (Operational Status) is **removed** from equipment card parameters. Remaining status fields: repair status and freeze status. *(Updated 16.04)*

Internal fleet equipment parameters:

1. Basic fixed info (TCO ID, Description, Service Zone, Class/Category, License Plate, Serial number, etc.) — Confirmed / Pending details
2. Dynamic parameters: Equipment Usage Status, **Cost Center**, CC/Division/Group/Department/Unit, Assigned location, Maintenance BP, Contact number / email, etc. — Confirmed
3. **Ownership field** — «Принадлежность»: **TCO Owned** / **Long term rented** — Confirmed *(Added 16.04)*
4. **Freeze fields** *(Added 16.04)*:
   - **Заморозка** (freeze flag)
   - **Причина заморозки** (freeze reason)
   - **Дата завершения заморозки** (freeze end date; empty = indefinite)
   > Visible and editable by FO and Admin. (FR-NEW-49)
5. Equipment specifications (picture, Chassis type, Drive type, Loading capacity, etc.) — Confirmed
6. Trackers' IDs (WIALON, IVMS, Vega, etc.) — Confirmed. Multiple trackers per unit supported *(Added)*
7. Planned engine hours (per shift/day) — editable by FO — Confirmed *(Added)*
8. Planned daily mileage (km/day) — editable by FO — Confirmed *(Added)*
9. Actual engine hours / mileage — sourced from tracker only; not manually editable — Confirmed *(Added)*

**Equipment card UI (13.04–17.04):**
- **Booking calendar**: visual calendar showing booked periods
- **Criticality flag**: displayed in card
- **Work schedule display**: split into day shift / night shift blocks
- **Weekend coverage**: default shift 06:00–06:00; Long Term — 07:00–07:00
- **Multiple photos**: supports uploading multiple photos per unit
- **Sidebar**: Comments block in sidebar
- **Tracker block**: add new trackers from equipment card UI
- **Schedule slider**: slider control for selecting time ranges in work schedule block *(Added 16.04)*

**GIS map integration in equipment list (17.04):**
- Equipment list (FO view) includes **checkboxes** to select units for display on the GIS map
- Map is rendered by GIS team (MAPH/Atlas) as an **iframe**; system passes equipment identifiers as parameters
- See FR-NEW-61–FR-NEW-67

**License plate and TCO number rules (13.04):**
- **License plate**: present on most wheeled equipment; may be absent on stationary units.
- **TCO number**: assigned at purchase; long-term rented equipment may not have one.

**Equipment Classification — three dimensions:**

| Dimension | Values |
|-----------|--------|
| Mobility | Self-propelled / Not self-propelled (Unwheeled) / Stationary |
| Ownership | TCO Owned / Long-term rented / On-demand rented (BP Showcase) |
| Usage Status | Assigned / Shared without Conditions / Shared with Conditions |

> **Note (08.04):** Mobility category NOT displayed in Requestor UI.
> **Note (08.04):** Stationary HDE excluded from Requestor search.
> **Note (13.04):** Shared with Conditions differs only in presence of mandatory Justification field.

**Equipment availability display rules:**
- Equipment under maintenance shown grayed out; label "Under maintenance until [Planned date from JDE]"
- Repair status sourced from JDE → DataLake integration — Pending details

---

### 5.1.1 Equipment Type-Specific Parameters *(Added 13.04)*

> Parameters define specific attributes and search filters for each equipment type. Preliminary — subject to final sign-off from TCO Fleet Owners.


---

### 5.2 Booking

Booking is linked to a single equipment item and has a booking range (datetime). Several bookings form a single request.

> **Architecture decision (09.04):** One Request → multiple RequestItems (one per equipment); each independently processed by FO.

#### 5.2.1 Internal Booking

Booking of TCO Owned (including Long-term rented) equipment. Requires only FO confirmation.

> **Mobilization (09.04):** Booking period starts from mobilization start. FO presses "Mobilization started" to fix actual start.

> **Updated (10.04):** FO can adjust booking period before and after confirmation. Requestor notified automatically.

#### 5.2.2 External Booking

**Scope change vs. original BRD:** On-demand BP equipment is NOT bookable in Phase 1. Displayed as catalog/showcase only.

- On-demand rented (BP Showcase) — catalog view only.
- Long-term rented — treated as internal fleet.
- ~~Full external booking workflow (CC Owner → External FO)~~ — Obsolete

#### 5.2.3 Booking Snapshot

At booking confirmation, system saves a snapshot of key equipment attributes. — Pending (OQ-37)

### 5.3 Request

#### 5.3.1 Regular Request

Requestor can add **1 or more** equipment items from any fleet; each is treated as a separate booking.

**Request fields (08.04):**

- **Work Order (JDE)** — mandatory for Maintenance / Railroad / Operations. One WO per request.
- **Work Center (WC)** — mandatory for JDE-based fleets; **optional for Logistics**.
- **Location** (text input) — for SCM Logistics replaces WO.
- **Work Description** — mandatory, ~50 chars.
- **Comments** — optional.
- **Priority** (P1–P4) — attribute of the Request / Work Order. *(OQ-NEW-I — CLOSED)*

> **WO pre-fill flow (16.04):** Block "Выбрать Work Order из JDE" on Submit Request page. See FR-NEW-48.

> **Default WO / WC (17.04):** For teams not using JDE (Logistics and others) — "Default Work Order" checkbox available; no JDE number required. Analogously "Default Work Center" for teams without WC. See FR-NEW-51.

**Request constraints (03.04):**
- Booking horizon: 1 month ahead (configurable).
- Max booking duration: 1 week (configurable).

#### 5.3.2 Service Work Request (within Work Order in JDE E1)

> **Pending clarification (13.04):** Auto-created as Draft when WO step reaches status 55 in JDE. SWP reviews and submits to FO. Formal confirmation required (OQ-NEW-C).

**Required WO data fields from DataLake (Updated 16.04):**

| Field | Description / Notes |
|-------|---------------------|
| Work Order number | Unique identifier in JDE E1 |
| Work Order name | Examples needed from TCO |
| Status | Clarify list of valid statuses |
| Status description | Text description *(Added 16.04)* |
| Priority | Priority value in JDE E1 — attribute of the WO |
| Work Center (Step code) | Type of work; e.g. "CRANE", "DTRK" |
| Step name | Text name of the step |
| Step volumes | Required quantity of equipment units per step *(Added 16.04)* |
| Date range | Start date / End date of the WO step *(Added 16.04)* |

---

## 6 Functional Requirements

### 6.1 User & Role Management

| ID | Requirement | Status |
|----|-------------|--------|
| FR-001 | System supports role-based access control | Confirmed |
| ~~FR-002~~ | ~~All TCO network users have Requestor access (with RWA initiator restriction for BP booking)~~ | Obsolete — superseded by updated Requestor definition |
| FR-003 | Administrators, Service Work Processors defined by AAD groups | Pending (SWP — OQ-NEW-C) |
| ~~FR-004~~ | ~~CC Owner / DOA privileges via PSWS/DOA integration~~ | Obsolete |
| FR-NEW-01 | All time-based parameters configurable via Admin Panel (not hardcoded) | Confirmed |
| FR-NEW-02 | FO can delegate privileges to any TCO employee for a date range; auto-revoke upon expiry | Pending cybersec sign-off (OQ-NEW-E) |

### 6.2 Admin Panel

| ID | Requirement | Status |
|----|-------------|--------|
| FR-005 | Admin creates fleets with unique names | Confirmed |
| FR-006 | Admin links internal fleet with AAD group | Confirmed |
| FR-007 | Admin manages external fleet owners | Confirmed |
| FR-008 | Admin updates existing fleets | Confirmed |
| FR-009 | Admin deletes fleets | Confirmed |
| FR-010 | Admin manages equipment template (baseline first; custom attributes → Backlog) | Pending / Backlog |
| FR-011 | Admin manages request template | Backlog |
| FR-012 | Admin manages booking template | Pending |
| FR-013 | Admin creates equipment (internal fleet) | Confirmed |
| FR-014 | Admin edits equipment (extended edit) | Confirmed |
| FR-015 | Admin deletes equipment | Confirmed |
| FR-016 | Admin accesses all dashboards and reports | Confirmed |
| FR-NEW-03 | Admin configures: FO timeout (24h/48h), booking horizon, max duration | Confirmed |
| FR-NEW-04 | Admin authorizes specific users to book Assigned equipment | Confirmed |
| FR-NEW-42 | Admin sets and updates fleet shared team email | Confirmed |
| FR-NEW-50 | Admin manages all reference/handbook values via Admin Panel | Confirmed |

### 6.3 Equipment Management

| ID | Requirement | Status |
|----|-------------|--------|
| FR-017 | Both internal and external FO can create equipment under own fleet | Confirmed |
| FR-018 | FO must indicate fleet of equipment; only own fleets selectable | Confirmed |
| FR-019 | FO can edit allowed equipment parameters; one user can own several fleets | Confirmed |
| FR-020 | FO can delete equipment (soft delete) | Confirmed |
| FR-021 | FO can freeze/unfreeze equipment for a period or indefinitely | Confirmed |
| FR-022 | Requestor / SWP can submit feedback on equipment with confirmed booking | Confirmed |
| FR-NEW-05 | FO uploads multiple photos; Requestor sees them in request form | Confirmed |
| FR-NEW-06 | Repair status from JDE → DataLake displayed in search and equipment list | Pending |
| FR-NEW-07 | Stationary HDE excluded from Requestor search | Confirmed |
| FR-NEW-08 | Assigned equipment: visible to all, bookable by authorized only | Confirmed |
| FR-NEW-09 | Shared with Conditions has visual color marker | Confirmed |
| FR-NEW-40 | Equipment card (FO view) has booking calendar visual | Confirmed |
| FR-NEW-41 | Multiple trackers per unit; FO can add trackers from equipment card UI | Confirmed |
| FR-NEW-49 | Freeze fields on equipment card: Заморозка, Причина заморозки, Дата завершения заморозки | Confirmed |

### 6.4 Request Management

| ID | Requirement | Status |
|----|-------------|--------|
| FR-023 | Requestor can create a Request | Confirmed |
| FR-024 | Request has unique ID and metadata | Confirmed |
| FR-025 | Requestor can view request details and status | Confirmed |
| FR-026 | Request statuses: Draft, Submitted, In Progress, Completed | Pending — aggregated status TBD (OQ-36) |
| FR-027 | Requestor can edit/cancel draft before submission | Confirmed |
| FR-028 | Service Work Requests auto-created with 'Draft' status | Pending (OQ-NEW-C) |
| FR-029 | Only SWP can manage (submit) Service Work Requests | Pending (OQ-NEW-C) |
| FR-030 | System supports draft saving | Confirmed |
| FR-NEW-10 | On-demand booking only; weekly schedule management out of scope | Confirmed |
| FR-NEW-11 | Requestor selects priority P1–P4 at request creation (priority = WO-level attribute) | Confirmed |
| FR-NEW-12 | WO mandatory for Maintenance/Railroad/Operations; optional for Logistics; Location replaces WO for SCM Logistics | Confirmed |
| FR-NEW-13 | "Work Description" mandatory (~50 chars) | Confirmed |
| FR-NEW-14 | "Comments" optional | Confirmed |
| FR-NEW-15 | Single form → auto-split into individual bookings per equipment item | Confirmed |
| FR-NEW-16 | Partial confirmation: confirmed items proceed independently from declined | Confirmed |
| FR-NEW-17 | Aggregated Request status auto-calculated from RequestItem statuses | Pending (OQ-36) |
| FR-NEW-48 | "Выбрать WO из JDE" block on Submit Request page; WO list from DataLake; auto-generate draft | Confirmed |
| FR-NEW-51 | **Default Work Order** option for teams not operating in JDE (Logistics and others): checkbox on request form; when selected — system does not require a JDE WO number. Analogously **Default Work Center** for teams without WC. *(Added 17.04)* | Confirmed |
| FR-NEW-64 | In the FO booking approval window: display an **equipment loading summary by dates** — a list of nearest active bookings for the given equipment unit (analogous to a ticket availability view). Allows FO to assess scheduling conflicts before approving. *(Added 20.04)* | Confirmed |

### 6.5 Booking Management

| ID | Requirement | Status |
|----|-------------|--------|
| FR-031 | Requestor can add/remove equipment items to a request; availability updated | Confirmed |
| FR-032 | System prioritizes internal fleet (business rule) | Confirmed |
| FR-033 | Requestor can add Shared equipment to request | Confirmed |
| ~~FR-034~~ | ~~System suggests external fleet based on decision matrix~~ | Obsolete |
| ~~FR-035~~ | ~~Requestor adds external fleet equipment if RWA initiator~~ | Obsolete |
| ~~FR-036~~ | ~~Justification required when adding external fleet equipment~~ | Obsolete |
| ~~FR-037~~ | ~~Cost Center required for external fleet equipment~~ | Obsolete |
| FR-038 | Each equipment item in request = separate booking | Confirmed |
| FR-039 | Each booking has unique ID, start/end datetimes | Confirmed |
| FR-040 | System validates availability before booking | Confirmed |
| FR-041 | Equipment attributes displayed in booking | Confirmed |
| FR-042 | Booking lifecycle: Draft, Submitted, Confirmed, Declined, Revoked, Terminated, Completed | Confirmed |
| FR-NEW-18 | Booking start = start of mobilization | Confirmed |
| FR-NEW-19 | No mandatory buffer between bookings | Confirmed |
| FR-NEW-20 | Unwheeled equipment: notification displayed to Requestor | Preliminary |

### 6.6 Approval Flow (Internal Fleet)

| ID | Requirement | Status |
|----|-------------|--------|
| FR-043 | Approver assigned automatically based on fleet ownership | Confirmed |
| FR-044 | Internal fleet booking confirmed/declined by FO | Confirmed |
| FR-045 | FO can confirm incoming bookings | Confirmed |
| FR-046 | FO can replace equipment before/after confirmation if booking not yet started | Confirmed |
| FR-047 | FO cannot replace if booking revoked, terminated, or past end date | Confirmed |
| FR-048 | FO can adjust booking date range **before or after** confirmation; Requestor notified automatically; no re-confirmation required *(Updated 10.04 — original: before confirmation only)* | Confirmed |
| FR-049 | FO can decline booking; availability updated | Confirmed |
| FR-050 | FO provides reason/comment when declining | Confirmed |
| FR-NEW-21 | FO must respond within timeout. Default: 24h reminder; 48h escalation | Confirmed |
| FR-NEW-22 | If FO has free equipment and declines — mandatory reason required | Confirmed |
| FR-NEW-23 | Reminder sent only to FO of the specific selected equipment | Confirmed |
| FR-NEW-24 | FO presses "Mobilization started" to record actual start time | Confirmed |

### 6.7 Approval Flow (External Fleet)

> **Major scope change (08.04):** On-demand BP not bookable. All FR-051..057 are Obsolete vs. original BRD.

| ID | Requirement | Status |
|----|-------------|--------|
| ~~FR-051..057~~ | ~~External fleet approval chain (CC Owner, External FO, PSWS, DOA)~~ | Obsolete |
| FR-NEW-25 | BP showcase: Requestor can view on-demand BP equipment; no booking action in Phase 1 | Confirmed |
| FR-NEW-26 | BP equipment prices/rates NOT displayed | Confirmed |
| FR-NEW-27 | BP populates own catalog cards | Confirmed |
| FR-NEW-28 | "Go to BP catalog" button after FO timeout | Confirmed |

### 6.8 Booking Lifecycle Management

| ID | Requirement | Status |
|----|-------------|--------|
| FR-058 | Requestor/SWP can revoke booking if not yet processed by FO | Confirmed |
| FR-059 | FO can terminate confirmed booking; reason required; availability updated | Confirmed |
| FR-060 | Requestor/SWP can terminate confirmed booking (internal only) | Confirmed |
| FR-061 | Requestor/SWP can extend confirmed booking | Confirmed |
| FR-NEW-29 | Three-step Unwheeled flow: Requestor → FO → Transportation Responsible → FO final | Confirmed |
| FR-NEW-30 | If transport unavailable for Unwheeled — FO proposes nearest available date | Pending |
| FR-NEW-31 | Booking can be closed early via "Close early" button | Pending |

### 6.9 Booking/Request Status Management

> **Conflict RESOLVED (14.04):** Auto status transition by datetime cancelled. All status changes are **manual only**.

| ID | Requirement | Status |
|----|-------------|--------|
| FR-062 | Booking → 'Submitted' once Request submitted | Confirmed |
| FR-063 | Booking → 'Confirmed' once confirmed by FO | Confirmed |
| FR-064 | Booking → 'Declined' once declined by FO | Confirmed |
| ~~FR-065~~ | ~~External booking → 'Partially Confirmed' by CC Owner~~ | Obsolete |
| ~~FR-066~~ | ~~External booking → 'Declined' by CC Owner~~ | Obsolete |
| FR-067 | Booking → 'Submitted' once extended by Requestor/SWP | Confirmed |
| FR-068 | Booking → 'Revoked' once revoked | Confirmed |
| FR-069 | Booking → 'Terminated' once terminated | Confirmed |
| ~~FR-070~~ | ~~Auto-transition to 'In Progress' by datetime~~ | Obsolete — manual close (see FR-NEW-32) |
| ~~FR-071~~ | ~~Auto-transition to 'Completed' by datetime~~ | Obsolete — manual close (see FR-NEW-32) |
| FR-072 | Request → 'Draft' when saved as draft | Confirmed |
| FR-073 | SWR → 'Draft' when imported from JDE | Pending (OQ-NEW-C) |
| FR-074 | Request → 'In Progress' when first booking (excl. Declined) goes In Progress | Confirmed |
| ~~FR-075~~ | ~~Request → 'Completed' by datetime~~ | Obsolete — see FR-NEW-44 |
| FR-NEW-32 | Booking closure is manual only — "Close" button by Requestor or FO | Confirmed |
| FR-NEW-33 | At close: Requestor inputs actual start/end time for utilization analytics | Confirmed |
| FR-NEW-44 | Request → 'Completed' when last active (non-Declined) booking closed | Confirmed |

### 6.10 Notifications

| ID | Requirement | Status |
|----|-------------|--------|
| FR-076 | System notifies SWP when SWR auto-created | Pending (OQ-NEW-C) |
| FR-077 | System notifies FO when request submitted | Confirmed |
| FR-078 | System notifies FO when booking revoked | Confirmed |
| FR-079 | System notifies Requestor/SWP of FO decision | Confirmed |
| FR-079a | System notifies Requestor when FO updates booking period | Confirmed |
| FR-080 | System notifies Requestor/SWP when FO terminates booking | Confirmed |
| FR-081 | System notifies FO when Requestor extends booking | Confirmed |
| ~~FR-082..090~~ | ~~External fleet notification chain~~ | Obsolete |
| FR-NEW-34 | 24h reminder to FO; 48h escalation to manager | Confirmed |
| FR-NEW-35 | Notification to Transportation Responsible for Unwheeled booking | Confirmed |
| FR-NEW-36 | FO notified when Transportation Responsible decides | Confirmed |
| FR-NEW-43 | Notifications delivered to fleet shared team email (and/or AD group personal emails) | Confirmed |

### 6.11 Search & Reporting

| ID | Requirement | Status |
|----|-------------|--------|
| FR-091 | Requestor/SWP can search and filter own requests | Confirmed |
| FR-092 | Approver (FO) can view all pending and completed approvals | Confirmed |
| FR-093 | Admin can view all requests | Confirmed |
| FR-094 | System generates reports on Requests/Bookings/Equipment with filters | Confirmed |
| FR-095 | Admin can view/download all reports | Confirmed |
| FR-096 | FO can view/download own approval reports | Confirmed |
| FR-097 | FO can view/download own equipment reports | Confirmed |
| FR-098 | SWP (and FOs) can view SWR grouped by Work Orders | Pending (OQ-NEW-C) |
| FR-NEW-37 | Audit trail and history on demand | Confirmed |
| FR-NEW-38 | Search: TCO equipment number + model mandatory; госномер if present | Confirmed |
| FR-NEW-39 | Dynamic search filters by equipment type (type → filter matrix in 5.1.1) | Pending / Partial |
| FR-NEW-45 | Dedicated "Completed Requests" page for Requestor, SWP, FO, Admin | Confirmed |
| FR-NEW-46 | Utilization reports: by day / department / equipment unit | Pending details |
| FR-NEW-47 | Work center reports (group by WC code) | Pending details |
| FR-NEW-52 | Requestor has access to **«История заявок»** (Request History) — a dedicated page listing all completed requests by this Requestor. Minimum columns: request number, equipment type, booking period, status, Work Order number. *(Added 17.04)* | Confirmed |

### 6.12 Dashboards / Map

| ID | Requirement | Status |
|----|-------------|--------|
| FR-099 | System displays utilization and coordinates from DataLake/PI (tracker data) | Confirmed |
| FR-100 | System displays digitized location of Work Orders from DataLake (JDE E1) | Confirmed |
| FR-NEW-53 | Utilization dashboard has **three tabs**: «Моточасы» (engine hours) / «Километраж» (mileage) / «Комбайн» (max of both). Active tabs depend on equipment type: HDE → Моточасы only; HDV → Километраж only; mixed equipment → all three. *(Added 17.04)* | Confirmed |
| FR-NEW-54 | Utilization cells use a **color gradient**: 0% — red → 50% — yellow → 100% — green. Smooth gradient, not discrete steps. *(Added 17.04)* | Confirmed |
| FR-NEW-55 | Repair days are marked in the dashboard with a **wrench icon** (not a color) to distinguish from low-utilization days. Repair data sourced from JDE (see Section 8.1): `commitment date` = start of repair, `planned date` = expected end, `completed date` = actual end. Repair days are **excluded from average utilization calculation**. Manual repair input for non-JDE equipment. *(Added 17.04)* | Confirmed (manual input) |
| FR-NEW-56 | Utilization dashboard includes a **Target value column** per equipment: maximum daily utilization threshold set by FO (e.g. 2 000 engine hours / 1 500 km per day). Displayed first after equipment name/type columns. *(Added 17.04)* | Confirmed |
| FR-NEW-57 | Utilization dashboard includes an **Average utilization column** for the selected period (repair days excluded). Table is **sortable by this column**. *(Added 17.04)* | Confirmed |
| FR-NEW-58 | Utilization dashboard supports **two time granularity views**: «По дням» (daily, current default) / «По месяцам» (monthly average per cell). User toggles via a control above the table. *(Added 17.04)* | Confirmed |
| FR-NEW-59 | Utilization dashboard left sidebar includes an **Ownership filter**: TCO Owned / Long-term rented. *(Added 17.04)* | Confirmed |
| FR-NEW-60 | Utilization data is sourced from **tracker telemetry only**, independent of bookings. Allows detecting equipment usage not linked to any booking. *(Added 17.04)* | Confirmed |
| FR-NEW-61 | In the FO Equipment section, equipment list includes **checkboxes** to select units for display on the GIS map. The GIS map is rendered as an **iframe** by the GIS team (MAPH/Atlas); system passes selected equipment identifiers as URL parameters. Map supports **expand** button for full-screen view. Search in equipment list by ТШО-номер. *(Added 17.04)* | Confirmed (GIS team involvement required) |
| FR-NEW-62 | GIS map marker **tooltip** on click: displays all available sensor data received from the last tracker synchronization. Data fields are defined by what the tracker transmits (speed, heading, coordinates, engine hours, odometer, ignition, battery, etc.). The displayed field list is subject to refinement based on business feedback. *(Updated 20.04 — original: WO number + today's utilization only)* | Pending alignment with GIS team |
| FR-NEW-63 | Utilization dashboard displays **booking count for the selected period** alongside the average utilization value. Allows analysis of correlation between equipment load (bookings) and actual utilization. *(Added 20.04)* | Confirmed |
| FR-NEW-65 | GIS map displays **speed and heading (direction)** for equipment equipped with GSM trackers. For equipment with LoRaWAN trackers (battery-powered, sync up to 3 times/day): heading is **not displayed** due to infrequent synchronization. *(Added 20.04)* | Confirmed |
| FR-NEW-66 | GIS map does **not display fuel level**. Fuel telemetry data will not be available from the trackers. *(Added 20.04)* | Confirmed |
| FR-NEW-67 | GIS map supports display of **historical route** for selected equipment for up to **7 days**. Data sourced from tracker telemetry via DataLake. *(Added 20.04, OQ-NEW-K CLOSED)* | Confirmed (to be aligned with GIS team) |

### 6.13 Audit & Compliance

| ID | Requirement | Status |
|----|-------------|--------|
| FR-101 | System logs all request creation, modifications, approvals, rejections | Confirmed |
| FR-102 | Audit trail accessible for compliance purposes | Confirmed |
| FR-103 | Exportable logs for legal/regulatory audits | Confirmed |

---

## 7 Non-Functional Requirements

| Area | Requirement | Status |
|------|-------------|--------|
| Availability | 99.5% uptime | Confirmed |
| Performance | High volume of requests per hour without degradation | Confirmed |
| Security | RBAC; Azure AD SSO | Confirmed |
| Compliance | Full audit history of all approvals / rejections / revokes / terminations | Confirmed |
| Scalability | Support internal and external partner use cases | Confirmed |
| Usability | Intuitive UI; UX reference: Booking.com | Confirmed |
| Users | ~200 users (per ACR assessment 07.04); No personal data (SPD) stored | Confirmed |
| Security tier | ACR assessment result: Low Risk (07.04) | Confirmed |
| Authentication | Azure SSO for internal users; separate account for external BP | Confirmed |

---

## 8 Integrations

### 8.1 JDE E1

Work Orders created in JDE E1 are imported into HDV/HDE Booking Tool as Service Work Requests (SWRs) once a defined status is reached. Each Work Order step is imported as a separate SWR. Only steps with approved equipment types (e.g. BHOE, CDECK, DTRK, FORKL, FTRK, GDRV, GMLOG, MLIFT, VTRK, etc.) are included.

**Proposed integration Approach (Updated 13.04):** 
The preferred solution is to source Work Order data from the Data Lake, populated from JDE E1.

Integration path:
JDE E1 → Data Lake → HDV/HDE Booking Tool
Expected synchronization frequency: approximately every 30 minutes
Status: Under clarification whether a consistent 30‑minute refresh can be guaranteed *(see OQ-NEW-P)*

If the required 30‑minute (or faster) update frequency cannot be supported via the Data Lake, a direct JDE E1 API integration provided by the JDE team will be considered as an alternative.

**JDE WO flow (Updated 13.04):** Steps: status ~50 (scheduler review) → status **55** (released to operations). Steps at status 55 → auto-import as SWR Drafts.

**Required Work Order fields from DataLake (Updated 16.04):**

| Field | Description / Notes |
|-------|---------------------|
| WO Number | Unique identifier in JDE E1 |
| WO Name | Examples needed from TCO |
| WO Status | Exact list of statuses TBD |
| WO Status Description | Text description *(Added 16.04)* |
| WO Priority | Priority = WO-level attribute *(OQ-NEW-I — CLOSED 17.04)* |
| Work Center (Step code) | Defines required equipment type |
| Step Name | Text name of the step |
| Step Volumes | Required quantity per step *(Added 16.04)* |
| Date Range | Start / End date of the WO step *(Added 16.04)* |

**Repair/Maintenance data fields from JDE (via DataLake) *(Added 17.04):***

| Field | Description |
|-------|-------------|
| Commitment date | Date when the maintenance WO was committed / accepted |
| Planned date | Expected completion date of maintenance work |
| Completed date | Actual completion date (set when maintenance WO is closed) |

> These fields are used to identify **repair days** in the utilization dashboard (FR-NEW-55). Repair days are excluded from average utilization calculation. For equipment not maintained via JDE.

**Status:** Pending — JDE trigger status TBD; DataLake path confirmed.

**Repair status (08.04):** Maintenance in progress / planned completion date displayed in Requestor's equipment search from DataLake. — Pending details.

### 8.2 DCT (Digital Contractor Timesheet)

**Status:** Obsolete — external fleet booking removed from scope.

- ~~RWA / Change Order Creation~~ — Obsolete
- ~~Tracker's data receival~~ — Obsolete

### 8.3 PSWS (TCO White Page)

**Status:** Confirmed — White Pages may be relevant for auto-populating user contact data. CC Owner role not applicable.

### 8.4 DOA (Delegation of Authority)

**Status:** Obsolete — CC Owner role not applicable.

### 8.5 DataLake

**Data Source and Utilization (22.04)**
Utilization data is obtained from **Cosmos DB via API**, sourced from **IVMS** and **WIALON** trackers. The equipment **Tracker ID** is used to correlate and merge utilization and location records within the Data Lake.

**Status:** Confirmed.

**GIS / Map integration (07.04, 17.04, 20.04, 22.04):** Integration with the ArcGIS platform using an **iframe and URL parameters**. The solution supports equipment ID synchronization and **Azure SSO pass‑through authentication**. Selected equipment IDs from portal checkboxes are passed directly to the GIS iframe.

Map visualization including **speed, heading, historical routes, and sensor data** is sourced from the **Cosmos DB via API**.

**Responsibilities (22.04)**

- **GIS team**: Owns all map rendering, behavior, and visualization logic
- **Booking Tool team**: Provides business requirements and equipment identifiers

**Expected tracker data fields (IVMS, confirmed 20.04):** `speed`, `latitude`, `longitude`, `heading`, `engine hours`, `odometer (kilometres)`, `ignition status`, `battery charge`.

---

## 9 Resolved Conflicts

| # | Conflict | Original BRD Position | Resolution |
|---|----------|-----------------------|------------|
| CONFLICT-01 | FO editing booking period after confirmation | FR-048: editing before confirmation only | Resolved (10.04): FO can edit before AND after confirmation. |
| CONFLICT-02 | Booking status auto-transition by datetime | FR-070/071/075: automatic transitions | Resolved (14.04): Manual close accepted. FR-070, FR-071, FR-075 → Obsolete. |
| CONFLICT-03 | External fleet booking workflow | FR-051..057: CC Owner → External FO | Resolved (08.04): On-demand BP = showcase only. All external booking FRs → Obsolete. |

---

## 10 Open Questions

| OQ ID | Description | Owner | Status |
|-------|-------------|-------|--------|
| OQ-29 | Dynamic filter matrix: equipment type → filter set | TCO | Pending — partially defined (see 5.1.1) |
| OQ-36 | Aggregated Request status state machine (exact mapping) | Team | Open |
| OQ-37 | Booking Snapshot — exact field list | Team | Open |
| OQ-41 | UI details for Unwheeled flow / manual close fields | Team | Open |
| OQ-NEW-A | Concrete field list for equipment cards per equipment type | TCO Fleet Owners (Asylbek) | Partially closed (13.04); remaining types — action Asylbek |
| OQ-NEW-B | Request / Equipment Template scope — baseline first; custom attrs → Backlog | TCO | Open |
| OQ-NEW-C | Service Work Processor role definition — graphist/scheduler? JDE trigger status? | TCO | Open |
| OQ-NEW-E | FO delegation — pending TCO Cybersecurity sign-off | TCO Cybersecurity | Open |
| OQ-NEW-F | Equipment type parameters not yet covered (Vacuum Truck and others) | TCO (Asylbek) | Open |
| OQ-NEW-H | Utilization booking threshold — can booking be submitted for equipment near 100% utilization? | TCO | Closed, FO decides |
| OQ-NEW-L | Final gradation of equipment categories and subcategories | Asylbek | Open (action from 20.04) |
| OQ-NEW-M | Final list of Work Centers | Asylbek | Open (action from 20.04) |
| OQ-NEW-N | Equipment type fields and data examples for all categories | TCO (Asylbek / colleagues) | Open (action from 20.04) |
| OQ-NEW-O | GIS map requirements: full requirements alignment with GIS team (MAPH/Atlas) | Duman / Team | Open — call to be organized by 23.04 |
| OQ-NEW-P | What is the maximum acceptable delay between a Work Order step reaching status 55 in JDE E1 and the corresponding Service Work Request appearing in the HDV/HDE Booking Tool? What update frequency do business users require to work effectively? | TCO | Open |
| OQ-NEW-Q | What is the maximum acceptable delay between tracker telemetry events (IVMS / WIALON) and utilization data appearing in the HDV/HDE dashboard? What refresh frequency do Fleet Owners require to make operational decisions? | TCO | Open |

---

## 11 New Requirements Summary

Requirements not present in the original BRD, identified during requirements gathering sessions (April 2026):

| ID | Summary | Session | Status |
|----|---------|---------|--------|
| FR-NEW-01 | All time parameters configurable via Admin Panel | 03.04 | Confirmed |
| FR-NEW-02 | FO delegation to any TCO employee with auto-revoke | 08.04 | Pending cybersec |
| FR-NEW-03 | Admin configures FO timeout thresholds | 03.04 | Confirmed |
| FR-NEW-04 | Admin authorizes users to book Assigned equipment | 08.04 | Confirmed |
| FR-NEW-05 | FO uploads multiple photos; Requestor sees them | 08.04, 14.04 | Confirmed |
| FR-NEW-06 | Repair status from JDE/DataLake in search and list | 08.04, 14.04 | Pending |
| FR-NEW-07 | Stationary HDE excluded from Requestor search | 08.04 | Confirmed |
| FR-NEW-08 | Assigned equipment: visible to all, bookable by authorized only | 08.04 | Confirmed |
| FR-NEW-09 | Shared with Conditions has visual color marker | 03.04 | Confirmed |
| FR-NEW-10 | On-demand only; no weekly schedules | 03.04 | Confirmed |
| FR-NEW-11 | Priority P1–P4 selected by Requestor; WO-level attribute | 03.04, 17.04 | Confirmed |
| FR-NEW-12 | WO mandatory for Maint/Railroad/Ops; optional for Logistics; Location for SCM | 08.04, 16.04 | Confirmed |
| FR-NEW-13 | "Work Description" mandatory (~50 chars) | 08.04 | Confirmed |
| FR-NEW-14 | "Comments" optional | 08.04 | Confirmed |
| FR-NEW-15 | Single form → auto-split into atomic bookings | 09.04 | Confirmed |
| FR-NEW-16 | Partial confirmation per RequestItem | 03.04 | Confirmed |
| FR-NEW-17 | Aggregated Request status auto-calculated | 09.04 | Pending |
| FR-NEW-18 | Booking start = mobilization start | 03.04 | Confirmed |
| FR-NEW-19 | No mandatory buffer between bookings | 09.04 | Confirmed |
| FR-NEW-20 | Unwheeled equipment warning in UI | 07.04 | Preliminary |
| FR-NEW-21 | FO timeout: 24h reminder / 48h escalation | 03.04, 06.04 | Confirmed |
| FR-NEW-22 | FO must state reason for decline if equipment available | 06.04 | Confirmed |
| FR-NEW-23 | Reminder only to FO of specific equipment | 07.04 | Confirmed |
| FR-NEW-24 | FO "Mobilization started" button | 09.04 | Confirmed |
| FR-NEW-25 | BP showcase: view-only catalog | 08.04 | Confirmed |
| FR-NEW-26 | BP prices not displayed | 08.04 | Confirmed |
| FR-NEW-27 | BP self-populates catalog cards | 08.04 | Confirmed |
| FR-NEW-28 | "Go to BP catalog" button after FO timeout | 08.04 | Confirmed |
| FR-NEW-29 | Three-step Unwheeled flow with Transportation role | 09.04 | Confirmed |
| FR-NEW-30 | FO proposes nearest date if transport unavailable | 03.04 | Pending |
| FR-NEW-31 | Early close button for Requestor/FO | 07.04 | Pending |
| FR-NEW-32 | Manual close only | 09.04 | Confirmed |
| FR-NEW-33 | Actual start/end time at close for utilization | 09.04 | Confirmed |
| FR-NEW-34 | FO reminder 24h / escalation 48h | 03.04, 06.04 | Confirmed |
| FR-NEW-35 | Notification to Transportation Responsible | 09.04 | Confirmed |
| FR-NEW-36 | FO notified of Transportation Responsible decision | 09.04 | Confirmed |
| FR-NEW-37 | Audit trail on demand | 07.04 | Confirmed |
| FR-NEW-38 | Search: TCO number + model mandatory; госномер conditional | 08.04, 13.04 | Confirmed |
| FR-NEW-39 | Dynamic search filters by equipment type | 08.04 | Pending / Partial |
| FR-NEW-40 | Booking calendar in equipment card (FO view) | 13.04 | Confirmed |
| FR-NEW-41 | Multiple trackers per unit; add from UI | 13.04, 14.04 | Confirmed |
| FR-NEW-42 | Shared team email per fleet (Admin-managed) | 13.04 | Confirmed |
| FR-NEW-43 | Notifications to fleet shared team email | 13.04 | Confirmed |
| FR-NEW-44 | Request → Completed when last active booking closed | 14.04 | Confirmed |
| FR-NEW-45 | "Completed Requests" page | 14.04 | Confirmed |
| FR-NEW-46 | Utilization reports by day / department / equipment | 14.04 | Pending |
| FR-NEW-47 | Work center reports | 14.04, 16.04 | Pending |
| FR-NEW-48 | "Выбрать WO из JDE" block; auto-generate draft | 16.04 | Confirmed |
| FR-NEW-49 | Freeze fields in equipment card | 16.04 | Confirmed |
| FR-NEW-50 | Admin manages all reference/handbook values | 16.04 | Confirmed |
| FR-NEW-51 | Default WO / Default WC for non-JDE users (Logistics and others) | 17.04 | Confirmed |
| FR-NEW-52 | "История заявок" page for Requestor (completed requests history) | 17.04 | Confirmed |
| FR-NEW-53 | Utilization dashboard — 3 tabs: Моточасы / Километраж / Комбайн | 17.04 | Confirmed |
| FR-NEW-54 | Utilization cells — color gradient: 0% red → 50% yellow → 100% green | 17.04 | Confirmed |
| FR-NEW-55 | Repair days — wrench icon; JDE fields (commitment/planned/completed date); excluded from average | 17.04 | Confirmed (manual) |
| FR-NEW-56 | Utilization dashboard — Target value column (FO-defined max per equipment) | 17.04 | Confirmed |
| FR-NEW-57 | Utilization dashboard — Average column; sortable by average | 17.04 | Confirmed |
| FR-NEW-58 | Utilization dashboard — Day/Month view toggle | 17.04 | Confirmed |
| FR-NEW-59 | Utilization dashboard — Ownership filter (TCO Owned / Long-term rented) | 17.04 | Confirmed |
| FR-NEW-60 | Utilization from trackers only; independent of bookings | 17.04 | Confirmed |
| FR-NEW-61 | GIS map — iframe; checkboxes in equipment list; ТШО search; expand button | 17.04 | Confirmed (GIS team) |
| FR-NEW-62 | GIS map tooltip — all available sensor data from last sync (updated from: WO + utilization only) | 17.04 / Updated 20.04 | Pending alignment with GIS team |
| FR-NEW-63 | Utilization dashboard — booking count for selected period next to average utilization | 20.04 | Confirmed |
| FR-NEW-64 | FO approval window — equipment loading summary by dates (list of nearby active bookings) | 20.04 | Confirmed |
| FR-NEW-65 | GIS map — speed and heading for GSM-tracked equipment; heading not shown for LoRaWAN | 20.04 | Confirmed |
| FR-NEW-66 | GIS map — fuel level not displayed | 20.04 | Confirmed |
| FR-NEW-67 | GIS map — historical route for 7 days | 20.04 | Confirmed (GIS team) |
