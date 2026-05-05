# HDV/HDE Booking Tool

# Business Requirement Document — v4

**Version:** 4.0  
**Date:** 2026-04-13  
**Based on:** BRD v3 (2026-04-10) + Requirements gathered in meeting 13.04 (~15:00)  
**Scope:** Phase 1 unless noted as Phase 2  
**Prepared by:** Telman Nurzhanov (SA)

### Changelog v3 → v4

| # | Раздел | Изменение |
|---|--------|-----------|
| 1 | 3. Glossary | Уточнено определение **Criticality**: критичное = требует сопровождения службы безопасности при перевозке |
| 2 | 3. Glossary | Уточнено **Shared with Conditions**: разница с without — только наличие поля Justification; отзыв FO — отдельный механизм |
| 3 | 4.2 Service Work Processor | Обновлена гипотеза роли: предположительно графисты/планировщики, обрабатывающие черновики Work Order из JDE (статус 55) |
| 4 | 4.3 Fleet Owner | Подтверждено: флот может иметь **2+ Fleet Owner-а** (сменщики); отображаются оба + shared team email |
| 5 | 5.1 Equipment | Добавлены специфические параметры по **8 типам техники**: погрузчик, кранманипулятор, генератор, компрессор, самосвал, фрактанк, трейлер, тягач |
| 6 | 5.1 Equipment | Fleet Owner: отображать двух сменщиков + **shared team email** в карточке; нотификации через AD-группы |
| 7 | 5.1 Equipment | Добавлена поддержка **нескольких трекеров** на одну единицу техники (разные функции: моточасы / GPS) |
| 8 | 5.1 Equipment | Утилизация: добавлены поля **плановые моточасы** и **плановый километраж в день** (редактирует FO); фактические — только из системы |
| 9 | 5.1 Equipment | Добавить в карточку техники: **календарь бронирования** (визуальный, забронированные периоды) и **cost center** |
| 10 | 5.1 Equipment | Уточнено: **госномер** есть не у всей техники; ТШО-номер присваивается только при покупке; арендная техника приходит со своим госномером |
| 11 | 6.2 Admin Panel | Шаблоны (FR-010/011/012): добавление **кастомных атрибутов** (колонок и фильтров) по типам техники без участия разработки — backlog после согласования базовых полей |
| 12 | 6.3 Equipment Management | Добавлен FR-NEW-40: booking calendar в карточке техники (FO) |
| 13 | 6.3 Equipment Management | Добавлен FR-NEW-41: поддержка нескольких трекеров на единицу техники |
| 14 | 6.3 Equipment Management | Добавлен FR-NEW-42: shared team email для флота как поле флота |
| 15 | 6.10 Notifications | FR-NEW-43: уведомления направляются на shared team email флота (через AD-группы) |
| 16 | 8.1 JDE E1 | Уточнён **путь интеграции**: JDE → DataLake (sync ~30 мин) → Booking Tool; контакт: Аяжан (TFM, группа 2835) |
| 17 | 9. Conflicts | CONFLICT-02 (статусы auto-transition) — остаётся открытым |
| 18 | 10. Open Questions | OQ-NEW-D **ЗАКРЫТ**: Shared with/without условия — разница только в Justification |
| 19 | 10. Open Questions | OQ-30 **ЧАСТИЧНО ЗАКРЫТ**: fleet может иметь 2+ FO (сменщики); shared email — механизм нотификации |
| 20 | 10. Open Questions | OQ-NEW-A **ЧАСТИЧНО ЗАКРЫТ**: параметры для Loader, Crane-Manipulator согласованы; остальные типы — Action Асылбек |
| 21 | 10. Open Questions | Добавлен OQ-NEW-E: делегирование FO — требует согласования с отделом кибербезопасности |
| 22 | 10. Open Questions | Добавлен OQ-NEW-F: типы техники Vacuum Truck и другие — параметры не разобраны |

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
- Each Fleet Owner independently defines the **100% utilization threshold** for each equipment type under their fleet (e.g., 2 hours of operation per day = 100%).
- **Maximum Utilization Capacity per Shift** — the maximum possible utilization of a given equipment unit within a working shift; serves as the 100% reference point for utilization calculations. *(Максимальная возможная утилизация в рабочую смену)*
- Utilization percentage is calculated per day, month, or any selected period.
- **🆕 [Updated 13.04]:** Some equipment types have **both** mileage and engine hours parameters. Planned values are entered by FO; actual values are sourced from trackers only (not manually editable).

**🆕 [Updated 13.04] Criticality** *(Критичность)* — equipment marked as critical requires **security escort (service de sécurité)** during transportation. Not a maintenance priority indicator. Displayed in equipment card.

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
7. 🆕 Close booking by pressing "Close" button (manual close only) — ✅ *(Закрытие брони по кнопке)*
8. 🆕 Book Assigned equipment only if authorized by Administrator — ✅ *(Бронирование закреплённой техники — только при авторизации Admin)*

### 4.2 Service Work Processor

> **⚠️ PARTIALLY CLARIFIED (13.04):** Based on meeting discussion, the Service Work Processor role is likely **graphists / schedulers** who work with Work Orders in JDE and process drafts once a Work Order step reaches a confirmed status (≈55 in JDE). They push Service Work Requests from Draft to Submitted for Fleet Owner processing. Formal confirmation from TCO required (OQ-NEW-C).

~~Access provision is managed by Azure Active Directory (AAD) group. Similar to Requestor, however, Service Work Processor works with Service Work Requests, created automatically in HDV/HDE booking tool.~~

### 4.3 Fleet Owner

For internal Fleet Owners access provision is managed by AAD group. There can be 2+ AAD groups to manage internal Fleet Owners (separate group for each fleet created by Administrator).

**🆕 [Updated 13.04]:** One fleet can have **two or more Fleet Owners** (e.g. shift workers / сменщики). Both owners are displayed in the equipment card. Each fleet has a **shared team email** address used as a single notification target (alongside personal emails via AD group). When a Fleet Owner changes role, Administrator reassigns the fleet without rebuilding the fleet structure.

Fleet Owners are allowed to:

1. 🆕 **[Updated 10.04]** Create new equipment under own fleet (allowed for **both internal and external** Fleet Owners) — ✅ for internal; 🔵 Phase 2 for external fleet onboarding workflow
2. Update equipments (only parameters that are allowed for editing by Administrator) — ✅
3. Delete equipment (soft delete) — ✅
4. Confirm / decline requests to book equipments under own fleet — ✅
5. 🆕 **[Updated 10.04 — CONFLICT-01 RESOLVED]** Update booking period **before or after confirmation**; Requestor is notified automatically; no re-confirmation required — ✅
6. Change equipment in booking — ✅
7. Terminate already confirmed bookings (only internal fleet) — ✅
8. Freeze / unfreeze own equipment from booking for a period or without end date — ✅
9. Monitor utilization of own equipment, if the equipment has tracker(s) — ✅
10. 🆕 Press "Mobilization started" button to fix mobilization start time — ✅ *(Кнопка «Мобилизация началась»)*
11. 🆕 Delegate FO privileges to any TCO employee for a specified period (without IT involvement) — ⚠️ Pending cybersecurity sign-off (OQ-NEW-E)
12. 🆕 One user can be an owner of several fleets (member of several AAD groups) — ✅
13. 🆕 **[Updated 13.04]** One fleet can have multiple Fleet Owners (shift workers); both displayed in equipment card with shared team email — ✅ *(OQ-30 partially closed)*

