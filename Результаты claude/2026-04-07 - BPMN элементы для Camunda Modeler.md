# BPMN элементы для Camunda Modeler
**Дата:** 2026-04-07
**Тип документа:** Справочник для построения BPMN в Camunda Modeler
**Использовать совместно с:** `2026-04-07 - Изменения BPMN на основе реестра требований.md`

---

## Как читать этот документ

- **Тип** — тип элемента в Camunda Modeler (как называется на панели или в контекстном меню)
- **Имя** — текст на элементе (Name)
- **От / К** — sequence flow (стрелки внутри пула)
- **Сообщение** — message flow (пунктирные стрелки между пулами)
- ⚠️ — особое внимание при создании

---

# Процесс 1: equipment request process v3
**Внутренний флот — Cat 1 (собственная TCO) + Cat 2 (долгосрочная аренда)**
**Пулы:** Requestor · Booking Tool System · Internal Fleet Owner

---

## Пул: Requestor

| # | Тип элемента | Имя элемента | Входящий поток (от) | Исходящий поток (к) | Примечание |
|---|-------------|-------------|-------------------|-------------------|-----------|
| R1 | **Start Event** | Open request window | — | R2 | |
| R2 | **Task** (User Task) | Choose equipment type | R1 | R3 | |
| R3 | **Task** (User Task) | Specify booking period (date/time from–to) | R2 | R4 | |
| R4 | **Task** (User Task) | Set priority (P1–P4) | R3 | R5 | |
| R5 | **Task** (User Task) | Submit request | R4 | R6 | → message flow → S1 |
| R6 | **Event-Based Gateway** | *(без имени)* | R5 | R7, R9, R11 | ⚠️ Именно Event-Based GW, не XOR |
| R7 | **Intermediate Catch Event** (Message) | Booking confirmed | R6 | R8 | Получает сообщение от S11 |
| R8 | **End Event** | Booking confirmed | R7 | — | |
| R9 | **Intermediate Catch Event** (Message) | Request rejected | R6 | R10 | Получает сообщение от S18 |
| R10 | **End Event** | Request closed | R9 | — | |
| R11 | **Intermediate Catch Event** (Message) | No internal fleet available | R6 | R12 | Получает сообщение от S7 |
| R12 | **Task** (User Task) | Fill Justification window (reason + cost center) | R11 | R13 | → message flow → S8 |
| R13 | **Intermediate Catch Event** (Message) | External booking initiated | R12 | R14 | Получает сообщение от S17 |
| R14 | **End Event** | External booking initiated | R13 | — | |

### Message flows — пул Requestor

| Направление | От | К | Событие-получатель |
|------------|----|----|-------------------|
| Requestor → System | R5 (Submit request) | S1 (Receive request) | — |
| Requestor → System | R12 (Fill Justification) | S8 (Receive Justification) | — |
| System → Requestor | S11 (Notify: confirmed) | R7 (Booking confirmed) | Message Catch |
| System → Requestor | S18 (Notify: rejected) | R9 (Request rejected) | Message Catch |
| System → Requestor | S7 (Notify: no internal fleet) | R11 (No internal fleet) | Message Catch |
| System → Requestor | S17 (Notify: external initiated) | R13 (External booking initiated) | Message Catch |

---

## Пул: Booking Tool System

