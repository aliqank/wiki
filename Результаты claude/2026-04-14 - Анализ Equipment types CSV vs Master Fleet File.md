# Анализ: Equipment types.csv vs FINAL Master Fleet File HDV HDE.xlsx

**Дата:** 2026-04-14  
**Файлы:**
- `Документация в процессе работы с требованиями/2026-04-14/Equipment types.csv`
- `Исходная документация от заказчика/Project documentation/FINAL Master fleet file HDV HDE.xlsx`

---

## Краткое резюме

| | Equipment types.csv | XLSX — лист Main | XLSX — лист directory all |
|---|---|---|---|
| Строк (единиц техники) | 785 | 785 | n/a (справочник) |
| Уникальных категорий | 37 | 37 | **71** |

**CSV и лист Main полностью совпадают** — одинаковые 785 единиц, одинаковые 37 категорий.  
**Лист "directory all"** содержит вдвое больше категорий (71), большинство из которых не используются в реальном флоте.

---

## 1. Категории CSV/Main: количество техники по типам

| Категория | Кол-во |
|---|---:|
| Frac tank | 91 |
| Trailer (mobile office, pump etc.) | 69 |
| Generator stationary | 67 |
| Lifting equipment motorized | 55 |
| Road construction and cleaning equipment | 52 |
| Lifting equipment electric | 47 |
| Trailer cargo | 44 |
| Generator portable | 39 |
| Pump motorized (do not use) | 36 |
| Heater portable | 32 |
| Air compressor motorized | 30 |
| Fire truck | 27 |
| Lifting (load gripping mechanism) | 26 |
| Lighting set, signal motorized | 22 |
| Food truck | 16 |
| Cleaning, washing tools etc. | 12 |
| Air compressor electric | 11 |
| HD truck (tractor) | 11 |
| Snow blower (do not use) | 11 |
| Firefighting equipment | 11 |
| Snow plough truck | 10 |
| Hook loader truck | 8 |
| Welder motorized (do not use) | 7 |
| Liquid vacuum truck | 7 |
| Flatbed/boom truck (do not use) | 7 |
| Dispossal equipment | 6 |
| Dump truck | 5 |
| Railroad equipment | 5 |
| Recovery/service truck | 4 |
| Wet/dry vacuum system (do not use) | 4 |
| Pump electric (do not use) | 4 |
| Chemical truck | 3 |
| Steam truck | 2 |
| Attachment | 1 |
| Lighting set, signal electric | 1 |
| Hydro excavator (do not use) | 1 |
| Welder electric (do not use) | 1 |
| **ИТОГО** | **785** |

---

## 2. Расхождения: CSV/Main vs directory all

### 2.1 Категории в CSV/Main, которых НЕТ в directory all

| Категория | Кол-во единиц | Примечание |
|---|---:|---|
| `Liquid vacuum truck` | 7 | Отсутствует в справочнике — нужно добавить |
| `Railroad equipment` | 5 | Опечатка в справочнике: написано `Railroad equipmet` (пропущена буква «n») |

**Итого:** 2 категории (12 единиц техники) без корректной записи в directory all.

### 2.2 Категории в directory all, которых НЕТ в CSV/Main (36 штук)

Эти категории присутствуют в справочнике, но ни одна единица техники им не назначена.  
Разбиты на смысловые группы:

**Грузоподъёмное оборудование (более детальная классификация)**
- `Boom lift`, `Manlift`, `Scissor Lift` → в Main объединены в `Lifting equipment motorized/electric`
- `Crawler crane`, `General crane (RT)`, `Mobile crane`, `Overhead crane` → в Main: `Lifting (load gripping mechanism)`
- `Boom truck` → в Main: нет прямого аналога (часть попала в `Flatbed/boom truck (do not use)`)
- `Telescopic handler` → в Main: часть под `Lifting equipment motorized`

**Строительная техника (более детальная)**
- `Backhoe loader`, `Excavator`, `Skid steer loader`, `Bulldozer` → в Main: `Road construction and cleaning equipment`

