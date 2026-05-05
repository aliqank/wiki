**Дата создания:** 2026-04-14  
**Последнее обновление:** 2026-04-14  
**Автор:** Тельман Нуржанов (SA)

---

# Архитектура БД — Booking Tool

## 1. Назначение и границы

**БД Booking Tool** хранит исключительно данные о бронировании техники.

| Хранится в этой БД | Хранится в других БД |
|---|---|
| Заявки (Request) | Пользователи (Users) |
| Брони (Booking) | Техника (Equipment) |
| Снапшоты атрибутов техники на момент подтверждения | Парки (Fleet) и Fleet Owner-ы |
| История статусов заявок и броней | Мониторинг и утилизация (трекеры) |
| Периоды заморозки техники | Уведомления |
| Делегирование прав FO | |
| Записи закрытия брони (фактическое время) | |
| Фидбек по броням | |
| Транспортные подзаявки (для негабарита) | |

Ссылки на пользователей, технику, парки и Fleet Owner-ов хранятся как **внешние ID** — без join-таблиц и без репликации атрибутов.

---

## 2. Принципы проектирования

- **Order / OrderLine pattern:** один Request → несколько Booking (по одному на каждую единицу техники).
- **Один Work Order на заявку:** поле `work_order_jde_id` живёт на уровне Request.
- **Снапшот при подтверждении:** при переходе Booking в статус `Confirmed` сохраняется снапшот ключевых атрибутов техники (для истории).
- **Внешние ID без join:** `user_id`, `equipment_id`, `fleet_id`, `fleet_owner_id` — только идентификаторы; атрибуты берутся из внешних сервисов по API.
- **Мягкое удаление:** техника и парки управляются в других БД; в этой БД записи не удаляются физически.

---

## 3. Таблицы

### 3.1 `request` — Заявка

Контейнер для одного набора броней, привязанный к одному Work Order.

| Колонка | Тип | Обязательность | Описание |
|---|---|---|---|
| `id` | UUID | PK | Уникальный ID заявки |
| `type` | ENUM | NOT NULL | `regular` / `service_work` |
| `status` | ENUM | NOT NULL | `draft`, `submitted`, `in_progress`, `completed`, `cancelled` |
| `work_order_jde_id` | VARCHAR | nullable | ID Work Order из JDE (обязателен для Maintenance/Railroad/Operations) |
| `location` | VARCHAR | nullable | Локация (обязательна для SCM Logistics вместо Work Order) |
| `work_description` | VARCHAR(255) | NOT NULL | Описание работ (~50 символов — мягкое ограничение) |
| `comments` | TEXT | nullable | Комментарий заявителя |
| `priority` | ENUM | NOT NULL | `P1`, `P2`, `P3`, `P4` |
| `created_by` | UUID | NOT NULL | Внешний ID пользователя (Requestor / SWP) |
| `created_at` | TIMESTAMP | NOT NULL | Время создания |
| `updated_at` | TIMESTAMP | NOT NULL | Время последнего изменения |

**Индексы:** `created_by`, `status`, `work_order_jde_id`

---

### 3.2 `booking` — Бронь

Одна бронь на одну единицу техники. Несколько броней образуют заявку.

| Колонка | Тип | Обязательность | Описание |
|---|---|---|---|
| `id` | UUID | PK | Уникальный ID брони |
| `request_id` | UUID | FK → request | Родительская заявка |
| `equipment_id` | UUID | NOT NULL | Внешний ID единицы техники |
| `fleet_id` | UUID | NOT NULL | Внешний ID парка |
| `fleet_owner_id` | UUID | nullable | Внешний ID FO, назначенного на эту бронь (может меняться при делегировании) |
| `status` | ENUM | NOT NULL | `draft`, `submitted`, `confirmed`, `declined`, `revoked`, `terminated`, `in_progress`, `closed` |
| `start_dt` | TIMESTAMP | NOT NULL | Начало бронирования (= начало мобилизации) |
| `end_dt` | TIMESTAMP | NOT NULL | Конец бронирования |
| `mobilization_started_at` | TIMESTAMP | nullable | Фактическое нажатие кнопки «Мобилизация началась» (FO) |
| `justification` | TEXT | nullable | Обоснование (обязательно для Assigned и Shared with Conditions) |
| `decline_reason` | TEXT | nullable | Причина отказа (FO) |
| `terminate_reason` | TEXT | nullable | Причина досрочного завершения |
| `closed_by` | UUID | nullable | Внешний ID пользователя, закрывшего бронь |
| `closed_at` | TIMESTAMP | nullable | Время закрытия |
| `created_at` | TIMESTAMP | NOT NULL | |
| `updated_at` | TIMESTAMP | NOT NULL | |

