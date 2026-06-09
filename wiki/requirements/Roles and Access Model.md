# Roles and Access Model

**Created:** 2026-06-09  
**Last updated:** 2026-06-09  
**Автор документов:** OpenCode

---

## Назначение

Документ фиксирует опубликованную модель ролей и доступа для HDV/HDE Booking Tool в Phase 1.

Документ нужен как единая reference-точка для:

- backend authorization;
- UI visibility rules;
- use case-моделирования;
- API permission mapping;
- анализа approval chains.

---

## Источники

Документ опирается на опубликованные материалы:

- [`wiki/brd/BRD.md`](../brd/BRD.md)
- [`wiki/glossary/Glossary.md`](../glossary/Glossary.md)
- [`wiki/api/booking/Booking API summary.md`](../api/booking/Booking%20API%20summary.md)
- [`wiki/requirements/usecases/Use Cases.md`](usecases/Use%20Cases.md)

---

## Роли в scope

Подтвержденные system/business roles в текущем scope:

- `Requestor`
- `Service Work Processor`
- `Fleet Owner`
- `FleetOwners' Supervisor`
- `Transportation Responsible`
- `Administrator`

Общая рамка:

- система использует role-based access control;
- provisioning ролей выполняется через Azure Active Directory / AAD groups;
- доступ должен определяться не только ролью, но и business context-ом конкретной сущности: own request, own fleet, supervisor queue, transport queue и т.д.

---

## Каталог ролей

| Роль | Назначение | Основной scope доступа | Provisioning / assignment | Ключевые действия |
|---|---|---|---|---|
| `Requestor` | Базовый пользователь, создающий и отправляющий обычные заявки на технику | Собственные `Regular Requests`, связанные booking item-ы, история собственных заявок, поиск внутренней техники | По BRD роль есть у пользователей сети TCO по умолчанию | Создать заявку, отправить заявку, продлить бронь, закрыть бронь вручную, оставить feedback по технике, смотреть usage rate забронированной техники, бронировать `Assigned` технику при отдельной авторизации |
| `Service Work Processor` | Специализированный пользователь для JDE-sourced Service Work Requests | Только JDE-сгенерированные `Service Work Requests`, связанные Work Orders и их processing flow | Dedicated system role | Просматривать SWR, управлять и отправлять SWR в обработку |
| `Fleet Owner` | Владелец / управляющий fleet-ом, принимающий решения по бронированиям и управляющий техникой флота | Очереди согласования и карточки броней по собственным fleet-ам, карточки собственной техники, usage rate по своей технике | AAD group per fleet; один пользователь может владеть несколькими fleet-ами; у одного fleet может быть 2+ FO | Создавать/редактировать/удалять технику своего fleet-а, confirm/decline booking, менять период, менять технику, terminate confirmed booking, freeze/unfreeze equipment, фиксировать `Mobilization started`, вручную закрывать бронь |
| `FleetOwners' Supervisor` | Финальный согласующий для `Long-term rented` после решения FO | Только queue и detail view long-term rented броней с pending шагом `SupervisorApproval` | Dedicated AAD-managed role | Получить уведомление, просмотреть justification, финально подтвердить бронь или отклонить ее с обязательным комментарием |
| `Transportation Responsible` | Команда, отвечающая за транспортировку `Unwheeled` техники | Transport queue и transport detail по заявкам, требующим транспортировки | Dedicated system role | Получать уведомление о transport request, подтверждать или отклонять транспортировку |
| `Administrator` | Процессный / системный администратор справочников, fleet-структуры и конфигурации | Admin Panel, handbook/reference data, fleet setup, system settings, отчетность | AAD group | Управлять fleet-ами и AAD groups, управлять техникой с расширенными правами, настраивать dynamic characteristics, reference values, configurable system parameters, shared team email, Assigned authorization, Supervisor timeout |

---

## Специальные access concepts

Помимо базовых ролей, в опубликованной модели используются специальные access concepts:

| Термин | Источник | Смысл |
|---|---|---|
| `Owner Access` | `wiki/glossary/Glossary.md` | Доступ к управлению fleet через `FleetManagePermissions.permissionType = Owner` |
| `Delegated Access` | `wiki/glossary/Glossary.md` | Доступ к управлению fleet через `FleetManagePermissions.permissionType = Delegated` |
| `Fleet Management Access` | `wiki/glossary/Glossary.md` | Обобщающий доступ к fleet для пользователей с `Owner` или `Delegated` assignment |

Практический смысл:

- часть FO-сценариев должна проверять не только роль `FleetOwner`, но и наличие права управлять именно данным fleet;
- для некоторых published artifacts разрешение может быть сформулировано как доступ пользователя к fleet management context, а не только как буквальное наличие роли `FleetOwner`.

---

## Approval responsibility model

### 1. TCO Owned

Approval chain:

`Requestor -> Fleet Owner -> Confirmed`

Смысл:

