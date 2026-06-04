# Список FR по Equipment блоку

**Дата:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)  
**Основание:** `2026-05-05 - HDV HDE BRD v13.md`, `2026-05-14 - Схема БД v11 (Azure SQL, Equipments, Bookings).md`

---

## Правило отбора

В основной список включены только те FR, которые закрываются:
- созданием таблиц `Equipment`-блока в схеме БД v11;
- CRUD-операциями по этим таблицам и связанным справочникам.

Если в BRD v13 нет достаточно явного FR для сущности из `Equipment`-блока, она вынесена в раздел `Additional FRs`.

---

## Основной список FR

| FR | Формулировка | Чем покрывается | Таблицы | Комментарий |
|---|---|---|---|---|
| FR-005 | Admin creates fleets with unique names | Создание таблицы + CRUD справочника/сущности Fleet | `Fleets` | Уникальность имени поддерживается бизнес-правилом и CRUD-операциями над флотами |
| FR-006 | Admin links internal fleet with AAD group | CRUD сущности Fleet | `Fleets` | Поддерживается полем `aadGroupId` |
| FR-007 | Admin manages external fleet owners | CRUD справочников и связанных сущностей | `Fleets`, `Users`, `BusinessPartners` | В BRD сформулировано общо; на уровне схемы это покрывается через данные флота, пользователей и BP |
| FR-008 | Admin updates existing fleets | CRUD сущности Fleet | `Fleets` | Прямое покрытие update-операциями |
| FR-009 | Admin deletes fleets | CRUD сущности Fleet | `Fleets` | Предполагается soft delete по audit conventions |
| FR-010 | Admin configures dynamic custom characteristics per equipment type via Admin Panel tool | CRUD EAV-модели и связки тип-свойство | `EquipmentTypes`, `Properties`, `MeasurementUnits`, `PropertyEnumValues`, `EquipmentTypeProperties`, `EquipmentProperties` | Прямое покрытие структуры динамических атрибутов |
| FR-013 | Admin creates equipment (internal fleet) | CRUD сущности Equipment | `Equipments` | Основная карточка техники и обязательные связи хранятся в `Equipments` |
| FR-014 | Admin edits equipment (extended edit) | CRUD сущности Equipment и связанных данных карточки | `Equipments`, `EquipmentStates`, `EquipmentPhotos`, `EquipmentProperties` | Расширенное редактирование включает статусы, фото и динамические свойства |
| FR-015 | Admin deletes equipment | CRUD сущности Equipment | `Equipments` | Предполагается soft delete по audit conventions |
| FR-NEW-04 | Admin authorizes specific users to book Assigned equipment | CRUD таблицы авторизаций | `EquipmentBookingAuthorizations`, `Equipments`, `Users` | Прямое покрытие списков авторизованных пользователей на единицу техники |
| FR-NEW-50 | Admin manages all reference/handbook values via Admin Panel | CRUD справочников Equipment-блока | `WorkCenters`, `EquipmentBrands`, `EquipmentModels`, `Locations`, `CostCenters`, `ServiceZones`, `Divisions`, `Groups`, `Departments`, `Sections`, `MeasurementUnits`, `Properties`, `PropertyEnumValues`, `MaintenancePartners`, `BusinessPartners`, `EquipmentTypes` | Это umbrella-FR для справочников и handbook-значений |
| FR-017 | Both internal and external FO can create equipment under own fleet | CRUD сущности Equipment | `Equipments`, `Fleets` | Табличная модель позволяет создавать технику с привязкой к флоту |
| FR-018 | FO must indicate fleet of equipment; only own fleets selectable | Создание связи Equipment -> Fleet + CRUD | `Equipments`, `Fleets`, `FleetManagePermissions` | Ограничение "only own fleets" требует прикладной фильтрации в CRUD/UI |
| FR-019 | FO can edit allowed equipment parameters; one user can own several fleets | CRUD карточки техники и связанных сущностей | `Equipments`, `EquipmentStates`, `EquipmentPhotos`, `EquipmentProperties`, `FleetManagePermissions` | Поддерживает редактирование разрешённых полей и владение несколькими флотами |
| FR-020 | FO can delete equipment (soft delete) | CRUD сущности Equipment | `Equipments` | Предполагается soft delete |
| FR-021 | FO can freeze/unfreeze equipment for a period or indefinitely | CRUD статусов техники | `EquipmentStates`, `Equipments` | Покрывается статусами с `startsAt`, `endsAt`, `actualEndsAt`; текущее состояние определяется по `EquipmentStates` |
| FR-022 | Requestor can submit feedback on equipment with confirmed booking | CRUD feedback-сущности | `EquipmentFeedbacks` | Хранение фидбэка по booking и equipment покрыто явно |
| FR-NEW-05 | FO uploads multiple photos; Requestor sees them in request form | CRUD фото техники | `EquipmentPhotos` | Множественные фото покрыты отдельной таблицей |
| FR-NEW-08 | Assigned equipment: visible to all users; bookable only by Admin-authorized users with mandatory justification; FO can approve or decline | Структура данных для типа использования и авторизаций | `Equipments`, `EquipmentBookingAuthorizations` | Из этого FR данным набором таблиц закрывается именно часть про Assigned-тип и авторизованных пользователей; justification/approval закрываются за пределами Equipment-блока |
| FR-NEW-49 | Freeze fields on equipment card: Заморозка, Причина заморозки, Дата завершения заморозки | CRUD статусов техники | `EquipmentStates`, `Equipments` | Поля заморозки покрываются статусной моделью |

