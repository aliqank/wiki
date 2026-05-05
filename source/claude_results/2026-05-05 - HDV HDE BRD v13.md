# HDV/HDE Booking Tool

# Business Requirement Document — v13

**Version:** 13.0  
**Date:** 2026-05-05  
**Based on:** BRD v12 (2026-04-28) + Requirements Update 05.05.2026  
**Prepared by:** Telman Nurzhanov (SA)

---

## Changelog v12 → v13

| # | Section | Change |
|---|---------|--------|
| 1 | 3. Glossary | "Long-term rented" updated: booking process is **no longer identical to TCO Owned**; now requires Requestor Justification per booking item + FleetOwners' Supervisor final approval after FO |
| 2 | 3. Glossary | "Approver" updated: FleetOwners' Supervisor added as Approver for Long-term rented bookings |
| 3 | 3. Glossary | New entry: **FleetOwners' Supervisor** — AAD-managed role; final approver for Long-term rented bookings after FO confirmation |
| 4 | 3. Glossary | New entry: **Justification (Long-term rented)** — mandatory field per booking item explaining why Requestor is not booking TCO Owned equipment |
| 5 | Status Reference | New status **Confirmed by FO** added — intermediate status for Long-term rented bookings pending FleetOwners' Supervisor approval |
| 6 | 4. Stakeholders & Roles | New section **4.7 FleetOwners' Supervisor** |
| 7 | 5.2.1 | TCO Owned and Long-term rented approval chains documented separately (no longer identical) |
| 8 | 6.1, FR-001 | FleetOwners' Supervisor added to list of RBAC roles |
| 9 | 6.2, FR-NEW-03 | Supervisor response timeout added to configurable Admin Panel parameters |
| 10 | 6.2 | FR-NEW-74 added: Admin configures FleetOwners' Supervisor timeout via Admin Panel |
| 11 | 6.4 | FR-NEW-71 added: mandatory Justification field per Long-term rented booking item |
| 12 | 6.5, FR-042 | Booking lifecycle updated: "Confirmed by FO" added to status list |
| 13 | 6.6 | FR-NEW-72, FR-NEW-73 added: FleetOwners' Supervisor approval step for Long-term rented |
| 14 | 6.9 | FR-NEW-77, FR-NEW-78 added: booking status transitions for Confirmed by FO and Supervisor decision |
| 15 | 6.10 | FR-NEW-75, FR-NEW-76 added: Supervisor notification flow |
| 16 | 10. Open Questions | OQ-NEW-S added: FleetOwners' Supervisor — open items (FO sees Justification in approval window confirmed; mixed-request behavior confirmed) |
| 17 | 11. New Requirements Summary | FR-NEW-71–78 added |
| 18 | Summary of Changes | Long-term rented row updated to reflect new approval chain |

---

## Changelog v11 → v12

| # | Section | Change |
|---|---------|--------|
| 1 | 3. Glossary, throughout | Term "Utilization / Утилизация" renamed to "Usage Rate / Процент использования" throughout the document; Glossary entry explicitly notes the previous term name |
| 2 | 6.4, 11. New Requirements Summary | FR-NEW-69 added: Priority field (P1–P4) displays tooltips describing each priority level; tooltip content (descriptions of P1–P4) TBD — to be provided by TCO; configured via Admin Panel |
| 3 | 6.5, 11. New Requirements Summary | FR-NEW-70 added: Equipment load summary displayed to Requestor during equipment selection for booking — list of active bookings for the selected date range, allowing Requestor to assess scheduling conflicts before submitting |

---

## Changelog v10 → v11

| # | Section | Change |
|---|---------|--------|
| 1 | 8.1, 10. Open Questions | OQ-NEW-P CLOSED: direct JDE API integration confirmed as final solution (23.04); DataLake fallback removed; target ~5-minute sync interval |
| 2 | 3. Glossary, 6.12, 8.5, 10. Open Questions | OQ-NEW-Q CLOSED: usage rate refresh once/day confirmed by all teams including Maintenance (23.04); GIS map tooltip usage rate — same frequency (once/day) |
| 3 | 5.2.1, 10. Open Questions | OQ-NEW-R CLOSED: no additional CC approval required when booking long-term rented equipment assigned to another department (cross-departmental booking); FO manages at own discretion (23.04) |
| 4 | 3. Glossary | Editorial fix: "Approver" definition updated — CC Owner removed (Obsolete since 10.04) |
| 5 | 3. Glossary, 6.12 | Editorial fix: "Combined / «Комбайн»" — bilingual label added for consistency across Glossary and FR-NEW-53 |
| 6 | 5.1 | Editorial fix: Unified database table column renamed from "BP (Long-term rented)" to "Long-term rented" |
| 7 | 1.2, 3. Glossary, 4.1, 5.1, 5.2.2, 6.7, 6.12 | Editorial fix: On-demand BP equipment terminology standardized to "On-demand BP (Showcase)" throughout document |
| 8 | 6.12 | Editorial fix: FR-NEW-64 removed from section 6.12 (duplicate; requirement belongs to section 6.4 / 6.6 only) |
| 9 | 8.1 | Editorial fix: Duplicate "Planned date" row removed from Repair/Maintenance data fields table |

---

## Summary of Changes vs. Original BRD

The table below reflects the most significant scope and design decisions made during requirements gathering sessions (April–May 2026) relative to the original BRD delivered by the client.

| Area | Original BRD | Current Version (v13) |
|------|-------------|----------------------|
| Booking scope | Full booking workflow for both internal (TCO) and external (BP) equipment | Internal and long-term rented equipment only. On-demand BP (Showcase) equipment is displayed as a catalog/showcase — no booking workflow for On-demand BP in Phase 1 |
| External approval chain | CC Owner → External Fleet Owner required for BP equipment bookings | CC Owner role is obsolete. All external booking FRs (FR-051–057) removed from scope |
| DCT integration | Required: RWA/Change Order creation; tracker data forwarding | Obsolete — external fleet booking removed from scope |
| PSWS integration | Required: CC Owner identification; user email from White Pages | CC Owner role removed; PSWS used for contact data only |
| DOA integration | Required: DOA processing of CC Owner bookings | Obsolete — external fleet booking removed from scope |
| Booking status transitions | Automatic by system datetime (FR-070, FR-071, FR-075) | Manual close only — status changes are triggered by user action, not datetime |
| FO date editing | Before confirmation only (FR-048 original) | Before and after confirmation; Requestor notified automatically; no re-confirmation required |
| Equipment classification | Internal fleet / External fleet | Three dimensions: Mobility (self-propelled / unwheeled / stationary) × Ownership (TCO Owned / Long-term rented / BP Showcase) × Usage Status (Assigned / Shared without Conditions / Shared with Conditions) |
| Equipment card — BP fields | Not described | Unified database for TCO and BP equipment; subset of fields optional for BP |
| Usage Rate | Basic monitoring via trackers (FR-099) | Full Usage Rate Dashboard: 3 tabs, color gradient, repair-day markers, target values, average, Work Center filter, ownership filter |
| GIS map | Basic coordinates display (FR-100) | Full GIS iframe integration (MAPH/Atlas team); speed/direction; historical route; tooltip with sensor data and current-day usage rate |
| New roles | — | Transportation Responsible (for unwheeled equipment transport); **FleetOwners' Supervisor** (final approver for long-term rented bookings) |
| New concepts | — | Mobilization period; Default Work Order / Work Center; equipment freeze mechanism; Booking snapshot; dynamic characteristics per equipment type; **Justification (Long-term rented)** |
| Fleet structure | One Fleet Owner per fleet | Fleet can have 2+ Fleet Owners with shared team email |
| Equipment templates | Static admin-managed templates | Dynamic characteristics: Admin configures per-type attributes via Admin Panel tool; shown automatically based on selected equipment type |
| Assigned equipment | Bookable only by pre-authorized users | Bookable by Admin-authorized users with mandatory justification; FO can decline |
| FO delegation | In-system delegation with date range | Delegation via AAD groups (outside system); no in-system feature |
| JDE integration | Not specified | Direct JDE E1 API (confirmed 23.04); target ~5-minute sync interval |
| **Long-term rented booking chain** | **Not specified** | **Requestor (+ Justification per booking item) → FO → FleetOwners' Supervisor → Confirmed. Differs from TCO Owned (FO only).** |

