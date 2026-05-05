# Анализ BPMN: equipment request process.bpmn

> Дата: 2026-04-06
> Файл: `2026-04-06 - Requirement gathering/equipment request process.bpmn`
> Основа для сравнения: BRD (FR-001–FR-103) + meeting notes 02.04 + meeting notes 03.04

---

## 1. Что смоделировано в диаграмме

### Участники (Pools)
| Pool | Роль |
|------|------|
| Requestor | Инициирует заявку |
| Booking tool system | Системная логика |
| Internal fleet owner | Согласование внутреннего флота |
| External fleet owner | Согласование внешнего флота |
| Cost center owner | Подтверждение затрат для внешнего флота |

### Верхнеуровневый поток
```
Requestor выбирает тип техники → создаёт заявку
       ↓ (message)
Система проверяет доступность внутреннего флота
       ↓
[XOR] Есть внутренняя техника?
  YES → публикует заявку Internal Fleet Owner
  NO  → открывает окно Justification для Requestor
              ↓
        Requestor заполняет Justification
              ↓
        CC Owner рассматривает
              ↓
        Подтверждение → публикует заявку External Fleet Owner
```

---

## 2. Что реализовано верно ✅

| # | Что правильно |
|---|---------------|
| ✅ 1 | Requestor как точка старта процесса |
| ✅ 2 | Приоритет внутреннего флота перед внешним (XOR-шлюз в системе) |
| ✅ 3 | Justification открывается только при отсутствии внутренней техники |
| ✅ 4 | Цепочка согласования для внешнего флота: CC Owner → External Fleet Owner |
| ✅ 5 | Fleet Owner (и внутренний, и внешний) видят заявку и принимают решение |
| ✅ 6 | После мобилизации — уведомление Requestor-у через систему |
| ✅ 7 | Нотификации на все ключевые события смоделированы в пуле системы |
| ✅ 8 | При отклонении External Fleet Owner — цикл обратно к созданию заявки |

---

## 3. Ошибки и несоответствия требованиям ❌

### 3.1 Структурные ошибки BPMN

| # | Проблема | Где |
|---|----------|-----|
| ❌ 1 | **Нет End Event** ни в одном пуле. BPMN-нарушение: процессы не имеют завершения | Все пулы |
| ❌ 2 | **Event_0tgoe6z** (Requestor) используется одновременно для двух разных сценариев: отклонение Internal Fleet Owner И отклонение CC Owner. Одно событие = один семантический смысл | Requestor pool |
| ❌ 3 | **Event_1iae9gl** (уведомление о подтверждении External FO) — catch event без исходящего потока. Не ясно, завершается ли процесс или нет | Requestor pool |
| ❌ 4 | **"mobilize equipment"** в пулах Fleet Owner — нет End Event и нет замыкающего потока в рамках пула | Internal + External FO pools |

---

### 3.2 Логические ошибки процесса

| # | Проблема | Требование |
|---|----------|-----------|
| ❌ 5 | **При отклонении Internal Fleet Owner** — заявка возвращается к "Create request" у Requestor. Это неверно: отклонение FO ≠ ошибка заявителя. Правильно: система должна попробовать других внутренних FO или эскалировать во внешний флот | R-25, meeting notes 03.04 |
| ❌ 6 | **При отклонении CC Owner** — тот же цикл обратно к "Create request". Некорректно: CC Owner отклонил → Requestor должен получить уведомление об отклонении и либо завершить процесс, либо скорректировать Justification | BRD FR-055, FR-066 |
| ❌ 7 | **После подтверждения Internal Fleet Owner** ("mobilize equipment") нет сигнала об окончании брони в системе и нет фиксации факта завершения | BRD FR-070, FR-071 |

---

### 3.3 Пропущенные функциональные блоки