### 4.4 CC Owner (incl. DOA)

> **[Confirmed 10.04]:** CC Owner confirmation is NOT required for Long-term rented equipment — this follows the internal booking flow. CC Owner is applicable only to on-demand external fleet booking, which is in DCT system.

CC owners are allowed to:

1. Confirm / decline external fleet equipment booking request — ❌ **Obsolete** *(CC Owner approval applies only to on-demand external fleet booking, which is done in DCT — not in this tool)*

### 4.5 Administrator

Administrator role will be provided to process owners – Logistics. Access provision is managed by Azure Active Directory (group). Users with an Administrator role are allowed to:

1. Create/edit internal fleet and link with AAD group — ✅
2. Create/edit external fleet and manage external fleet owners (send invitation, deactivate external user's account) — 🔵 Phase 2
3. Delete fleet — ✅
4. Create internal fleet equipment — ✅
5. Update equipment's parameters (extended edit) — ✅
6. Delete equipment (soft delete) — ✅
7. Manage 'Request' template — ⚠️ Partially clarified (OQ-NEW-B); **baseline fields first; custom attribute management is backlog**
8. Manage 'Equipment' template — ⚠️ Partially clarified (OQ-NEW-B); **adding custom attributes (columns/filters) per equipment type — backlog after baseline fields agreed**
9. Manage 'Booking' template — ⚠️ TBD (OQ-NEW-B)
10. Access to custom reportings — ✅
11. Access to utilization dashboard(s), if the equipment has tracker(s) — ✅
12. 🆕 Manage all configurable system parameters (booking horizon, max duration, FO timeout thresholds, etc.) via Admin Panel — ✅
13. 🆕 Authorize specific users to book Assigned equipment — ✅
14. 🆕 Manage delegation settings — ⚠️ Pending cybersecurity sign-off (OQ-NEW-E)
15. 🆕 **[Updated 13.04]** Manage fleet shared team email — ✅ *(Управление shared email флота)*

### 4.6 🆕 Transportation Responsible (Мобилизация/Транспортировка)

A dedicated system role for the team responsible for transporting non-motorized (Unwheeled) equipment (Heavy Ops / Maintenance team).

This role is allowed to:

1. 🆕 Receive notification about transportation request for Unwheeled equipment — ✅
2. 🆕 Confirm or decline the transportation request — ✅

---

## 5 Entities

### 5.1 Equipment

All equipments must have a mandatory attribute – fleet. The attribute will affect on which template the equipment will have (depending on internal and external fleet), who can create/edit/delete equipments (fleet owners), approval and processing flow of the equipment booking. — ✅

**🆕 Fleet naming (10.04):** Each fleet must have a unique **name** assigned at creation (e.g., "Heavy Ops", "Logistics", "Maintenance", "Production"). Equipment assignment is linked to a specific employee (not a department); when a Fleet Owner changes, the Administrator manually re-assigns the fleet to the new person.

**🆕 Fleet Owners display (13.04):** Equipment card displays **up to 2+ Fleet Owners** (shift workers) alongside their **shared team email**. Notifications are sent to shared team email and/or personal emails via AD group.

Internal fleet equipments have the next parameters:

1. Basic fixed info (TCO ID, Description, Service Zone, Class/Category, License Plate *(if present — not all equipment has one)*, Serial number, etc.) — ✅ ⚠️ *(Базовый перечень полей разбирается по типам техники — OQ-NEW-A частично закрыт)*
2. Dynamic (changeable) parameters: Equipment Usage Status, **Cost Center**, CC/Division/Group/Department, Assigned location, Maintenance BP, Contact number / email, etc. — ✅
3. Equipment specifications (picture, Chassis type, Drive type, Loading capacity, Generator / Compressor specifications) — ✅
4. Trackers' IDs if exist (WIALON tracker "Group", IVMS tracker "Registration number", Vega tracker "Device ID", etc) — ✅ **🆕 [Updated 13.04]: Multiple trackers per unit supported** (e.g. one for engine hours, one for GPS/mileage)
5. **🆕 [Updated 13.04]** Planned engine hours (per shift/day) — editable by FO — ✅
6. **🆕 [Updated 13.04]** Planned daily mileage (km/day) — editable by FO — ✅
7. **🆕 [Updated 13.04]** Actual engine hours / mileage — sourced from tracker only; not manually editable — ✅

**🆕 Equipment card UI additions (13.04):**
- **Booking calendar**: visual calendar in the equipment card (FO view) showing which periods are booked; confirmed required ✅
- **Criticality flag**: displayed in card; meaning = security escort required during transport ✅

**🆕 License plate and TCO number rules (13.04):**
- **License plate (госномер)**: present on most wheeled equipment; may be absent on some stationary units. Field optional/conditional.
- **TCO number (ТШО-номер)**: assigned when equipment is purchased for TCO; long-term rented equipment arrives with its own license plate and may not have a TCO number. Business partners (L-rent) may have neither.

**🆕 Equipment Classification (three dimensions):**

| Dimension | Values |
|-----------|--------|
| Mobility | Self-propelled / Not self-propelled (Unwheeled) / Stationary |
| Ownership | TCO Owned / Long-term rented / On-demand rented (BP Showcase) |
| Usage Status | Assigned / Shared without Conditions / Shared with Conditions |

> **Note (08.04):** Mobility category is NOT displayed in Requestor UI — Requestor sees a flat list of equipment types. ✅  
> **Note (08.04):** Stationary HDE is excluded from Requestor search — stored in system for FO use only. ✅  
> **Note (13.04 — OQ-NEW-D CLOSED):** Shared with Conditions differs from without Conditions only in the presence of a mandatory **Justification** field upon booking. FO can recall any booking at any time — this is a separate mechanism independent of conditions type. ✅

**🆕 Equipment availability display rules:**
- Equipment under maintenance is shown grayed out, not selectable; label "Under maintenance until [Planned date from JDE]" ✅
- Repair status and planned completion date are sourced from JDE → DataLake integration — ⚠️ (depends on integration)

---

### 🆕 5.1.1 Equipment Type-Specific Parameters (13.04)

> The parameters below define specific attributes and search filters for each equipment type. These are used both in the equipment card and in the Requestor search/filter form. All parameters are preliminary and subject to final sign-off from TCO Fleet Owners. Remaining types to be covered per OQ-NEW-F.

#### Loader / Погрузчик

| Parameter | Values / Format | Used in filter |
|-----------|----------------|----------------|
| Load capacity (грузоподъёмность) | Ranges: up to 2 t / 2–4.5 t / 4.5–8 t | ✅ Dropdown or range |
| Type | Fork (вилочный) / Bucket (ковшовый) | ✅ |
| Engine type | Electric / Diesel | ✅ |

#### Crane-Manipulator / Кранманипулятор

| Parameter | Values / Format | Used in filter |
|-----------|----------------|----------------|
| Vehicle load capacity | numeric (tons) | ✅ |
| Crane load capacity | numeric (tons) | ✅ |
| Boom length | numeric (meters) | ✅ |
| Body length (кузов) | numeric (meters) | ✅ |

#### Mobile Generator / Генератор передвижной

| Parameter | Values / Format | Used in filter |
|-----------|----------------|----------------|
| Power (мощность) | Dropdown: 30 / 60 / 80 / 100 / 120 / 130 / 200 / 250 kW (exact list TBD per fleet data) | ✅ Checkbox multi-select |
| Voltage (напряжение) | — | ⚠️ TBD |
| Current (ток) | — | ⚠️ TBD |
| Battery capacity (ёмкость батарей) | — | ⚠️ TBD |
| Mobility type | Wheeled / Non-wheeled / Stationary | ✅ |

> Action (Асылбек): provide final grouping of generator power categories from master fleet file.

#### Motor Compressor / Компрессор моторный

| Parameter | Values / Format | Used in filter |
|-----------|----------------|----------------|
| Pressure (давление) | numeric (atm); dropdown or range TBD | ✅ |
| Productivity (производительность) | numeric (m³/h); dropdown or range TBD | ✅ |

> Action (Асылбек): provide grouping of compressor pressure and productivity categories.

#### Dump Truck / Самосвал

| Parameter | Values / Format | Used in filter |
|-----------|----------------|----------------|
| Load capacity (тоннаж) | Dropdown or range (exact ranges TBD) | ✅ Primary filter |
| Body type (тип кузова) | Open / Closed (tent) — secondary; user can also reference photo | Via photo / optional |

#### Frac Tank / Фрактанк

| Parameter | Values / Format | Notes |
|-----------|----------------|-------|
| No type-specific filters | All frac tanks have the same volume | Any available unit can be selected |
| Dedicated transport vehicle | Kenworth Winch Truck (1 unit in fleet) | System note: if transport vehicle is unavailable, frac tank cannot be deployed |

> Decision point: transport vehicle for frac tank is the only Kenworth Winch Truck. System behavior when it is occupied — TBD (not currently part of booking workflow).

#### Cargo Trailer / Прицеп грузовой трейлер карго

| Parameter | Values / Format | Used in filter |
|-----------|----------------|----------------|
| Type | Low-frame (низкорамный) / Standard | ✅ |
| Load capacity | numeric (tons) | ✅ |
| Suspension system | Pneumatic / Standard | ✅ |

#### Semi-truck / Тягач HD

| Parameter | Values / Format | Notes |
|-----------|----------------|-------|
| No independent filter | Tractor unit is selected by FO based on trailer type | Requestor does not select tractor independently |

> **Decision (13.04):** Tractor head is determined by the trailer that needs to be towed. FO selects appropriate tractor. Requestor-facing filter for semi-truck is not required.

---

### 5.2 Booking

Booking is linked to a single equipment and have booking range (datetime). Several bookings form a single request. — ✅

> **🆕 Architecture decision (09.04):** Data model follows Order/OrderLine pattern: one Request contains multiple RequestItems (one per equipment); each RequestItem independently processed by FO and generates a separate Booking. ✅

#### 5.2.1 Internal Booking

Booking of TCO owned (internal fleet, including Long-term rented) equipment. Internal Fleet equipments require only confirmation from the side of Internal Fleet owner. Fleet owner can apply adjustments to the booking period and/or replace equipment if substitution exists. — ✅

> **🆕 Mobilization (09.04):** Booking period starts from mobilization start, not from arrival at work site. FO presses "Mobilization started" button to fix the actual start in the system. ✅

> **🆕 [Updated 10.04 — CONFLICT-01 RESOLVED]:** FO can adjust the booking period **both before and after confirmation**. Requestor is notified automatically upon any change. No re-confirmation from Requestor is required. ✅

#### 5.2.2 External Booking

**❗ CONFLICT / PHASE CHANGE vs. original BRD:**

Original BRD described a two-step approval flow (CC Owner → External FO) for external fleet equipment. **Meeting decision (08.04): On-demand BP equipment is NOT bookable through the system in Phase 1.** It is displayed as a catalog/showcase only.

- On-demand rented (BP Showcase) — catalog view only; no booking workflow in Phase 1. ✅
- Long-term rented — treated as **internal fleet**; no CC Owner step required. ✅ *(Confirmed 10.04)*
- Full external booking workflow (CC Owner → External FO → DCT/RWA) — ❌ **Obsolete**

#### 5.2.3 🆕 Booking Snapshot

At the moment of booking confirmation, the system saves a snapshot of key equipment attributes (model, license plate, fleet name). This ensures historical readability when equipment data is later updated. — ⚠️ Preliminary (OQ-37)

### 5.3 Request

Requests have 2 types: Regular Request, Service Work Request. The type affects on request template, mandatority of filling parameters and approvers' workflow. — ✅

#### 5.3.1 Regular Request

In a Regular Request Requestor can add **1 or more** equipments from any fleet; there is no restriction that all equipments must be from the same fleet. Each added equipment will be considered as separate booking and will be confirmed by Fleet Owners separately. — ✅

> **[Updated 10.04]:** Minimum equipment count in a request is **1** (not 2+).

The system will prioritize TCO owned (internal fleet) assets as a **business rule** (not technically enforced at current selection model). If FO does not respond within timeout — Requestor is redirected to the BP catalog.

**🆕 Request fields (08.04):**

- **Work Order (JDE)** — mandatory for Maintenance / Railroad / Operations fleets. One Work Order per request. ✅
- **Location** (text input) — replaces Work Order field for SCM Logistics fleet. ✅
- **Work Description** — mandatory, ~50 character limit. ✅
- **Comments** — optional, no hard limit. ✅
- **Priority** (P1–P4) — selected by Requestor. ✅

> **Note (10.04):** Within one request (Work Order), all equipment items share the same **location**, but may have **different booking time periods**.

**🆕 Request constraints (03.04):**

- Maximum booking horizon: 1 month ahead (configurable by Admin). ✅
- Maximum single booking duration: 1 week (configurable by Admin). ✅

**🆕 Form → Bookings split (09.04):** Requestor fills a single form for a Work Order with multiple equipment items. Upon submission, the system **automatically splits** the form into atomic individual bookings — one per equipment item. All bookings from one form are linked via the Work Order field. ✅

#### 5.3.2 Service Work Request (within Work Order in JDE E1)

> **⚠️ PARTIALLY CLARIFIED (13.04):** Service Work Requests are auto-created as Drafts in the tool when a Work Order step reaches a specific status in JDE. Based on discussion, Work Orders progress through statuses: ~50 (scheduler review) → **55 (released to operations)**. Steps at status 55 are candidates for import. Service Work Processor (likely a scheduler/graphist) reviews the draft and submits it to Fleet Owner. Each 'Step' in Work Order will be created as a separate Service Work Request. Formal confirmation from TCO required (OQ-NEW-C).

---

## 6 Functional Requirements

### 6.1 User & Role Management

| ID | Requirement | Status |
|----|-------------|--------|
| FR-001 | System supports role-based access control | ✅ Confirmed |
| FR-002 | All users in TCO network have access to the tool as a Requestor; to add external fleet equipment, the user must be a member of "RWA initiator" AAD group | ❌ **Obsolete** — booking of external fleet equipment is in DCT, not this tool |
| FR-003 | Administrators, Service Work Processors are defined by membership in the corresponding AAD groups | ⚠️ SWP pending (OQ-NEW-C) |
| FR-004 | The system will define if the user has CC Owner or DOA privileges as a result of integration with PSWS and DOA | ❌ **Obsolete** — CC Owner role is not applicable; external fleet booking is in DCT |
| 🆕 FR-NEW-01 | All time-based parameters (FO response timeout, booking horizon, max booking duration) are configurable via Admin Panel and are NOT hardcoded | ✅ Confirmed |
| 🆕 FR-NEW-02 | FO can delegate own privileges to any TCO employee for a specified date range (start + end date); delegation is automatically revoked upon expiry; delegation scope is configurable | ⚠️ Pending cybersecurity sign-off (OQ-NEW-E) |

### 6.2 Admin Panel

| ID | Requirement | Status |
|----|-------------|--------|
| FR-005 | Admin can create fleets (internal, external); each fleet must be given a unique name | ✅ Confirmed |
| FR-006 | Admin can link internal fleet with AAD group, so that members of that group can act as Fleet Owners | ✅ Confirmed |
| FR-007 | Admin can manage owners of created external fleets: send invitation to BP, deactivate existing external accounts | 🔵 Phase 2 |
| FR-008 | Admin can update existing fleets | ✅ Confirmed |
| FR-009 | Admin can delete existing fleets | ✅ Confirmed |
| FR-010 | Admin can manage template of equipment (internal and external): define parameters, formats, mandatority, editability by owners. **[Updated 13.04]** Baseline fields defined first; ability to add custom attributes (new columns/filters per equipment type) is **backlog** | ⚠️ Baseline first; custom attributes → 📋 Backlog |
| FR-011 | Admin can manage template of request. **[Updated 13.04]** Backlog after baseline fields agreed | 📋 Backlog (OQ-NEW-B) |
| FR-012 | Admin can manage template of booking | ⚠️ TBD (OQ-NEW-B) |
| FR-013 | Admin can create equipments of internal fleet | ✅ Confirmed |
| FR-014 | Admin can edit (extended edit) equipments | ✅ Confirmed |
| FR-015 | Admin can delete equipments | ✅ Confirmed |
| FR-016 | Admin has access to all dashboards and reports | ✅ Confirmed |
| 🆕 FR-NEW-03 | Admin can configure all system time parameters: FO response timeout (default 24h reminder / 48h escalation), max booking horizon, max booking duration | ✅ Confirmed |
| 🆕 FR-NEW-04 | Admin can authorize specific users (by employee ID or role) to book Assigned equipment and configure justification requirement | ✅ Confirmed |
| 🆕 FR-NEW-42 | **[New 13.04]** Admin can set and update **shared team email** for each fleet; used as notification target alongside personal emails via AD group | ✅ Confirmed |

### 6.3 Equipment Management

| ID | Requirement | Status |
|----|-------------|--------|
| FR-017 | **[Updated 10.04]** Both internal and external Fleet Owners can create equipment under their own fleet. Long-term rented equipment is created by the internal TCO team (not by BP). | ✅ Confirmed |
| FR-018 | Fleet owner must indicate 'Fleet' of the equipment; system allows selecting only fleets for which the user is a FO | ✅ Confirmed |
| FR-019 | Fleet owner can edit (allowed by Admin) equipments under his fleet; one user can be an owner of several fleets | ✅ Confirmed |
| FR-020 | Fleet owner can delete equipments under his fleet | ✅ Confirmed |
| FR-021 | Fleet owner can freeze / unfreeze own equipments for a period of time or without end date | ✅ Confirmed |
| FR-022 | Requestor / Service Work Processor can submit feedback on equipment with confirmed booking, starting from the booking start datetime | ✅ Confirmed |
| 🆕 FR-NEW-05 | FO can upload photos for equipment; Requestor sees equipment photo in the request form | ✅ Confirmed |
| 🆕 FR-NEW-06 | Equipment repair status (maintenance in progress / completion date) is sourced from JDE → DataLake integration and displayed in Requestor search results | ⚠️ Pending (JDE integration detail) |
| 🆕 FR-NEW-07 | Stationary HDE equipment is excluded from Requestor search results; stored in system for FO use only | ✅ Confirmed |
| 🆕 FR-NEW-08 | Assigned equipment is visible to all users in search, but can only be booked by users authorized by Admin; "Justification" field is mandatory for Assigned equipment booking | ✅ Confirmed |
| 🆕 FR-NEW-09 | Shared with Conditions equipment is displayed with a visual marker (color coding) | ✅ Confirmed |
| 🆕 FR-NEW-40 | **[New 13.04]** Equipment card (FO view) displays a **booking calendar** — visual representation of booked periods (e.g. color-coded days) to help FO plan maintenance and availability | ✅ Confirmed |
| 🆕 FR-NEW-41 | **[New 13.04]** An equipment unit can have **multiple trackers** linked simultaneously (e.g. engine hours tracker + GPS/mileage tracker); all tracker IDs stored in equipment card | ✅ Confirmed |

### 6.4 Request Management

| ID | Requirement | Status |
|----|-------------|--------|
| FR-023 | Requestor can create a Request | ✅ Confirmed |
| FR-024 | Request has a unique identifier and metadata (date, requester, status, etc.) | ✅ Confirmed |
| FR-025 | Requester can view request details and status | ✅ Confirmed |
| FR-026 | Request can have status: Draft, Submitted, In Progress, Completed | ⚠️ Aggregated statuses TBD (OQ-36) |
| FR-027 | Requester can edit or cancel a draft request before submission | ✅ Confirmed |
| FR-028 | Requests created by system automatically (Service Work Requests) will have 'Draft' status before submission by Service Work Processor | ⚠️ Pending (OQ-NEW-C) |
| FR-029 | Only users with role Service Work Processor can manage (submit) Service Work Requests | ⚠️ Pending (OQ-NEW-C) |
| FR-030 | System supports draft saving (continue later) | ✅ Confirmed |
| 🆕 FR-NEW-10 | TO-BE: only on-demand booking; weekly schedule management is out of scope | ✅ Confirmed |
| 🆕 FR-NEW-11 | Requestor selects a booking priority P1–P4 at the time of request creation | ✅ Confirmed |
| 🆕 FR-NEW-12 | Work Order field (JDE) is mandatory for Maintenance/Railroad/Operations fleets; for SCM Logistics fleet this field is replaced by a "Location" (free text) field; Work Order is one-to-one with a request | ✅ Confirmed |
| 🆕 FR-NEW-13 | "Work Description" field is mandatory in the request form (~50 character limit) | ✅ Confirmed |
| 🆕 FR-NEW-14 | "Comments" field is optional in the request form (no hard character limit) | ✅ Confirmed |
| 🆕 FR-NEW-15 | Requestor submits a single form for a Work Order with multiple equipment items; the system automatically splits the form into individual bookings (one per equipment item); all bookings are linked via the Work Order field | ✅ Confirmed |
| 🆕 FR-NEW-16 | Partial confirmation: confirmed items within a request proceed independently from declined items | ✅ Confirmed |
| 🆕 FR-NEW-17 | Aggregated Request status is calculated automatically based on statuses of its RequestItems | ⚠️ Details TBD (OQ-36) |

### 6.5 Booking Management

| ID | Requirement | Status |
|----|-------------|--------|
| FR-031 | Requestor can add/remove multiple equipment items into a single request; equipment availability in DB is updated | ✅ Confirmed |
| FR-032 | The system prioritizes internal fleet equipments | ✅ Confirmed (as business rule, not technically enforced) |
| FR-033 | Requestor can add internal fleet equipments with "Equipment Usage Status" in (Shared without conditions, Shared with conditions) | ✅ Confirmed |
| FR-034 | The system will suggest external fleet equipments based on a pre-set decision matrix | ❌ **Obsolete** — replaced by BP showcase model |
| FR-035 | Requestor can add external fleet equipment to Request, only if RWA initiator | ❌ **Obsolete** |
| FR-036 | When Requestor adds external fleet equipment to Request, he must provide "Justification" | ❌ **Obsolete** |
| FR-037 | When Requestor adds external fleet equipment, "Cost Center" must be filled | ❌ **Obsolete** |
| FR-038 | Each equipment item added to Request is treated as a separate booking | ✅ Confirmed |
| FR-039 | Each booking has its own unique booking ID, start and end datetimes | ✅ Confirmed |
| FR-040 | System validates availability of equipment before booking | ✅ Confirmed |
| FR-041 | Equipment attributes (picture, fleet, description, usage status, etc.) are displayed | ✅ Confirmed |
| FR-042 | Each booking has its own lifecycle (Draft, Submitted, Confirmed, Declined, Revoked, Terminated, Completed) | ✅ Confirmed |
| 🆕 FR-NEW-18 | Booking start datetime represents the start of **mobilization**, not arrival at work site | ✅ Confirmed |
| 🆕 FR-NEW-19 | No mandatory buffer between bookings; equipment availability is validated for the requested period only | ✅ Confirmed |
| 🆕 FR-NEW-20 | When Requestor adds Unwheeled (Not self-propelled) equipment, the system displays a notification: "Transportation equipment will be added to this booking type" | 🟡 Preliminary |

### 6.6 Approval Flow (Internal Fleet Equipment)

| ID | Requirement | Status |
|----|-------------|--------|
| FR-043 | System automatically assigns approver based on fleet ownership | ✅ Confirmed |
| FR-044 | If equipment belongs to internal fleet, booking can be confirmed or declined by Fleet Owner (member of corresponding AAD group) | ✅ Confirmed |
| FR-045 | Fleet owner can confirm incoming bookings of equipment in his fleet | ✅ Confirmed |
| FR-046 | Fleet owner can replace the equipment in the booking with another equipment before or after confirmation if booking date is not started or booking end date is in future | ✅ Confirmed |
| FR-047 | Fleet owner cannot replace the equipment if booking is revoked, terminated or booking end date is in past | ✅ Confirmed |
| FR-048 | **[Updated 10.04 — CONFLICT-01 RESOLVED]** Internal fleet owner can adjust booking date range **before or after confirmation**; Requestor is notified automatically; no re-confirmation from Requestor required; equipment availability in DB is updated. | ✅ Confirmed |
| FR-049 | Fleet owner can decline incoming bookings of equipment in his fleet; equipment availability in DB is updated | ✅ Confirmed |
| FR-050 | Fleet owner can indicate 'reason/comment' in case of declining | ✅ Confirmed |
| 🆕 FR-NEW-21 | FO must respond to a booking request within a configurable timeout period. Default: 24h — reminder notification; 48h — escalation to manager. Timeout is tracked per RequestItem | ✅ Confirmed |
| 🆕 FR-NEW-22 | If FO has free equipment of the requested type but declines, FO must provide a mandatory reason | ✅ Confirmed |
| 🆕 FR-NEW-23 | Reminder notification is sent only to the FO of the specific selected equipment item | ✅ Confirmed |
| 🆕 FR-NEW-24 | FO presses "Mobilization started" button to record the actual mobilization start time in the system | ✅ Confirmed |

### 6.7 Approval Flow (External Fleet Equipment)

> **❗ MAJOR SCOPE CHANGE vs. original BRD (08.04):** On-demand BP equipment is **NOT bookable through the system at all**. It is displayed as a showcase/catalog only. All FR-051..057 are **Obsolete**.

| ID | Requirement | Status |
|----|-------------|--------|
| FR-051 | External fleet booking requires CC owner confirmation | ❌ **Obsolete** |
| FR-052 | System identifies CC owner via PSWS integration | ❌ **Obsolete** |
| FR-053 | System allows DOAs of CC owners to confirm/decline bookings | ❌ **Obsolete** |
| FR-054 | System identifies if user is DOA of CC owner via PSWS and DOA integration | ❌ **Obsolete** |
| FR-055 | CC owner/DOA can put 'reason/comment' when declining | ❌ **Obsolete** |
| FR-056 | After CC owner confirmation, system requires External FO confirmation | ❌ **Obsolete** |
| FR-057 | External FO can confirm or decline the booking | ❌ **Obsolete** |
| 🆕 FR-NEW-25 | BP showcase (catalog): Requestor can view on-demand BP equipment by type; each entry shows contract owner and availability condition; no booking action available | ✅ Phase 1 |
| 🆕 FR-NEW-26 | BP equipment prices/rates are NOT displayed in the system | ✅ Confirmed |
| 🆕 FR-NEW-27 | BP populates their own equipment cards in the catalog; contract owner is indicated per entry | ✅ Confirmed |
| 🆕 FR-NEW-28 | After FO response timeout with no available internal equipment, Requestor sees a button "Go to BP catalog" in the request | ✅ Confirmed |

### 6.8 Booking Lifecycle Management

| ID | Requirement | Status |
|----|-------------|--------|
| FR-058 | Requestor/SWP can revoke (withdraw) booking if not yet processed by FO; equipment availability updated | ✅ Confirmed |
| FR-059 | FO of internal fleet can terminate already confirmed booking; can indicate reason; equipment availability updated | ✅ Confirmed |
| FR-060 | Requestor/SWP can terminate already confirmed booking; equipment availability updated | ✅ Confirmed (internal fleet only) |
| FR-061 | Requestor/SWP can extend previously confirmed booking; equipment availability updated | ✅ Confirmed |
| 🆕 FR-NEW-29 | **Three-step flow for Unwheeled equipment:** (1) Requestor creates request → FO; (2) FO adds transportation equipment → system sends request to "Transportation Responsible" role; (3) Transportation Responsible confirms/declines; (4) FO gives final approval only after transport confirmed | ✅ Confirmed |
| 🆕 FR-NEW-30 | If transportation equipment is unavailable for Unwheeled request, FO proposes the nearest available date | ⚠️ Decision pending |
| 🆕 FR-NEW-31 | Booking can be closed early by Requestor or FO via "Close early" button | ⚠️ Clarification pending |

### 6.9 Booking/Request Status Management

> **CONFLICT-02 (FR-070/FR-071 vs. R-75):**  
> Auto-transition by datetime (FR-070/071/075) vs. Manual close only (R-75, 09.04). **Resolution: currently treating R-75 (manual close) as accepted decision. Explicit sign-off from TCO required.**

| ID | Requirement | Status |
|----|-------------|--------|
| FR-062 | Booking will have 'Submitted' status once Request is submitted | ✅ Confirmed |
| FR-063 | Booking will have 'Confirmed' status once confirmed by FO | ✅ Confirmed |
| FR-064 | Booking will have 'Declined' status once declined by FO | ✅ Confirmed |
| FR-065 | Booking of external fleet will have 'Partially Confirmed' status once confirmed by CC Owner | ❌ **Obsolete** |
| FR-066 | Booking of external fleet will have 'Declined' status once declined by CC Owner | ❌ **Obsolete** |
| FR-067 | Status of booking returns to 'Submitted' once extended by Requestor/SWP | ✅ Confirmed |
| FR-068 | Booking will have 'Revoked' status once revoked by Requestor/SWP | ✅ Confirmed |
| FR-069 | Booking will have 'Terminated' status once terminated by Requestor/SWP/FO | ✅ Confirmed |
| FR-070 | Confirmed booking status changes to 'In Progress' once system datetime >= booking datetime | ❗ **Conflict** — see CONFLICT-02 |
| FR-071 | Confirmed booking status changes to 'Completed' once system datetime < booking datetime | ❗ **Conflict** — see CONFLICT-02 |
| FR-072 | Request will have 'Draft' status once saved as draft by Requestor/SWP | ✅ Confirmed |
| FR-073 | Service Work Request will have 'Draft' status once imported from JDE E1 | ⚠️ Pending (OQ-NEW-C) |
| FR-074 | Request will have 'In Progress' status once at least one Booking within the Request has status 'In Progress' | ⚠️ Aggregated statuses TBD (OQ-36) |
| FR-075 | Request will have 'Completed' status once system datetime >= maximum end datetime | ❗ **Conflict** — see CONFLICT-02 |
| 🆕 FR-NEW-32 | **Booking closure is manual only:** Requestor or FO presses "Close" button. Automatic closure by scheduled end time does NOT apply | ✅ Confirmed — supersedes FR-070/071/075 until sign-off |
| 🆕 FR-NEW-33 | At the time of closing, Requestor inputs **actual start time** and **actual end time** of work; data is used for utilization analytics | ✅ Confirmed |

### 6.10 Notifications

| ID | Requirement | Status |
|----|-------------|--------|
| FR-076 | System notifies SWP(s) once Service Work Request is created automatically via API | ⚠️ Pending (OQ-NEW-C) |
| FR-077 | System notifies FO once Requestor/SWP submits a Request with internal fleet equipment | ✅ Confirmed |
| FR-078 | System notifies FO once Requestor/SWP revokes booking of internal fleet equipment | ✅ (presumed confirmed) |
| FR-079 | System notifies Requestor/SWP about results once FO confirms or declines internal fleet booking | ✅ (presumed confirmed) |
| FR-079a | 🆕 **[Updated 10.04]** System notifies Requestor when FO updates the booking period | ✅ Confirmed |
| FR-080 | System notifies Requestor/SWP once FO terminates a confirmed internal fleet booking | ✅ (presumed confirmed) |
| FR-081 | System notifies FO once Requestor/SWP wants to extend internal fleet booking | ✅ (presumed confirmed) |
| FR-082..090 | External fleet notification chain (CC Owner, External FO) | ❌ **Obsolete** |
| 🆕 FR-NEW-34 | System sends reminder to FO of specific equipment after 24h of no response; escalation to FO's manager after 48h | ✅ Confirmed |
| 🆕 FR-NEW-35 | System notifies Transportation Responsible role when FO adds transportation equipment to an Unwheeled booking | ✅ Confirmed |
| 🆕 FR-NEW-36 | System notifies FO when Transportation Responsible confirms or declines transportation readiness | ✅ Confirmed |
| 🆕 FR-NEW-43 | **[New 13.04]** System notifications to Fleet Owners are delivered to the fleet's **shared team email** (and/or to members of the fleet's AD group personal emails); shared email is set per fleet by Admin | ✅ Confirmed |

