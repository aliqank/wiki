# Pencil.dev — UI Prompt — Booking Tool v1
**Дата:** 2026-04-07
**Язык промпта:** English
**Использование:** Вставить в Pencil.dev (или аналогичный AI-дизайн инструмент) для генерации макетов

---

## General Design Specification

```
Design a web application UI for TCO HDV/HDE Booking Tool — an enterprise heavy equipment booking
system for an oil & gas company (Tengizchevroil, Kazakhstan).

DESIGN SYSTEM:
- Layout: Desktop web, 1440px width
- Color scheme: Dark enterprise palette — deep navy (#0F1A2E) background for headers/nav,
  light gray (#F4F5F7) for page background, white (#FFFFFF) for cards/panels
- Accent colors: Blue (#1F6FEB) for primary actions, Orange (#F59E0B) for P1 priority and warnings,
  Green (#10B981) for confirmed/active status, Red (#EF4444) for rejected/overdue,
  Gray (#6B7280) for secondary text
- Typography: Inter or similar sans-serif. Headers: 24px bold. Section titles: 16px semibold.
  Body: 14px regular. Labels: 12px medium uppercase
- Components: rounded corners (8px), subtle shadows on cards, solid borders for inputs
- Icons: Lucide or similar line icon set. Equipment icons for each type.
- Priority chips: P1 = orange, P2 = yellow, P3 = blue, P4 = gray
- Status badges: rounded pill style
- Navigation: Fixed left sidebar (240px) + top header bar (64px)
- Language: Russian labels on all UI elements

ROLES:
- Requestor: employee who books equipment
- Fleet Owner (FO): department that owns and manages equipment fleet
```

---

## Screen 1 — Requestor Dashboard (R-01)

```
SCREEN: Requestor Dashboard
ROLE: Requestor
URL: /dashboard

LAYOUT:
- Fixed left sidebar (240px): Logo "HDV/HDE Booking" at top, user avatar + name + "Requestor"
  label, nav items: [Главная (active), Каталог техники, Мои заявки]
- Top header (64px): breadcrumb "Главная", notification bell icon with red badge "3",
  user menu dropdown
- Main content area: 2-row layout

ROW 1 — Stats cards (4 equal columns, gap 16px):
  Card 1: icon=clock, label="Ожидают ответа FO", value="4", subtitle="заявки в очереди", color accent=blue
  Card 2: icon=alert-triangle, label="Требуют вашего действия", value="2",
           subtitle="FO отказал — выберите другую технику", color accent=orange, highlighted border
  Card 3: icon=check-circle, label="Завершены (30 дней)", value="11", color accent=green
  Card 4: icon=x-circle, label="Отклонено", value="3", color accent=gray

ROW 2 — Two columns (60/40 split):
  LEFT PANEL "Требуют вашего действия" (orange left border):
    - Section title with orange dot indicator
    - 2 request rows, each showing:
      [equipment icon] "Погрузчик 4–8т · Флот Логистики" | P1 chip | "FO отказал: на ТО" |
      Button "Выбрать другую технику" (orange, outline)
  RIGHT PANEL "Активные брони":
    - Section title
    - 3 request rows: [type] [period from-to] [status badge "В работе"] [">"]
    - "Показать все" link at bottom

BUTTON: Floating primary button bottom-right "+ Новая заявка" (blue, rounded)
```

---

## Screen 2 — Equipment Catalog (R-02)

```
SCREEN: Equipment Catalog — Search & Select
ROLE: Requestor
URL: /catalog

LAYOUT:
- Left sidebar: navigation (same as dashboard)
- Top area: page title "Каталог техники" + subtitle "Выберите конкретную единицу для бронирования"
- Filter bar (full width, card with light background):
  [Тип техники ▼] [Дата начала 📅] [Дата окончания 📅] [Button "Найти" blue]
  [Expander "Характеристики ▼"] — when expanded shows dynamic fields:
    if Погрузчик: "Грузоподъёмность" segmented control [2-4т] [4-8т] [Любая]
    if Компрессор: "Давление (bar, мин)" number input + "Объём подачи (м³/мин, мин)" number input
    if Генератор: "Мощность (кВт, мин)" number input

RESULTS area: "Найдено 8 единиц" subtitle + grid of cards (3 columns)

EQUIPMENT CARD (show 6 cards in mockup):
  Card structure (white, 8px radius, hover shadow):
  - Top: equipment type icon (forklift/compressor/generator SVG icon) + type label
  - Middle:
    Model: "Hyster H4.0FT"
    Fleet Owner: "Флот Логистики • Иванов А."
    Characteristics: "⚖️ 4–8 т грузоподъёмность"
    Period: "✅ Свободна с 08.04 по 12.04"
  - Status badge bottom-left: "Shared" (green) or "Shared with Conditions" (yellow with tooltip icon)
  - Mobility indicator: if stationary — show truck icon + "⚠️ Потребуется трал"
  - Button bottom-right: "Выбрать" (blue primary button, full width of card)

Show 2 cards with "Shared" badge, 1 with "Shared with Conditions" badge,
1 card with stationary+tral warning icon

Empty state variation (show small): centered icon + "Техника не найдена. Измените параметры поиска."
```

