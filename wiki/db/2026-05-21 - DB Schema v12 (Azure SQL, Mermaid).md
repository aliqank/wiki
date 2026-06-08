# DB Schema v12 — Azure SQL Mermaid ER-диаграмма

**Created:** 2026-05-21  
**Last updated:** 2026-06-08  
**Version:** v12 (Azure SQL adaptation)

> Типы данных адаптированы под Azure SQL: `uniqueidentifier`, `datetime2(3)`, `bit`, `nvarchar(max)`.  
> `enum` заменены на `ref_*` таблицы.  
> `name JSON` заменён на `nameEn`, `nameRu`, `nameKz`.
> Для локализованных name-полей в текущем v12 предполагается, что `nameEn`, `nameRu`, `nameKz` являются обязательными (`not null`).
> Для всех основных mutable таблиц в v12 предполагаются **system-versioned temporal tables**, кроме `BookingStatuses`, `BookingRequestStatuses`, `EquipmentStates`, которые остаются явными history/event tables.

---

```mermaid
erDiagram

    ref_fleet_type {
        uniqueidentifier id PK
        nvarchar code
        nvarchar nameEn
        nvarchar nameRu
        nvarchar nameKz
        int sortOrder
    }

    ref_fleet_manage_permission_type {
        uniqueidentifier id PK
        nvarchar code
        nvarchar nameEn
        nvarchar nameRu
        nvarchar nameKz
        int sortOrder
    }

    ref_equipment_mobility_type {
        uniqueidentifier id PK
        nvarchar code
        nvarchar nameEn
        nvarchar nameRu
        nvarchar nameKz
        int sortOrder
    }

    ref_equipment_class {
        uniqueidentifier id PK
        nvarchar code
        nvarchar nameEn
        nvarchar nameRu
        nvarchar nameKz
        int sortOrder
    }

    ref_ownership_type {
        uniqueidentifier id PK
        nvarchar code
        nvarchar nameEn
        nvarchar nameRu
        nvarchar nameKz
        int sortOrder
    }

    ref_share_type {
        uniqueidentifier id PK
        nvarchar code
        nvarchar nameEn
        nvarchar nameRu
        nvarchar nameKz
        int sortOrder
    }

    ref_equipment_status_type {
        uniqueidentifier id PK
        nvarchar code
        nvarchar nameEn
        nvarchar nameRu
        nvarchar nameKz
        nvarchar iconUrl
        int sortOrder
    }

    ref_equipment_status_source {
        uniqueidentifier id PK
        nvarchar code
        nvarchar nameEn
        nvarchar nameRu
        nvarchar nameKz
        int sortOrder
    }

    ref_property_data_type {
        uniqueidentifier id PK
        nvarchar code
        nvarchar nameEn
        nvarchar nameRu
        nvarchar nameKz
        int sortOrder
    }

    ref_user_type {
        uniqueidentifier id PK
        nvarchar code
        nvarchar nameEn
        nvarchar nameRu
        nvarchar nameKz
        int sortOrder
    }

    ref_request_type {
        uniqueidentifier id PK
        nvarchar code
        nvarchar nameEn
        nvarchar nameRu
        nvarchar nameKz
        int sortOrder
    }

    ref_request_priority {
        uniqueidentifier id PK
        nvarchar code
        nvarchar nameEn
        nvarchar nameRu
        nvarchar nameKz
        nvarchar descriptionEn
        nvarchar descriptionRu
        nvarchar descriptionKz
        nvarchar color
        int sortOrder
    }

    ref_booking_request_status {
        uniqueidentifier id PK
        nvarchar code
        nvarchar nameEn
        nvarchar nameRu
        nvarchar nameKz
        int sortOrder
    }

    ref_request_closure_reason {
        uniqueidentifier id PK
        nvarchar code
        nvarchar nameEn
        nvarchar nameRu
        nvarchar nameKz
        int sortOrder
    }

    ref_booking_status {
        uniqueidentifier id PK
        nvarchar code
        nvarchar nameEn
        nvarchar nameRu
        nvarchar nameKz
        int sortOrder
    }

    ref_booking_closure_reason {
        uniqueidentifier id PK
        nvarchar code
        nvarchar nameEn
        nvarchar nameRu
        nvarchar nameKz
        int sortOrder
    }

    ref_booking_approval_type {
        uniqueidentifier id PK
        nvarchar code
        nvarchar nameEn
        nvarchar nameRu
        nvarchar nameKz
        int sortOrder
    }

    ref_booking_approval_status {
        uniqueidentifier id PK
        nvarchar code
        nvarchar nameEn
        nvarchar nameRu
        nvarchar nameKz
        int sortOrder
    }

    EquipmentTypes {
        uniqueidentifier id PK
        nvarchar nameEn
        nvarchar nameRu
        nvarchar nameKz
        nvarchar iconUrl
        uniqueidentifier mobilityTypeId FK
        bit requiresTransport
        uniqueidentifier equipmentClassId FK
        uniqueidentifier workCenterId FK
        int sortOrder
        datetime2 createdAt
        uniqueidentifier createdBy
        datetime2 updatedAt
        uniqueidentifier updatedBy
        bit isDeleted
        datetime2 deletedAt
        uniqueidentifier deletedBy
    }

    EquipmentBrands {
        uniqueidentifier id PK
        nvarchar nameEn
        nvarchar nameRu
        nvarchar nameKz
        int sortOrder
        datetime2 createdAt
        uniqueidentifier createdBy
        datetime2 updatedAt
        uniqueidentifier updatedBy
        bit isDeleted
        datetime2 deletedAt
        uniqueidentifier deletedBy
    }

    EquipmentModels {
        uniqueidentifier id PK
        uniqueidentifier brandId FK
        nvarchar nameEn
        nvarchar nameRu
        nvarchar nameKz
        int sortOrder
        datetime2 createdAt
        uniqueidentifier createdBy
        datetime2 updatedAt
        uniqueidentifier updatedBy
        bit isDeleted
        datetime2 deletedAt
        uniqueidentifier deletedBy
    }

    Fleets {
        uniqueidentifier id PK
        nvarchar nameEn
        nvarchar nameRu
        nvarchar nameKz
        uniqueidentifier fleetTypeId FK
        uniqueidentifier businessPartnerId FK
        nvarchar aadGroupId
        datetime2 createdAt
        uniqueidentifier createdBy
        datetime2 updatedAt
        uniqueidentifier updatedBy
        bit isDeleted
        datetime2 deletedAt
        uniqueidentifier deletedBy
    }

    WorkCenters {
        uniqueidentifier id PK
        nvarchar code
        nvarchar nameEn
        nvarchar nameRu
        nvarchar nameKz
        datetime2 createdAt
        uniqueidentifier createdBy
        datetime2 updatedAt
        uniqueidentifier updatedBy
        bit isDeleted
        datetime2 deletedAt
        uniqueidentifier deletedBy
    }

    Locations {
        uniqueidentifier id PK
        nvarchar code
        nvarchar nameEn
        nvarchar nameRu
        nvarchar nameKz
        int sortOrder
        datetime2 createdAt
        uniqueidentifier createdBy
        datetime2 updatedAt
        uniqueidentifier updatedBy
        bit isDeleted
        datetime2 deletedAt
        uniqueidentifier deletedBy
    }

    CostCenters {
        uniqueidentifier id PK
        nvarchar code
        nvarchar nameEn
        nvarchar nameRu
        nvarchar nameKz
        int sortOrder
        datetime2 createdAt
        uniqueidentifier createdBy
        datetime2 updatedAt
        uniqueidentifier updatedBy
        bit isDeleted
        datetime2 deletedAt
        uniqueidentifier deletedBy
    }

    ServiceZones {
        uniqueidentifier id PK
        nvarchar code
        nvarchar nameEn
        nvarchar nameRu
        nvarchar nameKz
        int sortOrder
        datetime2 createdAt
        uniqueidentifier createdBy
        datetime2 updatedAt
        uniqueidentifier updatedBy
        bit isDeleted
        datetime2 deletedAt
        uniqueidentifier deletedBy
    }

    Divisions {
        uniqueidentifier id PK
        nvarchar code
        nvarchar nameEn
        nvarchar nameRu
        nvarchar nameKz
        int sortOrder
        datetime2 createdAt
        uniqueidentifier createdBy
        datetime2 updatedAt
        uniqueidentifier updatedBy
        bit isDeleted
        datetime2 deletedAt
        uniqueidentifier deletedBy
    }

    Groups {
        uniqueidentifier id PK
        uniqueidentifier divisionId FK
        nvarchar code
        nvarchar nameEn
        nvarchar nameRu
        nvarchar nameKz
        int sortOrder
        datetime2 createdAt
        uniqueidentifier createdBy
        datetime2 updatedAt
        uniqueidentifier updatedBy
        bit isDeleted
        datetime2 deletedAt
        uniqueidentifier deletedBy
    }

    Departments {
        uniqueidentifier id PK
        uniqueidentifier groupId FK
        nvarchar code
        nvarchar nameEn
        nvarchar nameRu
        nvarchar nameKz
        int sortOrder
        datetime2 createdAt
        uniqueidentifier createdBy
        datetime2 updatedAt
        uniqueidentifier updatedBy
        bit isDeleted
        datetime2 deletedAt
        uniqueidentifier deletedBy
    }

    Sections {
        uniqueidentifier id PK
        uniqueidentifier departmentId FK
        nvarchar code
        nvarchar nameEn
        nvarchar nameRu
        nvarchar nameKz
        int sortOrder
        datetime2 createdAt
        uniqueidentifier createdBy
        datetime2 updatedAt
        uniqueidentifier updatedBy
        bit isDeleted
        datetime2 deletedAt
        uniqueidentifier deletedBy
    }

    FleetManagePermissions {
        uniqueidentifier id PK
        uniqueidentifier fleetId FK
        uniqueidentifier userId FK
        uniqueidentifier permissionTypeId FK
        datetime2 expiresAt
        datetime2 createdAt
        uniqueidentifier createdBy
        datetime2 updatedAt
        uniqueidentifier updatedBy
        bit isDeleted
        datetime2 deletedAt
        uniqueidentifier deletedBy
    }

    Equipments {
        uniqueidentifier id PK
        uniqueidentifier equipmentTypeId FK
        uniqueidentifier fleetId FK
        uniqueidentifier ownershipTypeId FK
        nvarchar tcoId
        nvarchar jdeId
        nvarchar stateNumber
        nvarchar vin
        nvarchar serialNumber
        nvarchar description
        uniqueidentifier brandId FK
        uniqueidentifier modelId FK
        uniqueidentifier shareTypeId FK
        bit isCritical
        int yearOfManufacture
        decimal plannedMotohourPerDay
        decimal plannedMileagePerDay
        uniqueidentifier serviceZoneId FK
        uniqueidentifier costCenterId FK
        uniqueidentifier baseLocationId FK
        uniqueidentifier sectionId FK
        datetime2 createdAt
        uniqueidentifier createdBy
        datetime2 updatedAt
        uniqueidentifier updatedBy
        bit isDeleted
        datetime2 deletedAt
        uniqueidentifier deletedBy
    }

    EquipmentPhotos {
        uniqueidentifier id PK
        uniqueidentifier equipmentId FK
        nvarchar url
        int sortOrder
        bit isPrimary
        datetime2 createdAt
        uniqueidentifier createdBy
        datetime2 updatedAt
        uniqueidentifier updatedBy
        bit isDeleted
        datetime2 deletedAt
        uniqueidentifier deletedBy
    }

    EquipmentStates {
        uniqueidentifier id PK
        uniqueidentifier equipmentId FK
        uniqueidentifier statusTypeId FK
        nvarchar reason
        date startsAt
        date endsAt
        date actualEndsAt
        datetime2 cancelledAt
        uniqueidentifier sourceId FK
        datetime2 createdAt
        uniqueidentifier createdBy
        datetime2 updatedAt
        uniqueidentifier updatedBy
        bit isDeleted
        datetime2 deletedAt
        uniqueidentifier deletedBy
    }

    MeasurementUnits {
        uniqueidentifier id PK
        nvarchar code
        nvarchar displayName
        int sortOrder
        datetime2 createdAt
        uniqueidentifier createdBy
        datetime2 updatedAt
        uniqueidentifier updatedBy
        bit isDeleted
        datetime2 deletedAt
        uniqueidentifier deletedBy
    }

    Properties {
        uniqueidentifier id PK
        nvarchar code
        nvarchar nameEn
        nvarchar nameRu
        nvarchar nameKz
        uniqueidentifier dataTypeId FK
        uniqueidentifier unitId FK
        datetime2 createdAt
        uniqueidentifier createdBy
        datetime2 updatedAt
        uniqueidentifier updatedBy
        bit isDeleted
        datetime2 deletedAt
        uniqueidentifier deletedBy
    }

    PropertyEnumValues {
        uniqueidentifier id PK
        uniqueidentifier propertyId FK
        nvarchar value
        int sortOrder
        datetime2 createdAt
        uniqueidentifier createdBy
        datetime2 updatedAt
        uniqueidentifier updatedBy
        bit isDeleted
        datetime2 deletedAt
        uniqueidentifier deletedBy
    }

    EquipmentTypeProperties {
        uniqueidentifier id PK
        uniqueidentifier equipmentTypeId FK
        uniqueidentifier propertyId FK
        bit isRequired
        bit isFilterable
        bit isVisibleInCard
        int sortOrder
        datetime2 createdAt
        uniqueidentifier createdBy
        datetime2 updatedAt
        uniqueidentifier updatedBy
        bit isDeleted
        datetime2 deletedAt
        uniqueidentifier deletedBy
    }

    EquipmentProperties {
        uniqueidentifier id PK
        uniqueidentifier equipmentId FK
        uniqueidentifier propertyId FK
        nvarchar valueString
        bigint valueInt
        decimal valueDecimal
        bit valueBit
        uniqueidentifier propertyEnumValueId FK
        datetime2 createdAt
        uniqueidentifier createdBy
        datetime2 updatedAt
        uniqueidentifier updatedBy
        bit isDeleted
        datetime2 deletedAt
        uniqueidentifier deletedBy
    }

    MaintenancePartners {
        uniqueidentifier id PK
        nvarchar nameEn
        nvarchar nameRu
        nvarchar nameKz
        nvarchar phoneNumber
        nvarchar email
        nvarchar address
        datetime2 createdAt
        uniqueidentifier createdBy
        datetime2 updatedAt
        uniqueidentifier updatedBy
        bit isDeleted
        datetime2 deletedAt
        uniqueidentifier deletedBy
    }

    EquipmentMaintenanceContracts {
        uniqueidentifier id PK
        uniqueidentifier equipmentId FK
        uniqueidentifier partnerId FK
        nvarchar serviceType
        nvarchar notes
        datetime2 createdAt
        uniqueidentifier createdBy
        datetime2 updatedAt
        uniqueidentifier updatedBy
        bit isDeleted
        datetime2 deletedAt
        uniqueidentifier deletedBy
    }

    EquipmentFeedbacks {
        uniqueidentifier id PK
        uniqueidentifier bookingId FK
        uniqueidentifier equipmentId FK
        nvarchar feedback
        datetime2 createdAt
        uniqueidentifier createdBy
        datetime2 updatedAt
        uniqueidentifier updatedBy
        bit isDeleted
        datetime2 deletedAt
        uniqueidentifier deletedBy
    }

    EquipmentBookingAuthorizations {
        uniqueidentifier id PK
        uniqueidentifier equipmentId FK
        uniqueidentifier userId FK
        datetime2 createdAt
        uniqueidentifier createdBy
        datetime2 updatedAt
        uniqueidentifier updatedBy
        bit isDeleted
        datetime2 deletedAt
        uniqueidentifier deletedBy
    }

    SystemSettings {
        nvarchar key PK
        nvarchar value
        datetime2 createdAt
        uniqueidentifier createdBy
        datetime2 updatedAt
        uniqueidentifier updatedBy
        bit isDeleted
        datetime2 deletedAt
        uniqueidentifier deletedBy
    }

    BusinessPartners {
        uniqueidentifier id PK
        nvarchar nameEn
        nvarchar nameRu
        nvarchar nameKz
        nvarchar description
        nvarchar bin
        nvarchar country
        nvarchar city
        nvarchar address
        nvarchar email
        nvarchar phoneNumber
        nvarchar externalId
        bit isActive
        datetime2 createdAt
        uniqueidentifier createdBy
        datetime2 updatedAt
        uniqueidentifier updatedBy
        bit isDeleted
        datetime2 deletedAt
        uniqueidentifier deletedBy
    }

    Users {
        uniqueidentifier id PK
        nvarchar badgeNumber
        nvarchar fullName
        nvarchar email
        nvarchar jobTitle
        uniqueidentifier departmentId FK
        uniqueidentifier businessPartnerId FK
        uniqueidentifier userTypeId FK
        bit isActive
        datetime2 createdAt
        uniqueidentifier createdBy
        datetime2 updatedAt
        uniqueidentifier updatedBy
        bit isDeleted
        datetime2 deletedAt
        uniqueidentifier deletedBy
    }

    BookingRequests {
        uniqueidentifier id PK
        int requestNumber
        uniqueidentifier requestTypeId FK
        uniqueidentifier statusId FK
        uniqueidentifier closureReasonId FK
        nvarchar workOrderNumber
        nvarchar location
        nvarchar workDescription
        nvarchar comments
        uniqueidentifier priorityId FK
        uniqueidentifier requestorId FK
        uniqueidentifier createdBy
        datetime2 createdAt
        datetime2 updatedAt
        uniqueidentifier updatedBy
        bit isDeleted
        datetime2 deletedAt
        uniqueidentifier deletedBy
    }

    Bookings {
        uniqueidentifier id PK
        uniqueidentifier requestId FK
        uniqueidentifier equipmentId
        uniqueidentifier fleetId
        uniqueidentifier workCenterId FK
        uniqueidentifier statusId FK
        datetime2 plannedStartDateTime
        datetime2 plannedEndDateTime
        datetime2 actualStartDateTime
        datetime2 actualEndDateTime
        nvarchar justification
        uniqueidentifier closureReasonId FK
        datetime2 createdAt
        uniqueidentifier createdBy
        datetime2 updatedAt
        uniqueidentifier updatedBy
        bit isDeleted
        datetime2 deletedAt
        uniqueidentifier deletedBy
    }

    BookingStatuses {
        uniqueidentifier id PK
        uniqueidentifier bookingId FK
        uniqueidentifier statusId FK
        uniqueidentifier closureReasonId FK
        nvarchar comment
        datetime2 createdAt
        uniqueidentifier createdBy
        datetime2 updatedAt
        uniqueidentifier updatedBy
    }

    BookingApprovals {
        uniqueidentifier id PK
        uniqueidentifier bookingId FK
        uniqueidentifier approvalTypeId FK
        uniqueidentifier userId FK
        uniqueidentifier statusId FK
        nvarchar comment
        int approvalOrder
        datetime2 createdAt
        uniqueidentifier createdBy
        datetime2 updatedAt
        uniqueidentifier updatedBy
    }

    BookingTransportations {
        uniqueidentifier id PK
        uniqueidentifier bookingId FK
        uniqueidentifier transportingBookingId FK
        datetime2 createdAt
        uniqueidentifier createdBy
        datetime2 updatedAt
        uniqueidentifier updatedBy
    }

    BookingRequestStatuses {
        uniqueidentifier id PK
        uniqueidentifier requestId FK
        uniqueidentifier statusId FK
        uniqueidentifier closureReasonId FK
        nvarchar comment
        datetime2 createdAt
        uniqueidentifier createdBy
        datetime2 updatedAt
        uniqueidentifier updatedBy
    }

    EquipmentBrands ||--o{ EquipmentModels : "brandId"
    EquipmentTypes ||--o{ Equipments : "equipmentTypeId"
    Fleets ||--o{ Equipments : "fleetId"
    Fleets ||--o{ FleetManagePermissions : "fleetId"
    BusinessPartners ||--o{ Fleets : "businessPartnerId"
    WorkCenters ||--o{ EquipmentTypes : "workCenterId"
    WorkCenters ||--o{ Bookings : "workCenterId"
    Locations ||--o{ Equipments : "baseLocationId"
    CostCenters ||--o{ Equipments : "costCenterId"
    ServiceZones ||--o{ Equipments : "serviceZoneId"
    Divisions ||--o{ Groups : "divisionId"
    Groups ||--o{ Departments : "groupId"
    Departments ||--o{ Sections : "departmentId"
    Departments ||--o{ Users : "departmentId"
    Sections ||--o{ Equipments : "sectionId"

    ref_equipment_mobility_type ||--o{ EquipmentTypes : "mobilityTypeId"
    ref_equipment_class ||--o{ EquipmentTypes : "equipmentClassId"
    ref_fleet_type ||--o{ Fleets : "fleetTypeId"
    ref_fleet_manage_permission_type ||--o{ FleetManagePermissions : "permissionTypeId"
    ref_ownership_type ||--o{ Equipments : "ownershipTypeId"
    ref_share_type ||--o{ Equipments : "shareTypeId"
    ref_equipment_status_type ||--o{ EquipmentStates : "statusTypeId"
    ref_equipment_status_source ||--o{ EquipmentStates : "sourceId"
    ref_property_data_type ||--o{ Properties : "dataTypeId"
    ref_user_type ||--o{ Users : "userTypeId"
    ref_request_type ||--o{ BookingRequests : "requestTypeId"
    ref_request_priority ||--o{ BookingRequests : "priorityId"
    ref_booking_request_status ||--o{ BookingRequests : "statusId"
    ref_booking_request_status ||--o{ BookingRequestStatuses : "statusId"
    ref_request_closure_reason ||--o{ BookingRequests : "closureReasonId"
    ref_request_closure_reason ||--o{ BookingRequestStatuses : "closureReasonId"
    ref_booking_status ||--o{ Bookings : "statusId"
    ref_booking_status ||--o{ BookingStatuses : "statusId"
    ref_booking_closure_reason ||--o{ Bookings : "closureReasonId"
    ref_booking_closure_reason ||--o{ BookingStatuses : "closureReasonId"
    ref_booking_approval_type ||--o{ BookingApprovals : "approvalTypeId"
    ref_booking_approval_status ||--o{ BookingApprovals : "statusId"

    EquipmentBrands ||--o{ Equipments : "brandId"
    EquipmentModels ||--o{ Equipments : "modelId"
    Equipments ||--o{ EquipmentStates : "equipmentId"
    Equipments ||--o{ EquipmentPhotos : "equipmentId"
    Equipments ||--o{ EquipmentProperties : "equipmentId"
    Equipments ||--o{ EquipmentMaintenanceContracts : "equipmentId"
    Equipments ||--o{ EquipmentFeedbacks : "equipmentId"
    Equipments ||--o{ EquipmentBookingAuthorizations : "equipmentId"
    MeasurementUnits ||--o{ Properties : "unitId"
    Properties ||--o{ EquipmentProperties : "propertyId"
    Properties ||--o{ PropertyEnumValues : "propertyId"
    EquipmentTypes ||--o{ EquipmentTypeProperties : "equipmentTypeId"
    Properties ||--o{ EquipmentTypeProperties : "propertyId"
    PropertyEnumValues ||--o{ EquipmentProperties : "propertyEnumValueId"
    MaintenancePartners ||--o{ EquipmentMaintenanceContracts : "partnerId"

    BookingRequests ||--o{ Bookings : "requestId"
    Equipments ||--o{ Bookings : "equipmentId"
    Fleets ||--o{ Bookings : "fleetId"
    Bookings ||--o{ BookingStatuses : "bookingId"
    Bookings ||--o{ BookingApprovals : "bookingId"
    Bookings ||--o| BookingTransportations : "bookingId"
    Bookings ||--o| BookingTransportations : "transportingBookingId"
    Bookings ||--o{ EquipmentFeedbacks : "bookingId"
    BookingRequests ||--o{ BookingRequestStatuses : "requestId"

    Users ||--o{ FleetManagePermissions : "userId"
    Users ||--o{ EquipmentBookingAuthorizations : "userId"
    Users ||--o{ BookingApprovals : "userId"
    Users ||--o{ BookingRequests : "requestorId"
    BusinessPartners ||--o{ Users : "businessPartnerId"
```
