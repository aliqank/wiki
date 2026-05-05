# Materials — TCO HDV/HDE Booking Tool

> Навигатор по всем файлам проекта. Обновлять при добавлении новых материалов.
> Медиафайлы (mp4, mp3, jpg, png) в список не включены — они не содержат текстового контента для анализа.

## Актуальные источники на сегодня

| Материал | Файл | Комментарий |
|----------|------|-------------|
| **Финальный BRD для разработки** | `wiki/brd/BRD.md` | **Главный актуальный BRD.** Публикационная версия в `wiki`, использовать как основной источник для dev-ready требований. |
| **Навигация по wiki** | `wiki/navigation.md` | Описание назначения `wiki/` и правил публикации финальных артефактов. |
| **Актуальный BRD-источник перед публикацией** | `claude_results/2026-05-05 - HDV HDE BRD v13.md` | Исходный аналитический артефакт, на основе которого обновлён `wiki/brd/BRD.md`. |
| **Актуальный рабочий глоссарий** | `claude_results/Глоссарий терминов.md` | Основной текстовый источник терминов до отдельной публикации заполненного `wiki/glossary/Glossary.md`. |
| **Правила работы по репозиторию** | `.claude/rules.md` | Актуальный процесс: черновики в `Документация...`, результаты в `claude_results/`, перенос в `wiki` только по команде пользователя. |
| **Актуальный доменный контекст** | `.claude/domain.md` | Краткая сводка по текущему scope Phase 1 и актуальным ролям / цепочкам согласования. |

---

## Workspace-правила и публикация

### `.claude/`

| Файл | Описание |
|------|----------|
| `.claude/CLAUDE.md` | Корневой указатель на рабочие правила, доменный контекст и навигатор по материалам. |
| `.claude/rules.md` | **Актуальные рабочие правила.** Язык, порядок навигации по материалам, правила оформления результатов, запрет на перенос в `wiki/` без команды пользователя. |
| `.claude/domain.md` | **Актуальный доменный контекст.** Scope Phase 1, роли, цепочки согласования, интеграции, источники истины. |
| `.claude/commands/learn.md` | Шаблон для сохранения контекста сессии в `working_docs/session_memory/`. |
| `.claude/settings.json` | Локальные настройки Claude-плагинов для чтения `pdf`, `xlsx`, `docx`. |

### `wiki/`

| Файл / папка | Описание |
|--------------|----------|
| `wiki/navigation.md` | **Правила назначения `wiki/`.** В `wiki/` хранятся только финальные, согласованные, ready-for-development артефакты. |
| `wiki/brd/BRD.md` | **Актуальный опубликованный BRD.** Использовать как главную версию BRD для разработки. |
| `wiki/glossary/Glossary.md` | Файл публикации финального глоссария. На текущий момент создан как контейнер, содержательное наполнение ещё не опубликовано. |
| `wiki/requirements/` | Раздел для финальных требований, публикуемых по отдельной команде пользователя. |
| `wiki/requirements/usecases/` | Раздел для финальных use cases. |
| `wiki/api/` | Раздел для финальных API-спецификаций. |

---

## client_docs

### BRD (Бизнес-требования)

| Файл | Описание |
|------|----------|
| `client_docs/BRD/HDV_HDE_BRD_RU.md` | Исходный BRD на русском языке от заказчика. Использовать как baseline/историю изменений, но **не как самый актуальный dev-ready документ**. Актуальная версия BRD опубликована в `wiki/brd/BRD.md`. |
| `client_docs/BRD/HDV HDE Business Requirements Document.md` | BRD на английском языке. Исходная версия, на основе которой сделан RU-перевод. |
| `client_docs/BRD/HDV HDE Business Requirements Document.docx` | То же, что выше, в формате Word. |

### Project Documentation (Проектная документация)

