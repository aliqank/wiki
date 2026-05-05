# HDV/HDE Booking Tool — Requirements Gathering Questions
# Вопросы для сбора требований

---

## 1. Current State / Текущее состояние (As-Is)

| # | Русский | English |
|---|---------|---------|
| 1 | Как сейчас осуществляется процесс бронирования техники — опишите шаги от возникновения потребности в технике до завершения бронирования. Опишите заинтересованных лиц и их зоны ответственности. | How is the equipment booking process currently carried out — describe the steps from identifying a need to completing the booking. Describe the stakeholders involved and their areas of responsibility. |
| 2 | Какие инструменты используются сейчас (Excel, email, мессенджеры, существующие системы и приложения, устные договорённости)? | What tools are currently used (Excel, email, messengers, existing systems and applications, verbal agreements)? |
| 3 | Можете поделиться примерами текущих заявок или форм, которые заполняются? С указанием обязательных и необязательных полей. | Can you share examples of current requests or forms that are filled out, indicating mandatory and optional fields? |
| 4 | Кто сейчас принимает решение о выделении техники и на каком основании? Какая должность? Как выглядит документ-основание? | Who currently makes the decision on equipment allocation and on what basis? What is their job title? What does the supporting document look like? |
| 5 | Как долго в среднем занимает процесс согласования от подачи заявки до получения техники? Если долго — каковы основные причины? | How long does the approval process take on average — from submission to receiving the equipment? If it takes long, what are the main reasons? |
| 6 | Как сейчас отслеживается статус заявки — есть ли какая-то система или всё через переписку? | How is the request status currently tracked — is there any system or is everything done through correspondence? |
| 7 | Как сейчас ведётся учёт техники — где хранится информация о наличии, статусе, параметрах и состоянии единиц? | How is equipment currently recorded — where is information about availability, status, parameters, and condition stored? |
| 8 | Требуется ли мониторинг и учёт уровня топлива? Если да — как это происходит сейчас? | Is fuel level monitoring and tracking required? If so, how is it currently handled? |

---

## 2. Pain Points / Боли и проблемы

| # | Русский | English |
|---|---------|---------|
| 1 | Какие основные проблемы вы испытываете в текущем процессе? | What are the main problems you experience in the current process? |
| 2 | Были ли случаи конфликта бронирований (двойное бронирование одной единицы)? | Have there been cases of booking conflicts (double booking of the same unit)? |
| 3 | Были ли случаи простоя техники из-за отсутствия видимости её доступности? | Have there been cases of equipment downtime due to lack of visibility of its availability? |
| 4 | Как часто возникают ситуации, когда нужная техника оказывается недоступна — и что происходит в таком случае? | How often does the required equipment turn out to be unavailable — and what happens in that case? |
| 5 | Есть ли проблемы с прозрачностью затрат — знаете ли вы, сколько тратится на аренду внешней техники? | Are there issues with cost transparency — do you know how much is spent on external equipment rental? |
| 6 | Какие ошибки или потери случались из-за отсутствия автоматизации? | What errors or losses have occurred due to the lack of automation? |
| 7 | Что происходит при поломке техники после того, как её забронировали? Есть ли процесс замены или уведомления заявителя? | What happens when equipment breaks down after it has been booked? Is there a process for replacement or notifying the requestor? |

---

## 3. Business Processes & Request Types / Бизнес-процессы и типы заявок

