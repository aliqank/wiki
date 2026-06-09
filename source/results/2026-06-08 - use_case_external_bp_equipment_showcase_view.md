**Created:** 2026-06-08 12:00  
**Last updated:** 2026-06-08 12:00  
**Author:** Telman Nurzhanov (SA)

---

# UC-REQ-BP-01 - Просмотр витрины техники внешних бизнес-партнеров

## 1. Назначение артефакта

Этот use case описывает read-only страницу витрины техники `On-demand BP (Showcase)` для Requestor.

Артефакт подготовлен как аналитический результат для дальнейшей реализации API и UI. В `wiki/` не публиковался.

---

## 2. Confirmed Facts

Подтверждено по `wiki/brd/BRD.md`:

1. `On-demand BP (Showcase)` не участвует в booking workflow в Phase 1.
2. Такая техника отображается только как showcase / catalog.
3. Цены и rates для `On-demand BP (Showcase)` не отображаются.
4. BP сам наполняет свои catalog cards.
5. Booking этой техники выполняется вне системы HDV/HDE Booking Tool.

Связанные BRD FR:
- `FR-NEW-25` - Requestor can view on-demand BP equipment; no booking action in Phase 1.
- `FR-NEW-26` - On-demand BP (Showcase) equipment prices/rates NOT displayed.
- `FR-NEW-27` - BP populates own catalog cards.
- `FR-NEW-28` - "Go to BP catalog" button after FO timeout.

---

## 3. Scope of This Use Case

Данный use case покрывает только:
- просмотр витрины техники внешних business partners;
- фильтрацию и поиск по showcase-данным;
- просмотр карточек/строк техники в read-only режиме.

Данный use case не покрывает:
- создание заявки;
- добавление техники в request;
- проверку availability на период;
- load summary;
- любые booking actions;
- переход в внешний BP catalog после FO timeout как отдельный сценарий.

---

## 4. Use Case Card

| Field | Value |
|---|---|
| Use case ID | `UC-REQ-BP-01` |
| Name | `Просмотр витрины техники внешних бизнес-партнеров` |
| Area | Страница `BP Showcase` / `External BP Equipment Showcase` |
| Actor | `Requestor` |
| Covered FR (BRD) | `FR-NEW-25`, `FR-NEW-26`, `FR-NEW-27` |
| Trigger | Пользователь открывает страницу витрины техники внешних business partners |
| Precondition | Пользователь авторизован; у пользователя есть доступ к просмотру showcase-страницы |
| Result | Пользователь видит read-only список `On-demand BP (Showcase)` техники и может открыть данные по технике без возможности что-либо сделать с ней |

---

## 5. Main Scenario

1. Пользователь открывает страницу `BP Showcase`.
2. Frontend загружает базовые reference-данные для фильтра business partners.
3. При первом открытии страницы frontend вызывает showcase search API.
4. Backend выбирает только технику с `ownershipType = OnDemand`.
5. Backend применяет фильтр по `businessPartnerId`, если он передан.
6. Backend применяет поиск только по `stateNumber`.
7. Backend возвращает paginated read-only список техники.
8. Frontend отображает витрину техники.
9. Для каждой единицы техники пользователь видит:
   - ГРНЗ;
   - тип техники;
   - бренд;
   - модель;
   - business partner;
   - preview photo, если есть;
   - базовую локацию и прочий разрешенный showcase-контекст.
10. Пользователь может открыть карточку / detail panel техники в read-only режиме.
11. Frontend не показывает кнопок `Add`, `Book`, `Select`, `Replace`, `Request`, `Submit` или любых других action controls.

---

## 6. Alternative Scenarios

1. Подходящая техника не найдена.
   Система показывает пустой результат витрины.

2. Не передан `businessPartnerId`.
   Система показывает общую витрину всех доступных `On-demand BP (Showcase)` business partners.

3. BP card заполнена частично.
   Система показывает только те поля, которые реально переданы BP и доступны для отображения в showcase.

4. Пользователь пытается выполнить действие с техникой.
   В этом сценарии действий нет; UI не должен предоставлять action controls.

---

## 7. API Delta vs `GET /equipment/search`

Ниже зафиксированы отличия от `wiki/api/booking/requestor/GET_equipment_search.md`.

### 7.1 Request Delta

Из запроса убрать:
- `equipmentTypeId`
- `plannedStartDateTime`
- `plannedEndDateTime`
- `ownershipType`
- `shareType`
- `fleetId`

