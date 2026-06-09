# UC-REQ-BP-01 - Просмотр витрины техники внешних бизнес-партнеров

**Created:** 2026-06-08  
**Last updated:** 2026-06-08  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка use case

| Поле | Значение |
|---|---|
| Область действия | Страница `BP Showcase` / `External BP Equipment Showcase` |
| Участник | Пользователь с ролью [`Requestor`](../../Roles%20and%20Access%20Model.md) |
| Покрываемые FR (BRD) | `FR-NEW-25`, `FR-NEW-26`, `FR-NEW-27` |
| Покрываемые FR (Additional list) | — |
| Предусловие | Пользователь авторизован; пользователь имеет доступ к странице showcase; в Phase 1 `On-demand BP (Showcase)` доступна только как read-only витрина |
| Триггер | Открытие страницы витрины техники внешних business partners |
| Ожидаемый результат | Пользователь видит read-only список `On-demand BP (Showcase)` техники, может фильтровать и просматривать карточки техники без booking actions |
| Используемые API | [`GET /equipment/bp-showcase`](../../../api/booking/requestor/GET_equipment_bp_showcase.md) |

---

## Основной сценарий

1. Пользователь открывает страницу `BP Showcase`.
2. Frontend открывает read-only витрину без контекста создания заявки и без привязки к booking period.
3. Frontend вызывает [`GET /equipment/bp-showcase`](../../../api/booking/requestor/GET_equipment_bp_showcase.md).
4. Backend возвращает только технику `On-demand BP (Showcase)`.
5. Если передан `businessPartnerId`, backend ограничивает выдачу техникой выбранного business partner.
6. Если передан `search`, backend выполняет поиск только по `stateNumber`.
7. Frontend отображает paginated showcase-список.
8. Для каждой единицы техники пользователь видит:
   - ГРНЗ;
   - тип техники;
   - бренд;
   - модель;
   - информацию о business partner, включая название, контакты и другие доступные showcase-реквизиты;
   - preview photo, если есть;
   - дополнительный showcase-контекст, разрешенный для отображения.
9. Пользователь может открыть карточку / detail panel выбранной техники в read-only режиме.
10. Frontend не показывает кнопок `Add`, `Book`, `Select`, `Request`, `Replace`, `Submit` или других booking actions.

---

## Альтернативные сценарии

1. Не передан `businessPartnerId`.
   Система показывает общую витрину всех доступных `On-demand BP (Showcase)` business partners.

2. Поиск не дал результатов.
   Система показывает пустой showcase-result.

3. BP card заполнена частично.
   Система показывает только реально доступные showcase-поля.

4. Пользователь пытается выполнить действие с техникой.
   В этом сценарии действия недоступны; UI остается строго read-only.

---

## Замечания

1. Use case не участвует в booking workflow и не использует `plannedStartDateTime` / `plannedEndDateTime`.
2. Цены и rates для `On-demand BP (Showcase)` не отображаются в соответствии с `FR-NEW-26`.
3. `FR-NEW-28` (`Go to BP catalog`) не входит в данный базовый use case и должен оформляться отдельным сценарием / CTA.
4. Для данной витрины внешний контекст строится вокруг `businessPartner`, а не вокруг `fleet`.
