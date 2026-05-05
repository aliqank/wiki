# HDV/HDE Booking Tool — Open Questions

**Version:** 1.0  
**Date:** 2026-04-21  
**Scope:** All currently open questions across the project  
**Prepared by:** Telman Nurzhanov (SA)

> This document consolidates all unresolved questions and pending decisions as of BRD v8 (2026-04-21). For resolved questions, refer to the BRD changelog and Section 9 (Resolved Conflicts).

---

## Summary

| OQ ID | Description | Owner | Priority | Due |
|-------|-------------|-------|----------|-----|
| OQ-29 | Dynamic filter matrix: remaining equipment type → filter set | TCO (Asylbek) | High | ASAP |
| OQ-36 | Aggregated Request status state machine | Team | Medium | TBD |
| OQ-37 | Booking Snapshot — exact field list | Team | Low | TBD |
| OQ-41 | UI details for Unwheeled flow / manual close fields | Team | Medium | TBD |
| OQ-NEW-A | Equipment card field list per equipment type (remaining types) | TCO (Asylbek) | High | ASAP |
| OQ-NEW-B | Request / Equipment Template scope | TCO | Medium | TBD |
| OQ-NEW-C | Service Work Processor role and JDE trigger status | TCO (Аяжан, Dias / SA) | High | TBD |
| OQ-NEW-E | FO delegation — cybersecurity sign-off | TCO Cybersecurity | High | TBD |
| OQ-NEW-F | Equipment type parameters: Vacuum Truck and others | TCO (Asylbek) | High | ASAP |
| OQ-NEW-H | Utilization booking threshold | TCO | Medium | TBD |
| OQ-NEW-L | Final equipment categories and subcategories | Asylbek | High | ASAP |
| OQ-NEW-M | Final list of Work Centers | Asylbek | High | ASAP |
| OQ-NEW-N | Equipment type fields and data examples | TCO (Asylbek / colleagues) | High | ASAP |
| OQ-NEW-O | GIS map requirements alignment with GIS team | Duman / Team | High | 23.04 |
| OQ-NEW-P | Assigned equipment: which types to exclude from Requestor filtering | TCO (Asylbek) | Medium | TBD |

---

## Detailed Descriptions

---

### OQ-29 — Dynamic filter matrix: remaining equipment types

**Description:** The filter set for each equipment type (what parameters are shown as search filters for Requestor) is partially defined in BRD Section 5.1.1. The matrix is not yet complete for all equipment types present in the TCO fleet.

**Current state:** Loader, Crane-Manipulator, Mobile Generator, Motor Compressor, Dump Truck, Frac Tank, Cargo Trailer, Semi-truck — defined (preliminarily). Vacuum Truck and other types — not yet defined (see OQ-NEW-F).

**Impact:** Affects FR-NEW-39 (dynamic search filters). Development of equipment search cannot be finalized without this information.

**Owner:** TCO (Asylbek)  
**Action:** Provide complete list of equipment categories with corresponding filter parameters.

---

### OQ-36 — Aggregated Request status state machine

**Description:** The exact rules for calculating the aggregated Request status from individual Booking statuses are not formally defined. For example: if one Booking is Confirmed and another is Declined, what is the Request status?

**Current state:** Known transitions:
- Request → In Progress: when at least one Booking (excl. Declined) is In Progress
- Request → Completed: when all active (non-Declined) Bookings are Closed (FR-NEW-44)

**Pending:** Full state machine covering partial confirmation, mixed statuses (some Confirmed + some Declined + some Pending), and edge cases.

**Impact:** Affects FR-026, FR-NEW-17. Required before UI design of Request status display.

**Owner:** Team  
**Action:** Formalize state machine diagram; align with design.

---

### OQ-37 — Booking Snapshot field list

**Description:** At booking confirmation, the system saves a snapshot of key equipment attributes (to preserve the state of equipment data at time of booking, in case the equipment card is later updated).

**Pending:** Exact list of fields to include in the snapshot.

**Impact:** Affects data model and storage design.

**Owner:** Team  
**Action:** Define list of equipment fields to be snapshotted at confirmation.

---

### OQ-41 — UI details for Unwheeled flow and manual close

**Description:** The three-step Unwheeled flow (Requestor → FO → Transportation Responsible → FO final confirmation) and the manual close flow ("Close" button) require detailed UI and field specification.

**Pending:**
- Fields shown to Requestor in Unwheeled notification
- FO decision form for Unwheeled (date proposals if transport unavailable)
- Fields for manual close (actual start/end time, notes)
- Whether "Close" and "Close early" are separate actions

**Impact:** Affects FR-NEW-29, FR-NEW-30, FR-NEW-31, FR-NEW-32, FR-NEW-33.

**Owner:** Team (SA + Design)  
**Action:** Detail UI flows for Unwheeled booking and manual close. Align with Figma prototype.

---

### OQ-NEW-A — Equipment card field list per equipment type (remaining)

**Description:** Specific parameters for each equipment type must be confirmed by Fleet Owners. Section 5.1.1 of the BRD contains a preliminary set for several types. The list is not complete.