---

## Screen 3 — Create Request (R-03)

```
SCREEN: Create Request Form
ROLE: Requestor
URL: /requests/new

LAYOUT:
- Two-column layout: main form (65%) + summary panel (35%)

MAIN FORM (white card, padding 32px):
  Title: "Новая заявка" h1

  SECTION "Выбранная техника" (light blue tinted card, 8px radius):
    [Forklift icon, 48x48]
    "Погрузчик 4–8т" — bold title
    "Hyster H4.0FT" — model
    "Флот Логистики · Иванов А.А." — fleet owner
    "08.04.2026, 08:00 — 12.04.2026, 17:00" — period
    "⚖️ 4–8 т" — characteristic chip
    [Link "Изменить технику →"] — right-aligned

  ALERT BANNER (yellow, only for stationary equipment):
    🚛 icon + "К заявке будет добавлен трал. Fleet Owner назначит транспорт после подтверждения."

  SECTION "Параметры заявки":
    Field: "JDE Work Order №" * (required)
      Input: text, placeholder "Введите номер WO, например: WO-2026-04812"
      Helper: "Номер заказа на работу из JDE E1. Используется FO для определения точки доставки."
    Field: "Приоритет" *
      Segmented control with 4 options:
        [P1 — Аварийный] [P2 — Срочный] [P3 — Плановый] [P4 — Низкий]
        P1 = orange background when selected, others = gray outline
    Field: "Описание работ" (optional)
      Textarea, placeholder "Краткое описание задачи (необязательно)", 3 rows

  Buttons row: [Подать заявку — blue primary, large] [← Назад в каталог — ghost button]

SUMMARY PANEL (right, sticky):
  Card "Сводка заявки":
    Equipment type + icon
    Period
    Fleet Owner
    Priority (if selected)
    Divider
    "Техника: подвижная" or "Техника: стационарная + трал"
```

---

## Screen 4 — My Requests List (R-04)

```
SCREEN: My Requests — List
ROLE: Requestor
URL: /requests

LAYOUT:
- Full-width table view
- Title "Мои заявки" + "+ Новая заявка" button (top right)
- Filter row (below title):
  [All Statuses ▼] [All Priorities ▼] [Date range picker] [Search by WO#]

TABLE COLUMNS:
  # | Тип техники | Fleet Owner | Период | Приоритет | Статус | Создана | Действия

TABLE ROWS (show 7 rows with different statuses):
  Row 1: Погрузчик 4-8т | Флот Логистики | 08.04–12.04 | [P1 orange chip] | [🟠 Требует действия] | 07.04 | [•••]
  Row 2: Генератор 250кВт | Флот Энергетики | 10.04–14.04 | [P2 yellow chip] | [🟡 Ожидает FO] + ⏱ 18ч | 07.04 | [•••]
  Row 3: Компрессор | Флот Логистики | 09.04–10.04 | [P3 blue chip] | [🟢 Подтверждено] | 06.04 | [•••]
  Row 4: Кран 25т | Флот ТО | 05.04–07.04 | [P1 orange chip] | [✅ Завершено] | 04.04 | [•••]
  Row 5: Погрузчик 2-4т | Флот Логистики | 14.04–15.04 | [P3 blue chip] | [🔵 Черновик] | 07.04 | [•••]
  Row 6: Генератор 100кВт | Флот Энергетики | 03.04–05.04 | [P2 yellow chip] | [🔴 Отклонено] | 02.04 | [•••]
  Row 7: Компрессор | Флот ТО | 11.04–13.04 | [P2 yellow chip] | [🟢 В работе] | 06.04 | [•••]

Row 1 (Action Required): highlighted with light orange left border
Row 2 (Pending FO): show inline timer "⏱ 18ч до эскалации" in orange text next to badge

STATUS BADGES (pill style):
  Черновик = gray | Ожидает FO = blue | Требует действия = orange |
  Подтверждено = teal | В работе = green | Завершено = dark green + check |
  Отклонено = red | Отозвана = light gray
```

---

## Screen 5 — Request Detail — Requestor (R-05)