### 6.11 Search & Reporting

| ID | Requirement | Status |
|----|-------------|--------|
| FR-091 | Requestor / SWP can search and filter own requests | ✅ Confirmed |
| FR-092 | Approver (CC Owner / FO) can view all pending and completed approvals | ✅ Confirmed |
| FR-093 | Admin can view all requests | ✅ Confirmed |
| FR-094 | System generates reports on Requests / Bookings / Equipments with capability to apply various filters | ✅ Confirmed |
| FR-095 | Admin can view/download reports on all Requests / Bookings / Equipments | ✅ Confirmed |
| FR-096 | Approver (CC Owner / FO) can view/download reports on pending and completed Requests / Bookings | ✅ Confirmed |
| FR-097 | FO can view/download reports on own equipments | ✅ Confirmed |
| FR-098 | SWPs (and selected FOs — Maintenance) can view Service Work Requests grouped in Work Orders | ⚠️ Pending (OQ-NEW-C) |
| 🆕 FR-NEW-37 | Audit trail and history of all bookings available on demand (not daily/weekly scheduled reports) | ✅ Confirmed |
| 🆕 FR-NEW-38 | Search results for equipment must display as mandatory fields: **TCO equipment number** + **model/make**; license plate (госномер) displayed **if present** (not all equipment has one) | ✅ Confirmed *(Updated 13.04: госномер conditional)* |
| 🆕 FR-NEW-39 | Search filters in equipment search are **dynamic**: only parameters relevant to the selected equipment type are displayed; filter matrix (equipment type → filter set) defined in Section 5.1.1 | ⚠️ Partially confirmed (baseline defined; remaining types OQ-NEW-F) |