| Файл | Описание |
|------|----------|
| `client_docs/Project documentation/FINAL Master fleet file HDV HDE.xlsx` | **Мастер-файл флота TCO.** Полный реестр единиц техники HDV/HDE: атрибуты, категории, статусы. Ключевой источник для проектирования БД и карточки оборудования. |
| `client_docs/Project documentation/HDV HDE process.drawio.svg` | **Диаграмма процесса** в формате draw.io (SVG). Визуализация текущего процесса управления техникой HDV/HDE. |
| `client_docs/Project documentation/HDV HDE process.pdf` | То же, что выше, экспортировано в PDF. |
| `client_docs/Project documentation/HDVHDE process flow 20.03.pdf` | Схема процесса (версия от 20.03). Ранняя версия process flow — может отличаться от актуальной. |
| `client_docs/Project documentation/New HDV HDE Management Procedure (5 component model).pdf` | **Процедура управления HDV HDE.** Описание 5-компонентной модели управления парком. Содержит операционные правила и процедуры. |
| `client_docs/Project documentation/IIoT Device Qualification Teltonika 250 and 650.xlsx` | Квалификация IIoT-устройств Teltonika (модели 250 и 650). Параметры трекеров, используемых на технике TCO. |
| `client_docs/Project documentation/Approved Acrhitecture diagram.jpg` | Утверждённая архитектурная диаграмма системы (изображение). |
| `client_docs/Project documentation/Approved Acrhitecture diagram_1.jpg` | Утверждённая архитектурная диаграмма системы (второй лист). |
| `client_docs/Project documentation/HDEV.jpg` | Изображение техники HDEV (для понимания предметной области). |
| `client_docs/Project documentation/HDEV_1.jpg` | Изображение техники HDEV (второй вид). |
| `client_docs/Project documentation/RE_ HDV _ E architecture diagram for device qualification.eml` | Email-переписка по архитектурной диаграмме для квалификации устройств. |
| `client_docs/Project documentation/RE_ HDV_E - Architecture Assurance.eml` | Email-переписка по архитектурным гарантиям системы HDV/HDE. |

---

## working_docs

### Встречи и вопросы

| Файл | Описание |
|------|----------|
| `working_docs/2026-03-10 Meeting - TCO New project.pdf` | PDF-материалы с первой встречи по проекту (10.03.2026). Первичное знакомство с проектом TCO. |
| `working_docs/2026-03-26 - Вопросы для заказчика.txt` | Список вопросов к заказчику (26.03.2026): утилизация, карточка техники, авторизация, доступ по IP, Service Work Request, JDE E1, уведомления, отчёты, список параметров техники. |
| `working_docs/2026-03-31 - Вопросы для заказчика.txt` | Список вопросов к заказчику (31.03.2026): ссылки на обучение, расписание daily, контакты команды, запрос встречи с пользователями (заявители + fleet owners + согласующие) для разбора AS-IS/TO-BE. |

### Встреча 19.03 — Материалы

| Файл | Описание |
|------|----------|
| `working_docs/2026-03-19 - Материалы со встречи/` | Папка с аудиозаписями встречи 19.03 (формат mp4/m4a). Текстового контента нет — только медиа. |

### Intro meeting 31.03

| Файл | Описание |
|------|----------|
| `working_docs/2026-03-31 - Intro meeting/` | Папка с фотографиями со вводной встречи (31.03.2026). Предположительно фото с whiteboard/flipchart. |

### Requirements Gathering Meeting 02.04

| Файл | Описание |
|------|----------|
| `working_docs/2026-04-02 - Requirements gathering meeting/Requirement gathering meeting transcription.txt` | **Транскрипция встречи по сбору требований (02.04).** Содержит: AS-IS процесс бронирования, роли (графисты, плановики, fleet owners), боли (конфликты бронирований, нет system of records, проблемы с утилизацией генераторов и компрессоров), внутренние vs внешние парки, cost centers, заморозка техники. **Ключевой источник AS-IS.** |
| `working_docs/2026-04-02 - Requirements gathering meeting/HDV, HDE Booking Tool - Requirements gathering questions.md` | **Вопросник для сбора требований (md-версия).** 7 блоков, 50+ вопросов: AS-IS, боли, бизнес-процессы, техника и парк, роли, внешние партнёры, внешние системы. Использовался на встрече 02.04. |
| `working_docs/2026-04-02 - Requirements gathering meeting/HDVHDE Booking Tool — Requirements Gathering Questions.doc` | То же, что выше, в формате Word. |
| `working_docs/2026-04-02 - Requirements gathering meeting/HDVHDE Booking Tool — Requirements Gathering Questions.pdf` | То же, что выше, в формате PDF. |
| `working_docs/2026-04-02 - Requirements gathering meeting/2026-04-02 - Резюме митинга по сбору требований.pdf` | **Резюме встречи 02.04** в PDF. Итоги и ключевые решения встречи по сбору требований. |
| `working_docs/2026-04-02 - Requirements gathering meeting/booking tool infographic.png` | Инфографика по booking tool (вариант 1). Визуальная схема системы. |
| `working_docs/2026-04-02 - Requirements gathering meeting/booking tool infographic 1.png` | Инфографика по booking tool (вариант 2). |
| `working_docs/2026-04-02 - Requirements gathering meeting/booking tool infographic 2.png` | Инфографика по booking tool (вариант 3). |