**Current state:** Partially closed 13.04 for: Loader, Crane-Manipulator, Mobile Generator, Motor Compressor, Dump Truck, Frac Tank, Cargo Trailer, Semi-truck. Remaining types outstanding.

**Impact:** Required for equipment card design, search filter implementation, and equipment onboarding.

**Owner:** TCO Fleet Owners (Asylbek as coordinator)  
**Action:** Asylbek to update and finalize the equipment categories / subcategories table. **Due: ASAP.**

---

### OQ-NEW-B — Request / Equipment Template scope

**Description:** Admin Panel supports managing templates for Request, Equipment, and Booking entities. The baseline set of fields is defined in the BRD. The question is whether Fleet Owners / Admin should be able to add custom attributes beyond the baseline.

**Current position:** Baseline first; custom attributes deferred to Backlog.

**Pending decision:** Confirm that no custom attributes are required in Phase 1. If any custom attributes are needed for go-live, they must be specified now.

**Impact:** Affects FR-010, FR-011, FR-012. Custom attributes significantly expand development scope.

**Owner:** TCO  
**Action:** TCO to confirm that baseline template (as defined in BRD) is sufficient for Phase 1 launch.

---

### OQ-NEW-C — Service Work Processor role definition and JDE trigger status

**Description:** The Service Work Processor (SWP) role is partially clarified. Based on meeting discussions (13.04), SWP is likely graphists/schedulers who manage Work Order steps in JDE and process auto-generated drafts in the Booking Tool.

**Pending:**
1. Formal confirmation of the SWP role definition from TCO
2. Exact JDE WO step status that triggers auto-import to HDV/HDE (currently assumed: status 55)
3. Confirmation of the backward integration requirement: should changes in HDV/HDE update the step status in JDE?

**Impact:** Affects FR-028, FR-029, FR-073, FR-076, FR-098; JDE integration design; notification flows.

**Owner:** TCO (Аяжан — Maintenance Plan Consultant, TFM 2835; Dias; SA team)  
**Action:** Organize clarification meeting with Аяжан and JDE team. Confirm role, JDE trigger status, and backward sync requirement.

---

### OQ-NEW-E — FO delegation — cybersecurity sign-off

**Description:** Fleet Owner can delegate their privileges to any TCO employee for a specified period (with auto-revoke upon expiry). This feature requires cybersecurity review and formal sign-off from TCO Cybersecurity team.

**Current state:** Feature is designed and included in BRD (FR-NEW-02, Admin FR-014). Pending formal approval.

**Impact:** If not approved, delegation feature is removed from scope. Affects FR-NEW-02 and Admin delegation management.

**Owner:** TCO Cybersecurity  
**Action:** TCO to initiate cybersecurity review. SA to provide feature description for the review.

---

### OQ-NEW-F — Equipment type parameters: Vacuum Truck and other types

**Description:** Parameters and search filters for several equipment types are not yet defined, including Vacuum Truck (VTRK) and others not covered in Section 5.1.1.

**Impact:** Blocks completion of OQ-29 (dynamic filter matrix) and OQ-NEW-A.

**Owner:** TCO (Asylbek)  
**Action:** Asylbek to provide parameters for all missing equipment types. **Due: ASAP.**

---

### OQ-NEW-H — Utilization booking threshold

**Description:** If a piece of equipment is approaching or at 100% utilization (based on tracker data), should the system still allow a new booking to be submitted?

**Current state:** No decision made. System currently validates availability (no overlapping confirmed bookings) but does not check utilization levels before allowing a new booking.

**Options:**
1. Allow booking regardless of utilization — FO decides
2. Show a warning to Requestor if utilization is high
3. Block booking if utilization exceeds a threshold (configurable)

**Impact:** Affects FR-040 (availability validation logic) and UX of the booking form.

**Owner:** TCO  
**Action:** TCO to decide on the policy for booking submission when utilization is near 100%.

---

### OQ-NEW-L — Final equipment categories and subcategories

**Description:** The current categorization of equipment (classes, categories, subcategories) used as a basis for handbook entries in the Admin Panel is preliminary. The final gradation needs to be confirmed.

**Context:** Categories and subcategories are managed as handbook values via Admin Panel (FR-NEW-50). The final list directly affects equipment card templates, search filters, and reporting groupings.

**Owner:** Asylbek  
**Action:** Asylbek to provide updated and confirmed list of equipment categories and subcategories. **Due: ASAP.**

---

### OQ-NEW-M — Final list of Work Centers

**Description:** The Work Center codes listed in the BRD Glossary (Section 3) are preliminary and subject to correction. The final list is required for Admin Panel handbook configuration and for the WC selection in request creation.

**Current preliminary list:** LRG, MGT, BHOE, DTRK, FTRK, GDRV, HYDR, OILT, VTRK, CDECK, CRANE, FORKL, GMLOG, MLIFT, MSINS.

**Owner:** Asylbek  
**Action:** Asylbek to provide final confirmed list of Work Center codes and descriptions. **Due: ASAP.**