| # | Что пропущено | Источник требования |
|---|--------------|---------------------|
| ❌ 8 | **Таймаут ответа Fleet Owner** — Fleet Owner должен ответить в течение 24–48 ч. Нет Timer Boundary Event на задачах "Review request" ни у Internal, ни у External FO | Meeting notes 03.04 (OQ-11) |
| ❌ 9 | **Напоминание и эскалация** при игнорировании заявки Fleet Owner (reminder → копия руководителю → авто-переход во внешний флот) | R-24, R-25 |
| ❌ 10 | **Указание времени бронирования** (дата/время начала, окончания) — ключевое поле, нигде не отражено в шагах Requestor | R-1, meeting notes 03.04 |
| ❌ 11 | **Приоритет заявки (P1–P4)** — Requestor не выбирает приоритет при создании заявки | R-15 |
| ❌ 12 | **JDE Work Order номер** — Requestor должен указывать номер WO при создании заявки | R-28 |
| ❌ 13 | **DPA (Delegated Authority)** — CC Owner может делегировать право подтверждения DPA. Не отражено в пуле Cost Center Owner | BRD FR-053, FR-054 |
| ❌ 14 | **Отзыв бронирования (Withdraw)** — Requestor должен иметь возможность отозвать заявку до подтверждения | BRD FR-058, FR-068 |
| ❌ 15 | **Частичное подтверждение** — в одной заявке может быть несколько единиц техники. Часть подтверждается, часть отклоняется. Заявка продолжает работу | R-27, BRD FR-065 |
| ❌ 16 | **Проверка наличия конкретной единицы vs типа техники** — Requestor выбирает тип, Fleet Owner назначает конкретную единицу. Это разграничение не отражено | Meeting notes 03.04 |
| ❌ 17 | **Justification при внутреннем флоте для Shared with Conditions** — фактически может потребоваться тоже (если FO устанавливает условия) | Meeting notes 03.04 |
| ❌ 18 | **Создание черновика RWA в DCT** после финального согласования внешней техники — интеграционный шаг не отражён | BRD §8.2 |

---

## 4. Предлагаемые улучшения

### 4.1 Структура пулов — добавить / переименовать

```
Текущие пулы:              Предлагаемые пулы:
──────────────────         ──────────────────────────────
Requestor                  Requestor
Booking tool system        Booking tool system
Internal fleet owner       Internal fleet owner
External fleet owner       External fleet owner (BP)
Cost center owner          Cost center owner / DPA   ← добавить DPA
                           DCT system               ← добавить для интеграции
```

---

### 4.2 Пул Requestor — уточнить задачи

**Было:**
```
Start → Choose equipment type → Create request
```

**Должно быть:**
```
Start → Choose equipment type
           ↓
        Specify booking period (date/time from–to)
           ↓
        Select priority (P1–P4)
           ↓
        Add JDE Work Order # (optional/mandatory TBD)
           ↓
        Submit request
           ↓
        [Waiting for response — Intermediate catch events]
           ↓
    ┌──────┬──────┬──────┬──────┐
 Confirmed Rejected Withdrawn  Partially confirmed
    ↓         ↓        ↓             ↓
  End    Retry/End   End        Handle each booking
```

---

### 4.3 Логика при отклонении Internal Fleet Owner

**Было:** отклонение FO → уведомление → "Create request" (некорректный цикл)

**Должно быть:**
```
Internal FO: reject request
       ↓
Booking tool: проверить, есть ли другие Internal FO с этим типом техники
       ↓
  [XOR] Есть ли другие FO?
     YES → переназначить заявку следующему FO
     NO  → уведомить Requestor "нет внутренней техники"
              → предложить внешний флот (открыть Justification)
              → или завершить процесс (End Event)
```

---

### 4.4 Таймаут Fleet Owner — добавить Timer Boundary Event

```
Internal Fleet Owner:
┌────────────────────────────────┐
│  Review request                │
│         ⏱ Timer (24–48 ч)     │ ← Boundary Event (non-interrupting → reminder)
│         ⏱ Timer (48–72 ч)     │ ← Boundary Event (interrupting → эскалация)
└────────────────────────────────┘
        ↓ (timeout + no response)
Booking tool: отправить reminder → копия руководителю
        ↓ (2-й таймаут)
Booking tool: автоматически эскалировать во внешний флот
```

---