| # | Русский | English |
|---|---------|---------|
| 1 | Какие бизнес-процессы связаны с бронированием техники — плановые работы, аварийные выезды, регулярные маршруты? Есть ли другие сценарии использования? | What business processes are related to equipment booking — planned works, emergency call-outs, regular routes? Are there other usage scenarios? |
| 2 | Чем отличается процесс бронирования внутренней техники от внешней? Укажите: процессы и цепочки согласования, кто согласует, формы заявки, передача данных в центр оплаты. | What is the difference between the booking process for internal and external equipment? Please cover: approval processes and chains, who approves, request forms, data transfer to the payment centre. |
| 3 | Что такое "Сервисная заявка" в контексте Work Order в JDE — опишите сценарий от начала до конца. По каким критериям и где инициируется? | What is a "Service Work Request" in the context of a Work Order in JDE — describe the scenario from start to finish. By what criteria and where is it initiated? |
| 4 | Какие типы работ инициируют потребность в HDV/HDE технике? Как техника разделяется по типам работ? | What types of work trigger the need for HDV/HDE equipment? How is equipment categorised by work type? |
| 5 | Можно ли бронировать технику с определённой регулярностью (например, еженедельно)? Как это выглядит сейчас? | Is it possible to book equipment on a recurring basis (e.g., weekly)? How is this currently handled? |
| 6 | Каков минимальный и максимальный срок от момента подачи заявки до даты/времени, на которое нужна бронь? | What is the minimum and maximum lead time between submitting a request and the required booking date/time? |
| 7 | На какой максимальный период может быть оформлено бронирование? | What is the maximum duration of a single booking? |
| 8 | Можно ли бронировать одну и ту же единицу техники несколько раз подряд? Есть ли ограничения? | Is it possible to book the same equipment unit multiple times consecutively? Are there any restrictions? |
| 9 | Можно ли в одной заявке забронировать несколько единиц техники с разными датами бронирования? | Is it possible to include multiple equipment units with different booking dates within a single request? |
| 10 | Можно ли в одной заявке добавить технику из разных парков (например, внутренний и внешний флот одновременно)? | Is it possible to add equipment from different fleets (e.g., internal and external) within a single request? |
| 11 | Как должна реагировать система, если в одной заявке одна бронь подтверждена, а другая — отклонена? Заявка остаётся активной или закрывается? | How should the system respond if one booking within a request is confirmed and another is rejected? Does the request remain active or close? |
| 12 | Что является подтверждением того, что бронирование состоялось — статус в системе, уведомление, документ? | What constitutes confirmation that a booking has been completed — a system status, a notification, a document? |
| 13 | Как должна работать приоритизация заявок — планируется ли ранжирование по степени важности проекта или срочности? Кто определяет приоритет? | How should request prioritisation work — is ranking by project importance or urgency planned? Who determines the priority? |
| 14 | Есть ли сезонность или пики спроса на технику в течение года? | Is there seasonality or peaks in equipment demand throughout the year? |

---

## 4. Equipment & Fleet / Техника и парк

| # | Русский | English |
|---|---------|---------|
| 1 | Какие типы техники входят в HDV/HDE — можете дать полный перечень категорий с характеристиками? Как разделяется техника на движимую и недвижимую (стационарные установки, генераторы и т.п.)? | What types of equipment are included in HDV/HDE — can you provide a full list of categories with specifications? How is equipment divided into mobile and stationary (fixed installations, generators, etc.)? |
| 2 | Какие параметры техники важны для заявителя при выборе (грузоподъёмность, тип привода, зона обслуживания)? | What equipment parameters are important for a Requestor when selecting (load capacity, drive type, service zone)? |
| 3 | Какие параметры техники важны для владельца парка при управлении? Предоставьте полный список параметров, которые должны быть отражены в карточке техники. | What equipment parameters are important for a Fleet Owner when managing the fleet? Please provide a full list of parameters to be reflected in the equipment card. |
| 4 | Должна ли карточка техники содержать историю бронирований — и если да, то для каких ролей она должна быть доступна (только Fleet Owner или также заявитель)? | Should the equipment card include booking history — if so, which roles should have access to it (Fleet Owner only, or also the Requestor)? |
| 4 | Сколько единиц техники в общей сложности — внутренней и внешней? | How many equipment units are there in total — internal and external? |
| 5 | Как часто меняется состав парка — добавляется новая техника, списывается старая? | How often does the fleet composition change — new equipment added, old equipment written off? |
| 6 | Что означает статус "Shared with conditions" для единицы техники — какие это могут быть условия? | What does the "Shared with conditions" status mean for an equipment unit — what conditions might apply? |
| 7 | Нужно ли учитывать водительский/операторский состав при бронировании техники? Привязывается ли водитель к конкретной единице техники в заявке? | Is it necessary to account for drivers or operators when booking equipment? Is a driver assigned to a specific unit within a request? |

---

