# Domain context — TCO HDV/HDE Booking Tool

## Project
**HDV/HDE Booking Tool** for the client **TCO (Tengizchevroil)**.
Domain: booking of heavy equipment and machinery at an oil field.
User role: **Business Analyst / System Analyst**.

## Environment
TCO enterprise environment: **Azure AAD, JDE E1, PSWS, DataLake, GIS (MAPH/Atlas)**.

## Scope baseline
- In Phase 1, only **TCO Owned** and **Long-term rented** units are booked through the system.
- **On-demand BP (Showcase)** does not participate in the booking workflow — shown as a catalog/showcase only.
- The external approval chain from the original BRD is not the target model for the current scope.

## Key entities
- **Equipment** — a single HDV/HDE unit
- **Fleet** — a pool of equipment
- **Request** — a booking request combining multiple booking items
- **Booking** — a booking for a specific equipment unit
- **Request Item** — an element of a request, processed as an independent booking

## Equipment classification

### Ownership
- **TCO Owned** — TCO-owned equipment, approval chain: `Requestor → Fleet Owner`
- **Long-term rented** — long-term rented equipment, approval chain: `Requestor → Fleet Owner → FleetOwners' Supervisor`
- **On-demand BP (Showcase)** — catalog only, no booking in the system

### Usage Status
- **Assigned**
- **Shared without Conditions**
- **Shared with Conditions**

### Mobility
- **Self-propelled**
- **Unwheeled**
- **Stationary**

## Roles
| Role | Description |
|------|-------------|
| Requestor | Submits equipment requests |
| Service Work Processor (SWP) | Works with JDE-sourced Service Work Requests |
| Fleet Owner (FO) | Manages the fleet and approves bookings |
| FleetOwners' Supervisor | Final approval for Long-term rented bookings after FO |
| Transportation Responsible | Handles transportation of unwheeled equipment |
| Admin | Configures the system and reference data |

## Approval chains
- **TCO Owned:** `Requestor → Fleet Owner → Confirmed`
- **Long-term rented:** `Requestor + Justification → Fleet Owner → Confirmed by FO → FleetOwners' Supervisor → Confirmed/Declined`
- **Unwheeled:** separate transportation scenario involving the `Transportation Responsible` role

## Key business rules
- Booking priority is internal TCO fleet first.
- **Justification** field is mandatory for **Long-term rented** at the booking item level.
- Access to booking **Assigned** equipment may be restricted via Admin settings.
- FO and Supervisor approval timeouts must be configurable via Admin Panel.
- Booking horizon, maximum duration, and other time parameters must not be hardcoded.
- Booking closure is manual only — no automatic completion by date/time.

## Integrations
- **Azure AAD** — authentication and role/group provisioning
- **JDE E1** — Work Orders / Service Work Requests
- **PSWS** — user contact data
- **DataLake / Cosmos DB via API** — telemetry, usage rate, GIS-related data
- **GIS (MAPH/Atlas)** — map via iframe integration

## Sources of truth
- **Final BRD for development:** `wiki/brd/BRD.md`
- **Wiki navigation:** `wiki/navigation.md`
- **Draft and working notes:** `working_docs/`
- **Prepared analytical results:** `results/`
- **Materials navigator:** `materials.md`

## Artifact publishing note
- Draft work is done in `working_docs/`.
- After analytical work, results are formatted as `.md` files in `results/`.
- Materials are moved to `wiki/` only on an explicit user command.
