# UC-COM-01 - Просмотр таймлайна брони

**Created:** 2026-06-05  
**Last updated:** 2026-06-08  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка use case

| Поле | Значение |
|---|---|
| Область действия | Страница `Мои заявки`, страница `Approvals`, detail / action view брони, карточка брони внутри заявки |
| Участник | Пользователь с ролью [`Requestor`](../../Roles%20and%20Access%20Model.md), [`ServiceWorkProcessor`](../../Roles%20and%20Access%20Model.md), [`FleetOwner`](../../Roles%20and%20Access%20Model.md), [`FleetOwnersSupervisor`](../../Roles%20and%20Access%20Model.md) или [`Admin`](../../Roles%20and%20Access%20Model.md), имеющий доступ к конкретной брони |
| Покрываемые FR (BRD) | `FR-NEW-37`, `FR-025`, `FR-092` |
| Покрываемые FR (Additional list) | — |
| Предусловие | Пользователь авторизован в системе; пользователь уже находится в контексте заявки или брони; в UI доступен `bookingId` целевой брони; пользователь имеет право просмотра этой брони |
| Триггер | Нажатие кнопки / ссылки `Timeline`, `История`, `Audit trail` или открытие одноименного блока в карточке брони |
| Ожидаемый результат | Пользователь видит упорядоченный timeline по выбранной брони без изменения состояния данных |
| Используемые API | [`GET /bookings/{id}/timeline`](../../../api/booking/common/GET_bookings_id_timeline.md), опционально supporting read API текущего экрана: [`GET /booking-requests/{id}`](../../../api/booking/requestor/GET_booking_requests_id.md), [`GET /approvals/bookings/{id}`](../../../api/booking/fleet-owner/GET_approvals_bookings_id.md) |

---

## Основной сценарий

1. Пользователь открывает страницу, где отображается заявка и связанные с ней брони, либо открывает detail / action view конкретной брони.
2. Frontend уже имеет контекст выбранной брони или получает `bookingId` из текущей строки / карточки.
3. Пользователь инициирует просмотр timeline по нужной брони:
   - из summary-блока брони на странице заявок;
   - либо из detail / action view брони;
   - либо из отдельной вкладки / accordion / modal внутри карточки брони.
4. Frontend вызывает [`GET /bookings/{id}/timeline`](../../../api/booking/common/GET_bookings_id_timeline.md).
5. Backend проверяет, что:
   - бронь существует;
   - текущий пользователь имеет право просмотра данной брони;
   - `bookingId` относится к доступному пользователю business context-у.
6. Backend собирает timeline из нескольких источников:
   - lifecycle history из `BookingStatuses`;
   - approval decisions из `BookingApprovals`;
   - execution milestones из factual booking fields при необходимости.
7. Backend сортирует timeline по `occurredAt` и возвращает единый event stream.
8. Frontend открывает timeline в выбранном UI-контейнере текущего экрана без обязательной навигации на отдельную страницу.
9. Пользователь просматривает события timeline, включая:
    - дату и время события;
    - описание события;
    - lifecycle status, если событие является status change;
    - approval decision, если событие относится к approval flow;
    - actor-а, выполнившего действие или зафиксированного как источник события;
   - комментарий, если он есть.
10. Пользователь использует timeline для понимания хода обработки брони, причин текущего состояния и последовательности согласований / исполнения.

---

## Альтернативные сценарии

1. Timeline открывается со страницы заявок без перехода в отдельную карточку брони.
   Frontend использует `bookingId` из текущего summary-блока / строки и показывает timeline в modal, drawer, side panel или embedded section.

2. Timeline открывается уже внутри карточки брони.
   Frontend использует `bookingId` текущей карточки и показывает timeline как отдельный блок / tab / accordion без повторной навигации.

3. По брони пока еще мало событий.
   Backend возвращает короткий список, например только `Draft` и `Submitted`; frontend все равно показывает timeline как валидный результат.

4. Пользователь не имеет доступа к выбранной брони.
   Backend возвращает `FORBIDDEN`; frontend не показывает timeline и отображает сообщение об ошибке.

5. Бронь не найдена.
   Backend возвращает `NOT_FOUND`; frontend показывает сообщение об ошибке и предлагает обновить страницу / контекст заявки.

6. В timeline нет approval events, потому что для данной брони еще не было решений согласующих.
   Frontend показывает только lifecycle events и execution milestones, если они есть.

---

## Замечания

1. Просмотр timeline является read-only сценарием и не меняет состояние брони или заявки.
2. Timeline должен отражать фактическую историю обработки одной конкретной брони, а не всей заявки целиком.
3. `Timeline` и `status history` не являются взаимозаменяемыми представлениями:
   - `status-history` = строгая история lifecycle status transitions;
   - `timeline` = более широкая агрегированная audit projection.
4. Timeline может быть открыт из нескольких UI-точек входа; отдельная dedicated page для него не является обязательной.
5. Один и тот же API `GET /bookings/{id}/timeline` должен обслуживать оба варианта UI:
   - со страницы заявок;
   - из карточки брони.
6. Права просмотра timeline определяются не местом открытия UI, а доступом пользователя к самой брони.
7. Если у брони есть approval chain из нескольких шагов, timeline должен показывать их в реальном порядке событий.
8. Для базового сценария новый API не требуется: покрытие уже обеспечено методом [`GET /bookings/{id}/timeline`](../../../api/booking/common/GET_bookings_id_timeline.md).