### 6.12 Dashboards / Map

| ID | Requirement | Status |
|----|-------------|--------|
| FR-099 | System has deployed/integrated visualization tool displaying utilization and coordinates from DataLake/PI (tracker data) | ✅ Confirmed |
| FR-100 | System has deployed/integrated visualization tool displaying digitized location of Work Orders from DataLake (JDE E1) | ✅ Confirmed |

### 6.13 Audit & Compliance

| ID | Requirement | Status |
|----|-------------|--------|
| FR-101 | System logs all request creation, modifications, approvals, rejections | ✅ Confirmed |
| FR-102 | Audit trail accessible for compliance purposes | ✅ Confirmed |
| FR-103 | Exportable logs for legal/regulatory audits | ✅ Confirmed |

---

## 7 Non-Functional Requirements

| Area | Requirement | Status |
|------|-------------|--------|
| Availability | 99.5% uptime | ✅ Confirmed |
| Performance | The system must support high volume of requests per hour without performance degradation | ✅ Confirmed |
| Security | Role-based access control (RBAC); system integrates with corporate authentication (SSO / Azure AD) | ✅ Confirmed |
| Compliance | Maintain audit history of all approvals / rejections / revokes / terminations | ✅ Confirmed |
| Scalability | Support both internal and external partner use cases | ✅ Confirmed |
| Usability | Intuitive UI for both requesters and approvers | ✅ Confirmed |
| 🆕 Users | ~200 users (per ACR assessment, 07.04); No PII data stored | ✅ Confirmed |
| 🆕 Security tier | ACR assessment result: Low Risk (07.04) | ✅ Confirmed |
| 🆕 Authentication | Azure SSO for internal users; separate account flow for external BP users (Phase 2) | ✅ Confirmed |
| 🆕 UX reference | UI simplicity reference: Booking.com | ✅ Confirmed |