---

## Additional FRs

Ниже перечислены короткие рабочие FR, для которых в BRD v13 нет достаточно явной отдельной формулировки, но которые следуют из `Equipment`-блока схемы БД v11 и потребуются для CRUD/UI.

| FR | Формулировка | Чем покрывается | Таблицы | Комментарий |
|---|---|---|---|---|
| AFR-01 | Admin manages equipment brands directory | CRUD справочника | `EquipmentBrands` | В BRD есть reference/handbook policy и общий FR-NEW-50, но нет отдельного явного FR по брендам |
| AFR-02 | Admin manages equipment models directory | CRUD справочника | `EquipmentModels` | Отдельный CRUD по моделям следует из схемы |
| AFR-03 | Admin manages organizational and location handbooks used in equipment card | CRUD справочников | `Locations`, `CostCenters`, `ServiceZones`, `Divisions`, `Groups`, `Departments`, `Sections` | В BRD перечислены как handbook-managed, но без отдельной детализации по каждому набору сущностей |
| AFR-04 | Admin manages maintenance partners directory | CRUD справочника | `MaintenancePartners` | В BRD есть упоминание Maintenance BP, но нет явного отдельного FR на CRUD этого справочника |
| AFR-05 | Admin manages business partners directory used in equipment data model | CRUD справочника | `BusinessPartners` | Требуется для внешних контрагентов в unified equipment model |
| AFR-06 | Admin manages equipment maintenance contracts by equipment, partner and service type | CRUD связующей сущности | `EquipmentMaintenanceContracts` | Для этой сущности в BRD v13 нет отдельного FR |
| AFR-07 | Admin manages equipment types including class, mobility, work center and sorting attributes | CRUD справочника/сущности типа техники | `EquipmentTypes` | BRD явно описывает dynamic characteristics, но не формулирует отдельный FR на CRUD самих типов техники |

---

## Примечания

- `FR-NEW-41` про multiple trackers не включён в основной список: в схеме БД v11 нет отдельной таблицы для tracker-ов.
- `FR-NEW-06`, `FR-NEW-07`, `FR-NEW-09`, `FR-NEW-40`, `FR-NEW-68` не включены в основной список, потому что они требуют не только таблиц и CRUD, но и дополнительной поисковой, интеграционной или UI-логики.
- `FR-NEW-42` не включён в основной список отдельно: в BRD это самостоятельный FR, но в v11 нет явного отдельного поля `shared team email` на уровне `Fleets`; ближайшая опора в схеме — `Users.sharedEmail`, поэтому требуется отдельное архитектурное уточнение.
