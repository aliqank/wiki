# **HDV/HDE Booking Tool**

# Business Requirement Document

## 1 Introduction

### 1.1 Purpose

This document defines the business requirements for a booking tool that allows users to create requests for HDV/HDE equipment. Requests may include multiple pieces of equipment, each of which is treated as a separate booking with its own approval workflow.

### 1.2 Scope

The tool will support bookings for company-owned equipment as well as equipment provided by business partners. It will cover request creation, approval flow, status tracking, reporting, and integration with external systems where applicable.

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

## 3 Glossary

**Request** – a container grouping multiple equipment bookings.

**Regular Request** – a request created by Requestor in HDV/HDE booking tool.

**Service Work Request** – a request created in HDV/HDE automatically via API based on Work Order created by Requestor in JDE E1 system.

**Booking** – individual booking within a request. One booking can contain one unique equipment booking. Treated individually within a request.

**Equipment** \- individual asset (HDV/HDE unit) that can be booked through the system. Each equipment item belongs to a fleet and inherits approval responsibility from the corresponding Fleet Owner. An equipment item has its own attributes (e.g., equipment ID, license, usage status, etc.) and is always treated as separate booking when included in a request. Template for equipment will differ depending on to which fleet it belongs to: internal (TCO owned) or external (Business Partner’s).

**Fleet** – logical group of equipment owned by a Fleet Owner (for internal: Maintenance, Construction, TFM, etc. for external, business partner’s name).

**Fleet Owner** – person responsible for approvals within a fleet.

**Approver** – Fleet Owner and/or Cost Center (CC) owner, who confirms/declines bookings.

## 4 Stakeholder & Roles

### 4.1 Requestor

All users in TCO network will by default have Requestor role to book equipments of the internal fleet (TCO owned HDV/HDE). Booking of equipments of the external fleet (BP owned HDV/HDE) is available only for users with ‘RWA initiator’ role in Digital Contractor Timesheet (DCT) tool. Requestors are allowed to:

1. Create a Regular Request  
2. Submit the Regular Request  
3. Prolong booking   
4. Submit feedback on the booked equipment  
5. Monitor utilization of the booked equipment for the booking period, if the equipment has tracker(s)  
6. Terminate booking (only external fleet)

### 4.2 Service Work Processor

Access provision is managed by Azure Active Directory (AAD) group. Similar to Requestor, however, Service Work Processor works with Service Work Requests, created automatically in HDV/HDE booking tool.

### 4.3 Fleet Owner

For internal Fleet Owners access provision is managed by AAD group. There can be 2+ AAD groups to manage internal Fleet Owners (separate group for each fleet created by Administrator). Fleet Owners are allowed to: 

1. Create new equipment under own fleet (allowed only for external Fleet Owners)  
2. Update equipments (only parameters that are allowed for editing by Adminitrator)  
3. Delete equipment (soft delete)  
4. Confirm / decline requests to book equipments under own fleet  
5. Update booking period before confirmation  
6. Change equipment in booking  
7. Terminate already confirmed bookings (only internal fleet)  
8. Freeze / unfreeze own equipment from booking for a period or without end date  
9. Monitor utilization of own equipment, if the equipment has tracker(s)

### 

### 4.4 CC Owner (incl. DOA)

There will be no specific AAD group to provide users CC owner’s grants. The HDV/HDE tool must identify (via integration with PSWS and DOA) if the user within the tool can conduct actions as CC owner or his DOA. CC owners are allowed to: 

1. Confirm / decline external fleet equipment booking request

### 

### 4.5 Administrator

Administrator role will be provided to process owners – Logistics. Access provision is managed by Azure Active Directory (group). Users with an Administrator role are allowed to:

1. Create/edit internal fleet and link with AAD group  
2. Create/edit external fleet and manage external fleet owners (send invitation, deactivate external user’s account)  
3. Delete fleet  
4. Create internal fleet equipment  
5. Update equipment’s parameters (extended edit)  
6. Delete equipment (soft delete)  
7. Manage ‘Request’ template  
8. Manage ‘Equipment’ template  
9. Access to custom reportings  
10. Access to utilization dashboard(s), if the equipment has tracker(s)

## 5 Entities

### 5.1 Equipment

All equipments must have a mandatory attribute – fleet. The attribute will affect on which template the equipment will have (depending on internal and external fleet), who can create/edit/delete equipments (fleet owners), approval and processing flow of the equipment booking. 