---

## 8 Integrations

### 8.1 JDE E1

Work Orders created in JDE E1 must be imported to HDV/HDE tool as Service Work Request(s), once a certain status is assigned to the Work Order. Work Order can contain several Steps; each step is created as a separate Service Work Request in HDV/HDE tool. Only Steps with certain types (BHOE, CDECK, DTRK, FORKL, FTRK-ХИНО, GDRV, GMLOG, MLIFT, VTRK, etc.) will be imported.

The systems must have backward integration which allows changing the status of 'Step' in JDE based on status change of the imported 'Request' in HDV/HDE or by Service Work Processor's manual actions.

**🆕 [Updated 13.04] Integration path:** JDE E1 → **DataLake** (sync every ~30 minutes) → HDV/HDE Booking Tool. Direct JDE API integration is NOT used; data is consumed from DataLake.

**🆕 [Updated 13.04] JDE Work Order flow:** Steps progress: status ~50 (scheduler review by graphist) → status **55** (released to operations). Steps at status 55 are candidates for auto-import as Service Work Request Drafts.

**🆕 [Updated 13.04] Integration contact:** **Аяжан** — Maintenance Plan Consultant, group TFM 2835 — is the SME for JDE integration and DataLake data structure. Action: schedule meeting to clarify API format, data schema, and exact trigger status.