---

### OQ-NEW-N — Equipment type fields and data examples for all categories

**Description:** For each equipment type, the team needs examples of actual field values and data formats (e.g. typical load capacity ranges, engine hour targets, typical mileage targets). This is needed for:
- Validating filter designs
- Populating test data
- Confirming field formats and dropdown options

**Context:** Discussed in meeting 20.04. Asylbek and TCO colleagues to fill in the equipment types table with real-world examples.

**Owner:** TCO (Asylbek and respective Fleet Owners)  
**Action:** Fill in the equipment types table with data examples per category. **Due: ASAP.**

---

### OQ-NEW-O — GIS map requirements alignment with GIS team

**Description:** The GIS map is implemented by the GIS team (MAPH/Atlas) as an iframe embedded in the Booking Tool. The Booking Tool team provides requirements, identifiers, and URL parameters. The GIS team implements all map behavior.

**Confirmed in 20.04 meeting:**
- Speed and direction displayed for GSM trackers; not for LoRaWAN
- Fuel level: not displayed
- Historical route: 7 days (FR-NEW-67)
- Tooltip: all sensor data from last sync (FR-NEW-62)
- Map fully on GIS team side

**Pending:**
- Exact URL parameter interface (how equipment IDs and filter parameters are passed)
- SSO/authentication pass-through mechanism
- Tooltip rendering: does the Booking Tool pass data to GIS iframe, or does GIS team fetch it independently?
- Historical route implementation: date range selector UI
- Confirmation of all FR-NEW-61–FR-NEW-67 with GIS team
- Timeline for GIS team delivery

**Owner:** Duman (Booking Tool team — to organize call with GIS team)  
**Action:** Organize call with GIS team (MAPH/Atlas). **Due: 23.04.**

---

### OQ-NEW-P — Assigned equipment: which types to exclude from Requestor search

**Description:** Assigned equipment is visible to all users but bookable only by authorized users (FR-NEW-08). In the 20.04 meeting, it was raised that certain Assigned equipment types (e.g. fire trucks) may be irrelevant for Requestors entirely — they are not transferable and Requestors would never need to see or select them.

**Question:** Which specific Assigned equipment types (or categories) should be hidden from Requestor search entirely vs. shown as visible-but-not-bookable?

**Impact:** Affects FR-NEW-07 (Stationary HDE excluded), FR-NEW-08 (Assigned equipment visibility), and search filter design.

**Owner:** TCO (Asylbek — to update equipment categories table with Assigned visibility flag)  
**Action:** Asylbek to review Assigned equipment types and indicate which should be excluded from Requestor view. Coordinate with update to OQ-NEW-L (categories table).

---

## Action Items from Meeting 20.04.2026

| # | Action | Owner | Due |
|---|--------|-------|-----|
| 1 | Update file of equipment categories/subcategories (final gradation) | Asylbek | ASAP |
| 2 | Provide complete list of Work Centers | Asylbek | ASAP |
| 3 | Fill in equipment type data examples (equipment types table) | TCO (Asylbek / colleagues) | ASAP |
| 4 | Organize call with GIS team (MAPH/Atlas) | Duman | 23.04 |
| 5 | Add booking count to utilization dashboard (FR-NEW-63) | Evgeny (design) | — |
| 6 | Add equipment loading summary to FO approval window (FR-NEW-64) | Evgeny (design) | — |
| 7 | Review Figma prototype independently and send feedback | All TCO participants | — |

---

## Closed Questions Reference

The following questions were resolved prior to BRD v8. Details in BRD Section 9 and version changelogs.

| OQ ID | Description | Resolution | Closed |
|-------|-------------|------------|--------|
| OQ-30 / R-48 | One fleet → multiple FOs | Fleet has 2+ FOs with shared team email. Conflict resolution rules between FOs: TBD | 13.04 |
| OQ-NEW-D | Shared with Conditions vs. without | Differ only in mandatory Justification field. Recall is always possible regardless of status | 13.04 |
| OQ-NEW-G | Work Center — attribute of equipment or type of work? | WC = type of work (step attribute), not equipment attribute | 16.04 |
| OQ-NEW-I | Priority attribution — Request vs. WC step vs. equipment? | Priority = attribute of Work Order (Request). All WC steps share the same WO priority | 17.04 |
| OQ-NEW-J | Department in reports — Fleet Owner's or Equipment's? | Reports use Equipment's department assignment | 17.04 |
| OQ-NEW-K | Route tracking — display movement route on GIS map? | Historical route confirmed: 7 days, data from trackers via DataLake | 20.04 |
| CONFLICT-01 | FO editing booking period after confirmation | FO can edit before AND after confirmation; Requestor notified automatically | 10.04 |
| CONFLICT-02 | Booking status auto-transition by datetime | Manual close accepted; auto-transitions removed (FR-070, FR-071, FR-075 → Obsolete) | 14.04 |
| CONFLICT-03 | External fleet booking workflow | On-demand BP = showcase only; external booking FRs → Obsolete | 08.04 |