Изменения по оставшимся параметрам:
- `search` - только поиск по `stateNumber`;
- вместо `fleetId` использовать `businessPartnerId`.

Итого ожидаемый набор query params для showcase API:
- `businessPartnerId` - optional;
- `workCenterId` - optional, если showcase UI действительно должен поддерживать такой фильтр;
- `hideInRepair` - not needed by default для read-only showcase, если не будет отдельного бизнес-требования;
- `propertyFilters` - optional only if business хочет dynamic showcase filtering;
- `page`;
- `limit`;
- `search` (только по `stateNumber`).

### 7.2 Response Delta

Из ответа убрать:
- `tcoId`
- `fleet`

В ответ добавить блок business partner:
- `businessPartner.id`
- `businessPartner.name`
- при необходимости `businessPartner.logoUrl` / `businessPartner.contactInfo` - только если такие поля реально есть в модели и согласованы отдельно.

Остальные поля showcase-списка могут быть унаследованы из search-ответа, если они не противоречат BRD scope.

### 7.3 Response Semantics Delta

Для showcase API:
- не нужна booking semantics around selected period;
- не нужны `hasBookingConflict` и `equipmentStateOnPeriod`, если страница действительно полностью read-only и не зависит от периода;
- не нужен `isBookable`, так как booking action отсутствует;
- не нужен `requiresJustification`, так как booking action отсутствует.

---

## 8. Proposed Read-Only Response Shape

Минимальный состав `value.items[]` для витрины:

| Field | Type | Comment |
|---|---|---|
| `id` | `uuid` | Equipment ID |
| `stateNumber` | `string` | Основной поисковый и display identifier |
| `equipmentTypeId` | `uuid` | |
| `equipmentTypeName` | `object` | `{ En, Ru, Kz }` |
| `brand` | `string` | |
| `model` | `string` | |
| `previewPhotoUrl` | `string \| null` | |
| `baseLocationName` | `object \| null` | |
| `workCenter` | `object \| null` | Только если показывается на UI |
| `businessPartner` | `object` | Основной внешний контекст |

Структура `businessPartner`:

| Field | Type | Comment |
|---|---|---|
| `id` | `uuid` | |
| `name` | `object` | `{ En, Ru, Kz }` |

---

## 9. Business Rules

1. В витрине отображается только техника `On-demand BP (Showcase)`.
2. Никакие booking actions в этом сценарии недоступны.
3. Цены, rates и коммерческие данные не показываются.
4. Showcase не использует период поиска, потому что техника не участвует в booking flow внутри системы.
5. Поиск по строке ограничен `stateNumber`.
6. Внешний контекст техники должен строиться вокруг `businessPartner`, а не `fleet`.
7. Карточка техники считается BP-provided content и может быть неполной.

---

## 10. Open Questions

1. Должен ли `businessPartnerId` быть обязательным фильтром, или страница показывает агрегированную витрину всех BP?
2. Нужны ли для showcase дополнительные reference-фильтры кроме `businessPartnerId`?
3. Нужен ли отдельный detail API для BP showcase card, или достаточно списка + side panel на тех же данных?
4. Нужно ли показывать `workCenter` на showcase-странице, если booking flow отсутствует?
5. Должны ли `InRepair` / `Frozen` состояния вообще отображаться в showcase, если пользователь не может ничего забронировать?

---

## 11. Suggestions According to BRD

1. Не перегружать `GET /equipment/search` этим сценарием.
   Лучше выделить отдельный read-only endpoint для showcase, потому что его семантика отличается от booking search:
   - нет периода;
   - нет availability logic;
   - нет add/select action;
   - фиксированный `ownershipType = OnDemand`.

2. Не возвращать booking-oriented поля в showcase API.
   Поля вроде `isBookable`, `requiresJustification`, `hasBookingConflict`, `bookingUnavailableReason`, `equipmentStateOnPeriod` только запутают UI, если страница строго витринная.

3. Отдельно продумать связь с `FR-NEW-28`.
   Кнопка `Go to BP catalog` после FO timeout должна быть оформлена как отдельный сценарий / CTA, а не как часть базового showcase browse-flow.

4. Явно маркировать BP showcase как external/non-bookable.
   Это снизит риск, что пользователь воспримет страницу как альтернативный internal booking search.