Internal fleet equipments have the next parameters:

1. Basic fixed info (TCO ID, Description, Service Zone, Class/Category, License, Serial number, etc.)  
2. Dynamic (changeable) parameters: Equipment Usage Status, CC/Division/Group/Department, Assigned location, Maintenance BP, Contact number / email, etc.)  
3. Equipment specifications (**picture**, Chassis type, Drive type, Loading capacity, Generator / Compressor specifications)

4. ### Trackers’ ID is exists (WIALON tracker “Group”, IVMS tracker “Registration number”, Vega tracker “Device ID”, etc)

### 

### 5.2 Booking 

Booking is linked to a single equipment and have booking range (datetime). Several bookings form a single request. 

#### 5.2.1 Internal Booking

Booking of TCO owned (internal fleet) equipment. Internal Fleet equipments require only confirmation from the side of Internal Fleet owner. Fleet owner can apply adjustments to the booking period and/or replace equipment if substitution exists. 

#### 5.2.2 External Booking

Booking of partner-provided (external fleet) equipment. External Fleet equipments require the next confirmations: 

1. CC owner or his DOA  
2. External Fleet Owner

### 5.3 Request

Requests have 2 types: Regular Request, Service Work Request. The type affects on request template, mandatority of filling parameters and approvers’ workflow.

#### 5.3.1 Regular Request

In a Regular Request Requestor can add 2+ equipments any fleet, there is no restriction that all equipments must be from the same fleet. Each added equipment will be considered as separate booking and will be confirmed by Fleet Owners separately. 

The system will prioritize TCO owned (internal fleet) assets. If unavailable, BP rentals (external fleet) will be suggested based on a pre-set decision matrix (cost, reliability, availability).

For internal fleet equipment added in the Request requestor must put booking range (datetime). 

For external fleet equipment added in the Request requestor must put “Justification” and Cost Center. The system must prefill “Cost Center” by Requestor’s cost center (taking information from PSWS), Requestor can manually change the prefilled data. 

#### 5.3.2 Service Work Request (within Work Order in JDE E1)

Unlike Regular Request, Service Work Requests are created automatically in the system as a Draft after Work order creation in JDE E1, and submitted by Service Work Processor. Each ‘Step’ in Work Order will be created as a separate Service Work Request in HDV/HDE booking tool. Service Work Processor assigns equipment(s) and submits the Request. 

## 6 Functional Requirements

### 6.1 User & Role Management

**FR-001:** System supports role-based access control

**FR-002:** All users in TCO network have an access to the tool as a Requestor, however, in order to add to the request external fleet equipment, the user must be a member of “RWA initiator” AAD group. 

**FR-003:** Administrators, Service Work Proccessors are defined by a membership in the corresponding AAD groups. 

**FR-004:** The system will define if the user has CC Owner or his DOA privileges as a result of integration with internal systems: PSWS (TCO White Page) and DOA (Delegation of Authority). 

### 6.2 Admin Panel

**FR-005:** Admin can create fleets (internal, external)

**FR-006:** Admin can link internal fleet with AAD group, so that members of that group can act as Fleet Owners in the system

**FR-007:** Admin can manage owners of created external fleets: send invitation to Business Partner to create account, deactivate existing external accounts

**FR-008:** Admin can update existing fleets 

**FR-009:** Admin can delete existing fleets

**FR-010:** Admin can manage template of equipment, which will be different for internal and external fleets: define parameters, formats (string, number, Boolean, drop-down, etc.), mandatority, if available for editing by owners). 

**FR-011:** Admin can manage template of request

**FR-012:** Admin can manage template of booking

**FR-013:** Admin can create equipments of internal fleet

**FR-014:** Admin can edit (extended edit) equipments

**FR-015:** Admin can delete equipments

**FR-016:** Admin has an access to all dashboards and reports

### 6.3 Equipments Management

**FR-017:** External fleet owner can create equipments if fills all mandatory parameters

**FR-018:** Fleet owner must indicate ‘Fleet’ of the equipment, the system allows to select only fleet, for which the user is a Fleet Owner. 

**FR-019:** Fleet owner can edit (allowed by Admin) equipments under his fleet. Note: One user can be an Owner of several fleets, is he is a member of several corresponding AAD groups.

**FR-020:** Fleet owner can delete equipments under his fleet. 