**Status:** ⚠️ Partially confirmed — JDE trigger status TBD; DataLake path confirmed; meeting with Аяжан required

🆕 **Repair status (08.04):** Repair status (maintenance in progress / planned completion date) — column "Planned End Date" in JDE — is sourced from DataLake and displayed in Requestor's equipment search. — ⚠️ Details TBD

### 8.2 DCT (Digital Contractor Timesheet)

#### 8.2.1 RWA / Change Order Creation
**Status:** ❌ **Obsolete** — booking of on-demand external equipment is done in DCT directly

#### 8.2.2 Tracker's data receival
**Status:** ❌ **Obsolete** — applicable only to external fleet booking workflow

### 8.3 PSWS (TCO White Page)

Identifies CC owner of cost centers for external fleet booking approval; provides preferred email address for notification recipients.

**Status:** ❌ **Obsolete** — CC Owner role is not applicable

> **Note (13.04):** White Pages may still be relevant for auto-populating user contact data (email, name) for Fleet Owners. Technical feasibility depends on shared email support in White Pages API — requires verification (some known bugs with shared emails in previous year). Alternative: manual entry of shared email in Admin Panel (FR-NEW-42).

### 8.4 DOA (Delegation of Authority)

Identifies if user is DOA of CC owner for external fleet booking processing.