**Транспорт и спецтехника**
- `Flatbed truck` → в Main: `Flatbed/boom truck (do not use)`
- `Forklift electric`, `Forklift motorized` → в Main: `Lifting equipment electric/motorized`
- `Fuel truck`, `Lubrication truck`, `Water truck`, `Waste collection truck` → в Main: нет прямых аналогов или вложены в другие типы
- `Vacuum truck` → в Main: `Liquid vacuum truck` (+ `Wet/dry vacuum system`)
- `Winch truck` → в Main: нет
- `Street sweeper` → в Main: `Cleaning, washing tools etc.`

**Мобильное оборудование**
- `Mobile crane`, `Mobile generator`, `Mobile lighting tower`, `Mobile office`, `Mobile pump`, `Mobile welding unit`, `Mobile workshop` → в Main: разбросаны по другим категориям

**Прочее (нет эквивалента в Main)**
- `Air breathing system`, `Airport support equipment`, `Concrete mixer truck`, `Jetting equipment`
- `Portable toilet`, `Vector unit`

---

## 3. Ключевые наблюдения

### 3.1 directory all — расширенный, но незаполненный справочник
- Из 71 категории только **5 первых строк** заполнены атрибутами (`Equipment Usage Status`, `HDV/HDE classification`, `Expected metrics from trackers`).
- Остальные 66 строк — без атрибутов. Справочник явно не завершён.

### 3.2 CSV/Main использует укрупнённую классификацию
- Реальный флот (785 единиц) использует только **37 категорий** из 71 возможных.
- Часть категорий directory all — это более детальное дробление того, что в Main объединено в широкие группы:
  - `Road construction and cleaning equipment` покрывает экскаваторы, бульдозеры, скрепера, лоудеры
  - `Lifting equipment motorized/electric` покрывает вилочные погрузчики, манлифты, ножничные подъёмники

### 3.3 Категории с пометкой "(do not use)"
В реальном флоте 11 категорий помечены как "do not use":
- `Flatbed/boom truck (do not use)` — 7 ед.
- `Snow blower (do not use)` — 11 ед.
- `Pump motorized/electric (do not use)` — 40 ед.
- `Hydro excavator (do not use)` — 1 ед.
- `Wet/dry vacuum system (do not use)` — 4 ед.
- `Welder motorized/electric (do not use)` — 8 ед.

Итого **71 единица техники** в устаревших/нежелательных категориях. Требует решения: переклассифицировать или оставить.

### 3.4 Опечатка в directory all
`Railroad equipmet` → правильно: `Railroad equipment`  
Из-за опечатки 5 единиц реальной техники не связаны с категорией справочника.

---

## 4. Вопросы для заказчика / команды

| # | Вопрос | Приоритет |
|---|---|---|
| Q1 | `Liquid vacuum truck` отсутствует в directory all — добавить? Или переименовать в `Vacuum truck`? | Высокий |
| Q2 | Исправить опечатку `Railroad equipmet` → `Railroad equipment` в directory all? | Высокий |
| Q3 | Какой из двух справочников является **мастером**: directory all (71 кат.) или фактические категории Main (37 кат.)? | Высокий |
| Q4 | 36 категорий directory all без техники — это плановые типы (Phase 2?) или устаревший черновик? | Средний |
| Q5 | Заполнить `Equipment Usage Status` / `HDV/HDE classification` / `Expected metrics` для всех 71 категорий? | Средний |
| Q6 | Техника в категориях "(do not use)" (71 ед.) — переклассифицировать в новые типы или заморозить? | Средний |

---

## 5. Рекомендация для БД Booking Tool

На основании анализа **рекомендуется использовать 37 фактических категорий** из CSV/Main как основу для справочника `equipment_types` в БД, с двумя корректировками:

1. Добавить `Liquid vacuum truck` (7 ед.) — есть в данных, нет в directory all.
2. Исправить опечатку: `Railroad equipment` (не `Railroad equipmet`).
3. Получить от TCO решение по категориям "(do not use)" — нужно ли их отображать Requestor-у в UI поиска.