| # | Тип элемента | Имя элемента | Входящий поток (от) | Исходящий поток (к) | Примечание |
|---|-------------|-------------|-------------------|-------------------|-----------|
| S1 | **Task** (Receive Task) | Receive request | — (message от R5) | S2 | |
| S2 | **Task** (Service Task) | Check internal fleet availability (Cat 1 + Cat 2) | S1 | S3 | |
| S3 | **Exclusive Gateway** | Free internal equipment? | S2 | S4 (yes), S7 (no) | |
| S4 | **Task** (Send Task) | Publish request to Internal Fleet Owners | S3 (yes), S14 (loop) | S5 | → message flow → F1 |
| S5 | **Boundary Event** (Timer, **non-interrupting**) | 24h — no response | прикреплён к S4 | S6 | ⚠️ cancelActivity = false |
| S5b | **Boundary Event** (Timer, **interrupting**) | 48h — escalate to external | прикреплён к S4 | S16 | ⚠️ cancelActivity = true |
| S6 | **Task** (Send Task) | Send reminder to Internal Fleet Owner | S5 | S6e | |
| S6e | **End Event** | Reminder sent | S6 | — | Параллельный конец (основной поток продолжается) |
| S8 | **Task** (Receive Task) | Wait for FO response | S4 → S8 (после публикации) | S9 | Получает message от F5 или F7 |
| S9 | **Exclusive Gateway** | FO decision? | S8 | S10 (confirmed), S13 (rejected) | |
| S10 | **Task** (Send Task) | Notify Requestor: booking confirmed | S9 (confirmed) | S11 | → message flow → R7 |
| S11 | **Task** (Service Task) | Update booking status: Confirmed | S10 | S12 | |
| S12 | **End Event** | Booking confirmed | S11 | — | |
| S13 | **Exclusive Gateway** | Other Internal FO available? | S9 (rejected) | S14 (yes), S15 (no) | R-34: не закрывать заявку |
| S14 | → S4 | *(поток "try next FO")* | S13 (yes) | S4 | Петля назад к S4 |
| S15 | → S16 | *(поток "no more FO")* | S13 (no) | S16 | |
| S16 | **Call Activity** | External Booking Process (Cat 3) | S5b, S15, S19 | S17 | Вызывает `external fleet booking process v1` |
| S17 | **Task** (Send Task) | Notify Requestor: external booking initiated | S16 | S17e | → message flow → R13 |
| S17e | **End Event** | Routed to external process | S17 | — | |
| S7 | **Task** (Send Task) | Notify Requestor: no internal fleet | S3 (no) | S8j | → message flow → R11 |
| S8j | **Task** (Receive Task) | Receive Justification from Requestor | S7 | S19 | Получает message от R12 |
| S19 | → S16 | *(поток к Call Activity)* | S8j | S16 | |
| S18 | **Task** (Send Task) | Notify Requestor: request fully rejected | *(если нужен явный Reject без внешнего флота)* | S18e | → message flow → R9 |
| S18e | **End Event** | Request rejected | S18 | — | |

> **Примечание по S8:** После публикации (S4) система ждёт ответа от FO. Sequence flow от S4 к S8 — основной поток. Таймеры S5 и S5b — Boundary Events, прикреплённые к S4.

### Message flows — пул System

| Направление | От | К |
|------------|----|----|
| System → Internal FO | S4 (Publish) | F1 (Receive notification) |
| System → Requestor | S7 (Notify: no fleet) | R11 |
| System → Requestor | S10 (Notify: confirmed) | R7 |
| System → Requestor | S17 (Notify: external) | R13 |
| System → Requestor | S18 (Notify: rejected) | R9 |

---

## Пул: Internal Fleet Owner

| # | Тип элемента | Имя элемента | Входящий поток (от) | Исходящий поток (к) | Примечание |
|---|-------------|-------------|-------------------|-------------------|-----------|
| F1 | **Task** (Receive Task) | Receive request notification | — (message от S4) | F2 | |
| F2 | **Task** (User Task) | Review request (priority P1–P4, period, equipment availability) | F1 | F3 | |
| F3 | **Exclusive Gateway** | Confirm / Reject? | F2 | F4 (Confirm), F6 (Reject) | |
| F4 | **Task** (User Task) | Assign specific equipment unit | F3 (Confirm) | F5 | |
| F5 | **Task** (User Task) | Confirm mobilization / handover | F4 | F5e | → message flow → S8 ("Confirmed") |
| F5e | **End Event** | Equipment assigned and mobilized | F5 | — | |
| F6 | **Task** (User Task) | Enter rejection reason *(mandatory — R-33)* | F3 (Reject) | F7 | ⚠️ Поле причины обязательно |
| F7 | **End Event** | Rejection sent (request stays open — R-34) | F6 | — | → message flow → S8 ("Rejected") |

### Message flows — пул Internal FO

| Направление | От | К |
|------------|----|----|
| Internal FO → System | F5 (Confirm mobilization) | S8 (Wait for FO response) |
| Internal FO → System | F7 (End: rejection sent) | S8 (Wait for FO response) |

---

# Процесс 2: external fleet booking process v1
**Внешний флот — Cat 3 (on-demand, новый SR)**
**Пулы:** Requestor · Booking Tool System · CC Owner · Contract Specialist

---

## Пул: Requestor