### 4.5 Жизненный цикл бронирования — добавить после мобилизации

```
Fleet Owner: mobilize equipment
       ↓
Booking tool: статус → "In Progress" (при наступлении даты начала)
       ↓
[Timer: дата окончания брони]
       ↓
Booking tool: статус → "Completed"
       ↓
End Event
```

---

### 4.6 Исправить End Events

В каждом пуле должны быть End Events:

| Пул | End Events |
|-----|-----------|
| Requestor | 1) Бронь подтверждена ✓  2) Бронь отклонена ✗  3) Заявка отозвана |
| Booking tool system | После каждого финального уведомления |
| Internal Fleet Owner | После мобилизации или после отклонения |
| External Fleet Owner | После мобилизации или после отклонения |
| Cost Center Owner | После подтверждения или отклонения |

---

## 5. Резюме: приоритеты доработки

| Приоритет | Доработка |
|-----------|-----------|
| 🔴 Критично | Добавить End Events во все пулы |
| 🔴 Критично | Разделить Event_0tgoe6z на два отдельных события |
| 🔴 Критично | Исправить логику при отклонении Internal Fleet Owner |
| 🔴 Критично | Добавить Timer Boundary Event на "Review request" (оба FO) |
| 🟡 Важно | Добавить шаги Requestor: период брони, приоритет P1–P4, номер WO |
| 🟡 Важно | Добавить DPA в пул CC Owner |
| 🟡 Важно | Добавить шаг "Create RWA draft in DCT" после подтверждения External FO |
| 🟡 Важно | Смоделировать жизненный цикл брони (In Progress → Completed) |
| 🟢 Желательно | Добавить возможность отзыва (Withdraw) для Requestor |
| 🟢 Желательно | Отразить частичное подтверждение заявки |
| 🟢 Желательно | Разграничить "выбор типа техники" (Requestor) и "назначение конкретной единицы" (Fleet Owner) |

---

## 6. Исправленная структура процесса (текстовая TO-BE схема)

```
REQUESTOR
  Start
    → Choose equipment type
    → Specify booking period (from / to)
    → Set priority (P1–P4)
    → Add JDE Work Order #
    → Submit request
    → [Wait]
         ├─ Confirmed (internal) ──────────────────────────────→ End ✓
         ├─ Confirmed (external) → Notify "submit RWA in DCT" → End ✓
         ├─ Rejected → [Retry or End]
         └─ Withdrawn by Requestor ───────────────────────────→ End

BOOKING TOOL SYSTEM
    Receive request
    → Check internal fleet availability
    → [XOR] Free internal equipment?
         YES → Publish to Internal Fleet Owners
                  ⏱ Timer 24h → Reminder (non-interrupt)
                  ⏱ Timer 48h → Auto-escalate to external (interrupt)
              → [Internal FO response]
                    Confirmed → notify Requestor → update status → End
                    Rejected  → try next FO or escalate external
         NO  → Open Justification window for Requestor
              → Route to CC Owner / DPA
              → [CC Owner response]
                    Confirmed → Publish to External Fleet Owner
                                   ⏱ Timer 24h → Reminder
                                   ⏱ Timer 48h → Escalate
                    Rejected  → notify Requestor → End
              → [External FO response]
                    Confirmed → create RWA draft in DCT → notify Requestor → End
                    Rejected  → notify Requestor → [Retry or End]

INTERNAL FLEET OWNER
    Receive request notification
    → Review request (see priority P1–P4, booking period, WO#)
    → [Confirm / Reject]
         Confirm → Assign specific equipment unit → Mobilize → End
         Reject  → Specify reason → End

COST CENTER OWNER / DPA
    Receive notification
    → Review request + Justification
    → [Confirm / Reject]
         Confirm → End (system routes to External FO)
         Reject  → Specify reason → End

EXTERNAL FLEET OWNER (BP)
    Receive request notification
    → Review request (see priority, period, Justification)
    → [Confirm / Reject]
         Confirm → Assign specific equipment unit → Mobilize → End
         Reject  → Specify reason → End
```
