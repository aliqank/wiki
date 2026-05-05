# HDV/HDE Booking Tool — Risk Register

**Version:** 1.0  
**Date:** 2026-04-10  
**Based on:** BRD v2 (2026-04-10), Requirements Registry v5, Gap Analysis, Meeting notes 02.04–09.04  
**Scope:** Phase 1 + Phase 2 risks identified

---

## Risk Scoring Matrix

| Probability \ Impact | Low | Medium | High |
|----------------------|-----|--------|------|
| **High** | Medium | High | **Critical** |
| **Medium** | Low | Medium | High |
| **Low** | Low | Low | Medium |

**Probability:** H = likely to occur; M = possible; L = unlikely  
**Impact:** H = blocks delivery or causes major rework; M = requires significant effort to resolve; L = manageable within normal work  
**Phase:** P1 = affects Phase 1 delivery; P2 = affects Phase 2 delivery; P1→P2 = decision needed in P1 for P2 to succeed

---

## Risk Categories

| Code | Category |
|------|----------|
| REQ | Requirements & Scope |
| INT | Integration |
| ARCH | Architecture & Technical |
| ORG | Organizational |
| PHASE | Phase Boundary |

---

## Risk Register

### Category: REQ — Requirements & Scope

| ID | Risk | Phase | Prob | Impact | Level | Mitigation | Owner | Status |
|----|------|-------|------|--------|-------|-----------|-------|--------|
| RSK-01 | **Unresolved conflict: booking closure trigger.** BRD FR-070/071/075 require automatic status transitions by datetime; meeting decision R-75 requires manual close only. State machine implementation cannot start until resolved. No sign-off from TCO. | P1 | H | H | 🔴 Critical | Escalate to TCO for formal sign-off at next meeting (10.04). Document decision as explicit BRD deviation. | BA | Open |
| RSK-02 | **Unresolved conflict: FO editing period after confirmation (OQ-40).** BRD FR-048 implies editing allowed only before confirmation. Meeting question: can FO edit after? Impacts booking state machine and UI logic. | P1 | H | H | 🔴 Critical | Raise with TCO at 10.04 meeting. If allowed — requires additional state transitions and notification logic. | BA | Open |
| RSK-03 | **39 new requirements (FR-NEW series) not formally signed off by TCO.** Identified during meetings but never reviewed as a consolidated list. Risk of scope creep or late rejection of implemented features. | P1 | M | H | 🟠 High | Present FR-NEW list to TCO for formal sign-off. Prioritize critical-path items (mobilization flow, Unwheeled, delegation). | BA | Open |
| RSK-04 | **Notifications block near-zero coverage (FR-076..090, 15 requirements).** Notification logic, channels (email / in-app), and triggers are undefined in requirements registry. Notifications are tightly coupled with state machine and approval flow. | P1 | H | H | 🔴 Critical | Conduct dedicated session to specify all notification triggers, recipients, and channels before development begins. | BA | Open |
| RSK-05 | **Admin Panel requirements largely absent from registry** (FR-005..016, ~12 requirements). Fleet management, AAD linking, template management, equipment CRUD — none formalized. Blocks Admin functional testing and delivery. | P1 | H | M | 🟠 High | Add Admin Panel block to upcoming requirements sessions. Prioritize: fleet creation, AAD group linking, equipment templates. | BA | Open |
| RSK-06 | **Service Work Processor (SWP) full workflow not specified.** Only auto-import from JDE (R-07) is documented. SWP role actions, SWR submission flow, backward JDE status sync, and SWR-grouped view (FR-098) are missing. | P1 | M | H | 🟠 High | Schedule dedicated session with Maintenance team on SWP workflow. Clarify JDE trigger statuses (Section 8.1 TBD). | BA | Open |
| RSK-07 | **Booking state machine incomplete.** BRD defines 14 status transitions (FR-062..075); registry covers only aggregated Request status (R-69) and manual close (R-75). Conflict between auto-transitions (BRD) and manual close (R-75) unsolved. | P1 | H | H | 🔴 Critical | Map all 14 transitions explicitly. Resolve CONFLICT-02 (RSK-01) first. Create state diagrams for Booking and Request separately. | BA + Dev | Open |
| RSK-08 | **Revoke / Terminate / Extend lifecycle operations not in registry** (BRD FR-058..061). These are core booking lifecycle actions. Extend architecture (booking_extensions) also has unresolved design question (OQ-34). | P1 | M | H | 🟠 High | Add FR-058..061 to registry as confirmed requirements. Clarify partial extension (OQ-34). | BA | Open |
| RSK-09 | **Reporting requirements not detailed** (FR-091..098). Who sees what, filters, export formats — undefined. Risk of delivering reports that don't meet user needs. | P1 | M | M | 🟡 Medium | Conduct brief requirements session on reports with Logistics (Admin) and FO representatives. | BA | Open |
| RSK-10 | **Filter matrix (equipment type → search filters) not agreed** (OQ-29). Dynamic search filters per equipment type require a matrix (~22 types) to be confirmed by TCO. Blocks UI development for search screen. | P1 | H | M | 🟠 High | Waiting on TCO (Асылбек, Турсынбек, Ерлан). Escalate if not received by 14.04. Set a deadline for input. | BA | Open |
| RSK-11 | **Multi-FO in one request design not confirmed** (OQ-30). R-26 states it should be possible; R-48 states multiple FOs can manage one fleet. Architectural and UX implications are significant (status display, notification routing). | P1 | M | M | 🟡 Medium | Clarify with TCO: is multi-FO in single request required for Phase 1? If yes, confirm data model. | BA | Open |
| RSK-12 | **Linked / bundled booking logic not resolved** (OQ-39). If equipment items within one request are interdependent (e.g., forklift + compressor as inseparable pair), system needs special handling when one is declined. | P1 | L | M | 🟢 Low | Clarify with business whether linked-package logic is required. If yes — significant scope addition. | BA + TCO | Open |
| RSK-13 | **Equipment attributes per type not defined** (OQ-32). Fields shown in FO equipment card depend on type. Without confirmed attribute lists, data model and UI cannot be finalized for Phase 1. | P1 | M | M | 🟡 Medium | Awaiting TCO (Асылбек). Work through 2–3 key types first (forklift, crane, compressor) to unblock development. | BA | Open |
| RSK-14 | **Reporting and compliance requirements not translated to NFR.** BRD Section 7 lists 99.5% uptime, performance, scalability — none formalized in requirements registry. Risk of scope misalignment with Dev. | P1 | M | M | 🟡 Medium | Add NFR section to registry. Confirm performance targets and uptime SLA with TCO. | BA | Open |