```
SCREEN: Request Detail (Requestor View)
ROLE: Requestor
URL: /requests/:id

LAYOUT: Single wide card, full content width

HEADER ROW:
  Left: "#REQ-2026-0047" (title) + [🟠 Требует действия] badge
  Right: Action buttons — [Редактировать] [Отозвать заявку ▼]

STATUS BANNER (orange, full width):
  ⚠️ "Fleet Owner отказал в бронировании: Техника на техническом обслуживании до 15.04."
  Button: [Выбрать другую технику →] (orange, right side)

TWO-COLUMN BODY (60/40):

LEFT COLUMN:
  CARD "Техника":
    Icon + "Погрузчик 4–8т" title
    Model: Hyster H4.0FT
    Fleet Owner: "Флот Логистики · Иванов А.А."
    Characteristics: ⚖️ 4–8 т
    Period: 📅 08.04.2026 08:00 – 12.04.2026 17:00
    Status: [🔴 Отказано FO]

  CARD "Параметры":
    WO: WO-2026-04812
    Приоритет: [P1 chip]
    Описание: "Плановое ТО насосного оборудования на TGPP-3"

RIGHT COLUMN:
  CARD "История событий" (timeline):
    ✅ green dot — "07.04 14:23 — Заявка создана"
    ✅ green dot — "07.04 14:24 — Отправлена Fleet Owner Иванов А.А."
    ⏳ blue dot — "07.04 16:00 — Ожидание ответа FO"
    ❌ red dot — "07.04 18:45 — Отклонена FO"
    ⚪ gray dot — "Ожидает вашего действия"

  CARD "Действия" (if In Progress status, show this variant):
    [✅ Завершить бронирование — green primary button]
    [⏩ Запросить продление — outline button]
```

---

## Screen 6 — FO Incoming Requests (FO-02)

```
SCREEN: Incoming Requests — Fleet Owner
ROLE: Fleet Owner
URL: /fo/requests

LAYOUT:
- Left sidebar navigation: Главная · Входящие заявки (active) · Мой парк
- Title "Входящие заявки" + [name FO] badge "Флот Логистики"
- Filter tabs: [Все (12)] [Новые (3)] [Подтверждённые (5)] [В работе (2)] [Завершённые (2)]

TABLE:
COLUMNS: Заявитель | Техника | Период | Приоритет | Тайм-аут | Статус | Трал | Действия

ROWS (show 5):
  Row 1 [highlighted red border]: Сидоров К.В. (Отдел ТО) | Погрузчик 4-8т Hyster | 08.04–12.04 |
         [P1 🔴] | ⏱ 2ч 14м (urgent red text) | [🆕 Новая] | — | [Рассмотреть]
  Row 2: Петров А.Н. (Логистика) | Генератор 250кВт | 10.04–14.04 | [P1 🔴] | ⏱ 6ч | [🆕 Новая] |
         🚛 (tral icon) | [Рассмотреть]
  Row 3: Иванова О.С. (Major Maint) | Компрессор 10 bar | 09.04–10.04 | [P2] | — | [🟢 Подтверждено] | — | [Детали]
  Row 4: Джаксыбеков М. (Эксплуатация) | Погрузчик 2-4т | 11.04–12.04 | [P3] | ⏱ 20ч | [🆕 Новая] | — | [Рассмотреть]
  Row 5: Бекова А.К. (ТО) | Кран 25т | 12.04–14.04 | [P2] | — | [🟢 В работе] | 🚛 | [Детали]

Row 1 (P1 + urgent timer): light red row background, red timer text "⏱ 2ч 14м"
Row 2 with tral icon: show 🚛 truck icon in tral column with tooltip "Требуется трал"

BADGE "3 новые заявки требуют ответа" — banner at top of table if pending exist
```

---

## Screen 7 — Request Detail — Fleet Owner (FO-03)