**FR-021:** Fleet owner can freeze / unfreeze own equipments for some period of time or without end date. 

**FR-022:** Requestor / Service Work Processor can submit a feedback on equipment, booking on which was confirmed by Approver(s) starting from the booking start datetime

### 6.4 Request Management

**FR-023:** Requestor can create a Request.

**FR-024:** Request has a unique identifier and metadata (date, requester, status, etc.).

**FR-025:** Requester can view request details and status.

**FR-026:** Request can have status: Draft, Submitted, In progress, Completed.

**FR-027:** Requester can edit or cancel a draft request before submission.

**FR-028:** Requests created by system automatically (Service Work Requests created via API) will have ‘Draft’ status before submission by Service Work Processor

**FR-029:** Only users with role Service Work Processor can manage (submit) Service Work Requests.

**FR-030:** System supports draft saving (continue later).

### 

### 6.5 Booking Management

**FR-031:** Requestor can add/remove multiple equipment items into a single request. Equipment availability in DB is updated. 

**FR-032:** The system prioritizes internal fleet equipments. 

**FR-033:** Requestor can add internal fleet equipments with “Equipment Usage Status” in (Shared without conditions, Shared with conditions)

**FR-034:** The system will suggest external fleet equipments based on a pre-set decision matrix (cost, reliability, availability). Custom logic \- TBD.

**FR-035:** Requestor can add external fleet equipment to Request, only if he is a member of “RWA initiator” AAD group. 

**FR-036:** When Requestor adds external fleet equipment to Request, he must provide “Justification”.

**FR-037:** When Requestor adds external fleet equipment to Request, “Cost Center” must be filled for that Booking. The system prefills the parameter with Requestor’s Cost Center (via PSWS integration), Requestor can edit prefilled parameters. 

**FR-038:** Each equipment item added to Request is treated as a separate booking.

**FR-039:** Each booking has its own unique booking ID, start and end datetimes.

**FR-040:** System validates availability of equipment before booking.

**FR-041:** Equipment attributes (picture, fleet, description, usage status, etc) are displayed.

**FR-042:** Each booking has its own lifecycle (Draft, Submitted, Confirmed, Declined, Revoked, Terminated, Completed)

### 6.6 Approval Flow (Internal Fleet Equipment)

**FR-043:** System automatically assigns approver based on fleet ownership.

**FR-044:** If equipment in the booking belongs to internal fleet, Booking can be confirmed or declined by user, who is owner of that Fleet (member of the corresponding AAD group). 

**FR-045:** Fleet owner can confirm incoming bookings of equipment in his fleet. 

**FR-046:** Fleet owner can replace the equipment in the booking with another equipment before or after confirmation if the booking date is not started or booking end date is in future. Equipment availability in DB is updated.

**FR-047:** Fleet owner cannot replace the equipment if booking is revoked (by Requestor), terminated (by Fleet Owner) or booking end date is in past. 

**FR-048:** Internal fleet owner can adjust booking date range before confirmation. Equipment availability in DB is updated.

**FR-049:** Fleet owner can decline incoming bookings of equipment in his fleet. Equipment availability in DB is updated.

**FR-050:** Fleet owner can indicate ‘reason/comment’ in case of declining.

### 6.7 Approval Flow (External Fleet Equipment)

**FR-051:** If equipment in the booking belongs to external fleet, the booking first requires confirmation of CC (cost center) owner, depending on which cost center is indicated in the booking

**FR-052:** The system automatically identifies if the user has CC owner privileges consuming information from PSWS 

**FR-053:** The system allows DOAs of CC owners confirm/decline bookings. Equipment availability in DB is updated in case of declining.

**FR-054:** The system automatically identifies if the user (DOA) can process the booking on the behalf of CC owner consuming information from PSWS and DOA systems.

**FR-055:** CC owner/DOA can put ‘reason/comment’ in case of declining the booking

**FR-056:** Once the booking is confirmed by ‘CC owner / DOA’, the system requires confirmation of Fleet owner (Business Partner)

**FR-057:** Fleet owner can confirm or decline the booking. Fleet owner can put ‘reason/comment’ in case of declining the booking. Equipment availability in DB is updated in case of declining.

### 6.8 Booking Lifecycle Management

**FR-058:** Requestor/Service Work Processor can revoke (withdraw) booking of any fleet equipment, if the booking is not processed (confirmed/declined) by Fleet Owner yet. Equipment availability in DB is updated.

