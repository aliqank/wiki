# Анализ BPMN: equipment request process v2
**Дата:** 2026-04-06
**Тип документа:** Gap-анализ BPMN-диаграммы
**Файл:** `2026-04-06 - Requirement gathering/equipment request process v2.bpmn`
**Инструмент:** Camunda Modeler 5.45.0

---

## 1. Структура диаграммы

| Пул | ID | Описание |
|-----|----|----------|
| Requestor | Process_0co7y9f | Заявитель |
| Booking tool system | Process_0w8eo2w | Системная логика |
| Internal fleet owner | Process_08ndols | Владелец внутреннего флота |
| External fleet owner (BP) | Process_0udk0r8 | Владелец внешнего флота |
| Cost center owner | Process_1gg88r7 | Владелец центра затрат |

---

## 2. Что исправлено по сравнению с v1 ✅

| # | Улучшение |
|---|-----------|
| 1 | **End Events добавлены** во все пулы (критическая ошибка v1 устранена) |
| 2 | **Приоритет P1–P4** — отдельная задача "Set priority (P1–P4)" в пуле Requestor |
| 3 | **Период бронирования** — задача "Specify booking period (date/time from–to)" |
| 4 | **Таймеры 24h / 48h** добавлены для ожидания ответа Internal FO и External FO |
| 5 | **Reminder-задача** перед эскалацией (до истечения 48h) |
| 6 | **Auto-escalate** при истечении 48h для обоих флотов |
| 7 | **Create RWA draft in DCT** — задача в System pool появилась |
| 8 | **Internal FO rejection** корректно ведёт в "Try next FO or escalate external" (в v1 был неверный loop на Create request) |
| 9 | **External FO rejection** корректно уведомляет Requestor |
| 10 | **Cost Center Owner pool** добавлен с ветками Confirm/Reject |

---

## 3. Оставшиеся ошибки

### 🔴 Критические (блокируют корректность BPMN 2.0)

#### Ошибка 1 — Неверный тип таймерных событий
**Проблема:** `Event_1s5x21e` ("24h") и `Event_10983c9` ("48h") — это `IntermediateCatchEvent` с двумя исходящими потоками каждое:
- 24h → Reminder
- 24h → 48h timer

**В BPMN 2.0 IntermediateCatchEvent может иметь только ОДИН исходящий поток.** Текущая конфигурация невалидна.

**Решение:**
```
Вариант A — Timer Boundary Events:
  Задача "Publish to Internal FOs"
    + Non-interrupting Timer Boundary (24h) → "Reminder"
    + Interrupting Timer Boundary (48h) → "Auto-escalate to external"

Вариант B — Event-Based Gateway:
  "Publish to Internal FOs" → Event-Based GW → {Timer 24h / Message "FO responded"}
```

#### Ошибка 2 — "Route to CC Owner" — тупик в Requestor pool
**Проблема:** Задача `Activity_14l78jo` "Route to CC Owner" имеет входящий поток, но **нет исходящего sequence flow**. Requestor заходит в эту задачу и процесс обрывается.

**Решение:** После "Route to CC Owner" добавить `IntermediateCatchEvent (Message)` "Waiting for CC Owner response" → ветвление по результату.

#### Ошибка 3 — "Route to CC Owner" находится в пуле Requestor (семантически неверно)
**Проблема:** Маршрутизацию к CC Owner делает **Requestor**, хотя это функция **системы**. Requestor только заполняет Justification; система сама определяет, куда его отправить.

**Решение:** Задачу "Route to CC Owner" перенести в **System pool**, убрать из Requestor pool. В Requestor pool оставить только ожидание ответа.

#### Ошибка 4 — Два message flow ведут в один MessageCatchEvent
**Проблема:** `Event_0vmp0fu` ("Rejected" в пуле Requestor) получает сообщения из двух источников:
- `Activity_1fe85j5` (Specify reason — External FO) → `Event_0vmp0fu`
- `Activity_1y3k5yd` (Notify Requestor — System, при отклонении External FO) → `Event_0vmp0fu`

**В BPMN 2.0 MessageIntermediateCatchEvent может получать сообщение только от одного источника.**

**Решение:** Оставить только один маршрут (System → Requestor). External FO должен отправлять отказ в **System**, а System уже уведомляет Requestor.

#### Ошибка 5 — XOR Gateway вместо Event-Based Gateway в Requestor pool
**Проблема:** `Gateway_0l32wdw` (после "Waiting for response") — это XOR Gateway с 4 исходящими потоками к разным событиям (Confirmed internal, Rejected, Confirmed external, Fill Justification). Логически это **ожидание одного из нескольких событий** — для этого нужен **Event-Based Gateway**.

**Решение:** Заменить XOR на `EventBasedGateway`, а за ним разместить Message/Timer Catch Events.