**Индексы:** `request_id`, `equipment_id`, `fleet_id`, `status`, `(equipment_id, start_dt, end_dt)` — для проверки доступности

---

### 3.3 `booking_close_record` — Закрытие брони

Фиксирует фактическое время работы при закрытии брони (используется для аналитики утилизации).

| Колонка | Тип | Обязательность | Описание |
|---|---|---|---|
| `id` | UUID | PK | |
| `booking_id` | UUID | FK → booking, UNIQUE | Одна запись на одну бронь |
| `actual_start_at` | TIMESTAMP | NOT NULL | Фактическое время начала работ |
| `actual_end_at` | TIMESTAMP | NOT NULL | Фактическое время окончания работ |
| `closed_by` | UUID | NOT NULL | Внешний ID пользователя (Requestor или FO) |
| `created_at` | TIMESTAMP | NOT NULL | |

---

### 3.4 `booking_snapshot` — Снапшот техники на момент подтверждения

Сохраняет ключевые атрибуты техники в момент перехода брони в `Confirmed`. Обеспечивает историческую читаемость при последующем изменении данных техники.

| Колонка | Тип | Обязательность | Описание |
|---|---|---|---|
| `id` | UUID | PK | |
| `booking_id` | UUID | FK → booking, UNIQUE | |
| `equipment_id` | UUID | NOT NULL | Внешний ID (копия) |
| `equipment_model` | VARCHAR | NOT NULL | Модель/марка техники |
| `license_plate` | VARCHAR | nullable | Госномер (если есть) |
| `tco_number` | VARCHAR | nullable | ТШО-номер (если есть) |
| `fleet_name` | VARCHAR | NOT NULL | Название парка |
| `snapshotted_at` | TIMESTAMP | NOT NULL | Время создания снапшота |

---

### 3.5 `booking_status_history` — История статусов брони

Полная история переходов статусов для аудита.

| Колонка | Тип | Обязательность | Описание |
|---|---|---|---|
| `id` | UUID | PK | |
| `booking_id` | UUID | FK → booking | |
| `old_status` | ENUM | nullable | Предыдущий статус (null при создании) |
| `new_status` | ENUM | NOT NULL | Новый статус |
| `changed_by` | UUID | NOT NULL | Внешний ID пользователя / `SYSTEM` |
| `changed_at` | TIMESTAMP | NOT NULL | |
| `comment` | TEXT | nullable | Причина перехода (если указана) |

**Индекс:** `booking_id`, `changed_at`

---

### 3.6 `request_status_history` — История статусов заявки

Аналогично `booking_status_history`, но для агрегированного статуса заявки.

| Колонка | Тип | Обязательность | Описание |
|---|---|---|---|
| `id` | UUID | PK | |
| `request_id` | UUID | FK → request | |
| `old_status` | ENUM | nullable | |
| `new_status` | ENUM | NOT NULL | |
| `changed_by` | UUID | NOT NULL | Внешний ID пользователя / `SYSTEM` |
| `changed_at` | TIMESTAMP | NOT NULL | |

**Индекс:** `request_id`, `changed_at`

---

### 3.7 `equipment_freeze` — Заморозка техники

Период, на который FO заморозил технику (недоступна для бронирования).

| Колонка | Тип | Обязательность | Описание |
|---|---|---|---|
| `id` | UUID | PK | |
| `equipment_id` | UUID | NOT NULL | Внешний ID техники |
| `fleet_owner_id` | UUID | NOT NULL | Внешний ID FO, создавшего заморозку |
| `freeze_from` | TIMESTAMP | NOT NULL | Начало заморозки |
| `freeze_until` | TIMESTAMP | nullable | Конец заморозки (null = бессрочно) |
| `reason` | TEXT | nullable | Причина заморозки |
| `is_active` | BOOLEAN | NOT NULL | Флаг активности (для soft-отмены) |
| `created_at` | TIMESTAMP | NOT NULL | |
| `updated_at` | TIMESTAMP | NOT NULL | |

**Индекс:** `equipment_id`, `(equipment_id, freeze_from, freeze_until)` — для проверки доступности

---

### 3.8 `fo_delegation` — Делегирование прав Fleet Owner