### Requirement Gathering — встречи 07.04

| Файл | Описание |
|------|----------|
| `working_docs/2026-04-07 - Requirement gathering/2026-04-07 13-04-36 - meeting notes - архитектура.txt` | Транскрипция встречи по GIS-интеграции с командой MAPH/Atlas (07.04, 13:04). Решения: iframe + URL параметры, синхронизация equipment ID, Azure SSO, Wialon заблокирован (cyber assessment). |
| `working_docs/2026-04-07 - Requirement gathering/2026-04-07 13-34-21 - meeting notes - кибербезопасность.txt` | Транскрипция ACR-оценки (07.04, 13:34). Результат: тир Low Risk. Нет PII. ~200 пользователей. Прайсинг не хранится. Ожидаем approve Светланы. |
| `working_docs/2026-04-07 - Requirement gathering/2026-04-07 15-04-24 - meeting notes - daily meeting.txt` | Daily standup + встреча с Balkindge (контрактная команда) (07.04, 15:04). Ключевые решения: DCT только для RoutMaint+MajMaint; только техника из контракта в BP-каталоге; Cat 3 вне контракта → Phase 2. |
| `working_docs/2026-04-07 - Requirement gathering/2026-04-07 16-01-47 - meeting notes - requirement gathering.txt` | Основная встреча по сбору требований (07.04, 16:01). Критическое изменение: broadcast-модель → targeted-выбор (Requestor видит конкретные единицы). Завершение брони: Requestor или FO. Трал Phase 1 — вручную. GDE WO — обязательное поле. |

### Requirements Gathering Meeting 06.04

| Файл | Описание |
|------|----------|
| `working_docs/2026-04-06 - Requirement gathering/equipment request process.bpmn` | BPMN v1. 5 пулов. Проанализирована 06.04 — выявлены 4 структурных ошибки, 3 логических ошибки, 11 пропущенных блоков. |
| `working_docs/2026-04-06 - Requirement gathering/equipment request process.png` | PNG-экспорт BPMN v1. |
| `working_docs/2026-04-06 - Requirement gathering/equipment request process v2.bpmn` | **BPMN v2** — обновлённая схема (добавлены End Events, P1-P4, таймеры 24h/48h, RWA, CC Owner pool). Проанализирована 06.04. Исправлено 10 пунктов из v1. Остаются: 7 критических ошибок (тип таймеров, Event-Based GW, dead-ends) + 5 важных логических замечаний. |
| `working_docs/2026-04-06 - Requirement gathering/equipment request process v2.png` | PNG-экспорт BPMN v2. |

### BPMN (актуальные схемы)

| Файл | Описание |
|------|----------|
| `working_docs/BPMN/equipment request process v3.bpmn` | **BPMN v3 — внутренний флот (Cat 1 + Cat 2).** Исправлены все структурные ошибки v2: Timer Boundary Events вместо IntermediateCatch, удалён CC Owner пул, удалён External FO пул (заменён Call Activity), реализован R-33 (причина отказа), R-34 (отказ не закрывает заявку), R-23 (24h/48h таймеры). 3 пула: Requestor / System / Internal FO. |
| `working_docs/BPMN/external fleet booking process v1.bpmn` | BPMN по внешнему флоту / BP-каталогу. Исторический аналитический материал: текущий Phase 1 scope предполагает `On-demand BP (Showcase)` как витрину без booking workflow. |
| `working_docs/BPMN/equipment request process v2.bpmn` | BPMN v2 (архив — заменена версией v3). |

### Requirements Gathering Meeting 03.04