| # | Тип элемента | Имя элемента | Входящий поток (от) | Исходящий поток (к) | Примечание |
|---|-------------|-------------|-------------------|-------------------|-----------|
| ER1 | **Start Event** | Justification submitted (entry from main process) | — | ER2 | |
| ER2 | **Event-Based Gateway** | Waiting for outcome | ER1 | ER3, ER5 | ⚠️ Event-Based GW |
| ER3 | **Intermediate Catch Event** (Message) | Request rejected by CC Owner | ER2 | ER4 | Получает сообщение от ES7 |
| ER4 | **End Event** | Request rejected | ER3 | — | |
| ER5 | **Task** (Receive Task) | Receive notification: bid options available | ER2 | ER6 | Получает message от ES10 |
| ER6 | **Task** (User Task) | Review bid options (Bidder 1/2/3: total sum + mob. date, anonymous — R-37) | ER5 | ER7 | |
| ER7 | **Task** (User Task) | Select preferred bid option | ER6 | ER8 | → message flow → ES11 |
| ER8 | **Intermediate Catch Event** (Message) | Booking confirmed (SR issued) | ER7 | ER9 | Получает сообщение от ES17 |
| ER9 | **End Event** | External booking confirmed | ER8 | — | |

### Message flows — пул Requestor (external process)

| Направление | От | К |
|------------|----|----|
| System → Requestor | ES7 (Notify: CC rejected) | ER3 |
| System → Requestor | ES10 (Notify: bids ready) | ER5 |
| System → Requestor | ES17 (Notify: confirmed) | ER8 |
| Requestor → System | ER7 (Select bid) | ES11 (Receive selection) |

---

## Пул: Booking Tool System

| # | Тип элемента | Имя элемента | Входящий поток (от) | Исходящий поток (к) | Примечание |
|---|-------------|-------------|-------------------|-------------------|-----------|
| ES1 | **Start Event** | Receive Justification + Cost Center | — | ES2 | Запускается из Call Activity v3 |
| ES2 | **Task** (Send Task) | Route request to CC Owner | ES1 | ES3 | → message flow → EC1 |
| ES3 | **Task** (Service Task) | Update status: Pending CC Owner approval | ES2 | ES4 | |
| ES4 | **Task** (Receive Task) | Receive CC Owner decision | ES3 | ES5 | Получает message от EC4 или EC6 |
| ES5 | **Exclusive Gateway** | CC Owner decision? | ES4 | ES6 (Rejected), ES8 (Approved) | |
| ES6 | **Task** (Send Task) | Notify Requestor: rejected by CC Owner (with reason) | ES5 (Rejected) | ES7 | → message flow → ER3 |
| ES7 | **Task** (Service Task) | Update status: Rejected | ES6 | ES7e | |
| ES7e | **End Event** | Request rejected by CC Owner | ES7 | — | |
| ES8 | **Task** (Send Task) | Notify Contract Specialist: start bid process | ES5 (Approved) | ES9 | → message flow → ECS1 |
| ES9 | **Task** (Service Task) | Update status: Pending external bid | ES8 | ES10r | |
| ES10r | **Task** (Receive Task) | Receive bid results from Contract Specialist | ES9 | ES10 | Получает message от ECS3 |
| ES10 | **Task** (Send Task) | Notify Requestor: bid options available | ES10r | ES11 | → message flow → ER5 |
| ES11 | **Task** (Receive Task) | Receive Requestor selection | ES10 | ES12 | Получает message от ER7 |
| ES12 | **Task** (Send Task) | Send selection to Contract Specialist | ES11 | ES13 | → message flow → ECS4 |
| ES13 | **Task** (Receive Task) | Receive SR number from Contract Specialist | ES12 | ES14 | Получает message от ECS6 |
| ES14 | **Task** (Service Task) | Link SR number to booking record (R-38) | ES13 | ES15 | |
| ES15 | **Task** (Service Task) | Create RWA draft in DCT *(⚠️ R-42 — уточняется)* | ES14 | ES16 | |
| ES16 | **Task** (Send Task) | Notify Requestor: external booking confirmed | ES15 | ES17 | → message flow → ER8 |
| ES17 | **Task** (Service Task) | Update booking status: Confirmed | ES16 | ES17e | |
| ES17e | **End Event** | External booking confirmed | ES17 | — | |

### Message flows — пул System (external process)

