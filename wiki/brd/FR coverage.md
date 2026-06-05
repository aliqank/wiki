# FR coverage

Источник: `wiki/brd/BRD.md`, раздел `6 Functional Requirements`.

Правило заполнения:
- `Покрыто = +`, если FR покрыт текущими Requestor use case-ами (`UC-REQ-01..05`, включая дочерние `UC-REQ-02.*`)
- `Покрыто = -`, если FR пока не покрыт use case-ами
- для всех FR с `+` проставлен `Спринт 2`
- для `FR-005..FR-010`, `FR-012..FR-015`, `FR-017..FR-021`, `FR-025` проставлен `Спринт 1`
- если FR относится к нескольким спринтам, указываются оба значения

## Спринты

| Спринт | Список FR |
|---|---|
| Спринт 1 | `FR-005`, `FR-006`, `FR-007`, `FR-008`, `FR-009`, `FR-010`, `FR-012`, `FR-013`, `FR-014`, `FR-015`, `FR-017`, `FR-018`, `FR-019`, `FR-020`, `FR-021`, `FR-025`, `FR-NEW-04`, `FR-NEW-05`, `FR-NEW-46`, `FR-NEW-49`, `FR-NEW-50`, `FR-NEW-53`, `FR-NEW-54`, `FR-NEW-55`, `FR-NEW-57`, `FR-NEW-58`, `FR-NEW-59`, `FR-NEW-60` |
| Спринт 2 | `FR-NEW-04`, `FR-NEW-50`, `FR-NEW-68`, `FR-NEW-08`, `FR-023`, `FR-024`, `FR-025`, `FR-026`, `FR-027`, `FR-030`, `FR-NEW-11`, `FR-NEW-12`, `FR-NEW-13`, `FR-NEW-14`, `FR-NEW-15`, `FR-NEW-16`, `FR-NEW-17`, `FR-NEW-51`, `FR-NEW-69`, `FR-NEW-71`, `FR-031`, `FR-032`, `FR-033`, `FR-038`, `FR-039`, `FR-040`, `FR-041`, `FR-042`, `FR-NEW-70`, `FR-058`, `FR-062`, `FR-068`, `FR-072`, `FR-091`, `FR-092`, `FR-NEW-37`, `FR-NEW-38`, `FR-NEW-39` |