## 5. Roles & Organisation / Роли и организация

| # | Русский | English |
|---|---------|---------|
| 1 | Кто в организации будет пользоваться системой — подразделения, роли, примерное количество людей? | Who in the organisation will use the system — departments, roles, approximate number of people? |
| 2 | Кто такой владелец парка на практике — это один человек или группа, есть ли заместители? | Who is a Fleet Owner in practice — is it one person or a group, are there deputies? |
| 3 | Как устроена структура Cost Center'ов — сколько их, кто является владельцем? | How is the Cost Center structure organised — how many are there, who is the owner? |
| 4 | Кто такой DOA на практике — это официальное делегирование или ситуативное? Как оформляется и хранится информация о делегировании? | Who is a DOA in practice — is this a formal delegation or situational? How is delegation information formalised and stored? |
| 5 | Кто будет Администратором системы со стороны бизнеса — Логистика? | Who will be the System Administrator from the business side — Logistics? |

---

## 6. External Partners / Внешние партнёры (Business Partners)

| # | Русский | English |
|---|---------|---------|
| 1 | Сколько внешних БП предоставляют технику — это 2–3 партнёра или десятки? Каков средний размер парка у одного БП? | How many external BPs provide equipment — is it 2–3 partners or dozens? What is the average fleet size per BP? |
| 2 | Как сейчас оформляются отношения с БП при аренде техники — договор, RWA, другое? Каковы типичные сроки действия договора с БП? | How are relations with BPs currently formalised when renting equipment — contract, RWA, other? What are the typical contract durations with BPs? |
| 3 | Что такое RWA — можете описать процесс его создания и назначение? Как выглядит документ? | What is RWA — can you describe the process of its creation and purpose? What does the document look like? |
| 4 | Есть ли у БП собственные системы учёта техники, с которыми потребуется взаимодействие? | Do BPs have their own equipment tracking systems that will need to be interacted with? |
| 5 | Оснащена ли техника БП трекерами — если да, то какими и чьими? Кто несёт ответственность за передачу данных трекеров? | Is BP equipment equipped with trackers — if so, what type and whose? Who is responsible for transmitting tracker data? |

---

## 7. External Systems & Data / Внешние системы и данные

| # | Русский | English |
|---|---------|---------|
| 1 | Какие внешние системы сейчас используются в смежных процессах (JDE E1, DCT, PSWS, DOA, трекеры)? Есть ли другие системы, не упомянутые в BRD? | What external systems are currently used in related processes (JDE E1, DCT, PSWS, DOA, trackers)? Are there other systems not mentioned in the BRD? |
| 2 | Для каждой из этих систем — какие данные из неё нужны и какие данные туда передаются? Укажите форматы и примеры входящих/исходящих данных. | For each of these systems — what data is needed from it and what data is sent back? Please provide formats and examples of incoming/outgoing data. |
| 3 | Есть ли уже API у этих систем или интеграция будет разрабатываться с нуля? Есть ли документация на API? | Do these systems already have APIs or will the integration be developed from scratch? Is API documentation available? |
| 4 | В каком формате предоставляются данные из трекеров (WIALON, IVMS, Vega) — Excel-выгрузки, API, Datalake? | In what format is data provided from trackers (WIALON, IVMS, Vega) — Excel exports, API, Datalake? |
| 5 | Есть ли ответственные со стороны владельцев внешних систем, с кем можно уточнять детали интеграции? | Are there responsible contacts on the side of these system owners who can clarify integration details? |
| 6 | Каковы ограничения или SLA внешних систем — доступность, частота обновления данных, лимиты API-запросов? | What are the limitations or SLAs of external systems — availability, data update frequency, API request limits? |

---

## 8. Notifications & Communication / Уведомления и коммуникации