#### Ошибка 6 — Reminder и Escalate задачи — тупики
**Проблема:** Следующие задачи не имеют исходящих потоков:
- `Activity_1lqswie` "Reminder" (Internal FO 24h)
- `Activity_1kzyrl8` "Auto-escalate to external" (Internal FO 48h)
- `Activity_0g0uvy7` "Reminder" (External FO 24h)
- `Activity_1my12pp` "Escalate" (External FO 48h)

**Решение:** Reminder должен возвращаться к ожиданию ответа. Escalate должен запускать следующую цепочку (выход из текущего ожидания и переход к эскалации).

#### Ошибка 7 — CC Owner rejection не уведомляет Requestor
**Проблема:** При отклонении CC Owner:
- CC Owner → `Activity_1ecqnxz` "Reject" → сообщение в System (`Activity_0ngzrns`)
- System → `Activity_0c44249` "Notify Requestor" → EndEvent

Но **нет message flow** от `Activity_0c44249` к Requestor pool. Requestor никогда не получит уведомление об отказе CC Owner.

**Решение:** Добавить message flow: `Activity_0c44249 → Event_0vmp0fu` (или отдельный catch event "CC Owner rejected" в Requestor pool).

---

### 🟡 Важные логические замечания

#### Замечание 1 — "Try next FO or escalate external" ведёт в EndEvent
`Activity_0pwt08u` заканчивается на `Event_1ig3rzc` (EndEvent). Но если это "попробовать следующего FO", процесс должен возвращаться к публикации запроса другому FO, а не заканчиваться.

**Решение:** Если есть следующий FO — loop back к "Publish to Internal Fleet Owners". Если FO-ов больше нет — переход к внешней эскалации (отдельный gateway).