| Направление | От | К |
|------------|----|----|
| System → CC Owner | ES2 (Route to CC Owner) | EC1 (Receive request) |
| System → Contract Specialist | ES8 (Notify CS) | ECS1 (Receive notification) |
| System → Requestor | ES6 (Notify: CC rejected) | ER3 |
| System → Requestor | ES10 (Notify: bids ready) | ER5 |
| System → Requestor | ES16 (Notify: confirmed) | ER8 |
| System → Contract Specialist | ES12 (Send selection) | ECS4 (Receive selection) |

---

## Пул: CC Owner

| # | Тип элемента | Имя элемента | Входящий поток (от) | Исходящий поток (к) | Примечание |
|---|-------------|-------------|-------------------|-------------------|-----------|
| EC1 | **Task** (Receive Task) | Receive request + Justification + Cost Center | — (message от ES2) | EC2 | |
| EC2 | **Task** (User Task) | Review request (business need, cost center, priority) | EC1 | EC3 | |
| EC3 | **Exclusive Gateway** | Confirm / Reject? | EC2 | EC4 (Confirm), EC5 (Reject) | |
| EC4 | **Task** (Send Task) | Approve external procurement | EC3 (Confirm) | EC4e | → message flow → ES4 ("Approved") |
| EC4e | **End Event** | Approved | EC4 | — | |
| EC5 | **Task** (User Task) | Enter rejection reason | EC3 (Reject) | EC6 | |
| EC6 | **End Event** | Rejected | EC5 | — | → message flow → ES4 ("Rejected") |

### Message flows — пул CC Owner

| Направление | От | К |
|------------|----|----|
| CC Owner → System | EC4 (Approve) | ES4 (Receive CC decision) |
| CC Owner → System | EC6 (End: rejected) | ES4 (Receive CC decision) |

---

## Пул: Contract Specialist (Central Contract Group)

| # | Тип элемента | Имя элемента | Входящий поток (от) | Исходящий поток (к) | Примечание |
|---|-------------|-------------|-------------------|-------------------|-----------|
| ECS1 | **Task** (Receive Task) | Receive notification: start bid process | — (message от ES8) | ECS2 | |
| ECS2 | **Task** (User Task) | Conduct external bid procedure (offline: DCT / emails / light bid) | ECS1 | ECS3 | |
| ECS3 | **Task** (User Task) | Enter bid results: Bidder 1/2/3 — total sum + mob. date (anonymous) | ECS2 | ECS3e | → message flow → ES10r |
| ECS3e | *(ждёт ответа)* | *(Contract Specialist ждёт выбора Requestor-а)* | ECS3 | ECS4 | Receive Task или просто ожидание message |
| ECS4 | **Task** (Receive Task) | Receive Requestor selection from system | ECS3e (message от ES12) | ECS5 | |
| ECS5 | **Task** (User Task) | Issue SR in DCT/SMART for selected BP | ECS4 | ECS6 | |
| ECS6 | **Task** (User Task) | Enter SR number into Booking Tool | ECS5 | ECS7 | → message flow → ES13 |
| ECS7 | **End Event** | SR issued and entered | ECS6 | — | |

### Message flows — пул Contract Specialist

| Направление | От | К |
|------------|----|----|
| Contract Specialist → System | ECS3 (Enter bids) | ES10r (Receive bids) |
| Contract Specialist → System | ECS6 (Enter SR) | ES13 (Receive SR) |

---

## Итоговая структура

```
Процесс v3 — внутренний флот
  Requestor:          14 элементов
  System:             19 элементов + 2 Boundary Events (24h / 48h)
  Internal FO:         7 элементов
  Всего sequence flows: ~25
  Всего message flows:  ~9

Процесс v1 — внешний флот
  Requestor:           9 элементов
  System:             17 элементов
  CC Owner:            6 элементов
  Contract Specialist: 7 элементов
  Всего sequence flows: ~26
  Всего message flows:  ~12
```

---

## Порядок создания в Camunda Modeler (рекомендация)

1. Создать Collaboration (не просто Process)
2. Добавить все пулы (Pools)
3. Нарисовать каждый пул слева направо по таблице, не думая пока о message flows
4. Добавить message flows между пулами в конце
5. Проверить: каждый Receive Task / Message Catch Event должен иметь ровно один входящий message flow
6. Нажать **Align Elements / Auto-layout** для причёсывания схемы
