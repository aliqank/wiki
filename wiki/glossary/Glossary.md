# Glossary

**Created:** 2026-05-15  
**Last updated:** 2026-06-03  
**Автор документов:** Telman Nurzhanov (SA)

---

В этот файл публикуется утвержденный глоссарий терминов и краткие описания ключевых таблиц БД, готовые к использованию в разработке.

Источники:
- `source/results/2026-05-05 - HDV HDE BRD v13.md`
- `wiki/db/2026-05-21 - DB Schema v12 (Azure SQL, Equipments, Booking).md`

---

## Business Terms

| Term | Definition |
|---|---|
| Request | Контейнер, объединяющий несколько бронирований техники; одно бронирование соответствует одной единице техники. |
| Unfinished Request | Незавершенная заявка: заявка, которая еще не перешла в terminal status и поэтому должна отображаться на странице `Мои заявки`. Для текущего scope термин означает request со статусом `Draft`, `Submitted` или `InProgress`. Заявки в статусе `Closed` относятся к completed/history flow и не входят в список незавершенных заявок. |
| Regular Request | Заявка, созданная пользователем вручную в HDV/HDE booking tool. |
| Service Work Request | Заявка, созданная автоматически по API на основе Work Order из JDE E1, когда шаг WO достигает нужного статуса. Доступна только роли Service Work Processor. |
| Booking | Отдельное бронирование внутри заявки, привязанное к одной конкретной единице техники. |
| Equipment | Единица техники HDV/HDE, доступная для поиска, просмотра и бронирования в системе. Каждая единица относится к определённому fleet. |
| Fleet | Логическая группа техники, закреплённая за Fleet Owner. Для внутренних флотов это, например, Maintenance, Construction, TFM; для внешних - BP/контрагент. |
| Fleet Owner (FO) | Пользователь, отвечающий за подтверждение или отклонение бронирований внутри своего флота. Один fleet может иметь двух и более FO. |
| Owner Access | Доступ пользователя к управлению fleet через `FleetManagePermissions.permissionType = Owner`. Означает, что пользователь является основным владельцем / управляющим данного флота в рамках booking-процессов. |
| Delegated Access | Доступ пользователя к управлению fleet через `FleetManagePermissions.permissionType = Delegated`. Означает, что пользователь действует по делегированному праву управления данным флотом. |
| Fleet Management Access | Обобщающий термин для доступа пользователя к fleet через `FleetManagePermissions` с типом `Owner` или `Delegated`. Используется в use cases и API, когда действие разрешено любому пользователю, имеющему право управлять данным fleet независимо от типа assignment. |
| FleetOwners' Supervisor | Роль, отвечающая за финальное согласование бронирований Long-term rented после подтверждения со стороны FO. |
| Approver | Согласующий по бронированию: Fleet Owner для внутренних цепочек и FleetOwners' Supervisor для финального решения по Long-term rented. |
| Justification | Обязательное текстовое обоснование. Используется для сценариев, где бронирование требует объяснения со стороны Requestor. |
| Justification (Long-term rented) | Обязательное текстовое поле на уровне booking item для Long-term rented техники; объясняет, почему не выбрана TCO Owned техника. |
| Mobilization | Период подготовки и перемещения техники до места работ. Период брони начинается с момента начала mobilization. |
| Assigned | Закреплённая техника: видна всем пользователям, но доступна к бронированию только заранее авторизованным пользователям с обязательным обоснованием. |
| Shared without Conditions | Общедоступная техника без дополнительных условий на бронирование. |
| Shared with Conditions | Общедоступная техника, для бронирования которой Requestor обязан указать обоснование. |
| TCO Owned | Техника, принадлежащая TCO. Для бронирования достаточно согласования Fleet Owner. |
| Long-term rented | Долгосрочно арендованная техника, управляемая внутренним FO. Требует justification, подтверждения FO и финального решения Supervisor. |
| On-demand BP (Showcase) | Техника BP, отображаемая в Phase 1 как витрина/каталог без полного booking-потока внутри системы. |
| Dynamic Characteristics | Пользовательские характеристики техники, настраиваемые по типу техники через Admin Panel и автоматически отображаемые в карточке, форме заявки и фильтрах. |
| Usage Rate | Показатель использования техники по пробегу, моточасам или обоим показателям в зависимости от типа техники. |
| Criticality | Признак критичности техники; означает необходимость security escort при транспортировке, а не приоритет ремонта. |
| Work Center | Рабочий центр или шаг Work Order. Привязан к шагу работ и используется для сопоставления типов техники и производственных задач. |
| Historical Route | История перемещения техники на карте за выбранный период на основе tracker-данных из DataLake. |
| Booking Extension Approval | Отдельный approval step на изменение `plannedEndDateTime` брони по запросу Requestor. Используется, когда изменение срока требует решения Fleet Owner и не должно смешиваться с базовым FO approval при первоначальном согласовании брони. |