**FR-059:** Fleet owner of internal fleet can terminate already confirmed booking. Fleet owner can indicate ‘comment/reason’ in case of termination. Equipment availability in DB is updated.

**FR-060:** Requestor/Service Work Processor can terminate already confirmed booking. Equipment availability in DB is updated.

**FR-061:** Requestor/Service Work Processor can extended previously confirmed booking. Equipment availability in DB is updated.

### 

### 6.9 Booking/Request Status Management

**FR-062:** Booking will have ‘Submitted’ status once Request is recently submitted

**FR-063:** Booking will have ‘Confirmed’ status once it is confirmed by Fleet Owner

**FR-064:** Booking will have ‘Declined’ status once it is Declined by Fleet Owner

**FR-065:** Booking of equipment of external fleet will have ‘Partially Confirmed’ status once it is confirmed by CC Owner

**FR-066:** Booking of equipment of external fleet will have ‘Declined’ status once it is declined by CC Owner

**FR-067:** Status of booking will return to ‘Submitted’ once it is extended by Requestor / Service Work Processor

**FR-068:** Booking will have ‘Revoked’ status once it is revoked by Requestor / Service Work Processor

**FR-069:** Booking will have ‘Terminated’ status once it is terminated by Requestor / Service Work Processor / Fleet Owner

**FR-070:** Confirmed booking status will change to ‘In progress’ status once system datetime \>= booking datetime

**FR-071:** Confirmed booking status will change to ‘Completed’ status once system datetime \< booking datetime

**FR-072:** Request will have ‘Draft’ status once it is saved as draft by Requestor / Service Work Processor

**FR-073:** Service Work Request will have ‘Draft’ status once it is recently imported from JDE E1

**FR-074:** Request will have ‘In Progress’ status once at least one Booking within the Request have status ‘In Progress’

**FR-075:** Request will have ‘Completed’ status once system datetime \>= maximum end datetime

### 6.10 Notifications

**FR-076:** The system notifies Service Work Processor(s) once Service Work Request is created in the system automatically via API (Call to action)

**FR-077:** The system notifies Fleet Owner once the Requestor / Service Work Processor submits a Request with booking of internal fleet equipment (Call to action)

**FR-078:** The system notifies Fleet Owner once the Requestor / Service Work Processor revokes booking of internal fleet equipment (Info)

**FR-079:** The system notifies Requestor / Service Work Processor about results of once Fleet Owner confirms or declines the booking of internal fleet equipment (Result: Success or Fail)

**FR-080:** The system notifies Requestor / Service Work Processor once Fleet Owner terminates the booking of internal fleet equipment (Fail result)

**FR-081:** The system notifies Fleet Owner once the Requestor / Service Work Processor wants to extend the booking of internal fleet equipment (Call to action)

**FR-082:** The system notifies CC owner(s) once the Requestor / Service Work Processor submits a Request with booking of external fleet equipment (Call to action)

**FR-083:** The system notifies CC owner once the Requestor / Service Work Processor revokes a booking of external fleet equipment before receival of CC owner’s confirmation (Info)

**FR-084:** The system notifies Requestor / Service Work Processor once CC owner declines a booking of external fleet equipment (Fail result)

**FR-085:** The system notifies Fleet Owner once CC owner confirms a booking of external fleet equipment (Call to action)

**FR-086:** The system notifies CC owner and Fleet Owner once the Requestor / Service Work Processor revokes a booking of external fleet equipment before receival of Fleet owner’s confirmation but after receival of CC owner’s confirmation (Info)

**FR-087:** The system notifies Requestor / Service Work Processor once Fleet owner declines a booking of external fleet equipment (Fail result)

**FR-088:** The system notifies Requestor / Service Work Processor once Fleet owner confirms a booking of external fleet equipment (Success result and Call to action to submit RWA in DCT)

**FR-089:** The system notifies Fleet owner once Requestor / Service Work Processor terminates already confirmed booking of external fleet equipment (Info)

**FR-090:** The system notifies CC owner once Requestor / Service Work Processor wants to extend already confirmed booking of external fleet equipment (Call to action)

### 6.11 Search & Reporting

**FR-091:** Requestor / Service Work Processor can search and filter own requests

**FR-092:** Approver (CC Owner / Fleet Owner) can view all pending and completed approvals.

**FR-093:** Admin can view all requests