#### Замечание 2 — После Mobilize процесс просто заканчивается
В пулах Internal FO и External FO: `Assign equipment → Mobilize → EndEvent`. Отсутствуют:
- Создание JDE Work Order (номер WO#)
- Отслеживание статуса бронирования (Active → Completed)
- Возврат техники / закрытие бронирования

**Решение:** После Mobilize добавить задачу "Notify System: equipment mobilized" с message flow в System pool; в System pool добавить задачи "Create JDE WO#" и "Update booking status → Active".

#### Замечание 3 — "Create new request" после Rejected ведёт в EndEvent
После получения сообщения "Rejected" Requestor переходит к "Create new request" → EndEvent. Логически должен быть loop back к началу заполнения заявки (или явный вопрос: подать новую заявку?).

**Решение:** Из "Create new request" провести поток обратно к `StartEvent` или к "Choose equipment type".

#### Замечание 4 — DPA (делегирование) CC Owner не отражено
По требованиям BRD (FR-047–FR-050): CC Owner может делегировать полномочия DPA. В текущей схеме этот сценарий отсутствует.

**Решение:** Добавить в пул Cost Center Owner задачу "Delegate to DPA" с ветвлением: самостоятельно / делегировать.

#### Замечание 5 — Нет возможности отозвать / отменить заявку
Requestor не может отозвать заявку после подачи (FR-038). Нет ни события, ни задачи "Withdraw request".

**Решение:** Добавить в Requestor pool Message Boundary Event (non-interrupting) "Withdraw request" на этапе ожидания ответа.

---

### 🟢 Рекомендации (nice-to-have)

| # | Рекомендация |
|---|-------------|
| 1 | Значения таймеров (24h / 48h) вынести как параметры: `${timerReminder}` / `${timerEscalation}` — должны задаваться через Admin panel, не hardcode |
| 2 | Добавить задачу "Attach JDE WO#" в System pool после подтверждения бронирования |
| 3 | Смоделировать сценарий частичного подтверждения (Shared with Conditions) |
| 4 | Добавить нотацию: нужен ли трейлер для доставки техники (auto-detect по типу оборудования) |
| 5 | Пометить, что Internal FO может подтвердить только **часть** запрошенных единиц |

---

## 4. Итоговая таблица приоритетов

| Приоритет | # | Элемент | Что исправить |
|-----------|---|---------|---------------|
| 🔴 Критично | 1 | Timer 24h/48h (IntermediateCatchEvent → 2 flows) | Заменить на Timer Boundary Events |
| 🔴 Критично | 2 | "Route to CC Owner" — тупик | Добавить outgoing flow + Waiting event |
| 🔴 Критично | 3 | "Route to CC Owner" в пуле Requestor | Перенести в System pool |
| 🔴 Критично | 4 | Два message flow → один catch event | Убрать прямой поток от Ext FO к Requestor |
| 🔴 Критично | 5 | XOR вместо Event-Based Gateway | Заменить тип gateway |
| 🔴 Критично | 6 | Reminder / Escalate — тупики | Добавить исходящие потоки |
| 🔴 Критично | 7 | CC Owner rejection не уведомляет Requestor | Добавить message flow в Requestor pool |
| 🟡 Важно | 8 | "Try next FO" ведёт в EndEvent | Loop back к публикации или эскалация |
| 🟡 Важно | 9 | После Mobilize — нет жизненного цикла | Добавить JDE WO#, статус Active |
| 🟡 Важно | 10 | "Create new request" ведёт в EndEvent | Loop back к началу |
| 🟡 Важно | 11 | DPA не отражено | Добавить ветку делегирования в CC Owner pool |
| 🟡 Важно | 12 | Нет Withdraw/Cancel | Добавить Boundary Event в ожидание ответа |
| 🟢 Желательно | 13 | Таймеры hardcoded | Параметризовать через Admin panel |
| 🟢 Желательно | 14 | JDE WO# не привязан | Добавить задачу в System pool |
| 🟢 Желательно | 15 | Частичное подтверждение | Смоделировать Shared with Conditions |

---

## 5. Исправленная текстовая схема TO-BE процесса

```
REQUESTOR
  [Start] Open request window
  → Choose equipment type
  → Specify booking period (date/time from–to)
  → Set priority (P1–P4)
  → [optionally: Attach JDE WO# / location / notes]
  → Submit request
  → [Event-Based GW] waiting for one of:
      - [Msg Catch] Confirmed (internal)  → [End] booking confirmed
      - [Msg Catch] Confirmed (external)  → [End] booking confirmed
      - [Msg Catch] Rejected / Timed out  → Create new request → [loop to Start]
      - [Msg Catch] Need justification    → Fill Justification window
                                          → [Event-Based GW] waiting for CC Owner result:
                                              - Confirmed → [Msg Catch] Confirmed (external) → [End]
                                              - Rejected  → [Msg Catch] Rejected → Create new request
  + [Boundary Event] Withdraw request → [End] cancelled

BOOKING TOOL SYSTEM
  [Msg Catch] Receive request
  → Check internal fleet availability
  → [GW] Free internal equipment?
      YES → Publish to Internal Fleet Owners
            + Timer Boundary (24h, non-interrupting) → Send Reminder to FO
            + Timer Boundary (48h, interrupting)     → Auto-escalate to external
            → [Msg Catch] FO responded:
                - Confirmed → Notify Requestor (internal confirmed)
                              → Update booking status → Create JDE WO# → [End]
                - Rejected  → [GW] Next FO available?
                              YES → loop to Publish to Internal FOs
                              NO  → [GW] No internal FO — escalate external
      NO  → Open Justification window for Requestor
            → [Msg Catch] Justification received
            → Route to CC Owner
            → [Msg Catch] CC Owner responded:
                - Confirmed → Publish to External Fleet Owner (BP)
                              + Timer Boundary (24h) → Reminder
                              + Timer Boundary (48h) → Escalate
                              → [Msg Catch] Ext FO responded:
                                  - Confirmed → Create RWA draft in DCT
                                               → Notify Requestor (external confirmed)
                                               → Update booking status → Create JDE WO# → [End]
                                  - Rejected  → Notify Requestor (rejected) → [End]
                - Rejected  → Notify Requestor (CC Owner rejected) → [End]

INTERNAL FLEET OWNER
  [Msg Catch] Receive request notification
  → Review request (priority P1–P4, booking period)
  → [GW] Confirm / Reject
      Confirm → Assign specific equipment unit
               → Mobilize
               → Notify System: equipment mobilized → [End]
      Reject  → Specify reason → Notify System: rejected → [End]

EXTERNAL FLEET OWNER (BP)
  [Msg Catch] Receive request notification
  → Review request (priority, period, Justification)
  → [GW] Confirm / Reject
      Confirm → Assign specific equipment unit
               → Mobilize
               → Notify System: equipment mobilized → [End]
      Reject  → Specify reason → Notify System: rejected → [End]

COST CENTER OWNER
  [Msg Catch] Receive notification + Justification
  → Review request + Justification
  → [optional: Delegate to DPA]
  → [GW] Confirm / Reject
      Confirm → Confirm → Notify System → [End]
      Reject  → Specify reason → Notify System: rejected → [End]
```

---

## 6. Краткий итог

**v2 значительно лучше v1**: устранены все критические структурные ошибки (End Events), добавлены таймеры, RWA, P1-P4, период бронирования.

**Главные задачи для v3:**
1. Исправить тип таймерных событий (→ Timer Boundary Events)
2. Заменить XOR Gateway на Event-Based Gateway в Requestor pool
3. Исправить "Route to CC Owner" (перенести в System, добавить ожидание)
4. Устранить два message flow на один catch event
5. Добавить исходящие потоки у Reminder / Escalate задач
6. Добавить уведомление Requestor при отказе CC Owner