---

## Status Reference

| Status | Description |
|--------|-------------|
| Confirmed | Requirement confirmed in meetings; no conflicts |
| Pending | Requirement accepted; details TBD or pending sign-off |
| Obsolete | Cancelled or replaced by another requirement |
| Backlog | Confirmed direction; deferred after baseline implementation |
| Conflict | Contradiction between original BRD and meeting decisions; requires explicit sign-off |

**Booking statuses:**

| Status | Description |
|--------|-------------|
| Draft | Booking item created but request not yet submitted |
| Submitted | Booking item submitted to FO for approval |
| Confirmed by FO | FO confirmed the booking; **for Long-term rented only** — pending FleetOwners' Supervisor final approval *(Added v13)* |
| Confirmed | Booking fully confirmed. For TCO Owned: confirmed by FO. For Long-term rented: confirmed by FleetOwners' Supervisor |
| Declined | Declined by FO or (for Long-term rented) by FleetOwners' Supervisor |
| Revoked | Revoked by Requestor before FO action |
| Terminated | Terminated after confirmation by FO or Requestor |
| Completed | Manually closed via "Close" button |

---

## 1 Introduction

### 1.1 Purpose

This document defines the business requirements for a booking tool that allows users to create requests for HDV/HDE equipment. Requests may include multiple pieces of equipment, each of which is treated as a separate booking with its own approval workflow.

### 1.2 Scope

The tool will support bookings for company-owned equipment as well as long-term rented equipment. It will cover request creation, approval flow, status tracking, reporting, and integration with external systems where applicable.

> **Scope decision (08.04):** Booking through the tool is available only for TCO Owned and Long-term rented equipment. On-demand BP (Showcase) is displayed as a catalog/showcase only — no booking workflow for On-demand BP in the system. This is a significant scope reduction vs. the original BRD. See Section 6.7 for detail.

---

## 2 Business Context

### 2.1 Current Challenges

- Lack of centralized booking and collaboration between departments on shared resources
- Low transparency on total number of units and spend
- No standardized process or guidelines
- No Process Owner(s)
- Lack of visibility due to manual usage rate monitoring

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

**Service Work Request** – a request created in HDV/HDE automatically via API based on a Work Order created in JDE E1, once the Work Order step reaches a certain status. Visible only to users with the Service Work Processor (SWP) role. *(OQ-NEW-C — PARTIALLY CLOSED 22.04)*

**Booking** – individual booking within a request. One booking is linked to one unique equipment item. Treated individually within a request.

**Equipment** – individual asset (HDV/HDE unit) that can be booked through the system. Each equipment item belongs to a fleet and inherits approval responsibility from the corresponding Fleet Owner.