---

## Availability And Conflict Terms

### Hard Availability Restriction

Абсолютное ограничение доступности техники, при котором действие с бронью или выбор техники должно быть отклонено backend-ом независимо от наличия или отсутствия competing bookings.

Типовые примеры:
- по `EquipmentStates` для выбранного периода есть актуальный статус `Decommissioned`, `Frozen` или `InRepair`;
- техника не допускается к обычному booking flow по обязательным бизнес-правилам, например `OnDemand` техника вне scope соответствующего сценария;
- для Assigned техники отсутствует обязательная authorization-запись в `EquipmentBookingAuthorizations`, если сценарий требует такую проверку.

Явные примеры:
- Пользователь пытается добавить технику в draft-заявку, но по `EquipmentStates` у этой техники на весь выбранный период есть активная запись `InRepair`. Результат: backend возвращает `EQUIPMENT_NOT_AVAILABLE`.
- Fleet Owner пытается изменить период брони, но техника уже имеет актуальный статус `Frozen` на новый диапазон дат. Результат: изменение периода блокируется независимо от других броней.
- Requestor выбирает Assigned технику, но для пары `equipmentId + userId` нет записи в `EquipmentBookingAuthorizations`. Результат: техника считается недоступной по hard restriction даже если в `Bookings` нет пересечений.

Практический смысл:
- hard availability restriction блокирует add/edit/submit/confirm/change-period/change-equipment сценарии;
- такие ограничения должны возвращать ошибку уровня `EQUIPMENT_NOT_AVAILABLE` или эквивалентную бизнес-ошибку;
- competing bookings сами по себе не являются hard availability restriction.

### Booking Conflict Context

Информационный контекст о пересечении одной брони с другими активными бронями той же техники. Используется для принятия решения пользователем, но сам по себе не является автоматическим запретом на действие.

Типовые примеры:
- в `Bookings` уже есть другая запись по тому же `equipmentId` со статусом `Submitted`, `Confirmed` или `InProgress`, и ее диапазон пересекается с новым или текущим периодом брони;
- для одной техники существует несколько competing bookings, которые должны быть показаны через load summary, conflict indicator или отдельный conflict list.

Явные примеры:
- Requestor ищет экскаватор на период `20.05 08:00 - 22.05 18:00`, а в `Bookings` уже есть другая бронь этой же техники со статусом `Confirmed` на период `21.05 09:00 - 21.05 20:00`. Результат: техника остается доступной для выбора, но UI показывает conflict indicator и load summary.
- Fleet Owner подтверждает бронь, у которой по тому же `equipmentId` уже есть другая активная бронь в статусе `Submitted`. Результат: confirm не блокируется автоматически; FO принимает решение на основании conflict context.
- Fleet Owner меняет период брони, и новый `plannedEndDateTime` начинает пересекаться с другой бронью той же техники в статусе `InProgress`. Результат: backend возвращает conflict context для UI, но не обязан отклонять изменение только из-за самого overlap.

Практический смысл:
- booking conflict context должен быть рассчитан и показан в UI как summary или detail view;
- наличие conflict context не должно автоматически блокировать add/edit/submit/change-period/confirm, если отсутствуют hard availability restrictions;
- окончательное решение по conflict context принимает Requestor или Fleet Owner в зависимости от сценария.

---

## Equipment Tables

