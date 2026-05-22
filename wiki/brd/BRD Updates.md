# BRD Updates

**Created:** 2026-05-20  
**Last updated:** 2026-05-20  
**Автор документов:** Telman Nurzhanov (SA)

---

## Назначение

Файл фиксирует бизнес-уточнения и изменения правил, появившиеся после согласования и подписания `BRD.md`.

Правило проекта:

- `BRD.md` не редактируется после согласования;
- все новые договоренности, уточнения и отклонения от BRD фиксируются только в этом файле;
- при конфликте между `BRD.md` и этим файлом для дальнейшей аналитики, UI/API-документации и реализации используется более поздняя запись из этого файла.

---

## Updates Register

| ID | Date | Area | Type | Summary | Affects BRD |
|---|---|---|---|---|---|
| `BRD-U-001` | 2026-05-20 | Request lifecycle | Business rule clarification | Request lifecycle uses single terminal status `Closed`; pre-start closure uses `requestClosureReason = Closed`, post-start closure uses `Completed` | `FR-026`, `FR-NEW-17`, `FR-NEW-44` |
| `BRD-U-002` | 2026-05-20 | Booking lifecycle | Business rule clarification | Draft booking item cancelled with parent request uses `Closed` + booking closure reason `Cancelled` | `FR-027`, `FR-042`, `FR-NEW-17` |
| `BRD-U-003` | 2026-05-20 | Request lifecycle | Business rule clarification | Separate request-level withdraw flow allowed for `Submitted` request before first FO confirmation in the request | `FR-025`, `FR-058`, `FR-NEW-17` |

---

## Equipment Additional FRs

Ниже зафиксированы только новые FR по Equipment-блоку, которых нет как достаточно явных самостоятельных FR в согласованном `BRD.md`.

Источник:

- `source/results/2026-05-15 - Список FR по Equipment блоку.md`

Назначение раздела:

- поддерживать только post-BRD additions поверх согласованной версии BRD;
- не дублировать уже существующие FR из `BRD.md`;
- использовать как рабочий реестр новых Equipment FR для CRUD/UI/API scope.

### New Additional FRs

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

## BRD-U-001 - Request terminal status semantics

### Причина

Текущая трактовка `Completed` на уровне заявки воспринимается как успешное выполнение, хотя request-level статус в системе должен отражать завершение жизненного цикла заявки, а не бизнес-успешность каждой брони.

### Новое правило

Для request-level статусов используется следующий набор:

- `Draft`
- `Submitted`
- `InProgress`
- `Closed`

Для request-level closure reasons используется следующий набор:

- `Closed`
- `Completed`

### Семантика terminal statuses / closure reasons

- `Closed` + `requestClosureReason = Closed`
  Используется для request-level отмены draft-заявки до submit, а также для submitted-заявки, которая была закрыта до перехода в `InProgress`.

- `Closed` + `requestClosureReason = Completed`
  Используется если заявка была отправлена и в ней больше не осталось активных броней. Причины завершения отдельных броней могут быть разными: `Completed`, `Revoked`, `Declined`, `Terminated`.

### Примеры

1. Все брони `Revoked` / `Declined` до старта.
   Итоговый request status = `Closed`, `requestClosureReason = Completed`.

2. Одна бронь `Closed`, другая `Terminated`.
   Итоговый request status = `Closed`, `requestClosureReason = Completed`.

3. Draft-заявка отменена пользователем.
   Итоговый request status = `Closed`, `requestClosureReason = Closed`.

### Правило приоритета

Если после submit больше нет активных item-ов, request status = `Closed`, `requestClosureReason = Completed`.

### Влияние на документацию и реализацию

Эта запись должна использоваться при обновлении:

- request status rules;
- API-документации list/history/report endpoints;
- UI-лейблов и фильтров по request status;
- backend-логики пересчета `BookingRequest.status`;
- модели reference-статусов заявки в БД и API.

---

## BRD-U-002 - Booking Cancelled for cancelled draft request

### Причина

Если draft-заявка отменяется, связанные draft booking item-ы не должны оставаться в статусе `Draft`, потому что это затрудняет анализ черновиков и аудит их конечного состояния.

### Новое правило

При отмене draft-заявки через `POST /booking-requests/{id}/cancel`:

- `BookingRequest.status` переводится в `Closed`, `requestClosureReason = Closed`;
- все связанные booking item-ы в статусе `Draft` переводятся в `Closed` с `closureReason = Cancelled` либо удаляются;
- по каждому измененному booking item создается запись в `BookingStatuses`.

### Семантика `Booking closureReason = Cancelled`

- `Booking.closureReason = Cancelled` используется только для booking item-ов, отмененных вместе с родительской draft-заявкой до submit;
- `Booking.closureReason = Cancelled` не используется для submitted booking item;
- после submit requestor-initiated cancel на уровне item по-прежнему выражается статусом `Revoked`.

### Разграничение с соседними статусами

- `Draft`
  Живой, еще не отмененный draft booking item.

- `Closed` + `closureReason = Cancelled`
  Draft booking item, завершенный отменой родительской draft-заявки.

- `Revoked`
  Submitted booking item, отозванный requestor-ом до решения FO.

### Влияние на документацию и реализацию

Эта запись должна использоваться при обновлении:

- booking status reference list;
- API `POST /booking-requests/{id}/cancel`;
- request/booking history and audit trail;
- аналитики по draft lifecycle.

---

## BRD-U-003 - Withdraw submitted request before first FO confirmation

### Причина

Requestor должен иметь отдельный request-level сценарий для отзыва уже отправленной заявки целиком до того, как Fleet Owner подтвердил первую бронь из этой заявки.

### Новое правило

Разрешается отдельный use case `Отзыв заявки requestor-ом` для заявки в статусе `Submitted`, если ни один booking item этой заявки еще не был подтвержден Fleet Owner.

### Условия допустимости

- request status = `Submitted`;
- у заявки нет booking item-ов, которые уже находятся в `Confirmed`, `InProgress`, `Closed` или уже прошли необратимый terminal transition;
- requestor отзывает заявку до первого FO confirmation по любому item-у.

### Ожидаемое поведение

- request-level действие отзывает все еще не обработанные booking item-ы заявки;
- такие item-ы переходят в `Closed` с terminal причиной `Revoked`;
- request status пересчитывается после массового revoke item-ов;
- после того как активных item-ов не остается, request переходит в `Closed`.

### Разграничение с соседними сценариями

- `UC-REQ-03` - отмена draft-заявки до submit;
- `UC-REQ-04` - отзыв отдельной брони requestor-ом;
- `UC-REQ-05` - отзыв всей submitted-заявки до первого FO confirmation.

### Влияние на документацию и реализацию

Эта запись должна использоваться при обновлении:

- use case request withdrawal;
- UI-кнопки request-level withdraw для submitted requests;
- API orchestration / dedicated endpoint для массового revoke item-ов в request.