| FR | Описание | Покрыто | Use case | Спринт |
|---|---|---|---|---|
| FR-001 | System supports role-based access control. Roles: Requestor, Service Work Processor, Fleet Owner, FleetOwners' Supervisor, Administrator, Transportation Responsible | - | — | — |
| FR-003 | Administrators, Service Work Processors, FleetOwners' Supervisors defined by AAD groups | - | — | — |
| FR-NEW-01 | All time-based parameters configurable via Admin Panel | - | — | — |
| FR-005 | Admin creates fleets with unique names | + | Реализация админ панели и стр. оборудования | Спринт 1 |
| FR-006 | Admin links internal fleet with AAD group | + | Реализация админ панели и стр. оборудования | Спринт 1 |
| FR-007 | Admin manages external fleet owners | + | Реализация админ панели и стр. оборудования | Спринт 1 |
| FR-008 | Admin updates existing fleets | + | Реализация админ панели и стр. оборудования | Спринт 1 |
| FR-009 | Admin deletes fleets | + | Реализация админ панели и стр. оборудования | Спринт 1 |
| FR-010 | Admin configures dynamic custom characteristics per equipment type via Admin Panel tool | + | Реализация админ панели и стр. оборудования | Спринт 1 |
| FR-012 | Admin manages booking template | + | Реализация админ панели и стр. оборудования | Спринт 1 |
| FR-013 | Admin creates equipment (internal fleet) | + | Реализация админ панели и стр. оборудования | Спринт 1 |
| FR-014 | Admin edits equipment (extended edit) | + | Реализация админ панели и стр. оборудования | Спринт 1 |
| FR-015 | Admin deletes equipment | + | Реализация админ панели и стр. оборудования | Спринт 1 |
| FR-016 | Admin accesses all dashboards and reports | - | — | — |
| FR-NEW-03 | Admin configures via Admin Panel: FO timeout, booking horizon, max duration, FleetOwners' Supervisor response timeout | - | — | — |
| FR-NEW-04 | Admin authorizes specific users to book Assigned equipment | + | Реализация админ панели и стр. оборудования; UC-REQ-02 - Создание новой заявки (Draft-first); UC-REQ-02.4 - Добавление техники в draft-заявку; UC-REQ-02.5 - Редактирование брони или замена техники в draft | Спринт 1, Спринт 2 |
| FR-NEW-42 | Admin sets and updates fleet shared team email | - | — | — |
| FR-NEW-50 | Admin manages all reference/handbook values via Admin Panel | + | Реализация админ панели и стр. оборудования; UC-REQ-02 - Создание новой заявки (Draft-first); UC-REQ-02.3 - Поиск техники для добавления в заявку | Спринт 1, Спринт 2 |
| FR-NEW-68 | System dynamically shows only type-specific characteristics in request form and equipment search filters | + | UC-REQ-02 - Создание новой заявки (Draft-first); UC-REQ-02.3 - Поиск техники для добавления в заявку | Спринт 2 |
| FR-NEW-74 | Admin configures FleetOwners' Supervisor response timeout via Admin Panel | - | — | — |
| FR-017 | Both internal and external FO can create equipment under own fleet | + | Реализация админ панели и стр. оборудования | Спринт 1 |
| FR-018 | FO must indicate fleet of equipment; only own fleets selectable | + | Реализация админ панели и стр. оборудования | Спринт 1 |
| FR-019 | FO can edit allowed equipment parameters; one user can own several fleets | + | Реализация админ панели и стр. оборудования | Спринт 1 |
| FR-020 | FO can delete equipment (soft delete) | + | Реализация админ панели и стр. оборудования | Спринт 1 |
| FR-021 | FO can freeze/unfreeze equipment for a period or indefinitely | + | Реализация админ панели и стр. оборудования | Спринт 1 |
| FR-022 | Requestor / SWP can submit feedback on equipment with confirmed booking | - | — | — |
| FR-NEW-05 | FO uploads multiple photos; Requestor sees them in request form | + | Реализация админ панели и стр. оборудования | Спринт 1 |
| FR-NEW-06 | Repair status from JDE -> DataLake displayed in search and equipment list | - | — | — |
| FR-NEW-07 | Stationary HDE excluded from Requestor search | - | — | — |
| FR-NEW-08 | Assigned equipment: visible to all users; bookable only by Admin-authorized users with mandatory justification; FO can approve or decline | + | UC-REQ-02 - Создание новой заявки (Draft-first); UC-REQ-02.4 - Добавление техники в draft-заявку; UC-REQ-02.5 - Редактирование брони или замена техники в draft; UC-REQ-02.6 - Отправка draft-заявки | Спринт 2 |
| FR-NEW-09 | Shared with Conditions has visual color marker | - | — | — |
| FR-NEW-40 | Equipment card (FO view) has booking calendar visual | - | — | — |
| FR-NEW-41 | Multiple trackers per unit; FO can add trackers from equipment card UI | - | — | — |
| FR-NEW-49 | Freeze fields on equipment card | + | Реализация админ панели и стр. оборудования | Спринт 1 |
| FR-023 | Requestor can create a Request | + | UC-REQ-02 - Создание новой заявки (Draft-first); UC-REQ-02.1 - Создание пустого draft заявки; UC-REQ-02.6 - Отправка draft-заявки | Спринт 2 |
| FR-024 | Request has unique ID and metadata | + | UC-REQ-01 - Просмотр моих заявок; UC-REQ-02 - Создание новой заявки (Draft-first); UC-REQ-02.1 - Создание пустого draft заявки | Спринт 2 |
| FR-025 | Requestor can view request details and status | + | UC-REQ-01 - Просмотр моих заявок; UC-REQ-03 - Отмена draft-заявки requestor-ом; UC-REQ-04 - Отзыв брони requestor-ом; UC-REQ-05 - Отзыв submitted-заявки requestor-ом; UC-COM-01 - Просмотр таймлайна брони | Спринт 1, Спринт 2 |
| FR-026 | Request statuses: Draft, Submitted, In Progress, Completed | + | UC-REQ-01 - Просмотр моих заявок; UC-REQ-02 - Создание новой заявки (Draft-first); UC-REQ-02.6 - Отправка draft-заявки | Спринт 2 |
| FR-027 | Requestor can edit/cancel draft before submission | + | UC-REQ-02 - Создание новой заявки (Draft-first); UC-REQ-02.1 - Создание пустого draft заявки; UC-REQ-02.2 - Заполнение и редактирование шапки заявки; UC-REQ-02.5 - Редактирование брони или замена техники в draft; UC-REQ-02.6 - Отправка draft-заявки; UC-REQ-02.7 - Отмена draft-брони; UC-REQ-03 - Отмена draft-заявки requestor-ом; UC-REQ-07 - Отмена draft-брони requestor-ом со страницы Мои заявки | Спринт 2 |
| FR-028 | Service Work Requests auto-created with Draft status; visible to SWP role only | - | — | — |
| FR-029 | Only SWP can manage Service Work Requests | - | — | — |
| FR-030 | System supports draft saving | + | UC-REQ-02 - Создание новой заявки (Draft-first); UC-REQ-02.1 - Создание пустого draft заявки | Спринт 2 |
| FR-NEW-10 | On-demand booking only; weekly schedule management out of scope | - | — | — |
| FR-NEW-11 | Requestor selects priority P1-P4 at request creation | + | UC-REQ-02 - Создание новой заявки (Draft-first); UC-REQ-02.2 - Заполнение и редактирование шапки заявки | Спринт 2 |
| FR-NEW-12 | WO mandatory for some departments; optional for Logistics; Location replaces WO for SCM Logistics | + | UC-REQ-02 - Создание новой заявки (Draft-first); UC-REQ-02.2 - Заполнение и редактирование шапки заявки | Спринт 2 |
| FR-NEW-13 | Work Description mandatory | + | UC-REQ-02 - Создание новой заявки (Draft-first); UC-REQ-02.2 - Заполнение и редактирование шапки заявки | Спринт 2 |
| FR-NEW-14 | Comments optional | + | UC-REQ-02 - Создание новой заявки (Draft-first); UC-REQ-02.2 - Заполнение и редактирование шапки заявки | Спринт 2 |
| FR-NEW-15 | Single form auto-splits into individual bookings per equipment item | + | UC-REQ-02 - Создание новой заявки (Draft-first); UC-REQ-02.4 - Добавление техники в draft-заявку | Спринт 2 |
| FR-NEW-16 | Partial confirmation: confirmed items proceed independently from declined | + | UC-REQ-02 - Создание новой заявки (Draft-first); UC-REQ-02.6 - Отправка draft-заявки | Спринт 2 |
| FR-NEW-17 | Aggregated Request status auto-calculated from RequestItem statuses | + | UC-REQ-02 - Создание новой заявки (Draft-first); UC-REQ-02.6 - Отправка draft-заявки; UC-REQ-04 - Отзыв брони requestor-ом; UC-REQ-05 - Отзыв submitted-заявки requestor-ом | Спринт 2 |
| FR-NEW-48 | Select WO from JDE block on Submit Request page; WO list from JDE E1 via direct API; auto-generate draft | - | — | — |
| FR-NEW-51 | Default Work Order / Default Work Center option | + | UC-REQ-02 - Создание новой заявки (Draft-first); UC-REQ-02.2 - Заполнение и редактирование шапки заявки | Спринт 2 |
| FR-NEW-64 | Equipment loading summary in FO approval window | - | — | — |
| FR-NEW-69 | Priority field displays tooltips/hints for each priority level | + | UC-REQ-02 - Создание новой заявки (Draft-first); UC-REQ-02.2 - Заполнение и редактирование шапки заявки | Спринт 2 |
| FR-NEW-71 | Long-term rented equipment item requires mandatory Justification | + | UC-REQ-02 - Создание новой заявки (Draft-first); UC-REQ-02.4 - Добавление техники в draft-заявку; UC-REQ-02.5 - Редактирование брони или замена техники в draft; UC-REQ-02.6 - Отправка draft-заявки | Спринт 2 |
| FR-031 | Requestor can add/remove equipment items to a request; availability updated | + | UC-REQ-02 - Создание новой заявки (Draft-first); UC-REQ-02.3 - Поиск техники для добавления в заявку; UC-REQ-02.4 - Добавление техники в draft-заявку; UC-REQ-02.5 - Редактирование брони или замена техники в draft; UC-REQ-02.7 - Отмена draft-брони; UC-REQ-07 - Отмена draft-брони requestor-ом со страницы Мои заявки | Спринт 2 |
| FR-032 | System prioritizes internal fleet | + | UC-REQ-02 - Создание новой заявки (Draft-first); UC-REQ-02.3 - Поиск техники для добавления в заявку | Спринт 2 |
| FR-033 | Requestor can add Shared equipment to request | + | UC-REQ-02 - Создание новой заявки (Draft-first); UC-REQ-02.3 - Поиск техники для добавления в заявку; UC-REQ-02.4 - Добавление техники в draft-заявку | Спринт 2 |
| FR-038 | Each equipment item in request = separate booking | + | UC-REQ-02 - Создание новой заявки (Draft-first); UC-REQ-02.4 - Добавление техники в draft-заявку; UC-REQ-02.5 - Редактирование брони или замена техники в draft; UC-REQ-02.7 - Отмена draft-брони; UC-REQ-07 - Отмена draft-брони requestor-ом со страницы Мои заявки | Спринт 2 |
| FR-039 | Each booking has unique ID, start/end datetimes | + | UC-REQ-02 - Создание новой заявки (Draft-first); UC-REQ-02.4 - Добавление техники в draft-заявку; UC-REQ-02.5 - Редактирование брони или замена техники в draft | Спринт 2 |
| FR-040 | System validates availability before booking | + | UC-REQ-02 - Создание новой заявки (Draft-first); UC-REQ-02.3 - Поиск техники для добавления в заявку; UC-REQ-02.5 - Редактирование брони или замена техники в draft | Спринт 2 |
| FR-041 | Equipment attributes displayed in booking | + | UC-REQ-02 - Создание новой заявки (Draft-first); UC-REQ-02.3 - Поиск техники для добавления в заявку; UC-REQ-02.5 - Редактирование брони или замена техники в draft | Спринт 2 |
| FR-042 | Booking lifecycle: Draft, Submitted, Confirmed by FO, Confirmed, Declined, Revoked, Terminated, Completed | + | UC-REQ-04 - Отзыв брони requestor-ом | Спринт 2 |
| FR-NEW-18 | Booking start = start of mobilization | - | — | — |
| FR-NEW-19 | No mandatory buffer between bookings | - | — | — |
| FR-NEW-20 | Unwheeled equipment: notification displayed to Requestor | - | — | — |
| FR-NEW-70 | Equipment load summary displayed to Requestor during equipment selection | + | UC-REQ-02 - Создание новой заявки (Draft-first); UC-REQ-02.3.1 - Просмотр load summary при выборе техники | Спринт 2 |
| FR-043 | Approver assigned automatically based on fleet ownership | - | — | — |
| FR-044 | Internal fleet booking confirmed/declined by FO | - | — | — |
| FR-045 | FO can confirm incoming bookings | - | — | — |
| FR-046 | FO can replace equipment before/after confirmation if booking not yet started | - | — | — |
| FR-047 | FO cannot replace if booking revoked, terminated, or past end date | - | — | — |
| FR-048 | FO can adjust booking date range before or after confirmation | - | — | — |
| FR-049 | FO can decline booking; availability updated | - | — | — |
| FR-050 | FO provides reason/comment when declining | - | — | — |
| FR-NEW-21 | FO must respond within timeout | - | — | — |
| FR-NEW-22 | If FO has free equipment and declines, reason is mandatory | - | — | — |
| FR-NEW-23 | Reminder sent only to FO of the specific selected equipment | - | — | — |
| FR-NEW-24 | FO presses Mobilization started to record actual start time | - | — | — |
| FR-NEW-72 | Long-term rented booking moves to Confirmed by FO after FO confirmation and goes to Supervisor | - | — | — |
| FR-NEW-73 | FleetOwners' Supervisor reviews Long-term rented booking in Confirmed by FO status | - | — | — |
| FR-NEW-25 | On-demand BP equipment is view-only in Phase 1 | - | — | — |
| FR-NEW-26 | On-demand BP equipment prices/rates not displayed | - | — | — |
| FR-NEW-27 | BP populates own catalog cards | - | — | — |
| FR-NEW-28 | Go to BP catalog button after FO timeout | - | — | — |
| FR-058 | Requestor/SWP can revoke booking if not yet processed by FO | + | UC-REQ-04 - Отзыв брони requestor-ом; UC-REQ-05 - Отзыв submitted-заявки requestor-ом | Спринт 2 |
| FR-059 | FO can terminate confirmed booking; reason required; availability updated | - | — | — |
| FR-060 | Requestor/SWP can terminate confirmed booking (internal only) | - | — | — |
| FR-061 | Requestor/SWP can extend confirmed booking | - | — | — |
| FR-NEW-29 | Three-step Unwheeled flow | - | — | — |
| FR-NEW-30 | If transport unavailable for Unwheeled, FO proposes nearest available date | - | — | — |
| FR-NEW-31 | Booking can be closed early via Close early button | - | — | — |
| FR-062 | Booking -> Submitted once Request submitted | + | UC-REQ-02 - Создание новой заявки (Draft-first); UC-REQ-02.6 - Отправка draft-заявки | Спринт 2 |
| FR-063 | TCO Owned booking -> Confirmed; Long-term rented booking -> Confirmed by FO once confirmed by FO | - | — | — |
| FR-064 | Booking -> Declined once declined by FO | - | — | — |
| FR-067 | Booking -> Submitted once extended by Requestor/SWP | - | — | — |
| FR-068 | Booking -> Revoked once revoked | + | UC-REQ-04 - Отзыв брони requestor-ом | Спринт 2 |
| FR-069 | Booking -> Terminated once terminated | - | — | — |
| FR-072 | Request -> Draft when saved as draft | + | UC-REQ-02 - Создание новой заявки (Draft-first); UC-REQ-02.1 - Создание пустого draft заявки | Спринт 2 |
| FR-073 | SWR -> Draft when imported from JDE | - | — | — |
| FR-074 | Request -> In Progress when first booking goes In Progress | - | — | — |
| FR-NEW-32 | Booking closure is manual only | - | — | — |
| FR-NEW-33 | At close Requestor inputs actual start/end time for usage rate analytics | - | — | — |
| FR-NEW-44 | Request -> Completed when last active booking closed | - | — | — |
| FR-NEW-77 | Long-term rented booking -> Confirmed by FO upon FO confirmation | - | — | — |
| FR-NEW-78 | Long-term rented booking -> Confirmed or Declined upon Supervisor decision | - | — | — |
| FR-076 | System notifies SWP when SWR auto-created | - | — | — |
| FR-077 | System notifies FO when request submitted | - | — | — |
| FR-078 | System notifies FO when booking revoked | - | — | — |
| FR-079 | System notifies Requestor/SWP of FO decision | - | — | — |
| FR-079a | System notifies Requestor when FO updates booking period | - | — | — |
| FR-080 | System notifies Requestor/SWP when FO terminates booking | - | — | — |
| FR-081 | System notifies FO when Requestor extends booking | - | — | — |
| FR-NEW-34 | 24h reminder to FO; 48h escalation to manager | - | — | — |
| FR-NEW-35 | Notification to Transportation Responsible for Unwheeled booking | - | — | — |
| FR-NEW-36 | FO notified when Transportation Responsible decides | - | — | — |
| FR-NEW-43 | Notifications delivered to fleet shared team email | - | — | — |
| FR-NEW-75 | System notifies FleetOwners' Supervisor when Long-term rented booking reaches Confirmed by FO | - | — | — |
| FR-NEW-76 | System notifies Requestor and FO when FleetOwners' Supervisor makes a decision | - | — | — |
| FR-091 | Requestor/SWP can search and filter own requests | + | UC-REQ-01 - Просмотр моих заявок | Спринт 2 |
| FR-092 | Approver can view all pending and completed approvals | + | UC-COM-01 - Просмотр таймлайна брони | Спринт 2 |
| FR-093 | Admin can view all requests | - | — | — |
| FR-094 | System generates reports on Requests/Bookings/Equipment with filters | - | — | — |
| FR-095 | Admin can view/download all reports | - | — | — |
| FR-096 | FO can view/download own approval reports | - | — | — |
| FR-097 | FO can view/download own equipment reports | - | — | — |
| FR-098 | SWP and FOs can view SWR grouped by Work Orders | - | — | — |
| FR-NEW-37 | Audit trail and history on demand | + | UC-COM-01 - Просмотр таймлайна брони | Спринт 2 |
| FR-NEW-38 | Search: TCO equipment number + model mandatory; госномер if present | + | UC-REQ-02 - Создание новой заявки (Draft-first); UC-REQ-02.3 - Поиск техники для добавления в заявку | Спринт 2 |
| FR-NEW-39 | Dynamic search filters by equipment type | + | UC-REQ-02 - Создание новой заявки (Draft-first); UC-REQ-02.3 - Поиск техники для добавления в заявку | Спринт 2 |
| FR-NEW-45 | Dedicated Completed Requests page for Requestor, SWP, FO, Admin | - | — | — |
| FR-NEW-46 | Usage Rate reports: by day / department / equipment unit | + | Реализация админ панели и стр. оборудования | Спринт 1 |
| FR-NEW-47 | Work center reports | - | — | — |
| FR-NEW-52 | Requestor has access to Request History page listing completed requests | - | — | — |
| FR-099 | System displays usage rate data and coordinates from DataLake/PI | - | — | — |
| FR-100 | System displays digitized location of Work Orders from DataLake (JDE E1) | - | — | — |
| FR-NEW-53 | Usage Rate Dashboard has three tabs | + | Реализация админ панели и стр. оборудования | Спринт 1 |
| FR-NEW-54 | Usage Rate cells use a color gradient | + | Реализация админ панели и стр. оборудования | Спринт 1 |
| FR-NEW-55 | Repair days marked with wrench icon and excluded from average | + | Реализация админ панели и стр. оборудования | Спринт 1 |
| FR-NEW-56 | Usage Rate Dashboard includes Target value column | - | — | — |
| FR-NEW-57 | Usage Rate Dashboard includes Average usage rate column | + | Реализация админ панели и стр. оборудования | Спринт 1 |
| FR-NEW-58 | Usage Rate Dashboard supports daily / monthly granularity | + | Реализация админ панели и стр. оборудования | Спринт 1 |
| FR-NEW-59 | Usage Rate Dashboard includes Ownership filter | + | Реализация админ панели и стр. оборудования | Спринт 1 |
| FR-NEW-60 | Usage rate data sourced from tracker telemetry only | + | Реализация админ панели и стр. оборудования | Спринт 1 |
| FR-NEW-61 | FO Equipment section includes checkboxes to display units on GIS map | - | — | — |
| FR-NEW-62 | GIS map marker tooltip displays sensor data and current-day usage rate | - | — | — |
| FR-NEW-63 | Usage Rate Dashboard displays booking count for selected period | - | — | — |
| FR-NEW-65 | GIS map displays speed and heading for GSM trackers | - | — | — |
| FR-NEW-66 | GIS map does not display fuel level | - | — | — |
| FR-NEW-67 | GIS map supports historical route for up to 7 days | - | — | — |
| FR-101 | System logs all request creation, modifications, approvals, rejections | - | — | — |
| FR-102 | Audit trail accessible for compliance purposes | - | — | — |
| FR-103 | Exportable logs for legal/regulatory audits | - | — | — |

## AFR coverage

Источник: `wiki/brd/new FR's/Список FR по equipment.md`, раздел `Additional FRs`.

| AFR | Описание | Покрыто | Use case | Спринт |
|---|---|---|---|---|
| AFR-01 | Admin manages equipment brands directory | - | — | Спринт 1 |
| AFR-02 | Admin manages equipment models directory | - | — | Спринт 1 |
| AFR-03 | Admin manages organizational and location handbooks used in equipment card | - | — | Спринт 1 |
| AFR-04 | Admin manages maintenance partners directory | - | — | Спринт 1 |
| AFR-05 | Admin manages business partners directory used in equipment data model | - | — | Спринт 1 |
| AFR-06 | Admin manages equipment maintenance contracts by equipment, partner and service type | - | — | Спринт 1 |
| AFR-07 | Admin manages equipment types including class, mobility, work center and sorting attributes | - | — | Спринт 1 |