**FR-094:** System generates reports on Requests / Bookings / Equipments with capability to apply various filters

**FR-095:** Admin can view/download reports on all Requests / Bookings / Equipments

**FR-096:** Approver (CC Owner / Fleet Owner) can view/download reports on pending and completed Requests / Bookings 

**FR-097:** Fleet Owner can view/download reports on own equipments

**FR-098:** Service Work Processors (and selected Fleet Owners \- Maintenance) can view Service Work Requests grouped in Work Orders. 

### 6.12 Dashboards / Map

**FR-099:** The system has deployed inside / integrated visualization tool which is enabled to use data from DataLake/PI (on trackers) to display information about utilization (applying custom logic) and coordinates.

**FR-100:** The system has deployed inside / integrated visualization tool which is enabled to use data from DataLake (JDE E1) to display digitized location of Work Orders.

### 6.13 Audit & Compliance

**FR-101:** System logs all request creation, modifications, approvals, rejections.

**FR-102:** Audit trail accessible for compliance purposes.

**FR-103:** Exportable logs for legal/regulatory audits.

## 7 Non-Functional Requirements

**Availability:** 99.5% uptime.

**Performance:** The system must support high volume of requests per hour without performance degradation.

**Security:** Role-based access control (RBAC). System integrates with corporate authentication (SSO).

**Compliance:** Maintain audit history of all approvals/rejections/revokes/terminations.

**Scalability:** Support both internal and external partner use cases.

**Usability:** Intuitive UI for both requesters and approvers.

## 8 Integrations

### 8.1 JDE E1

Work Orders created in JDE E1 must be imported to HDV/HDE tool as a Service Work Request(s), once the certain (TBD) status is assigned to Work Order. Work Order can contain several Steps, each step is created as a separate Service Work Request in HDV/HDE tool. Only Steps with certain types (BHOE, CDECK, DTRK, FORKL, FTRK-ХИНО, GDRV, GMLOG, MLIFT, VTRK, etc.) will be imported to HDV/HDE tool as a Service Work Requests.

The systems must have backward integration which allows to change the status of ‘Step’ in JDE based on status change of the imported ‘Request’ in HDV/HDE or by Service Work Processor’s manual actions. 

### 8.2 DCT (Digital Contractor Timesheet)

#### *8.2.1 RWA / Change Order Creation*

When external fleet’s equipment booking receives all (CC owners’ and Fleet Owner’s) approval, the system must create draft of RWA in DCT (via POST API) and receive ID of RWA (synchronous or asynchronous). 

**Note:** Receival of RWA ID can be synchronous, if HDV/HDE will send all mandatory attributes of RWA, so that DCT system can assign ID. If not, API on the side of HDV/HDE tool must be implemented, which will be consumed by DCT tool: DCT will send RWA ID and parameter of request/equipment (TBD).

When external fleet’s equipment booking prolongation receives all approval (CC owners’ and Fleet owner’s), the system must create Change Order in DCT (via POST API). ID of previously created RWA exists in the request body of API.

#### *8.2.2 Tracker’s data receival* 

Once external fleet equipments will be equipped by trackers, the system must be designed to send data to DCT, so that DCT can actual working hours versus reported by Business Partner hours. 

8.3 PSWS (TCO white page)

If equipment added to the booking is external fleet’s equipment, it first requires approval from CC owner indicated in the request. Once the user enters the HDV/HDE tool, the system must identify if entered user is a CC owner of any external fleet’s (BP’s) equipment, which is pending for CC owner’s approval. If yes, the system must be designed to display explicitly list of equipments which are pending for that user’s approval. Information about CC owners of cost centers will be provided from PSWS.

Information from PSWS is also required to save user’s preferred e-mail address which is required to identify correct notification recipients.

8.4 DOA (Delegation of Authority)

Once the user enters the HDV/HDE tool, the system must identify if entered user is a DOA of CC owner of any external fleet’s (BP’s) equipment, which is pending for CC owner’s approval. If yes, the system must be designed to display explicitly list of equipments which are pending for CC owner’s approval, so that DOA of the CC owner can process the request.

8.5 Datalake / PI

From Datalake utilization information from IVMS and WIALON trackers (the list of source can be extended if new trackers will be equipped) will be received. “Trackers’ ID” will exist in the list of parameters of equipments allowing the system to merge existing equipments with information from Datalake on utilization and location.