```
SCREEN: Request Detail (Fleet Owner View)
ROLE: Fleet Owner
URL: /fo/requests/:id

HEADER:
  Left: "#REQ-2026-0047" + [🆕 Новая] badge
  Right: ⏱ "Осталось: 2 ч 14 мин" (orange countdown) + [P1 🔴] chip

ALERT BANNER (blue info):
  📋 "Запрос от Сидоров К.В. · Отдел ТО · WO-2026-04812"

TWO-COLUMN BODY:

LEFT (65%):
  CARD "Запрошенная техника (из вашего парка)":
    [Forklift icon] Погрузчик 4–8т
    Модель: Hyster H4.0FT (S/N: HY-2024-1876)
    Характеристики: ⚖️ 4–8 т
    Период: 08.04.2026 08:00 – 12.04.2026 17:00
    Доступность: [✅ Свободна в запрошенный период] (green badge)

  CARD "Параметры заявки":
    Заявитель: Сидоров К.В., Отдел ТО
    Work Order: WO-2026-04812
    Приоритет: P1 — Аварийный
    Описание: "Плановое ТО насосного оборудования на TGPP-3"

  CARD "Транспорт" (show only for stationary equipment):
    🚛 Требуется транспорт для доставки
    Статус: [⚠️ Не назначен]
    [Назначить транспорт] button (outline, blue)

RIGHT (35%):
  CARD "Принять решение":
    [✅ Принять заявку — green primary button, full width, large]
    [✕ Отклонить — red outline button, full width]

    Divider "или"

    Note: "После принятия — уведомление будет отправлено заявителю"

  CARD "История":
    Timeline: Создана · Отправлена вам · Ожидание ответа (active)
```

---

## Screen 8 — My Fleet (FO-05)

```
SCREEN: My Fleet
ROLE: Fleet Owner
URL: /fo/fleet

TITLE: "Мой парк · Флот Логистики"
SUBTITLE STATS row: "Всего: 24 ед. · Доступно: 17 · Занято: 5 · На ТО: 2"

FILTER row: [Все типы ▼] [Статус ▼] [Подвижная/Стационарная ▼] + [Search] + [+ Добавить технику] button blue

TABLE:
COLUMNS: Тип | Модель / S/N | Характеристики | Движимость | Share-статус | Текущий статус | Действия

ROWS (show 6):
  Row 1: 🚜 Погрузчик | Hyster H4.0FT / HY-2024-1876 | ⚖️ 4–8 т | 🚗 Подвижная |
         [Shared 🟢] | [✅ Свободна] | [✏ Ред.] [❄ Заморозить]
  Row 2: 🚜 Погрузчик | Toyota 8FBN25 / TY-2023-0432 | ⚖️ 2–4 т | 🚗 Подвижная |
         [Shared with Conditions 🟡] | [🔒 Занята до 12.04] | [✏] [❄]
  Row 3: ⚡ Генератор | Caterpillar D250 / CAT-2022-7821 | ⚡ 250 кВт | 📦 Стационарная |
         [Shared 🟢] | [✅ Свободна] | [✏] [❄]
  Row 4: 💨 Компрессор | Atlas Copco XAS185 / AC-2021-3341 | 💨 10 bar · 185 м³/ч | 📦 Стационарная |
         [Assigned 🔵] | [🔒 Занята до 15.04] | [✏] [❄]
  Row 5: 🏗 Кран | Liebherr LTM1025 / LH-2019-0098 | 🏗 25 т | 🚗 Подвижная |
         [Shared 🟢] | [❄ На ТО (заморожена)] | [✏] [▶ Разморозить]
  Row 6: ⚡ Генератор | FG Wilson P100 / FG-2023-5512 | ⚡ 100 кВт | 📦 Стационарная |
         [Shared 🟢] | [✅ Свободна] | [✏] [❄]

Share-status badges: Assigned=blue, Shared=green, Shared with Conditions=amber
```

---

## Screen 9 — Equipment Card Form (FO-06)

```
SCREEN: Equipment Card — Add / Edit
ROLE: Fleet Owner
URL: /fo/fleet/new  or  /fo/fleet/:id/edit

TITLE: "Добавить технику" (or "Редактировать технику")

FORM LAYOUT — two columns inside white card:

LEFT COLUMN:
  Section "Основная информация":
    Field: Тип техники * [dropdown: Погрузчик / Кран / Генератор / Компрессор / Другое]
    Field: Производитель * [text input, e.g. "Hyster"]
    Field: Модель * [text input, e.g. "H4.0FT"]
    Field: Серийный номер * [text input]
    Field: Госномер [text input, optional]

  Section "Движимость":
    Radio group: [🚗 Подвижная] [📦 Стационарная]
    Note (if stationary): "ℹ️ При бронировании этой техники FO назначит транспорт для доставки"

RIGHT COLUMN:
  Section "Технические характеристики" (dynamic by type):
    if Погрузчик:
      "Грузоподъёмность (т)" [number input]
    if Генератор:
      "Мощность (кВт)" [number input]
    if Компрессор:
      "Давление (bar)" [number input]
      "Объём подачи (м³/ч)" [number input]

  Section "Настройки доступности":
    "Share-статус" [radio cards with descriptions]:
      [Assigned — только для вашего отдела]
      [Shared — доступна всем отделам]  ← selected, highlighted
      [Shared with Conditions — с ограничениями]

    "Статус техники" toggle:
      [🟢 Активна] / [❄️ Заморожена (ТО/ремонт)]

FORM FOOTER:
  [Сохранить — blue primary] [Отмена — ghost]
  (only on edit): [🗑 Удалить технику — red link, right-aligned]
```