---

### Category: INT — Integrations

| ID | Risk | Phase | Prob | Impact | Level | Mitigation | Owner | Status |
|----|------|-------|------|--------|-------|-----------|-------|--------|
| RSK-15 | **JDE E1 trigger status for SWR import is TBD** (BRD Section 8.1). The specific Work Order status that triggers Service Work Request creation in HDV/HDE is undefined. Blocks development of the entire SWP workflow. | P1 | H | H | 🔴 Critical | Resolve with TCO + JDE team. Required for R-07/R-30 implementation. Add to 10.04 agenda. | BA + Dev | Open |
| RSK-16 | **JDE E1 backward integration not detailed.** BRD requires bidirectional sync: HDV/HDE status changes must update JDE Step status. API contract undefined; no dev estimate possible. | P1 | M | H | 🟠 High | Scope a technical session with JDE integration team. Define API contract (fields, statuses, direction). | Dev + BA | Open |
| RSK-17 | **WIALON tracker integration blocked.** Wialon is on the cyber security restricted list (ACR assessment 07.04). Utilization dashboard and map functionality affected if block is permanent. IVMS and Vega in scope as fallback. | P1 | M | M | 🟡 Medium | Track Wialon cyber assessment status. Design DataLake integration to be tracker-agnostic so IVMS/Vega work now and Wialon can be added later. | Dev + Cyber | Open |
| RSK-18 | **GIS/Atlas (MAPH) integration pending security approval** (07.04 meeting). Approved architecture: iframe + URL params, Azure SSO pass-through. Waiting for Светлана's approval. Blocks map screen development. | P1 | M | M | 🟡 Medium | Follow up with MAPH team on approval timeline. Design UI map container to be stub-replaceable. | BA | Open |
| RSK-19 | **PSWS integration needed in Phase 1 for email notifications.** BRD Section 8.3 requires PSWS to identify preferred email addresses. Even if CC Owner flow is Phase 2, email lookup is needed for internal FO notifications. | P1 | M | H | 🟠 High | Clarify with TCO: is PSWS integration in scope for Phase 1? Minimum viable: email lookup endpoint. | BA + Dev | Open |
| RSK-20 | **DataLake / PI integration details undefined** (BRD Section 8.5). Tracker data sources (IVMS, WIALON, Vega), data format, refresh rate, and Tracker ID matching logic are unspecified. | P2 | L | M | 🟢 Low | Defer detailed spec to Phase 2 planning. Ensure equipment model has Tracker ID fields (already in BRD 5.1). | Dev | Open |
| RSK-21 | **DCT RWA/Change Order integration unspecced for Phase 2.** API is POST, sync vs async TBD (BRD Section 8.2.1). No API contract defined. Phase 2 delivery timeline depends on DCT team availability. | P2 | M | M | 🟡 Medium (P2) | Begin API contract discussion with DCT team in parallel with Phase 1. Avoid Phase 2 last-minute integration crunch. | Dev + DCT | Open |