| Файл | Описание |
|------|----------|
| `working_docs/2026-04-03 - Requirements gathering meeting/meeting notes.txt` | **Полная транскрипция встречи 03.04** (324 строки). Содержит: решение об отказе от еженедельных графиков (только on-demand бронирование); финальное определение трёх категорий техники (Assigned / Shared / Shared with Conditions); приоритеты P1–P4; временны́е ограничения (горизонт 1 мес., длительность 1 нед.); логистика доставки (трал); тайм-аут Fleet Owner (24–48 ч) с эскалацией; UX-разграничение Requestor/Fleet Owner; уточнения по JDE E1 и трекерам. **Ключевой источник для встречи 03.04.** |
| `working_docs/2026-04-03 - Requirements gathering meeting/meeting chat notes.txt` | **Чат-заметки со встречи 03.04.** Ключевые договорённости: максимальный срок бронирования должен быть настраиваемым (не hardcode), только через панель администратора; уточнение по статусам "Shared" и "Shared with Conditions". |
| `working_docs/2026-04-03 - Requirements gathering meeting/service work order request.bpmn` | **BPMN-диаграмма процесса Service Work Order Request.** Формальная схема процесса создания заявки на сервисные работы (BPMN 2.0). |
| `working_docs/2026-04-03 - Requirements gathering meeting/service work order request.png` | То же, что выше, экспортировано в PNG. |
| `working_docs/2026-04-03 - Requirements gathering meeting/image.png` | Изображение с доски / слайда со встречи 03.04 (вариант 1). |
| `working_docs/2026-04-03 - Requirements gathering meeting/image (1).png` | Изображение с доски / слайда со встречи 03.04 (вариант 2). |

### Анализ от 08.04 — Требования и архитектура

| Файл | Описание |
|------|----------|
| `working_docs/2026-04-08/Заметки.md` | **Заметки по уточнению данных.** Обновление глоссария HDV/HDE классификации; уточнение определений "Equipment usage status"; новые требования: модальное окно с графиком техники (R-47), несколько FO для одного парка (R-48); вопросы по полям Master Fleet File (Operation status, Service zone, Location, CC Code, Division, Department, Criticality и т.д.). |

---

## Быстрый навигатор по задачам