---

## Modal 1 — Reject Request (M-FO1)

```
MODAL: Отклонить заявку
TRIGGER: "Отклонить" button on FO-03
SIZE: 480px wide, centered overlay with dark backdrop

CONTENT:
  Header: "Отклонить заявку #REQ-2026-0047" + ✕ close button

  Info row (light gray):
    "Заявитель: Сидоров К.В. · Погрузчик 4–8т · 08.04–12.04"

  Field: "Причина отказа *" (required)
    Textarea, 4 rows, placeholder "Укажите причину отказа..."
    Quick-select chips below textarea:
      [На техническом обслуживании] [Занята в этот период] [Не в наличии] [Другое]

  Warning note (yellow box):
    "⚠️ После отказа заявка вернётся к Requestor-у для выбора другой техники. Причина будет показана заявителю."

  Footer buttons:
    [✕ Подтвердить отказ — red primary, disabled until reason filled]
    [Отмена — ghost]
```

---

## Modal 2 — Complete Booking (M-R1)

```
MODAL: Завершить бронирование
TRIGGER: "Завершить" button on R-05 (status: In Progress)
SIZE: 440px wide

CONTENT:
  Header: "Завершить бронирование?" + ✕ close

  Equipment summary card (light gray):
    [Forklift icon] Погрузчик 4–8т · Hyster H4.0FT
    Период: 08.04 – 12.04.2026

  Info note (blue):
    "✅ После завершения техника будет помечена как доступная и сможет использоваться другими."

  Footer:
    [✅ Подтвердить завершение — green primary]
    [Отмена — ghost]
```

---

## Modal 3 — Extend Booking Request (M-R2)

```
MODAL: Запрос на продление
TRIGGER: "Запросить продление" on R-05 (status: Confirmed/In Progress)
SIZE: 500px wide

CONTENT:
  Header: "Запрос на продление бронирования" + ✕

  Current booking info (readonly chip row):
    "Текущий период: 08.04.2026 08:00 – 12.04.2026 17:00"

  Field: "Продлить до *"
    DateTime picker (date + time), must be > current end time

  Field: "Комментарий" (optional)
    Textarea, 2 rows, placeholder "Причина продления (необязательно)"

  Warning banner (amber, show conditionally if next booking exists):
    "⚠️ У данной техники следующее бронирование: Иванова О.С. с 13.04 08:00.
     FO примет решение с учётом этого."

  Footer:
    [Отправить запрос — blue primary]
    [Отмена — ghost]
```

---

## Style Reference (для применения ко всем экранам)

```
COLORS (hex):
  Background page: #F4F5F7
  Sidebar: #0F1A2E
  Sidebar text active: #FFFFFF
  Sidebar text inactive: #8B9DC3
  Header: #FFFFFF with border-bottom #E5E7EB
  Card background: #FFFFFF
  Card border: #E5E7EB
  Primary blue: #1F6FEB
  Success green: #10B981
  Warning orange: #F59E0B
  Danger red: #EF4444
  Text primary: #111827
  Text secondary: #6B7280
  Text disabled: #9CA3AF

STATUS BADGE STYLES (pill, 6px radius, 12px font):
  Черновик: bg #F3F4F6, text #6B7280
  Ожидает FO: bg #DBEAFE, text #1D4ED8
  Требует действия: bg #FEF3C7, text #D97706, border #F59E0B
  Подтверждено: bg #D1FAE5, text #065F46
  В работе: bg #ECFDF5, text #047857, left accent border green
  Завершено: bg #F0FDF4, text #166534
  Отклонено: bg #FEE2E2, text #991B1B
  Новая (FO inbox): bg #EFF6FF, text #1D4ED8

PRIORITY CHIPS:
  P1: bg #FEF3C7, text #92400E, dot 🔴
  P2: bg #FFFBEB, text #B45309
  P3: bg #EFF6FF, text #1E40AF
  P4: bg #F9FAFB, text #6B7280

EQUIPMENT TYPE ICONS (use outline icons):
  Погрузчик: forklift icon
  Генератор: zap / lightning icon
  Компрессор: wind / air icon
  Кран: crane / tool icon

TRAL / TRANSPORT INDICATOR:
  Truck icon (filled amber) + tooltip "Требуется транспорт для доставки"
  If assigned: truck icon (green) + "Трал назначен"
  If not assigned: truck icon (red outline) + "Транспорт не назначен"
```