---

### Category: ARCH — Architecture & Technical

| ID | Risk | Phase | Prob | Impact | Level | Mitigation | Owner | Status |
|----|------|-------|------|--------|-------|-----------|-------|--------|
| RSK-22 | **Complex two-level state machine** (Request + RequestItem/Booking). Order/OrderLine pattern confirmed (R-68), but aggregated status logic (OQ-36) and conflict with auto-transitions (RSK-01) mean state machine design is unstable. Rework risk is high if decisions change mid-dev. | P1 | H | H | 🔴 Critical | Freeze state machine design only after CONFLICT-02 (RSK-01) and OQ-36 are resolved. Produce formal state diagrams before sprint start. | Dev + BA | Open |
| RSK-23 | **Unwheeled three-step flow adds architectural complexity.** FR-NEW-29 introduces a 4-actor sub-workflow (Requestor → FO → Transportation role → FO again) with its own notification chain and state transitions. Not in original BRD — no prior validation. | P1 | M | M | 🟡 Medium | Model this as a sub-process (or sub-state) within internal booking approval. Confirm with TCO that Transportation role sign-off is mandatory for all Unwheeled equipment types. | Dev + BA | Open |
| RSK-24 | **Booking snapshot design pending** (OQ-37). If equipment attributes change after booking confirmation, historical booking data may become unreadable. Snapshot requires additional storage model decisions. | P1 | L | M | 🟢 Low | Confirm with Dev team: is JSON snapshot in Booking record sufficient? Low cost to implement early; high cost to retrofit. Recommend implementing in Phase 1. | Dev | Open |
| RSK-25 | **FO delegation mechanism is non-trivial.** FR-NEW-02 requires: arbitrary TCO employee as delegate, configurable scope, date-bounded auto-revoke, without IT involvement. AAD-based access model may not support this natively. | P1 | M | M | 🟡 Medium | Assess if AAD group membership can support delegation, or if a custom delegation table is needed. Define delegation scope limits early. | Dev + BA | Open |
| RSK-26 | **Dynamic filter matrix adds frontend complexity** (FR-NEW-39). 22+ equipment types with different filter sets. If matrix is confirmed late (OQ-29), UI development is blocked or must be reworked. | P1 | H | M | 🟠 High | Define a default minimal filter set to unblock development. Apply final matrix when TCO confirms (target: 14.04). | Dev + BA | Open |

---

### Category: ORG — Organizational

| ID | Risk | Phase | Prob | Impact | Level | Mitigation | Owner | Status |
|----|------|-------|------|--------|-------|-----------|-------|--------|
| RSK-27 | **No formal Process Owner for the system.** BRD Business Objective lists "Process ownership by Logistics" but no named individual is assigned as decision authority. Multiple open questions (OQ-29, OQ-30, OQ-36, OQ-40) require business sign-off; unclear who owns them. | P1 | M | H | 🟠 High | Confirm Process Owner (Logistics representative) and their decision-making authority. All OQ sign-offs must route through this person. | PM + TCO | Open |
| RSK-28 | **Key SMEs have limited availability.** Матрица фильтров (OQ-29) awaiting Асылбек/Турсынбек/Ерлан. Master Fleet File fields (OQ-26/27) awaiting Гульфат. Delays in input from SMEs directly block UI/data model. | P1 | M | M | 🟡 Medium | Set explicit deadlines for SME input. Escalate to PM if missed. Identify backup contacts per topic. | BA | Open |
| RSK-29 | **Transportation Responsible role (Heavy Ops) not integrated into sign-off process.** FR-NEW-29 introduces a new system role for Ерлан's team; they have not formally reviewed or confirmed the Unwheeled flow. | P1 | M | M | 🟡 Medium | Schedule a focused session with Heavy Ops team to validate three-step Unwheeled flow before development starts. | BA | Open |
| RSK-30 | **External BP users and account management not scoped for Phase 2.** FR-007 (Admin invites BP, manages accounts) and BP showcase catalog population require a BP-side onboarding process that has not been designed. | P2 | M | M | 🟡 Medium (P2) | Include BP onboarding process design in Phase 2 planning scope. Identify TCO contact owner for BP relations. | BA + PM | Open |

---

### Category: PHASE — Phase Boundary