FO делегирует свои права на определённый срок другому сотруднику TCO (без участия IT).

| Колонка | Тип | Обязательность | Описание |
|---|---|---|---|
| `id` | UUID | PK | |
| `fleet_id` | UUID | NOT NULL | Внешний ID парка |
| `delegated_from` | UUID | NOT NULL | Внешний ID FO-делегатора |
| `delegated_to` | UUID | NOT NULL | Внешний ID сотрудника-получателя |
| `valid_from` | DATE | NOT NULL | Начало делегирования |
| `valid_until` | DATE | NOT NULL | Конец делегирования (обязательно, автоотзыв по истечении) |
| `is_active` | BOOLEAN | NOT NULL | Флаг активности |
| `revoked_at` | TIMESTAMP | nullable | Время досрочного отзыва |
| `revoked_by` | UUID | nullable | Кто отозвал (Admin или сам FO) |
| `created_at` | TIMESTAMP | NOT NULL | |

**Статус:** ⚠️ Pending cybersecurity sign-off (OQ-NEW-E)

---

### 3.9 `transport_subrequest` — Транспортная подзаявка

Создаётся FO для брони негабаритной (Unwheeled) техники, когда требуется транспортировка. Transportation Responsible подтверждает/отказывает.

| Колонка | Тип | Обязательность | Описание |
|---|---|---|---|
| `id` | UUID | PK | |
| `booking_id` | UUID | FK → booking | Родительская бронь (для Unwheeled equipment) |
| `transport_equipment_id` | UUID | NOT NULL | Внешний ID транспортного средства (тягач/трал) |
| `transport_responsible_id` | UUID | nullable | Внешний ID ответственного за транспортировку |
| `status` | ENUM | NOT NULL | `pending`, `confirmed`, `declined` |
| `decline_reason` | TEXT | nullable | |
| `created_at` | TIMESTAMP | NOT NULL | |
| `updated_at` | TIMESTAMP | NOT NULL | |

---

### 3.10 `booking_feedback` — Фидбек по брони

Отзыв заявителя или SWP о забронированной технике. Доступен с момента подтверждения брони.

| Колонка | Тип | Обязательность | Описание |
|---|---|---|---|
| `id` | UUID | PK | |
| `booking_id` | UUID | FK → booking | |
| `submitted_by` | UUID | NOT NULL | Внешний ID пользователя |
| `rating` | SMALLINT | nullable | Оценка (диапазон TBD) |
| `comment` | TEXT | nullable | Текстовый комментарий |
| `submitted_at` | TIMESTAMP | NOT NULL | |

---

## 4. Связи между таблицами

```
request (1) ──────────── (N) booking
                                │
              ┌─────────────────┼──────────────────────────┐
              │                 │                          │
    booking_snapshot    booking_close_record    booking_status_history
    (0..1)              (0..1)                  (N)
              │
              ├─── transport_subrequest (0..1)
              └─── booking_feedback (0..N)

request (1) ──── (N) request_status_history

equipment_freeze — отдельная таблица, не связана с booking напрямую
fo_delegation   — отдельная таблица, не связана с booking напрямую
```

---

## 5. Перечень ENUM-значений

### `request.type`
`regular` | `service_work`

### `request.status`
`draft` | `submitted` | `in_progress` | `completed` | `cancelled`

### `booking.status`
`draft` | `submitted` | `confirmed` | `declined` | `revoked` | `terminated` | `in_progress` | `closed`

> ⚠️ **CONFLICT-02 (открыт):** автоматические переходы `confirmed → in_progress → closed` по дате vs. ручное закрытие (FR-NEW-32). До получения sign-off от TCO — только ручное закрытие.

### `transport_subrequest.status`
`pending` | `confirmed` | `declined`

---

## 6. Открытые вопросы по БД

| # | Вопрос | Связь с BRD |
|---|---|---|
| 1 | Точный перечень агрегированных статусов заявки (`request.status`) | OQ-36 |
| 2 | Автоматические переходы статусов брони vs. ручное закрытие | CONFLICT-02 |
| 3 | Делегирование FO — ограничения по объёму прав (вся бронь / отдельные действия) | OQ-NEW-E |
| 4 | Снапшот брони — полный список атрибутов для сохранения | OQ-37 |
| 5 | Механизм отзыва FO подтверждённой брони (Recall) — хранится как отдельное событие или как смена статуса? | BRD §5.1 |
