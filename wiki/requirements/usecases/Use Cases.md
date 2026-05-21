# Use Cases

**Created:** 2026-05-15  
**Last updated:** 2026-05-21  
**Автор документов:** Telman Nurzhanov (SA)

---

Раздел содержит финальные use case-сценарии, подготовленные для разработки и детализации UI/API-потока.

Связанный документ с общими правилами статусов:

- `../Aggregated Request Status Rules.md`

## Список use cases

| ID | Наименование | Область действия | Покрываемые FR | Основные API |
|---|---|---|---|---|
| `UC-REQ-01` | Просмотр моих заявок | Страница `Мои заявки` | `BRD: FR-025, FR-091` | `GET /booking-requests/my`, `GET /booking-requests/{id}` |
| `UC-REQ-02` | Создание новой заявки (Draft-first) | Модальное окно `Новая заявка` | `BRD: FR-023, FR-027, FR-030, FR-031, FR-038, FR-040, FR-NEW-04, FR-NEW-08, FR-NEW-11, FR-NEW-12, FR-NEW-13, FR-NEW-14, FR-NEW-15, FR-NEW-16, FR-NEW-17, FR-NEW-38, FR-NEW-39, FR-NEW-48, FR-NEW-50, FR-NEW-51, FR-NEW-68, FR-NEW-71` | `POST /booking-requests`, `PATCH /booking-requests/{id}`, `GET /equipment/search`, `POST /booking-requests/{id}/items`, `POST /booking-requests/{id}/submit` |
| `UC-REQ-02.1` | Создание пустого draft заявки | Кнопка `Создать заявку` | `BRD: FR-023, FR-027, FR-030` | `POST /booking-requests` |
| `UC-REQ-02.2` | Заполнение и редактирование шапки заявки | Модальное окно `Новая заявка` | `BRD: FR-027, FR-NEW-11, FR-NEW-12, FR-NEW-13, FR-NEW-14, FR-NEW-48, FR-NEW-51` | `PATCH /booking-requests/{id}` |
| `UC-REQ-02.3` | Поиск техники для добавления в заявку | Окно `Добавить технику` | `BRD: FR-031, FR-040, FR-NEW-38, FR-NEW-39, FR-NEW-50, FR-NEW-68` | `GET /equipment/search`, reference APIs |
| `UC-REQ-02.4` | Добавление техники в draft-заявку | Окно `Добавить технику` | `BRD: FR-031, FR-038, FR-NEW-04, FR-NEW-08, FR-NEW-15, FR-NEW-71` | `POST /booking-requests/{id}/items` |
| `UC-REQ-02.5` | Редактирование брони или замена техники в draft | Карточка booking item в draft | `BRD: FR-027, FR-031, FR-038, FR-040, FR-NEW-04, FR-NEW-08, FR-NEW-71` | `PATCH /booking-requests/{id}/items/{bookingId}`, `GET /equipment/search` |
| `UC-REQ-02.6` | Отправка draft-заявки | Модальное окно `Новая заявка` | `BRD: FR-023, FR-027, FR-NEW-08, FR-NEW-16, FR-NEW-17, FR-NEW-71` | `POST /booking-requests/{id}/submit` |
| `UC-REQ-02.7` | Удаление draft-брони | Карточка booking item в draft | `BRD: FR-027, FR-031, FR-038` | `DELETE /booking-requests/{id}/items/{bookingId}`, `GET /booking-requests/{id}` |
| `UC-REQ-03` | Отмена draft-заявки requestor-ом | Страница `Мои заявки` / detail view draft-заявки | `BRD: FR-025, FR-027` | `POST /booking-requests/{id}/cancel`, `GET /booking-requests/my`, `GET /booking-requests/{id}` |
| `UC-REQ-04` | Отзыв брони requestor-ом | Страница `Мои заявки` / карточка заявки / карточка booking item в отправленной заявке | `BRD: FR-025, FR-042, FR-058, FR-068, FR-078, FR-NEW-17` | `GET /booking-requests/my`, `POST /bookings/{id}/revoke`, `GET /booking-requests/{id}` |
| `UC-REQ-05` | Отзыв submitted-заявки requestor-ом | Страница `Мои заявки` / карточка submitted-заявки | `BRD: FR-025, FR-058, FR-NEW-17; Additional: BRD-U-003` | `GET /booking-requests/my`, `GET /booking-requests/{id}`, request-level withdraw API / orchestration *(TBD)* |
| `UC-FO-01` | Просмотр списка броней Fleet Owner | Страница `Approvals` → view `Bookings` | `BRD: FR-092, FR-043` | `GET /approvals/bookings`, `GET /approvals/bookings/{id}` |
| `UC-FO-02` | Подтверждение брони Fleet Owner (базовый сценарий) | Страница `Approvals` → detail / action view брони | `BRD: FR-043, FR-045, FR-063` | `GET /approvals/bookings/{id}`, `POST /approvals/bookings/{id}/confirm` |
| `UC-FO-03` | Просмотр списка заявок Fleet Owner | Страница `Approvals` → view `Requests` | `BRD: FR-092, FR-094, FR-043; Additional: BRD-U-001` | `GET /approvals/requests` |

---

## Файлы

- `Requestor/UC-REQ-01 - Просмотр моих заявок.md`
- `Requestor/UC-REQ-02 - Создание новой заявки (Draft-first)/UC-REQ-02 - Создание новой заявки (Draft-first).md`
- `Requestor/UC-REQ-02 - Создание новой заявки (Draft-first)/UC-REQ-02.1 - Создание пустого draft заявки.md`
- `Requestor/UC-REQ-02 - Создание новой заявки (Draft-first)/UC-REQ-02.2 - Заполнение и редактирование шапки заявки.md`
- `Requestor/UC-REQ-02 - Создание новой заявки (Draft-first)/UC-REQ-02.3 - Поиск техники для добавления в заявку.md`
- `Requestor/UC-REQ-02 - Создание новой заявки (Draft-first)/UC-REQ-02.4 - Добавление техники в draft-заявку.md`
- `Requestor/UC-REQ-02 - Создание новой заявки (Draft-first)/UC-REQ-02.5 - Редактирование брони или замена техники в draft.md`
- `Requestor/UC-REQ-02 - Создание новой заявки (Draft-first)/UC-REQ-02.6 - Отправка draft-заявки.md`
- `Requestor/UC-REQ-02 - Создание новой заявки (Draft-first)/UC-REQ-02.7 - Удаление draft-брони.md`
- `Requestor/UC-REQ-03 - Отмена draft-заявки requestor-ом.md`
- `Requestor/UC-REQ-04 - Отзыв брони requestor-ом.md`
- `Requestor/UC-REQ-05 - Отзыв submitted-заявки requestor-ом.md`
- `Fleet Owner/UC-FO-01 - Просмотр списка броней Fleet Owner.md`
- `Fleet Owner/UC-FO-02 - Подтверждение брони Fleet Owner (базовый сценарий).md`
- `Fleet Owner/UC-FO-03 - Просмотр списка заявок Fleet Owner.md`