| # | Русский | English |
|---|---------|---------|
| 1 | Через какие каналы должны приходить уведомления — email, Microsoft Teams, внутренний портал, SMS? | Through what channels should notifications be sent — email, Microsoft Teams, internal portal, SMS? |
| 2 | Кто и о каких событиях должен получать уведомления (подача заявки, согласование, отклонение, завершение бронирования)? | Who should be notified and about which events (request submission, approval, rejection, booking completion)? |
| 3 | Должна ли быть возможность настройки уведомлений пользователем — включение/отключение по типу события? | Should users be able to configure their notification preferences — enabling/disabling by event type? |
| 4 | Нужны ли напоминания при отсутствии реакции согласующего — и если да, через какое время и кому эскалировать? | Are reminders needed when an approver has not responded — if so, after how long and to whom should escalation go? |

---

## 9. Goals & Expected Outcome / Цели и ожидаемый результат

| # | Русский | English |
|---|---------|---------|
| 1 | Какой главный результат вы хотите получить от внедрения системы — что изменится через год после запуска? | What is the main result you want to achieve from implementing the system — what will change one year after launch? |
| 2 | По каким метрикам вы будете оценивать успех системы (время согласования, утилизация техники, затраты)? Какие показатели наиболее важны? | What metrics will you use to evaluate the success of the system (approval time, equipment utilisation, costs)? Which indicators are most important? |
| 3 | Есть ли конкретные целевые показатели — например, сократить время согласования с X дней до Y? | Are there specific target indicators — for example, reduce approval time from X days to Y? |
| 4 | Что для вас является абсолютным приоритетом в системе, а без чего можно обойтись на первом этапе? | What is an absolute priority for you in the system, and what can be deferred to a later phase? |
| 5 | Есть ли примеры похожих систем — внутри компании или у других организаций — которые нравятся как эталон? | Are there examples of similar systems — within the company or at other organisations — that you consider a benchmark? |
| 6 | Как планируется работа с требованиями после согласования документа — как будут уточняться и улучшаться требования к системе в ходе разработки? | How is it planned to work with requirements after the document is approved — how will requirements be refined and improved during development? |

---

## 10. Dashboards & Reporting / Дашборды и отчётность

| # | Русский | English |
|---|---------|---------|
| 1 | Что понимается под "утилизацией" техники в контексте системы — это процент времени использования от доступного, пробег, моточасы, или что-то другое? Как утилизация рассчитывается сейчас? | What is meant by equipment "utilisation" in the context of the system — is it percentage of time in use, mileage, engine hours, or something else? How is utilisation currently calculated? |
| 2 | Какие отчёты необходимы в системе? Опишите каждый: что отображается, за какой период, для какой роли, в каком формате (таблица, график, выгрузка)? | What reports are required in the system? Describe each one: what is displayed, for what period, for which role, and in what format (table, chart, export)? |
| 3 | Какие дашборды необходимы — например, карта местоположения техники, график утилизации, сводка по заявкам? Для каких ролей? | What dashboards are required — for example, equipment location map, utilisation chart, request summary? For which roles? |
| 4 | Нужна ли возможность выгрузки данных из отчётов (Excel, CSV, PDF)? | Is the ability to export report data required (Excel, CSV, PDF)? |
| 5 | За какой период должны храниться данные для отчётности и дашбордов? | For what period should data be retained for reporting and dashboards? |

---

## 11. Security & Access / Безопасность и доступ

| # | Русский | English |
|---|---------|---------|
| 1 | Какие требования к авторизации в системе — логин/пароль, SSO, многофакторная аутентификация (SMS, приложение)? Есть ли интеграция с корпоративным провайдером идентификации (Azure AD, Teams)? | What are the authentication requirements — login/password, SSO, multi-factor authentication (SMS, app)? Is there integration with a corporate identity provider (Azure AD, Teams)? |
| 2 | Должен ли доступ к системе ограничиваться по местоположению (IP-адрес, корпоративная сеть, VPN)? Возможен ли доступ извне корпоративной сети — например, для внешних БП? | Should access to the system be restricted by location (IP address, corporate network, VPN)? Should external access be possible — for example, for external BPs? |
| 3 | Какие требования к логированию действий пользователей — что именно должно фиксироваться и как долго хранится лог? | What are the requirements for user action logging — what exactly should be recorded and how long should logs be retained? |

---

## 12. UX & Interface / UX и интерфейс