| ID | Risk | Phase | Prob | Impact | Level | Mitigation | Owner | Status |
|----|------|-------|------|--------|-------|-----------|-------|--------|
| RSK-31 | **Phase 1 / Phase 2 boundary not formally documented.** The decision to move on-demand BP booking (FR-034..037, FR-051..057, DCT RWA) to Phase 2 was made in 08.04 meeting but not captured in any signed-off scoping document. Risk: disagreement on what is expected in Phase 1 delivery. | P1→P2 | H | H | 🔴 Critical | Create a formal Phase 1/Phase 2 scope split document. Get sign-off from TCO before development sprint starts. Include in BRD v2 as a signed appendix. | BA + PM | Open |
| RSK-32 | **CC Owner / DOA integration — architecture decisions needed in Phase 1.** Even though CC Owner flow is Phase 2, the data model (cost_center, CC owner linkage in Booking) must be designed in Phase 1 to avoid breaking changes later. PSWS integration may also be partially needed in Phase 1 (see RSK-19). | P1→P2 | M | M | 🟡 Medium | Include CC data fields in Phase 1 data model as placeholders. Agree on PSWS integration scope for Phase 1 (email only vs. full CC lookup). | Dev + BA | Open |
| RSK-33 | **DCT / RWA integration data model must be planned in Phase 1.** Booking record must contain fields needed for future RWA creation (justification, cost center, contractor ID). If not included in Phase 1 schema, Phase 2 will require migration. | P1→P2 | M | M | 🟡 Medium | Identify RWA-required fields from DCT API spec. Add as nullable fields in Phase 1 booking schema. | Dev + BA | Open |
| RSK-34 | **DataLake / Utilization dashboard (FR-099..100) completely unspecced.** Visualization tool selection (embedded BI vs custom), DataLake connection method, and KPI definitions are undefined. If dashboard is expected in Phase 1, it is a significant undiscovered scope item. | P1 / P2 | M | H | 🟠 High | Confirm with TCO: is utilization dashboard in Phase 1 or Phase 2 scope? If Phase 1 — immediately begin requirements gathering for this block. | BA + PM | Open |

---

## Summary Dashboard

### By Risk Level

| Level | Count | Risk IDs |
|-------|-------|---------|
| 🔴 Critical | 5 | RSK-01, RSK-02, RSK-07, RSK-15, RSK-22, RSK-31 |
| 🟠 High | 9 | RSK-03, RSK-04, RSK-05, RSK-06, RSK-08, RSK-10, RSK-16, RSK-19, RSK-26, RSK-27, RSK-34 |
| 🟡 Medium | 13 | RSK-09, RSK-11, RSK-13, RSK-14, RSK-17, RSK-18, RSK-21, RSK-23, RSK-25, RSK-28, RSK-29, RSK-30, RSK-32, RSK-33 |
| 🟢 Low | 3 | RSK-12, RSK-20, RSK-24 |

### By Phase

| Phase | Critical | High | Medium | Low |
|-------|---------|------|--------|-----|
| Phase 1 | RSK-01, RSK-02, RSK-07, RSK-15, RSK-22, RSK-31 | RSK-03, RSK-04, RSK-05, RSK-06, RSK-08, RSK-10, RSK-16, RSK-19, RSK-26, RSK-27, RSK-34 | RSK-09, RSK-11, RSK-13, RSK-14, RSK-17, RSK-18, RSK-23, RSK-25, RSK-26, RSK-28, RSK-29, RSK-32, RSK-33 | RSK-12, RSK-24 |
| Phase 2 | — | RSK-21, RSK-34 | RSK-20, RSK-30 | — |
| Phase 1 → Phase 2 | RSK-31 | — | RSK-32, RSK-33 | — |

---

## Top Priority Actions (Phase 1 — Immediate)

| # | Action | Linked Risks | Target |
|---|--------|-------------|--------|
| 1 | Get TCO sign-off: **manual close vs. auto datetime transitions** | RSK-01, RSK-07, RSK-22 | 10.04 meeting |
| 2 | Get TCO sign-off: **FO editing booking period after confirmation** (OQ-40) | RSK-02 | 10.04 meeting |
| 3 | Create and sign off **Phase 1 / Phase 2 scope split document** | RSK-31, RSK-33, RSK-34 | This week |
| 4 | Resolve **JDE E1 trigger status** for SWR import | RSK-15, RSK-06 | Next technical session |
| 5 | Conduct dedicated **Notifications requirements session** | RSK-04 | Before dev sprint |
| 6 | Receive and validate **filter matrix** from TCO (OQ-29) | RSK-10, RSK-26 | Deadline 14.04 |
| 7 | Confirm **PSWS integration scope** for Phase 1 (email only vs. full) | RSK-19, RSK-32 | Before architecture freeze |
| 8 | Confirm **utilization dashboard scope** (Phase 1 or Phase 2?) | RSK-34 | This week |
| 9 | Present **FR-NEW list** (39 requirements) to TCO for formal sign-off | RSK-03 | Next stakeholder meeting |
| 10 | Confirm **Process Owner** (named individual from Logistics) | RSK-27 | Immediately |