**Status:** ❌ **Obsolete** — CC Owner/DOA flow is not applicable

### 8.5 DataLake / PI

From DataLake: utilization information from IVMS and WIALON trackers (extendable to other trackers). Equipment "Tracker ID" field allows system to merge equipment records with DataLake utilization and location data.

**Status:** ✅ Confirmed

> **Note (07.04):** WIALON integration is **blocked** pending cyber security assessment (Wialon is currently on the restricted list). IVMS and Vega trackers are in scope. — ⚠️

🆕 **GIS / Map integration (07.04):** Integration with MAPH/Atlas GIS tool via iframe + URL parameters; equipment ID synchronization; Azure SSO pass-through. — ⚠️ Pending cyber assessment approval (Светлана)

---

## 9 Open Conflicts Summary

| # | Conflict | BRD Position | Meeting Decision | Status |
|---|----------|-------------|-----------------|--------|
| ~~CONFLICT-01~~ | ~~FO editing booking period after confirmation~~ | ~~FR-048: editing allowed "before confirmation"~~ | ~~OQ-40: open~~ | ✅ **RESOLVED (10.04):** FO can edit before AND after confirmation. FR-048 updated. |
| CONFLICT-02 | Booking status auto-transition by datetime | FR-070/071/075: automatic status change by datetime | R-75 (09.04): manual close only | ❗ Sign-off from TCO required; update state machine |
| CONFLICT-03 | External fleet booking workflow | FR-051..057: full CC Owner → External FO flow | 08.04: on-demand BP = showcase only; full booking flow is in DCT | ✅ Resolved — FR-051..057 Obsolete |

---

## 10 Open Questions