| # | Русский | English |
|---|---------|---------|
| 1 | На каких языках должен быть интерфейс системы — русский, английский, казахский? Все три обязательны или часть опциональна? Какой язык является основным? | In what languages should the system interface be available — Russian, English, Kazakh? Are all three mandatory or is some optional? Which is the primary language? |
| 2 | Нужна ли мобильная версия системы или достаточно адаптивного веб-интерфейса? Есть ли пользователи, работающие преимущественно с телефона (например, полевые операторы, водители)? | Is a mobile version of the system required, or is a responsive web interface sufficient? Are there users who work primarily from a phone (e.g., field operators, drivers)? |
| 3 | Есть ли корпоративный UI-гайд, дизайн-система или брендбук, которым должна соответствовать система? | Is there a corporate UI guide, design system, or brand book that the system must comply with? |
| 4 | Какие пользователи наименее технически подготовлены? Как они сейчас работают с цифровыми инструментами — есть ли у них опыт работы с корпоративными системами? | Which users are the least technically proficient? How do they currently work with digital tools — do they have experience with corporate systems? |
| 5 | Есть ли требования к доступности интерфейса (например, для людей с ограниченными возможностями)? | Are there any accessibility requirements for the interface (e.g., for users with disabilities)? |

---

## 13. MVP & Delivery Phases / MVP и фазы поставки

| # | Русский | English |
|---|---------|---------|
| 1 | Согласно Software Delivery Timeline, фаза сбора требований завершается 17 апреля, а разработка управления внутренним флотом стартует уже 30 марта параллельно. Все ли требования по внутреннему флоту зафиксированы и согласованы для старта разработки? | According to the Software Delivery Timeline, requirements gathering ends on 17 April, while internal fleet management development starts on 30 March in parallel. Have all internal fleet requirements been captured and agreed upon for the development start? |
| 2 | Фаза "Booking and allocation for the internal fleet" (17-Apr → 29-May) идёт раньше внешнего флота. Какой минимальный функционал бронирования должен быть готов к 29 мая — что является критерием завершения этой фазы? | The "Booking and allocation for the internal fleet" phase (17-Apr → 29-May) precedes the external fleet. What is the minimum booking functionality required by 29 May — what are the completion criteria for this phase? |
| 3 | "Fleet maintenance & inspections" выделена в отдельную фазу (22-Jun → 03-Aug). Что входит в этот модуль — регламентное ТО, внеплановые ремонты, инспекции безопасности? Есть ли отдельные требования, которые ещё не зафиксированы в BRD? | "Fleet maintenance & inspections" is a separate phase (22-Jun → 03-Aug). What does this module include — scheduled maintenance, unplanned repairs, safety inspections? Are there separate requirements not yet captured in the BRD? |
| 4 | Дашборды и аналитика запланированы на последнюю неделю перед деплоем (27-Jul → 03-Aug). Это финальный инструмент или промежуточный MVP? Каков ожидаемый объём функционала на старте? | Dashboards and analytics are scheduled for the last week before deployment (27-Jul → 03-Aug). Is this the final tool or an intermediate MVP? What is the expected scope of functionality at launch? |
| 5 | Планируется ли пилотный запуск на ограниченной группе пользователей перед полным развёртыванием — и если да, кто войдёт в пилотную группу и по каким критериям? | Is a pilot launch planned with a limited user group before full deployment — if so, who will be included and on what criteria? |
| 6 | Каков критерий готовности системы к переходу в Prod — кто подписывает приёмку и по каким показателям? | What is the system readiness criterion for transition to Prod — who signs off acceptance and based on what indicators? |

---

## 14. Data Migration / Миграция данных

| # | Русский | English |
|---|---------|---------|
| 1 | Есть ли данные, которые необходимо перенести в новую систему до запуска — реестры техники, исторические заявки, данные о парках и флотах? | Is there data that needs to be migrated to the new system before launch — equipment registers, historical requests, fleet and park data? |
| 2 | В каком формате хранятся текущие данные — Excel, база данных, бумажные журналы, другие системы? | In what format is the current data stored — Excel, database, paper records, other systems? |
| 3 | Насколько актуальны и полны текущие данные — есть ли дубли, пропуски, устаревшие записи? Кто отвечает за качество данных при переносе? | How accurate and complete is the current data — are there duplicates, gaps, or outdated records? Who is responsible for data quality during migration? |
| 4 | Кто со стороны бизнеса будет отвечать за подготовку, проверку и загрузку мастер-данных (справочник техники, флоты, пользователи)? | Who on the business side will be responsible for preparing, validating, and loading master data (equipment catalogue, fleets, users)? |