| Table | Description | Key Relations |
|---|---|---|
| `EquipmentTypes` | Справочник типов техники. Хранит класс техники, признак необходимости транспортировки, mobility type, work center и порядок сортировки. | `WorkCenters`, `ref_equipment_mobility_type`, `ref_equipment_class` |
| `Fleets` | Справочник флотов. Хранит название флота, тип флота и AAD group Fleet Owner; ownership больше не хранится прямо в записи флота. | `ref_fleet_type`, `FleetManagePermissions` |
| `WorkCenters` | Справочник рабочих центров, используемый при типизации техники и booking-контексте. | Используется в `EquipmentTypes`, `Bookings` |
| `EquipmentBrands` | Справочник брендов/производителей техники. | Используется в `EquipmentModels`, `Equipments` |
| `EquipmentModels` | Справочник моделей техники с привязкой к бренду. | `EquipmentBrands`, `Equipments` |
| `Locations` | Справочник базовых локаций техники. | Используется в `Equipments` |
| `CostCenters` | Справочник cost center для привязки техники к организационной структуре. | Используется в `Equipments` |
| `ServiceZones` | Справочник сервисных зон техники. | Используется в `Equipments` |
| `Divisions` | Справочник division в оргструктуре. | Родитель для `Groups` |
| `Groups` | Справочник groups в оргструктуре. | `Divisions`, родитель для `Departments` |
| `Departments` | Справочник departments. Используется, в том числе, для отчётности и оргструктуры пользователей. | `Groups`, родитель для `Sections`, используется в `Users` |
| `Sections` | Справочник sections / unit-level привязки техники. | `Departments`, используется в `Equipments` |
| `FleetManagePermissions` | Таблица владения и делегирования управления fleet для конкретных пользователей, включая тип assignment и срок действия доступа. | `Fleets`, `Users`, `ref_fleet_manage_permission_type` |
| `Equipments` | Основная таблица карточек техники. Хранит принадлежность, тип, статус, идентификаторы, бренд/модель, критичность, плановые показатели и оргпривязки. | `EquipmentTypes`, `Fleets`, `EquipmentBrands`, `EquipmentModels`, `ServiceZones`, `CostCenters`, `Locations`, `Sections`, reference tables статусов/типов |
| `EquipmentPhotos` | Фотографии единицы техники с признаком primary и сортировкой. | `Equipments` |
| `EquipmentStates` | История статусов техники: заморозка, ремонт, вывод из эксплуатации и другие статусы с периодами действия. | `Equipments`, `ref_equipment_status_type`, `ref_equipment_status_source` |
| `MeasurementUnits` | Справочник единиц измерения для динамических характеристик. | Используется в `Properties` |
| `Properties` | Справочник определений динамических свойств техники. | `MeasurementUnits`, `ref_property_data_type`, используется в `EquipmentTypeProperties`, `EquipmentProperties`, `PropertyEnumValues` |
| `PropertyEnumValues` | Справочник допустимых enum-значений для properties. | `Properties`, используется в `EquipmentProperties` |
| `EquipmentTypeProperties` | Настройка того, какие свойства доступны для конкретного типа техники, включая обязательность, видимость и filterability. | `EquipmentTypes`, `Properties` |
| `EquipmentProperties` | Значения динамических характеристик для конкретной единицы техники. | `Equipments`, `Properties`, `PropertyEnumValues` |
| `MaintenancePartners` | Справочник сервисных/ремонтных партнёров. | Используется в `EquipmentMaintenanceContracts` |
| `EquipmentMaintenanceContracts` | Связка техники с maintenance partner и видом сервиса. | `Equipments`, `MaintenancePartners` |
| `EquipmentFeedbacks` | Отзывы по технике, оставленные в контексте конкретной брони. | `Equipments`, `Bookings` |
| `EquipmentBookingAuthorizations` | Таблица авторизаций пользователей на бронирование Assigned техники. | `Equipments`, `Users` |

---

## Service Tables

| Table | Description | Key Relations |
|---|---|---|
| `SystemSettings` | Таблица системных настроек вида key-value для параметров, которые не должны быть захардкожены. | Используется прикладной логикой и Admin Panel |
| `Users` | Справочник пользователей системы, включая ФИО, email, должность, department, тип пользователя и BP-привязку. | `Departments`, `BusinessPartners`, `FleetManagePermissions`, `EquipmentBookingAuthorizations`, `BookingApprovals` |
| `BusinessPartners` | Справочник внешних компаний/контрагентов. | Используется в `Users` и сценариях внешних сущностей |

---

## Booking Tables

| Table | Description | Key Relations |
|---|---|---|
| `BookingRequests` | Заголовок заявки на бронирование: request-level данные, инициатор, тип заявки, приоритет, WO business-поля и агрегированный статус. | Связан с `Users`, `BookingRequestStatuses`, `Bookings` |
| `Bookings` | Отдельные booking items внутри заявки. Хранят оборудование, период, текущий lifecycle status и фактические атрибуты выполнения брони. | `BookingRequests`, `Equipments`, `WorkCenters`, `BookingStatuses`, `BookingApprovals` |
| `BookingTransportations` | Таблица transport linkage между основной бронью и бронью, выполняющей транспортировку. | `Bookings` x2 |
| `BookingApprovals` | Журнал шагов согласования брони. Хранит отдельные решения FO и Supervisor с типом шага, результатом, комментарием, очередностью и аудитом. | `Bookings`, `Users`, `ref_booking_approval_type`, `ref_booking_approval_status` |
| `BookingStatuses` | История статусов individual booking с периодами и источником изменения. | `Bookings` |
| `BookingRequestStatuses` | История статусов request-level сущности. | `BookingRequests` |

---

## Notes

| Topic | Clarification |
|---|---|
| Shared team email | В текущей схеме v12 отдельное поле для shared team email не зафиксировано; если бизнес-атрибут останется в scope, для него потребуется отдельное уточнение модели. |
| Dynamic characteristics | Глоссарий и таблицы EAV отражают новую модель dynamic characteristics; это замена старой идеи статических шаблонов. |
| Assigned authorization | Возможность бронировать Assigned технику отделена от общей видимости техники и хранится в `EquipmentBookingAuthorizations`. |
| On-demand BP | On-demand BP техника присутствует в бизнес-модели, но не должна участвовать в обычных `Bookings` по правилу схемы v12. |