| Задача | Файл(ы) |
|--------|---------|
| Понять требования системы | `wiki/brd/BRD.md` |
| Понять правила публикации финальных артефактов | `wiki/navigation.md`, `.claude/rules.md` |
| Понять текущий scope и роли | `.claude/domain.md` |
| Понять AS-IS процесс | `2026-04-02.../Requirement gathering meeting transcription.txt` |
| Список вопросов для стейкхолдеров | `2026-04-02.../HDV, HDE Booking Tool - Requirements gathering questions.md` |
| Данные по составу флота | `Project documentation/FINAL Master fleet file HDV HDE.xlsx` |
| Схема процесса (draw.io) | `Project documentation/HDV HDE process.drawio.svg` |
| BPMN Service Work Order | `2026-04-03.../service work order request.bpmn` |
| BPMN Equipment Request v3 (внутренний флот, актуальный) | `BPMN/equipment request process v3.bpmn` |
| BPMN External Fleet v1 (исторический материал) | `BPMN/external fleet booking process v1.bpmn` |
| Анализ изменений BPMN (v2→v3 + новый внешний) | `claude_results/2026-04-07 - Изменения BPMN на основе реестра требований.md` |
| BPMN Equipment Request (TO-BE архив v2) | `2026-04-06.../equipment request process v2.bpmn` |
| Типы техники по разным категориям | `2026-04-06.../Типы техники по разным категориям.txt` — классификация по 3 измерениям: движимость / владение / share-тип |
| Вопросы для встречи 07.04 (от TCO) | `2026-04-06.../Вопросы для встречи 2026-04-07.txt` — повестка и темы для следующей встречи, составлена TCO |
| Вопросы для встречи 06.04 | `claude_results/2026-04-06 - Вопросы для встречи 06.04.md` |
| Summary встреч 06.04 | `claude_results/2026-04-06 - Summary встреч 06.04.md` |
| Реестр требований и открытых вопросов (06.04) | `claude_results/2026-04-06 - Реестр требований и открытых вопросов.md` — версия до встреч 07.04 |
| Реестр требований (07.04 вечер, историческая версия) | `claude_results/2026-04-07 - Реестр требований актуальный.md` — версия на 07.04: R-47–R-62, deprecated блок, OQ-26–OQ-30 |
| Summary встреч 07.04 | `claude_results/2026-04-07 - Summary встреч 07.04.md` — 4 встречи: архитектура/GIS, ACR, daily+контракт, requirement gathering |
| Вопросы для встречи 07.04 | `claude_results/2026-04-07 - Вопросы для встречи 07.04.md` |
| **Вопросы для встречи 08.04** | `claude_results/2026-04-07 - Вопросы для встречи 08.04.md` — фокус: внешний флот, тайм-аут FO, документы от TCO |
| **Список экранов UI v1** | `claude_results/2026-04-07 - Список экранов UI v1.md` — 12 экранов + 5 модалов для Requestor и FO; описание каждого экрана с требованиями |
| **Pencil промпт UI v1** | `claude_results/2026-04-07 - Pencil prompt UI v1.md` — английский промпт для Pencil.dev: 9 экранов + 3 модала с детальными спецификациями |
| **Use Cases базовый список** | `claude_results/2026-04-08 - Use Cases базовый список.md` — 30 UC: Requestor (UC-01–10), FO (UC-11–24), System (UC-25–27), Admin (UC-28–30). Без внешних BP. |
| **Архитектура БД базовая** | `claude_results/2026-04-08 - Архитектура БД базовая.md` — 10 коллекций MongoDB: users, fleets, equipment_types, equipment, requests, bookings, booking_extensions, notifications, audit_logs, system_settings. Поля, индексы, ERD, примечания для dev. |
| **Резюме встреч 08.04** | `claude_results/2026-04-08 - Резюме встреч 08.04.md` — daily meeting (внешний флот = витрина, Long-term rent = internal, Assigned видна всем, делегирование FO) + UI design review (фидбэк по Figma-макетам, разбор Master Fleet File) |
| **Требования к доработке дизайна** | `claude_results/2026-04-08 - Требования к доработке дизайна.md` — 33 пункта D-01–D-33 по итогам демонстрации Figma; экраны Requestor (поиск техники, форма заявки) и FO (список заявок, оборудование); матрица фильтров TBD |
| Реестр требований (08.04 v2, историческая версия) | `claude_results/2026-04-08 - Реестр требований и открытых вопросов v2.md` — версия на 08.04: BP = витрина (R-49–R-53), Assigned видна всем (R-54), делегирование FO (R-55), UI-требования (R-56–R-66), OQ-16 закрыт, новые OQ-27–OQ-29 |
| Реестр требований (08.04, историческая версия) | `claude_results/2026-04-08 - Реестр требований и открытых вопросов.md` — версия на 08.04: добавлены R-47 (график техники), R-48 (несколько FO), обновлена терминология на "техника для перемещения Unwheeled техники", добавлены определения HDV/HDE классификации и Equipment usage status, обновлены OQ-22–OQ-26, добавлен новый блок "Fleet Management" |
| **Архитектура БД — анализ Master Fleet File** | `claude_results/2026-04-08 - Анализ Master Fleet File и архитектура БД.md` — полный анализ XLSX (786 equipment, 19,075 Cost Centers, справочники); детальное описание полей (85 колонок) по 4 группам; дизайн БД с SQL (sущности, связи, индексы); процесс синхронизации; 10 открытых вопросов для TCO |
| Открытые договорённости последней встречи | `2026-04-03.../meeting chat notes.txt` |
| Процедура управления парком | `Project documentation/New HDV HDE Management Procedure (5 component model).pdf` |
| Параметры трекеров Teltonika | `Project documentation/IIoT Device Qualification Teltonika 250 and 650.xlsx` |
| Архитектурная диаграмма | `Project documentation/Approved Acrhitecture diagram.jpg` |
| **Глоссарий терминов** | `claude_results/Глоссарий терминов.md` — **актуальный рабочий глоссарий**: роли, сущности, категории техники, статусы, системы интеграции, аббревиатуры |
| **Матрица фильтров и карточка техники FO** | `claude_results/2026-04-09 - Матрица фильтров и карточка техники FO.md` — матрица «тип техники → набор фильтров» (22 типа, динамические фильтры); карточка FO: блок A (27 общих полей), блок B (12 технических спец-полей), блок C (11 контрактных); сводная таблица по типам; 5 открытых вопросов |
| **Анализ архитектуры модуля Booking** | `claude_results/2026-04-09 - Анализ архитектуры модуля Booking.md` — разбор рассуждения по архитектуре; ERD с явной сущностью `request_items`; статус-машины на двух уровнях (Request + RequestItem); рекомендации: snapshot техники, statusHistory[], conflict-check; 4 открытых вопроса |
| **Жизненный цикл заявки и броней** | `claude_results/2026-04-09 - Жизненный цикл заявки и броней.md` — аргументированный анализ против модели «1 заявка = несколько FO»; ERD, state diagrams, sequence diagram, Gantt, ASCII-wireframes UI; 3 альтернативных варианта модели с оценкой; 5 вопросов для бизнеса |
| Требования к доработке дизайна v2 (историческая версия) | `claude_results/2026-04-10 - Требования к доработке дизайна v2.md` — D-01–D-54: обновление по встречам 08.04–10.04; экраны Requestor и FO; флоу Unwheeled техники (D-34–D-36); закрытие заявки (D-37–D-39); роль «ответственный за транспортировку» (D-40–D-42); итоги встречи с дизайнером 10.04 (D-43–D-54) |
| **Поля карточки техники для дизайнера** | `claude_results/2026-04-10 - Поля карточки техники для дизайнера.md` — структура карточки по 7 ключевым типам техники (Lifting motorized, Generator portable, Air compressor, Dump truck, Frac tank, Trailer cargo, HD truck) с примерами данных; шаблон строки поиска Requestor-а; матрица «тип → ключевые параметры» |
| Архитектура БД Booking Tool (14.04, историческая версия) | `claude_results/2026-04-14 - Архитектура БД Booking Tool.md` — схема БД, ответственной только за данные о бронировании: 10 таблиц (request, booking, booking_snapshot, booking_close_record, booking_status_history, request_status_history, equipment_freeze, fo_delegation, transport_subrequest, booking_feedback); внешние ID для users/equipment/fleet; ENUM-статусы; открытые вопросы |
| **Схема БД Equipment-блок v1 (23.04)** | `claude_results/2026-04-23 - Схема БД Equipment-блок v1.md` — 16 таблиц Equipment-блока: EquipmentTypes, Fleets, FleetDelegations, Equipments (TCO+BP unified), EquipmentPhotos, EAV-блок (MeasurementUnits/Properties/PropertyEnumValues/EquipmentTypeProperties/EquipmentProperties), EquipmentTrackers, EquipmentFreezes, EquipmentRepairHistory, MaintenancePartners, EquipmentMaintenanceContracts, SystemSettings, Users; принятые решения и 3 открытых вопроса |
| **Схема БД Booking-блок v1 (24.04)** | `claude_results/2026-04-24 - Схема БД Booking-блок v1.md` — 4 таблицы Booking-блока: BookingRequests, Booking, BookingStatusHistory, RequestStatusHistory; принятые решения, связи с Equipment-блоком, 4 открытых вопроса (OQ-DB-5,8,9,10) |
| **Схема БД v2 сводная (24.04)** | `claude_results/2026-04-24 - Схема БД v2 (Equipment + Booking).md` — 20 таблиц = Equipment-блок (16) + Booking-блок (4, обновлён); новые решения: Booking→Bookings, статусы EquipmentChanged/Extended, previousSnapshot JSONB в BookingStatusHistories вместо отдельной BookingSnapshot; закрыты OQ-DB-5, OQ-DB-8, OQ-DB-10 |
| **Схема БД v3 сводная (24.04, актуальная)** | `claude_results/2026-04-24 - Схема БД v3 (Equipment + Booking).md` — **актуальная**: закрыт OQ-DB-9 (транспортировка unwheeled); Bookings расширена: поле `transportBookingId` (self-ref FK) + статус `TransportConfirmed`; previousSnapshot покрывает транспортный откат; добавлен OQ-DB-11 (скоуп транспортной брони) |
| **Резюме встречи по сбору требований 16.04** | `claude_results/2026-04-17 - Резюме встречи по сбору требований 16.04.md` — уточнение модели данных (Request→WC→Equipment→Fleet→FO); изменения карточки оборудования (заморозка, принадлежность, справочники); интеграция с JDE (выбор WO, draft заявки, поля DataLake); особенности Логистики (WO/WC необязательны); обновления дизайна; 9 action items; 2 открытых вопроса |
| **Резюме встречи 17.04** | `claude_results/2026-04-17 - Резюме встречи 17.04.md` — GIS-карта (iframe, чекбоксы, tooltip, route tracking TBD); дашборд утилизации (3 таба, градиент, ремонтные дни из JDE, целевое значение, среднее, месячный вид, фильтр принадлежности); Default WO/WC для non-JDE; история заявок для Requestor; 8 action items |
| **Требования к доработке дизайна v3** | `claude_results/2026-04-17 - Требования к доработке дизайна v3.md` — последняя оформленная версия требований к доработке дизайна: D-55–D-72, GIS-карта, дашборд утилизации, Default WO, история заявок |
| **HDV HDE BRD v6** | `claude_results/2026-04-17 - HDV HDE BRD v6.md` — BRD по итогам встречи 16.04 |
| **HDV HDE BRD v13 (актуальный)** | `claude_results/2026-05-05 - HDV HDE BRD v13.md` — **актуальная версия BRD**: новая роль FleetOwners' Supervisor (4.7), цепочка Long-term rented: Requestor (+ Justification) → FO → Confirmed by FO → Supervisor → Confirmed, новые FR-NEW-71–78, OQ-NEW-S |
| **Структура описания API** | `claude_results/2026-05-05 - Структура описания API.md` — шаблон для дальнейшего описания API-методов: карточка метода, требования, логика, permissions, настройки, ошибки, параметры, пример запроса, возвращаемые данные и пример ответа |
| **Шаблон обертки результата API** | `claude_results/2026-05-05 - Шаблон обертки результата API.md` — шаблон общего result wrapper: `value`, `isSuccess`, `errors`, структура ошибки и примеры JSON-ответов |
| **Шаблон результата пагинации API** | `claude_results/2026-05-05 - Шаблон результата пагинации API.md` — шаблон `PaginatedResult`: `items`, `total` и пример полного ответа API с пагинацией внутри общей обёртки |
| **Wiki API: GET /equipment-types** | `wiki/api/admin/equipment-types/GET - equipment-types.md` — выделенное dev-ready описание API-метода списка типов техники для Admin Panel; включает query params, логику, структуру ответа, ошибки и привязку к БД v5 |
| **Wiki BRD (опубликованная версия)** | `wiki/brd/BRD.md` — **публикационная dev-ready версия BRD** в `wiki`; содержательно соответствует актуальному `BRD v13` |
| **HDV HDE BRD v7** | `claude_results/2026-04-17 - HDV HDE BRD v7.md` — изменения из встречи 17.04 — Priority (OQ-NEW-I CLOSED, атрибут WO), Department (OQ-NEW-J CLOSED, по Equipment), Default WO/WC для non-JDE (FR-NEW-51), история заявок для Requestor (FR-NEW-52), дашборд утилизации 3 таба + градиент + ремонт из JDE (FR-NEW-53–60), GIS-карта iframe+чекбоксы+tooltip (FR-NEW-61–62), новый OQ-NEW-K (route tracking) |
| **Резюме встречи 20.04** | `claude_results/2026-04-20 - Резюме встречи 20.04.md` — демонстрация дизайна; GIS-карта (скорость/направление для JSM, топливо Phase1 нет, исторический маршрут да); дашборд утилизации (новое: количество бронирований D-NEW-1); окно согласования FO (загруженность техники по датам D-NEW-2); обязательность полей карточки для BP-техники; 9 action items; 3 открытых вопроса; новый участник Валерий (мониторинг) |
| **HDV HDE BRD v8** | `claude_results/2026-04-21 - HDV HDE BRD v8.md` — изменения из встречи 20.04 — OQ-NEW-K закрыт (маршрут 7 дней), поля BP-техники уточнены, GIS tooltip обновлён (все данные датчиков), FR-NEW-63–67 (бронирования в дашборде, загруженность техники в окне FO, скорость/направление GIS, топливо не показывать, исторический маршрут). Без emoji-маркеров — версия для согласования с руководством. Секция "Key Changes vs. Original BRD" с таблицей было/стало. |
| **Open Questions v1** | `claude_results/2026-04-21 - Open questions v1.md` — реестр всех открытых вопросов по состоянию на 21.04: OQ-29, OQ-36, OQ-37, OQ-41, OQ-NEW-A–P. Подробный контекст, impact и action items по каждому вопросу. Action items со встречи 20.04. Справочник закрытых вопросов. |
| **Требования к модулю бронирования техники** | `claude_results/2026-04-27 - Требования к модулю бронирования техники.md` — FRD модуля бронирования на основе BRD v11: сущности Request/Booking, роли, функциональные требования (создание заявок, поиск техники, согласование, жизненный цикл статусов, уведомления), бизнес-правила, интеграции JDE/PSWS/AAD, открытые вопросы. |
| **Event Storming — список событий (29.04)** | `claude_results/2026-04-29 - Event Storming - список событий.md` — 64 доменных события на основе BRD v12; блоки: Booking (Request, Booking, Unwheeled-поток, уведомления, обратная связь) и Equipment (техника, флот, интеграции). |