---

## 15. Compliance & Regulatory / Комплаенс и регуляторика

| # | Русский | English |
|---|---------|---------|
| 1 | Есть ли требования к хранению данных со стороны регулятора или внутренней политики компании — например, данные не должны покидать определённый регион или страну? | Are there data storage requirements from regulators or internal company policy — for example, data must not leave a specific region or country? |
| 2 | Нужна ли электронная подпись (ЭЦП) для подтверждения заявок, или достаточно нажатия кнопки в системе? | Is a digital signature (EDS) required for approving requests, or is a button click within the system sufficient? |
| 3 | Есть ли требования к архивированию закрытых заявок и бронирований — на какой срок, в каком формате, с возможностью поиска? | Are there requirements for archiving closed requests and bookings — for how long, in what format, with search capability? |
| 4 | Распространяются ли на систему требования корпоративной информационной безопасности (IS политики ТШО)? Проводился ли IS-assessment для аналогичных систем? | Do corporate information security requirements (TCO IS policies) apply to the system? Has an IS assessment been conducted for similar systems? |
| 5 | Требуется ли согласование системы с юридическим или compliance-департаментом перед запуском? | Is approval from the legal or compliance department required before launch? |

---

## 16. Exception Scenarios / Сценарии исключений и граничные случаи

| # | Русский | English |
|---|---------|---------|
| 1 | Что должна делать система при недоступности внешней системы (JDE E1, PSWS, DOA) — ставить операцию в очередь, блокировать действие, уведомлять администратора? | What should the system do if an external system (JDE E1, PSWS, DOA) is unavailable — queue the operation, block the action, notify the administrator? |
| 2 | Что происходит с очередью согласования, если Fleet Owner или CC Owner уволился, заболел или длительно недоступен? Кто получает его задачи? | What happens to the approval queue if a Fleet Owner or CC Owner resigns, falls ill, or is unavailable for a long time? Who takes over their tasks? |
| 3 | Что происходит с активными бронированиями, если единица техники переводится в другой флот или проходит процедуру списания? | What happens to active bookings if an equipment unit is transferred to another fleet or goes through a write-off procedure? |
| 4 | Возможна ли экстренная отмена уже начавшегося бронирования (статус In Progress)? Кто имеет право это сделать и требуется ли обоснование? | Is emergency cancellation of an already started booking (In Progress status) possible? Who has the authority to do so, and is justification required? |
| 5 | Как система должна обрабатывать конфликт, если два заявителя одновременно пытаются забронировать одну и ту же единицу на одно время? | How should the system handle a conflict if two requestors simultaneously try to book the same unit for the same time? |
| 6 | Что происходит, если внешний БП не отреагировал на подтверждённую CC Owner заявку в течение N дней — есть ли автоматическая эскалация или отмена? | What happens if an external BP does not respond to a CC Owner-approved request within N days — is there automatic escalation or cancellation? |

---

## 17. Performance & Maintenance / Производительность и сопровождение

| # | Русский | English |
|---|---------|---------|
| 1 | Сколько пользователей могут работать в системе одновременно — каков ожидаемый пик? | How many users can work in the system simultaneously — what is the expected peak load? |
| 2 | Каков допустимый SLA на отклик системы — максимальное время загрузки страницы, обработки заявки? | What is the acceptable SLA for system response time — maximum page load time, request processing time? |
| 3 | Кто будет осуществлять техническую поддержку системы после запуска — внутренняя команда или подрядчик? Как будет организован service desk? | Who will provide technical support after launch — an internal team or a contractor? How will the service desk be organised? |
| 4 | Каков процесс внесения изменений в систему после запуска — кто инициирует, кто согласует, какой цикл выпуска обновлений? | What is the change management process after launch — who initiates, who approves, what is the release cycle? |