| OQ ID | Description | Owner | Status |
|-------|-------------|-------|--------|
| OQ-29 | Dynamic filter matrix: equipment type → filter set | TCO | ⚠️ Partially defined (see 5.1.1); remaining types → OQ-NEW-F |
| ~~OQ-30 / R-48~~ | ~~One fleet → multiple Fleet Owners: rules and conflicts~~ | — | ✅ **PARTIALLY CLOSED (13.04):** Fleet has 2+ FOs (shift workers), displayed with shared email. Specific conflict resolution rules (e.g. who gets notification) TBD |
| OQ-36 | Aggregated Request status mapping (exact state machine) | Team | ⚠️ Open |
| OQ-37 | Booking Snapshot — exact field list | Team | ⚠️ Open |
| OQ-39 | Split logic | — | ✅ Closed |
| OQ-40 | FO editing booking period after confirmation | — | ✅ **Closed (10.04)** |
| OQ-41 | UI details for Unwheeled flow / manual close fields | Team | ⚠️ Open |
| **OQ-NEW-A** | **Concrete field list for equipment cards per equipment type** | TCO Fleet Owners | ⚠️ **Partially closed (13.04):** Loader and Crane-Manipulator defined. Remaining: Generator, Compressor, Dump Truck, Frac Tank, Trailer, Truck — Action Асылбек |
| **OQ-NEW-B** | **Manage Request / Equipment Template scope** — baseline fields first; custom attributes capability → backlog | TCO | ⚠️ Open — baseline in progress |
| **OQ-NEW-C** | **Service Work Processor role definition** — is this a user role (graphist/scheduler) or system component? Trigger status in JDE? | TCO (Аяжан, Dias / BRD author) | ⚠️ Open — hypothesis updated; meeting with Аяжан required |
| ~~OQ-NEW-D~~ | ~~Shared with Conditions vs. Shared without Conditions — is this distinction still meaningful?~~ | — | ✅ **CLOSED (13.04):** Difference = Justification field required. FO recall is independent mechanism. Both types confirmed. |
| **OQ-NEW-E** | **Delegation of FO privileges** — must be approved by TCO Cybersecurity department before implementation | TCO Cybersecurity / Project Team | ⚠️ Open — pending sign-off |
| **OQ-NEW-F** | **Equipment type parameters not yet covered:** Vacuum Truck, and other types from master fleet file | TCO (Асылбек) | ⚠️ Open — Action: Асылбек prepares parameter table for all remaining types |

---

## 11 Requirements Not in Original BRD (New from Meetings)

| New ID | Summary | Source | Status |
|--------|---------|--------|--------|
| FR-NEW-01 | All time parameters configurable via Admin Panel | 03.04 | ✅ |
| FR-NEW-02 | FO delegation to any TCO employee with auto-revoke | 08.04 | ⚠️ Cybersec pending |
| FR-NEW-03 | Admin configures FO timeout thresholds | 03.04 | ✅ |
| FR-NEW-04 | Admin authorizes users to book Assigned equipment | 08.04 | ✅ |
| FR-NEW-05 | FO uploads photos; Requestor sees photo | 08.04 | ✅ |
| FR-NEW-06 | Repair status from JDE/DataLake displayed in search | 08.04 | ⚠️ |
| FR-NEW-07 | Stationary HDE excluded from Requestor search | 08.04 | ✅ |
| FR-NEW-08 | Assigned equipment: visible to all, bookable by authorized only | 08.04 | ✅ |
| FR-NEW-09 | Shared with Conditions has visual color marker | 03.04 | ✅ |
| FR-NEW-10 | On-demand only; no weekly schedules | 03.04 | ✅ |
| FR-NEW-11 | Priority P1–P4 selected by Requestor | 03.04 | ✅ |
| FR-NEW-12 | Work Order mandatory; Location replaces it for SCM | 08.04 | ✅ |
| FR-NEW-13 | "Work Description" mandatory (~50 chars) | 08.04 | ✅ |
| FR-NEW-14 | "Comments" optional | 08.04 | ✅ |
| FR-NEW-15 | Single form → auto-split into atomic bookings via WO | 09.04 | ✅ |
| FR-NEW-16 | Partial confirmation per RequestItem | 03.04 | ✅ |
| FR-NEW-17 | Aggregated Request status auto-calculated | 09.04 arch | ⚠️ |
| FR-NEW-18 | Booking start = mobilization start | 03.04 | ✅ |
| FR-NEW-19 | No mandatory buffer between bookings | 09.04 | ✅ |
| FR-NEW-20 | Unwheeled equipment warning notification in UI | 07.04 | 🟡 |
| FR-NEW-21 | FO timeout: 24h reminder / 48h escalation per RequestItem | 03.04, 06.04 | ✅ |
| FR-NEW-22 | FO must state reason for decline if equipment available | 06.04 | ✅ |
| FR-NEW-23 | Reminder to FO of specific equipment only | 07.04 | ✅ |
| FR-NEW-24 | FO "Mobilization started" button | 09.04 | ✅ |
| FR-NEW-25 | BP showcase: view-only catalog by type | 08.04 | ✅ |
| FR-NEW-26 | BP prices not displayed | 08.04 | ✅ |
| FR-NEW-27 | BP self-populates catalog cards | 08.04 | ✅ |
| FR-NEW-28 | "Go to BP catalog" button after FO timeout | 08.04 | ✅ |
| FR-NEW-29 | Three-step Unwheeled flow with Transportation role | 09.04 | ✅ |
| FR-NEW-30 | FO proposes nearest date if transport unavailable | 03.04 | ⚠️ |
| FR-NEW-31 | Early close button for Requestor/FO | 07.04 | ⚠️ |
| FR-NEW-32 | Manual close only (no auto-transition by datetime) | 09.04 | ✅ |
| FR-NEW-33 | Actual start/end time entered at close for utilization | 09.04 | ✅ |
| FR-NEW-34 | FO reminder 24h / escalation 48h | 03.04, 06.04 | ✅ |
| FR-NEW-35 | Notification to Transportation Responsible for Unwheeled | 09.04 | ✅ |
| FR-NEW-36 | FO notified of Transportation Responsible decision | 09.04 | ✅ |
| FR-NEW-37 | Audit trail on demand | 07.04 | ✅ |
| FR-NEW-38 | Search: TCO number + model mandatory; госномер if present | 08.04, **13.04** | ✅ |
| FR-NEW-39 | Dynamic search filters by equipment type | 08.04 | ⚠️ Partial |
| **FR-NEW-40** | **Booking calendar in equipment card (FO view)** | **13.04** | ✅ |
| **FR-NEW-41** | **Multiple trackers per equipment unit** | **13.04** | ✅ |
| **FR-NEW-42** | **Shared team email per fleet (Admin-managed)** | **13.04** | ✅ |
| **FR-NEW-43** | **Notifications delivered to fleet shared team email** | **13.04** | ✅ |