**Fleet** – logical group of equipment owned by a Fleet Owner (for internal: Maintenance, Construction, TFM, etc.; for external: business partner's name). Each fleet has a unique name assigned at creation.

**Fleet Owner (FO)** – person responsible for approvals within a fleet. One fleet can have 2+ Fleet Owners (e.g. shift workers). A fleet has a shared team email address used for system notifications. Delegation is managed via AAD group membership (outside the system).

**FleetOwners' Supervisor** — *(Added v13)* — person responsible for final approval of Long-term rented equipment bookings. Role is managed via AAD group. Receives notification once FO confirms a Long-term rented booking (status: Confirmed by FO). Can confirm or decline the booking; decline moves booking immediately to Declined with mandatory comment. Has no approval authority over TCO Owned equipment bookings.

**Approver** – Fleet Owner, who confirms/declines bookings within the internal fleet; FleetOwners' Supervisor, who provides final approval for Long-term rented bookings after FO confirmation. *(Updated v13; CC Owner role — Obsolete 10.04)*

**Justification (Long-term rented)** — *(Added v13)* — mandatory free-text field filled by Requestor at the individual booking item level when the booked equipment is Long-term rented. Must explain why the Requestor is not booking TCO Owned equipment instead. Displayed to FO in the FO approval window. Reviewed by FleetOwners' Supervisor as part of the final approval step.

**Mobilization** *(Мобилизация)* – the period of preparation/movement of equipment before it arrives at the work site. Booking period starts from mobilization start, not from arrival. Added during requirements gathering.

**Equipment categories — Usage Status:**
- **Assigned** – equipment assigned to a specific department; visible to all users. Bookable only by **Admin-authorized users** with a mandatory **Justification** field; Fleet Owner can decline. Not available in the general shared pool. *(Закреплённая техника. Clarified 22.04: Admin pre-authorization required; mandatory justification added as additional requirement)*
- **Shared without Conditions** – equipment available for general booking without any preconditions. *(Общедоступная без условий)*
- **Shared with Conditions** – equipment available for booking, but the Requestor must provide a **Justification** field. Used for specialized equipment where FO needs context (e.g. specific compressors, chemical tanks). Fleet Owner can recall any booking at any time regardless of this status — recall is a separate mechanism. *(Общедоступная с условиями; обоснование обязательно. Решение закрыто 13.04)*

**Equipment Ownership Types:**
- **TCO Owned** – internally owned by TCO. Approval chain: FO only.
- **Long-term rented** – rented long-term; managed by internal TCO Fleet Owner (not by BP). Booking requires: (1) Requestor Justification per booking item, (2) FO confirmation, (3) FleetOwners' Supervisor final approval. *(Updated v13 — booking process is no longer identical to TCO Owned)*
- **On-demand BP (Showcase)** – rented on demand; displayed as catalog/showcase only in Phase 1.

**Dynamic Characteristics** *(Динамические характеристики)* – custom equipment attributes configured per equipment type via Admin Panel tool. These characteristics are shown automatically in the equipment card, request creation form, and search filters based on the selected equipment type. Replaces the static template approach. *(Added 22.04)*

**Usage Rate** *(Процент использования)* — *(Previously referred to as "Utilization / Утилизация" in BRD versions prior to v12.)* — daily efficiency of equipment usage; reflects how effectively the company manages its equipment.
- Tracked by **mileage** (wheeled equipment) or **engine hours** (other equipment types), or **both** depending on equipment category.
- Each Fleet Owner independently defines the **100% usage rate threshold** for each equipment type (maximum moto-hours or km per 24 hours = 100%).
- **Maximum Usage Rate Capacity per Shift** — maximum possible usage rate within a working shift; serves as the 100% reference point.
- The usage rate percentage is calculated per day, month, or any selected period.
- Some equipment types have **both** mileage and engine hours parameters. Planned values are entered by FO; actual values are sourced from trackers only (not manually editable). *(Updated 13.04)*
- Usage rate data is sourced from **trackers only**, independent of whether a booking exists in the system. This allows detecting equipment usage not linked to any system request. *(Updated 17.04)*
- Usage rate dashboard data is **refreshed once at the end of the working day**; dashboard shows data for the **previous day**. Confirmed for all teams including Logistics and Maintenance. *(OQ-NEW-Q — CLOSED 23.04)*

**Usage Rate Calculation Formulas** *(Updated 17.04):*

| Metric type | Formula |
|-------------|---------|
| Engine hours only | `usage_rate = engine_time / target_mh_per_day` |
| Mileage only | `usage_rate = mileage / target_km_per_day` |
| Both (hours + km) | `usage_rate = max(engine_time / target_mh_per_day, mileage / target_km_per_day)` |
| Cap rule | If `usage_rate > 1` → display as 1 (100%) |

> **Business goal (14.04):** Eliminate equipment downtime. Primary metric — how effectively equipment is used.

**Usage Rate Dashboard tabs** *(Added 17.04):*
- **Engine hours** — engine hours usage rate only. Applicable to HDE equipment.
- **Mileage** — mileage usage rate only. Applicable to HDV equipment.
- **Combined** *(«Комбайн»)* — maximum of the two metrics. For equipment with both engine hours and mileage.

**Usage Rate average calculation** *(Added 17.04):*
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

All users in TCO network will by default have Requestor role to book equipment of the internal fleet (TCO Owned + Long-term rented HDV/HDE). On-demand BP (Showcase) equipment is displayed as a showcase/catalog only — booking of On-demand BP equipment is handled in DCT, not in this tool.

Requestors are allowed to:

1. Create a Regular Request — Confirmed
2. Submit the Regular Request — Confirmed
3. Prolong booking — Confirmed
4. Submit feedback on the booked equipment — Confirmed
5. Monitor usage rate of the booked equipment for the booking period, if equipment has tracker(s) — Confirmed
6. ~~Terminate booking (only external fleet)~~ — Obsolete (external fleet booking removed from scope)
7. Close booking by pressing "Close" button (manual close only) — Confirmed *(Added)*
8. Book Assigned equipment if authorized by Admin; mandatory justification required; Fleet Owner approves/declines — Confirmed
9. View own request history (completed requests) — Confirmed *(Added 17.04)*

### 4.2 Service Work Processor

> **Clarified (22.04):** The Service Work Processor role is a **dedicated system role** that grants access to JDE-sourced requests (Service Work Requests). Users with the SWP role can view, manage, and submit Service Work Requests created automatically from JDE Work Orders. Users **without** the SWP role do not see JDE-sourced requests — they work with Regular Requests using the Default Work Order option. Formal confirmation from TCO on JDE trigger status still required *(OQ-NEW-C — PARTIALLY CLOSED)*.

Service Work Processors are allowed to:

1. View Service Work Requests (JDE-sourced, auto-created as Draft) — Confirmed
2. Submit Service Work Requests to Fleet Owner for processing — Confirmed
3. View SWRs grouped by Work Orders — Pending (OQ-NEW-C)

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
9. Monitor usage rate of own equipment via Usage Rate Dashboard — Confirmed
10. Press "Mobilization started" button to fix mobilization start time — Confirmed *(Added)*
11. Delegate FO privileges via AAD group membership (managed by AAD Admin outside the system; no in-system delegation feature) — Confirmed/Closed *(Updated 22.04 — OQ-NEW-E CLOSED)*
12. One user can be an owner of several fleets — Confirmed *(Added)*
13. One fleet can have multiple Fleet Owners; both displayed with shared team email — Confirmed *(Updated 13.04)*

> **Note (v13):** For Long-term rented equipment bookings, FO confirmation moves the booking to "Confirmed by FO" status. Final confirmation requires FleetOwners' Supervisor approval. FO decision alone does not constitute full confirmation for Long-term rented.

### 4.4 CC Owner (incl. DOA)

> **Obsolete (10.04):** CC Owner confirmation is NOT required for Long-term rented equipment. CC Owner is applicable only to on-demand external fleet booking, which is handled in DCT system.

CC owners are allowed to:

1. ~~Confirm / decline external fleet equipment booking request~~ — Obsolete

### 4.5 Administrator

Administrator role will be provided to process owners – Logistics. Access provision is managed by Azure Active Directory (group). Users with an Administrator role are allowed to:

1. Create/edit internal fleet and manage AAD groups — Confirmed
2. Create/edit external fleet and manage external fleet owners — Confirmed
3. Delete fleet — Confirmed
4. Create internal fleet equipment — Confirmed
5. Update equipment's parameters (extended edit) — Confirmed
6. Delete equipment (soft delete) — Confirmed
7. Manage dynamic characteristics per equipment type via Admin Panel tool — Confirmed *(Updated 22.04, replaces static template approach)*
8. Manage 'Request' template — Obsolete (replaced by dynamic characteristics per equipment type) *(Updated 22.04)*
9. Manage 'Booking' template — Pending
10. Access to custom reportings — Confirmed
11. Access to usage rate dashboard(s), if equipment has tracker(s) — Confirmed
12. Manage all configurable system parameters via Admin Panel — Confirmed *(Added)*
13. Authorize specific users to book Assigned equipment — Confirmed *(Updated 22.04 — remains valid; mandatory justification added as additional requirement)*
14. ~~Manage delegation settings~~ — Obsolete *(Updated 22.04 — delegation via AAD groups outside system)*
15. Manage fleet shared team email — Confirmed *(Updated 13.04)*
16. Manage reference/handbook values (equipment classes, categories, subcategories, manufacturers, models, companies, locations, work centers, divisions, groups, departments, units) via Admin Panel — Confirmed *(Added 16.04)*
17. Configure FleetOwners' Supervisor response timeout via Admin Panel — Confirmed *(Added v13)*

### 4.6 Transportation Responsible *(Мобилизация/Транспортировка)*

A dedicated system role for the team responsible for transporting non-motorized (Unwheeled) equipment (Heavy Ops / Maintenance team). Added during requirements gathering.

This role is allowed to:

1. Receive notification about transportation request for Unwheeled equipment — Confirmed
2. Confirm or decline the transportation request — Confirmed

### 4.7 FleetOwners' Supervisor *(Added v13)*

A dedicated system role for the manager responsible for final approval of Long-term rented equipment bookings. Role is managed via AAD group (same provisioning approach as Fleet Owner role). Has no approval authority over TCO Owned equipment bookings.

FleetOwners' Supervisor is allowed to:

1. Receive notification when a Long-term rented booking reaches "Confirmed by FO" status — Confirmed
2. View booking details including the Requestor's Justification (why not TCO Owned) — Confirmed
3. Confirm the booking — final approval; booking moves to "Confirmed" — Confirmed
4. Decline the booking with mandatory comment — booking moves immediately to "Declined"; Requestor and FO are notified — Confirmed

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

**Dynamic characteristics per equipment type (22.04):** Admin configures custom attributes (dynamic characteristics) per equipment type via Admin Panel tool. These attributes are displayed automatically in the equipment card, request creation form, and search filters based on the selected equipment type. Base data — from the provided equipment database (DB from TCO). This approach replaces the static template concept.

**Unified database (confirmed 20.04):** TCO-owned and long-term rented equipment share a single database. For long-term rented equipment, a subset of fields is optional (null allowed):

| Field | TCO Owned | Long-term rented |
|-------|-----------|-----------------|
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
10. **Dynamic characteristics** — custom attributes per equipment type, configured via Admin Panel tool *(Added 22.04)*

**Equipment card UI (13.04–17.04):**
- **Booking calendar**: visual calendar showing booked periods
- **Criticality flag**: displayed in card
- **Sharing condition**: Assigned / Shared without Conditions / Shared with Conditions — displayed in card *(Figma update pending)*
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
| Ownership | TCO Owned / Long-term rented / On-demand BP (Showcase) |
| Usage Status | Assigned / Shared without Conditions / Shared with Conditions |

> **Note (08.04):** Mobility category NOT displayed in Requestor UI.
> **Note (08.04):** Stationary HDE excluded from Requestor search.
> **Note (13.04):** Shared with Conditions differs only in presence of mandatory Justification field.
> **Note (22.04):** Assigned equipment differs from Shared with Conditions in that it is not available in the general shared pool; bookable only by Admin-authorized users; requires mandatory justification; FO more likely to decline.

**Equipment availability display rules:**
- Equipment under maintenance shown grayed out; label "Under maintenance until [Planned date from JDE]"
- Repair status sourced from JDE → DataLake integration — Pending details

---

### 5.1.1 Equipment Type-Specific Parameters *(Updated 22.04)*

> **Approach confirmed (22.04):** Equipment type-specific parameters (dynamic characteristics) are configured via the Admin Panel tool. Admin can add custom attributes for each equipment type. These attributes appear automatically in the equipment card, request form, and search filters. Base data sourced from TCO equipment database provided by the client. This approach replaces the earlier concept of static, manually curated field lists per type.
>
> **Scope of work:** Development of the Admin Panel tool for managing dynamic characteristics. Filters in the request/search form display only characteristics relevant to the selected equipment type.

---

### 5.2 Booking

Booking is linked to a single equipment item and has a booking range (datetime). Several bookings form a single request.

> **Architecture decision (09.04):** One Request → multiple RequestItems (one per equipment); each independently processed by FO.

#### 5.2.1 Internal Booking

Booking of TCO Owned or Long-term rented equipment. Approval chains differ by ownership type. *(Updated v13 — chains are no longer identical)*

**TCO Owned booking chain:**

> Requestor → FO confirmation → **Confirmed**

- Requires only FO confirmation.
- No Justification required (except for Assigned equipment — see Section 3 Glossary).

**Long-term rented booking chain:** *(Updated v13)*

> Requestor (+ Justification per booking item) → FO confirmation → **Confirmed by FO** → FleetOwners' Supervisor approval → **Confirmed**

- Requestor must fill in a **Justification** field at the individual booking item level — mandatory, explains why TCO Owned equipment is not being booked.
- FO reviews the booking and the Justification; confirms or declines.
- If FO confirms → booking moves to **"Confirmed by FO"** status and is queued for FleetOwners' Supervisor review.
- FleetOwners' Supervisor reviews and makes final decision:
  - **Approve** → booking moves to "Confirmed"
  - **Decline** → booking immediately moves to "Declined"; Requestor and FO are notified with Supervisor's mandatory comment.
- FO can still decline at the FO stage (same rules as for TCO Owned).

> **Mobilization (09.04):** Booking period starts from mobilization start. FO presses "Mobilization started" to fix actual start.

> **Updated (10.04):** FO can adjust booking period before and after confirmation. Requestor notified automatically.

> **Cross-departmental booking (23.04):** When booking long-term rented equipment assigned to another department (with re-invoicing to the requester's Cost Center), no additional Cost Center Owner approval is required. Fleet Owner manages such cross-departmental bookings at own discretion. FleetOwners' Supervisor approval still applies. *(OQ-NEW-R — CLOSED 23.04)*

> **Mixed Request behavior (v13):** A single Request may contain both TCO Owned and Long-term rented booking items. Each item follows its own approval chain independently. The Request remains "In Progress" until all booking items are resolved. TCO Owned items may reach "Confirmed" while Long-term rented items are still at "Confirmed by FO" (pending Supervisor). This is expected behavior.

#### 5.2.2 External Booking

**Scope change vs. original BRD:** On-demand BP (Showcase) equipment is NOT bookable in Phase 1. Displayed as catalog/showcase only.

- On-demand BP (Showcase) — catalog view only.
- Long-term rented — treated as internal fleet with extended approval chain (see 5.2.1).
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

> **Partially clarified (22.04):** Auto-created as Draft when WO step reaches a defined status in JDE. Visible only to users with the **Service Work Processor** role. SWP reviews and submits to FO. Users without the SWP role do not see SWRs and use the Default Work Order option instead. Formal confirmation of JDE trigger status required *(OQ-NEW-C — PARTIALLY CLOSED)*.

**Required WO data fields from JDE E1 (Updated 16.04):**

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
| FR-001 | System supports role-based access control. Roles: Requestor, Service Work Processor, Fleet Owner, FleetOwners' Supervisor, Administrator, Transportation Responsible. *(Updated v13 — FleetOwners' Supervisor added)* | Confirmed |
| ~~FR-002~~ | ~~All TCO network users have Requestor access (with RWA initiator restriction for BP booking)~~ | Obsolete — superseded by updated Requestor definition |
| FR-003 | Administrators, Service Work Processors, FleetOwners' Supervisors defined by AAD groups. Confirmed for role definition; Pending for JDE trigger status only (OQ-NEW-C) *(Updated v13)* | Pending (OQ-NEW-C) |
| ~~FR-004~~ | ~~CC Owner / DOA privileges via PSWS/DOA integration~~ | Obsolete |
| FR-NEW-01 | All time-based parameters configurable via Admin Panel (not hardcoded) | Confirmed |
| ~~FR-NEW-02~~ | ~~FO can delegate privileges to any TCO employee for a date range; auto-revoke upon expiry~~ | Obsolete *(22.04 — delegation via AAD groups outside system; no in-system feature required)* |

### 6.2 Admin Panel

| ID | Requirement | Status |
|----|-------------|--------|
| FR-005 | Admin creates fleets with unique names | Confirmed |
| FR-006 | Admin links internal fleet with AAD group | Confirmed |
| FR-007 | Admin manages external fleet owners | Confirmed |
| FR-008 | Admin updates existing fleets | Confirmed |
| FR-009 | Admin deletes fleets | Confirmed |
| FR-010 | Admin configures dynamic custom characteristics per equipment type via Admin Panel tool; these characteristics appear automatically in equipment card, request form, and search filters based on equipment type *(Updated 22.04 — replaces static template concept)* | Confirmed |
| FR-011 | ~~Admin manages request template~~ | Obsolete *(22.04 — replaced by dynamic characteristics per equipment type)* |
| FR-012 | Admin manages booking template | Pending |
| FR-013 | Admin creates equipment (internal fleet) | Confirmed |
| FR-014 | Admin edits equipment (extended edit) | Confirmed |
| FR-015 | Admin deletes equipment | Confirmed |
| FR-016 | Admin accesses all dashboards and reports | Confirmed |
| FR-NEW-03 | Admin configures via Admin Panel: FO timeout (24h/48h), booking horizon, max duration, **FleetOwners' Supervisor response timeout** *(Updated v13)* | Confirmed |
| FR-NEW-04 | Admin authorizes specific users to book Assigned equipment | Confirmed *(22.04 — remains valid; mandatory justification added as additional requirement)* |
| FR-NEW-42 | Admin sets and updates fleet shared team email | Confirmed |
| FR-NEW-50 | Admin manages all reference/handbook values via Admin Panel | Confirmed |
| FR-NEW-68 | System dynamically shows only type-specific characteristics (dynamic attributes) in request creation form and equipment search filters, based on the equipment type selected by the Requestor. *(Added 22.04)* | Confirmed |
| FR-NEW-74 | Admin configures **FleetOwners' Supervisor response timeout** via Admin Panel (analogous to FO timeout). Default value TBD. Upon timeout — escalation logic TBD. *(Added v13)* | Confirmed (escalation logic Pending) |

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
| FR-NEW-08 | Assigned equipment: visible to all users; bookable only by Admin-authorized users with mandatory justification; FO can approve or decline. | Confirmed |
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
| FR-026 | Request statuses: Draft, Submitted, In Progress, Completed | Confirmed *(OQ-36 — CLOSED 22.04; resolved by dev team)* |
| FR-027 | Requestor can edit/cancel draft before submission | Confirmed |
| FR-028 | Service Work Requests auto-created with 'Draft' status; visible to SWP role only | Pending (OQ-NEW-C — trigger status TBD) |
| FR-029 | Only SWP can manage (submit) Service Work Requests | Confirmed *(Updated 22.04)* |
| FR-030 | System supports draft saving | Confirmed |
| FR-NEW-10 | On-demand booking only; weekly schedule management out of scope | Confirmed |
| FR-NEW-11 | Requestor selects priority P1–P4 at request creation (priority = WO-level attribute) | Confirmed |
| FR-NEW-12 | WO mandatory for Maintenance/Railroad/Operations; optional for Logistics; Location replaces WO for SCM Logistics | Confirmed |
| FR-NEW-13 | "Work Description" mandatory (~50 chars) | Confirmed |
| FR-NEW-14 | "Comments" optional | Confirmed |
| FR-NEW-15 | Single form → auto-split into individual bookings per equipment item | Confirmed |
| FR-NEW-16 | Partial confirmation: confirmed items proceed independently from declined | Confirmed |
| FR-NEW-17 | Aggregated Request status auto-calculated from RequestItem statuses | Confirmed *(OQ-36 — CLOSED 22.04)* |
| FR-NEW-48 | "Выбрать WO из JDE" block on Submit Request page; WO list from JDE E1 via direct API; auto-generate draft | Confirmed |
| FR-NEW-51 | **Default Work Order** option for teams not operating in JDE (Logistics and others): checkbox on request form; when selected — system does not require a JDE WO number. Analogously **Default Work Center** for teams without WC. *(Added 17.04)* | Confirmed |
| FR-NEW-64 | In the FO booking approval window: display an **equipment loading summary by dates** — a list of nearest active bookings for the given equipment unit (analogous to a ticket availability view). Allows FO to assess scheduling conflicts before approving. *(Added 20.04)* | Confirmed |
| FR-NEW-69 | Priority field (P1–P4) in the request creation form must display **tooltips/hints** for each priority level, describing the business meaning of that priority. Tooltip content (descriptions of P1–P4) is **TBD — to be provided by TCO**. Priority descriptions are configured via Admin Panel. *(Added 28.04)* | Confirmed (pending priority descriptions from TCO) |
| FR-NEW-71 | When a Requestor adds a **Long-term rented** equipment item to a request, the system requires a mandatory **Justification** field for that booking item — a free-text explanation of why TCO Owned equipment is not being booked. The Justification is entered at the individual booking item level (not at the request level). It is displayed to FO in the FO approval window and to FleetOwners' Supervisor in the Supervisor approval window. The request cannot be submitted if a Long-term rented booking item has no Justification. *(Added v13)* | Confirmed |

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
| FR-042 | Booking lifecycle: Draft, Submitted, **Confirmed by FO** (Long-term rented only — pending Supervisor), Confirmed, Declined, Revoked, Terminated, Completed *(Updated v13 — "Confirmed by FO" added)* | Confirmed |
| FR-NEW-18 | Booking start = start of mobilization | Confirmed |
| FR-NEW-19 | No mandatory buffer between bookings | Confirmed |
| FR-NEW-20 | Unwheeled equipment: notification displayed to Requestor | Preliminary |
| FR-NEW-70 | When the Requestor selects an equipment item and specifies a booking date range, the system displays an **equipment load summary** for the selected period — a list of existing confirmed/submitted bookings for that equipment unit. Allows the Requestor to assess scheduling conflicts and make an informed selection before submitting. *(Added 28.04)* | Confirmed |

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
| FR-NEW-72 | For **Long-term rented** equipment: after FO confirms a booking, the booking moves to **"Confirmed by FO"** status and is routed to FleetOwners' Supervisor for final approval. FO confirmation alone does not finalize the booking. *(Added v13)* | Confirmed |
| FR-NEW-73 | FleetOwners' Supervisor reviews Long-term rented booking in "Confirmed by FO" status, including Requestor's Justification. Supervisor can: **Confirm** → booking moves to "Confirmed"; **Decline** → booking moves immediately to "Declined"; Supervisor must provide a mandatory comment on decline. Requestor and FO are notified of the Supervisor's decision. *(Added v13)* | Confirmed |

### 6.7 Approval Flow (External Fleet)

> **Major scope change (08.04):** On-demand BP (Showcase) not bookable. All FR-051..057 are Obsolete vs. original BRD.

| ID | Requirement | Status |
|----|-------------|--------|
| ~~FR-051..057~~ | ~~External fleet approval chain (CC Owner, External FO, PSWS, DOA)~~ | Obsolete |
| FR-NEW-25 | On-demand BP (Showcase): Requestor can view on-demand BP equipment; no booking action in Phase 1 | Confirmed |
| FR-NEW-26 | On-demand BP (Showcase) equipment prices/rates NOT displayed | Confirmed |
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
| FR-063 | **TCO Owned** booking → 'Confirmed' once confirmed by FO. **Long-term rented** booking → 'Confirmed by FO' once confirmed by FO (see FR-NEW-77). *(Updated v13)* | Confirmed |
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
| FR-NEW-33 | At close: Requestor inputs actual start/end time for usage rate analytics | Confirmed |
| FR-NEW-44 | Request → 'Completed' when last active (non-Declined) booking closed | Confirmed |
| FR-NEW-77 | **Long-term rented** booking → 'Confirmed by FO' upon FO confirmation (intermediate status). Booking remains in this status until FleetOwners' Supervisor acts. During this period, FO date adjustments remain possible. *(Added v13)* | Confirmed |
| FR-NEW-78 | **Long-term rented** booking → 'Confirmed' upon FleetOwners' Supervisor approval; → 'Declined' upon Supervisor decline. No further states between Confirmed by FO and Supervisor decision. *(Added v13)* | Confirmed |

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
| FR-NEW-75 | System notifies **FleetOwners' Supervisor** when a Long-term rented booking reaches "Confirmed by FO" status — triggering the Supervisor review step. Notification contains: booking details, equipment info, Requestor name, Justification text. *(Added v13)* | Confirmed |
| FR-NEW-76 | System notifies **Requestor and FO** when FleetOwners' Supervisor makes a decision on a Long-term rented booking (Confirmed or Declined). Notification contains Supervisor's comment (mandatory on decline). *(Added v13)* | Confirmed |

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
| FR-NEW-39 | Dynamic search filters by equipment type — shown automatically based on dynamic characteristics configured per type via Admin Panel *(Confirmed 22.04 — OQ-29 CLOSED)* | Confirmed |
| FR-NEW-45 | Dedicated "Completed Requests" page for Requestor, SWP, FO, Admin | Confirmed |
| FR-NEW-46 | Usage Rate reports: by day / department / equipment unit | Pending details |
| FR-NEW-47 | Work center reports (group by WC code) | Pending details |
| FR-NEW-52 | Requestor has access to **«История заявок»** (Request History) — a dedicated page listing all completed requests by this Requestor. Minimum columns: request number, equipment type, booking period, status, Work Order number. *(Added 17.04)* | Confirmed |

### 6.12 Dashboards / Map

| ID | Requirement | Status |
|----|-------------|--------|
| FR-099 | System displays usage rate data and coordinates from DataLake/PI (tracker data) | Confirmed |
| FR-100 | System displays digitized location of Work Orders from DataLake (JDE E1) | Confirmed |
| FR-NEW-53 | Usage Rate Dashboard has **three tabs**: «Моточасы» (engine hours) / «Километраж» (mileage) / «Комбайн» / Combined (max of both). Active tabs depend on equipment type: HDE → Моточасы only; HDV → Километраж only; mixed equipment → all three. *(Added 17.04)* | Confirmed |
| FR-NEW-54 | Usage Rate cells use a **color gradient**: 0% — red → 50% — yellow → 100% — green. Smooth gradient, not discrete steps. *(Added 17.04)* | Confirmed |
| FR-NEW-55 | Repair days are marked in the dashboard with a **wrench icon** (not a color) to distinguish from low-usage-rate days. Repair data sourced from JDE (see Section 8.1): `commitment date` = start of repair, `planned date` = expected end, `completed date` = actual end. Repair days are **excluded from average usage rate calculation**. Manual repair input for non-JDE equipment. *(Added 17.04)* | Confirmed (manual input) |
| FR-NEW-56 | Usage Rate Dashboard includes a **Target value column** per equipment: maximum daily usage rate threshold set by FO (e.g. 2 000 engine hours / 1 500 km per day). Displayed first after equipment name/type columns. *(Added 17.04)* | Confirmed |
| FR-NEW-57 | Usage Rate Dashboard includes an **Average usage rate column** for the selected period (repair days excluded). Table is **sortable by this column**. *(Added 17.04)* | Confirmed |
| FR-NEW-58 | Usage Rate Dashboard supports **two time granularity views**: «По дням» (daily, current default) / «По месяцам» (monthly average per cell). User toggles via a control above the table. *(Added 17.04)* | Confirmed |
| FR-NEW-59 | Usage Rate Dashboard left sidebar includes an **Ownership filter**: TCO Owned / Long-term rented. *(Added 17.04)* | Confirmed |
| FR-NEW-60 | Usage rate data is sourced from **tracker telemetry only**, independent of bookings. Allows detecting equipment usage not linked to any booking. *(Added 17.04)* | Confirmed |
| FR-NEW-61 | In the FO Equipment section, equipment list includes **checkboxes** to select units for display on the GIS map. The GIS map is rendered as an **iframe** by the GIS team (MAPH/Atlas); system passes selected equipment identifiers as URL parameters. Map supports **expand** button for full-screen view. Search in equipment list by ТШО-номер. *(Added 17.04)* | Confirmed (GIS team involvement required) |
| FR-NEW-62 | GIS map marker **tooltip** on click: displays all available sensor data received from the last tracker synchronization, **plus current-day usage rate** (Maintenance team requirement). Data fields: speed, heading, coordinates, engine hours, odometer, ignition, battery, current-day usage rate value. Usage rate in tooltip refreshed **once per day** (end of working day), consistent with dashboard frequency. *(Updated 23.04 — OQ-NEW-Q CLOSED)* | Pending alignment with GIS team *(OQ-NEW-O — Open)* |
| FR-NEW-63 | Usage Rate Dashboard displays **booking count for the selected period** alongside the average usage rate value. Allows analysis of correlation between equipment load (bookings) and actual usage rate. *(Added 20.04)* | Confirmed |
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

**Integration approach (Updated 23.04):**

**Confirmed solution (23.04):** Direct API integration with JDE E1 (bypassing DataLake for WO data). Target synchronization interval: ~5 minutes. This approach has been confirmed as the final integration solution for WO data to meet business requirements for urgent Work Orders. *(OQ-NEW-P — CLOSED 23.04)*

Integration path:
- **Confirmed:** JDE E1 → Direct API → HDV/HDE Booking Tool (target: ~5 min interval)

**JDE WO flow (Updated 13.04):** Steps: status ~50 (scheduler review) → status **55** (released to operations). Steps at status 55 → auto-import as SWR Drafts. SWP reviews and submits.

**Required Work Order fields from JDE (Updated 16.04):**

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

> These fields are used to identify **repair days** in the Usage Rate Dashboard (FR-NEW-55). Repair days are excluded from average usage rate calculation.

**Status:** Confirmed — Direct JDE API integration confirmed (23.04). JDE trigger status (WO step 55) — see OQ-NEW-C (PARTIALLY CLOSED 22.04).

**Repair status (08.04):** Maintenance in progress / planned completion date displayed in Requestor's equipment search from DataLake. — Pending details.

### 8.2 DCT (Digital Contractor Timesheet)

**Status:** Obsolete — external fleet booking removed from scope.

- ~~RWA / Change Order Creation~~ — Obsolete
- ~~Tracker's data receival~~ — Obsolete

### 8.3 PSWS (TCO White Page)

**Status:** Confirmed — White Pages used for auto-populating user contact data. CC Owner role not applicable.

### 8.4 DOA (Delegation of Authority)

**Status:** Obsolete — CC Owner role not applicable.

### 8.5 DataLake

**Data Source and Usage Rate (22.04)**
Usage rate data is obtained from **Cosmos DB via API**, sourced from **IVMS** and **WIALON** trackers. The equipment **Tracker ID** is used to correlate and merge usage rate and location records within the Data Lake.

**Usage rate refresh frequency (23.04):**
- Usage rate data is updated **once at the end of the working day**.
- Dashboard shows **previous day's** data.
- Confirmed for all teams: Logistics and Maintenance. *(OQ-NEW-Q — CLOSED 23.04)*

**Status:** Confirmed.

**GIS / Map integration (07.04–22.04):** Integration with the ArcGIS platform using an **iframe and URL parameters**. The solution supports equipment ID synchronization and **Azure SSO pass-through authentication**. Selected equipment IDs from portal checkboxes are passed directly to the GIS iframe.

Map visualization including **speed, heading, historical routes, sensor data, and current-day usage rate** is sourced from the **Cosmos DB via API**.

**Responsibilities (22.04)**

- **GIS team**: Owns all map rendering, behavior, and visualization logic
- **Booking Tool team**: Provides business requirements and equipment identifiers

**Tracker types and synchronization frequency (22.04):**

| Tracker type | Sync frequency | Notes |
|-------------|----------------|-------|
| Teltonika (GSM) | Real-time (continuous) | Last known position shown when device is off |
| LoRaWAN (GPS-only) | ~3 times/day | Battery-powered; low sync frequency |
| Additional GPS sensors | ~3 times/day | Battery-powered; planned for equipment without telemetry (e.g. Kamaz) |

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
| OQ-29 | Dynamic filter matrix: equipment type → filter set | Team | **CLOSED 22.04** — handled by dynamic characteristics admin tool; FR-NEW-39 Confirmed |
| OQ-36 | Aggregated Request status state machine (exact mapping) | Team | **CLOSED 22.04** — resolved by dev team |
| OQ-37 | Booking Snapshot — exact field list | Team | Open |
| OQ-41 | UI details for Unwheeled flow / manual close fields | Team | Open |
| OQ-NEW-A | Concrete field list for equipment cards per equipment type | TCO Fleet Owners (Asylbek) | **CLOSED 22.04** — covered by dynamic characteristics admin tool; base data from TCO equipment DB |
| OQ-NEW-B | Request / Equipment Template scope | TCO | **CLOSED 22.04** — dynamic characteristics pulled automatically per equipment type; no manual template selection |
| OQ-NEW-C | Service Work Processor role — SWP role defined: users with SWP see JDE requests; users without use Default WO. JDE trigger status (status 55 confirmation) still TBD | TCO | **PARTIALLY CLOSED 22.04** |
| OQ-NEW-E | FO delegation mechanism | TCO Cybersecurity | **CLOSED 22.04** — delegation via AAD groups; no in-system feature required |
| OQ-NEW-F | Equipment type parameters not yet covered (Vacuum Truck and others) | TCO (Asylbek) | **CLOSED 22.04** — covered by dynamic characteristics admin tool |
| OQ-NEW-H | Usage rate booking threshold — can booking be submitted for equipment near 100% usage rate? | TCO | Closed — FO decides |
| OQ-NEW-L | Final gradation of equipment categories and subcategories | Asylbek | Open (action from 20.04) |
| OQ-NEW-M | Final list of Work Centers | Asylbek | Open (action from 20.04) |
| OQ-NEW-N | Equipment type fields and data examples for all categories | TCO (Asylbek / colleagues) | **CLOSED 22.04** — covered by dynamic characteristics admin tool |
| OQ-NEW-O | GIS map requirements: full requirements alignment with GIS team (MAPH/Atlas) | Duman / Team | Open — call scheduled 23.04 |
| OQ-NEW-P | JDE update frequency: direct JDE API confirmed as final solution. Target ~5-minute sync interval. DataLake approach not used for WO data. Decision driven by business requirement for urgent WO processing. | Ayan / JDE team | **CLOSED 23.04** |
| OQ-NEW-Q | Usage Rate dashboard update frequency. All teams confirmed: once at end of working day (previous day data) — satisfies Logistics and Maintenance. GIS map tooltip usage rate: same frequency (once per day). | Team / Maintenance | **CLOSED 23.04** |
| OQ-NEW-R | Cross-departmental booking of long-term rented equipment: no additional CC Owner approval required when one department books equipment assigned to another department (re-invoicing to requester's CC). FO manages at own discretion. FleetOwners' Supervisor approval still applies (separate from CC Owner). | Team | **CLOSED 23.04** *(Note updated v13: Supervisor approval is a separate step, distinct from CC Owner — OQ-NEW-R closure remains valid)* |
| OQ-NEW-S | **FleetOwners' Supervisor — open items (v13).** Confirmed: (1) Supervisor role is AAD-managed; (2) Supervisor decline → immediately Declined, no return to FO; (3) FO sees Justification in approval window; (4) Mixed Request (TCO Owned + Long-term rented) hangs In Progress until all items resolved — expected behavior. **Remaining open:** (a) default and max Supervisor timeout value — TBD with TCO; (b) escalation logic upon Supervisor timeout — TBD; (c) FleetOwners' Supervisor scope: one Supervisor per fleet, or one global Supervisor? | Team / TCO | Open *(Added v13)* |

---

## 11 New Requirements Summary

Requirements not present in the original BRD, identified during requirements gathering sessions (April–May 2026):

| ID | Summary | Session | Status |
|----|---------|---------|--------|
| FR-NEW-01 | All time parameters configurable via Admin Panel | 03.04 | Confirmed |
| ~~FR-NEW-02~~ | ~~FO delegation to any TCO employee with auto-revoke~~ | 08.04 | **Obsolete 22.04** — delegation via AAD groups outside system |
| FR-NEW-03 | Admin configures FO timeout thresholds + Supervisor timeout | 03.04 / **Updated v13** | Confirmed |
| FR-NEW-04 | Admin authorizes specific users to book Assigned equipment | 08.04 | **Confirmed** — remains valid; mandatory justification added |
| FR-NEW-05 | FO uploads multiple photos; Requestor sees them | 08.04, 14.04 | Confirmed |
| FR-NEW-06 | Repair status from JDE/DataLake in search and list | 08.04, 14.04 | Pending |
| FR-NEW-07 | Stationary HDE excluded from Requestor search | 08.04 | Confirmed |
| FR-NEW-08 | Assigned equipment: visible to all, bookable by Admin-authorized users with mandatory justification; FO can decline | 08.04 | Confirmed |
| FR-NEW-09 | Shared with Conditions has visual color marker | 03.04 | Confirmed |
| FR-NEW-10 | On-demand only; no weekly schedules | 03.04 | Confirmed |
| FR-NEW-11 | Priority P1–P4 selected by Requestor; WO-level attribute | 03.04, 17.04 | Confirmed |
| FR-NEW-12 | WO mandatory for Maint/Railroad/Ops; optional for Logistics; Location for SCM | 08.04, 16.04 | Confirmed |
| FR-NEW-13 | "Work Description" mandatory (~50 chars) | 08.04 | Confirmed |
| FR-NEW-14 | "Comments" optional | 08.04 | Confirmed |
| FR-NEW-15 | Single form → auto-split into atomic bookings | 09.04 | Confirmed |
| FR-NEW-16 | Partial confirmation per RequestItem | 03.04 | Confirmed |
| FR-NEW-17 | Aggregated Request status auto-calculated | 09.04 | **Confirmed 22.04** (OQ-36 Closed) |
| FR-NEW-18 | Booking start = mobilization start | 03.04 | Confirmed |
| FR-NEW-19 | No mandatory buffer between bookings | 09.04 | Confirmed |
| FR-NEW-20 | Unwheeled equipment warning in UI | 07.04 | Preliminary |
| FR-NEW-21 | FO timeout: 24h reminder / 48h escalation | 03.04, 06.04 | Confirmed |
| FR-NEW-22 | FO must state reason for decline if equipment available | 06.04 | Confirmed |
| FR-NEW-23 | Reminder only to FO of specific equipment | 07.04 | Confirmed |
| FR-NEW-24 | FO "Mobilization started" button | 09.04 | Confirmed |
| FR-NEW-25 | On-demand BP (Showcase): view-only catalog | 08.04 | Confirmed |
| FR-NEW-26 | On-demand BP (Showcase) prices not displayed | 08.04 | Confirmed |
| FR-NEW-27 | BP self-populates catalog cards | 08.04 | Confirmed |
| FR-NEW-28 | "Go to BP catalog" button after FO timeout | 08.04 | Confirmed |
| FR-NEW-29 | Three-step Unwheeled flow with Transportation role | 09.04 | Confirmed |
| FR-NEW-30 | FO proposes nearest date if transport unavailable | 03.04 | Pending |
| FR-NEW-31 | Early close button for Requestor/FO | 07.04 | Pending |
| FR-NEW-32 | Manual close only | 09.04 | Confirmed |
| FR-NEW-33 | Actual start/end time at close for usage rate analytics | 09.04 | Confirmed |
| FR-NEW-34 | FO reminder 24h / escalation 48h | 03.04, 06.04 | Confirmed |
| FR-NEW-35 | Notification to Transportation Responsible | 09.04 | Confirmed |
| FR-NEW-36 | FO notified of Transportation Responsible decision | 09.04 | Confirmed |
| FR-NEW-37 | Audit trail on demand | 07.04 | Confirmed |
| FR-NEW-38 | Search: TCO number + model mandatory; госномер conditional | 08.04, 13.04 | Confirmed |
| FR-NEW-39 | Dynamic search filters by equipment type (via dynamic characteristics admin tool) | 08.04 / **Confirmed 22.04** | Confirmed |
| FR-NEW-40 | Booking calendar in equipment card (FO view) | 13.04 | Confirmed |
| FR-NEW-41 | Multiple trackers per unit; add from UI | 13.04, 14.04 | Confirmed |
| FR-NEW-42 | Shared team email per fleet (Admin-managed) | 13.04 | Confirmed |
| FR-NEW-43 | Notifications to fleet shared team email | 13.04 | Confirmed |
| FR-NEW-44 | Request → Completed when last active booking closed | 14.04 | Confirmed |
| FR-NEW-45 | "Completed Requests" page | 14.04 | Confirmed |
| FR-NEW-46 | Usage Rate reports by day / department / equipment | 14.04 | Pending |
| FR-NEW-47 | Work center reports | 14.04, 16.04 | Pending |
| FR-NEW-48 | "Выбрать WO из JDE" block; WO via direct JDE API; auto-generate draft | 16.04 | Confirmed |
| FR-NEW-49 | Freeze fields in equipment card | 16.04 | Confirmed |
| FR-NEW-50 | Admin manages all reference/handbook values | 16.04 | Confirmed |
| FR-NEW-51 | Default WO / Default WC for non-JDE users (Logistics and others) | 17.04 | Confirmed |
| FR-NEW-52 | "История заявок" page for Requestor (completed requests history) | 17.04 | Confirmed |
| FR-NEW-53 | Usage Rate Dashboard — 3 tabs: Моточасы / Километраж / Комбайн (Combined) | 17.04 | Confirmed |
| FR-NEW-54 | Usage Rate cells — color gradient: 0% red → 50% yellow → 100% green | 17.04 | Confirmed |
| FR-NEW-55 | Repair days — wrench icon; JDE fields (commitment/planned/completed date); excluded from average | 17.04 | Confirmed (manual) |
| FR-NEW-56 | Usage Rate Dashboard — Target value column (FO-defined max per equipment) | 17.04 | Confirmed |
| FR-NEW-57 | Usage Rate Dashboard — Average column; sortable by average | 17.04 | Confirmed |
| FR-NEW-58 | Usage Rate Dashboard — Day/Month view toggle | 17.04 | Confirmed |
| FR-NEW-59 | Usage Rate Dashboard — Ownership filter (TCO Owned / Long-term rented) | 17.04 | Confirmed |
| FR-NEW-60 | Usage rate from trackers only; independent of bookings | 17.04 | Confirmed |
| FR-NEW-61 | GIS map — iframe; checkboxes in equipment list; ТШО search; expand button | 17.04 | Confirmed (GIS team) |
| FR-NEW-62 | GIS map tooltip — sensor data from last sync + current-day usage rate; refresh once/day (OQ-NEW-Q CLOSED) | 17.04 / **Updated 23.04** | Pending (GIS team — OQ-NEW-O) |
| FR-NEW-63 | Usage Rate Dashboard — booking count for selected period next to average usage rate | 20.04 | Confirmed |
| FR-NEW-64 | FO approval window — equipment loading summary by dates (list of nearby active bookings) | 20.04 | Confirmed |
| FR-NEW-65 | GIS map — speed and heading for GSM-tracked equipment; heading not shown for LoRaWAN | 20.04 | Confirmed |
| FR-NEW-66 | GIS map — fuel level not displayed | 20.04 | Confirmed |
| FR-NEW-67 | GIS map — historical route for 7 days | 20.04 | Confirmed (GIS team) |
| FR-NEW-68 | Admin Panel tool for configuring dynamic custom characteristics per equipment type; system shows only type-specific characteristics in request form and search filters | **22.04** | Confirmed |
| FR-NEW-69 | Priority field (P1–P4) in request creation form — tooltips/hints describing each priority level; content (P1–P4 descriptions) TBD — to be provided by TCO; configured via Admin Panel | **28.04** | Confirmed (pending P1–P4 descriptions from TCO) |
| FR-NEW-70 | Requestor booking flow — equipment load summary for selected date range: list of existing confirmed/submitted bookings for the selected equipment unit; allows Requestor to assess conflicts before submitting | **28.04** | Confirmed |
| FR-NEW-71 | Long-term rented booking item — mandatory **Justification** field per booking item (not per request): free-text, why TCO Owned is not being booked. Displayed to FO and FleetOwners' Supervisor in their respective approval windows. Request cannot be submitted without Justification for Long-term rented items. | **05.05** | Confirmed |
| FR-NEW-72 | Long-term rented booking — after FO confirms, booking moves to **"Confirmed by FO"** status and is routed to FleetOwners' Supervisor. FO confirmation alone does not finalize Long-term rented bookings. | **05.05** | Confirmed |
| FR-NEW-73 | FleetOwners' Supervisor approval — can Confirm (→ "Confirmed") or Decline (→ "Declined" immediately, mandatory comment). Requestor and FO notified of decision. | **05.05** | Confirmed |
| FR-NEW-74 | Admin configures FleetOwners' Supervisor response timeout via Admin Panel. Default value TBD. Escalation logic upon timeout TBD. | **05.05** | Confirmed (timeout value and escalation Pending — OQ-NEW-S) |
| FR-NEW-75 | Notification to FleetOwners' Supervisor when Long-term rented booking reaches "Confirmed by FO" status. Includes: booking details, equipment info, Requestor name, Justification text. | **05.05** | Confirmed |
| FR-NEW-76 | Notification to Requestor and FO when FleetOwners' Supervisor makes decision on Long-term rented booking. Includes Supervisor comment (mandatory on decline). | **05.05** | Confirmed |
| FR-NEW-77 | Booking status transition: Long-term rented booking → **"Confirmed by FO"** upon FO confirmation (intermediate status pending Supervisor review). | **05.05** | Confirmed |
| FR-NEW-78 | Booking status transition: Long-term rented booking → **"Confirmed"** upon Supervisor approval; → **"Declined"** upon Supervisor decline. | **05.05** | Confirmed |