- пользователь создает и отправляет booking request;
- Fleet Owner принимает базовое решение;
- после положительного решения бронь может перейти к исполнению и далее в `InProgress` / `Closed`.

### 2. Long-term rented

Approval chain:

`Requestor + Justification -> Fleet Owner -> Confirmed by FO -> FleetOwners' Supervisor -> Confirmed / Declined`

Смысл:

- `Requestor` обязан указать `Justification` на уровне booking item;
- решение FO для этой категории не является финальным;
- финальное решение принимает `FleetOwners' Supervisor`.

### 3. Unwheeled transport flow

Approval chain:

`Requestor -> Fleet Owner -> Transportation Responsible -> Fleet Owner final context / execution continuation`

Смысл:

- для техники, требующей транспортировки, появляется отдельная транспортная роль;
- транспортное решение хранится как approval step транспортной роли;
- published API summary уже выделяет отдельный transport UI/API block.

---

## Access by functional area

| Functional area | Main roles |
|---|---|
| Request creation and own booking management | `Requestor`, `Service Work Processor` |
| Fleet approval and booking execution management | `Fleet Owner` |
| Final approval for long-term rented | `FleetOwners' Supervisor` |
| Transport decision flow for unwheeled equipment | `Transportation Responsible` |
| Admin Panel, references, configuration, fleet setup | `Administrator` |
| Audit / timeline / status history | Зависит от доступа к конкретной брони; published artifacts допускают `Requestor`, `ServiceWorkProcessor`, `FleetOwner`, `FleetOwnersSupervisor`, `Admin` при наличии доступа к объекту |

---

## API surface by role

Ниже зафиксирована high-level role-to-API mapping по опубликованной сводке API.

| Роль | Основные published API areas |
|---|---|
| `Requestor` | `booking/requestor/*`, `booking/reference/*`, часть `booking/common/*`, usage/reporting read flows для собственного контекста |
| `Service Work Processor` | Requestor-like read/write flows для `Service Work Requests`; отдельные ограничения определяются типом заявки и business context-ом JDE-sourced requests |
| `Fleet Owner` | `booking/fleet-owner/*`, часть `booking/common/*`, FO approval queues и execution actions |
| `FleetOwners' Supervisor` | `booking/supervisor/*`, часть `booking/common/*` |
| `Transportation Responsible` | `booking/transport/*` |
| `Administrator` | `api/admin/*`, reporting/configuration/reference management |

---

## Role boundaries and non-goals

1. `CC Owner` не входит в текущую актуальную role model для booking flow.
   В BRD эта роль помечена как obsolete для текущего scope.

2. `On-demand BP (Showcase)` не создает отдельную booking approval role внутри Phase 1.
   Этот поток в текущем scope является showcase-only и не участвует в обычном internal booking workflow.

3. Delegation не является in-system feature.
   Для `Fleet Owner` делегирование управляется через AAD group membership вне системы.

4. Access decision не должен строиться только на текстовом названии роли.
   Для большинства действий нужен дополнительный business context:
   - принадлежит ли request пользователю;
   - относится ли booking к fleet-у пользователя;
   - находится ли booking в supervisor queue;
   - относится ли booking к transport queue.

---

## Role-to-entity access summary

| Role | `BookingRequests` | `Bookings` | `Equipments` | `Fleets` | `Approvals` | `Reports / Dashboards` |
|---|---|---|---|---|---|---|
| `Requestor` | Own regular requests | Own bookings | Search/view in booking context | Reference / search context only | No approval authority | Own booking usage context; own history |
| `Service Work Processor` | JDE-sourced service work requests in accessible context | Bookings inside SWR context | Search/view in booking context | Reference / search context only | No FO/Supervisor authority | SWR-related operational context |
| `Fleet Owner` | Requests relevant to own fleet bookings | Bookings of own fleets | Own fleet equipment management | Own fleets | FO approval authority | Own fleet usage rate |
| `FleetOwners' Supervisor` | Indirectly through long-term rented booking context | Only long-term rented bookings in supervisor queue | Read-only context as needed for approval | No fleet management | Final approval authority for long-term rented | Not primary reporting role |
| `Transportation Responsible` | Indirectly through transport queue | Transport-related booking context | Read-only context for transport decision | No fleet management | Transport decision authority | Not primary reporting role |
| `Administrator` | Cross-cutting admin/reporting context | Cross-cutting admin/reporting context | Extended management rights | Full setup/management | No business approval role by default; manages configuration, not operational approval chain | Full admin/reporting access |

---

## Notes for implementation

1. UI visibility should be derived from both role and object context.
2. Backend authorization should validate both role and ownership / queue membership.
3. `Requestor`, `Fleet Owner`, `FleetOwners' Supervisor`, and `Transportation Responsible` are operational roles in workflow.
4. `Administrator` is a configuration and governance role, not the default operational approver in booking chains.
5. Any future dedicated document on permissions should refine this model into endpoint-level authorization rules, but must not contradict this published role catalog.
