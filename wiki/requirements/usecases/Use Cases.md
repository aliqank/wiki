# Use Cases

**Created:** 2026-05-15  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

Раздел содержит финальные use case-сценарии, подготовленные для разработки и детализации UI/API-потока.

## Список use cases

| ID | Наименование | Область действия | Покрываемые FR | Основные API |
|---|---|---|---|---|
| `UC-REQ-01` | Просмотр моих заявок | Страница `Мои заявки` | `BRD: FR-025, FR-091` | `GET /booking-requests/my`, `GET /booking-requests/{id}` |
| `UC-REQ-02` | Создание новой заявки (Draft-first) | Модальное окно `Новая заявка` | `BRD: FR-023, FR-027, FR-030, FR-031, FR-038, FR-040, FR-NEW-04, FR-NEW-08, FR-NEW-11, FR-NEW-12, FR-NEW-13, FR-NEW-14, FR-NEW-15, FR-NEW-16, FR-NEW-17, FR-NEW-38, FR-NEW-39, FR-NEW-48, FR-NEW-50, FR-NEW-51, FR-NEW-68, FR-NEW-71` | `POST /booking-requests`, `PATCH /booking-requests/{id}`, `GET /equipment/search`, `POST /booking-requests/{id}/items`, `POST /booking-requests/{id}/submit` |
| `UC-REQ-02.1` | Создание пустого draft заявки | Кнопка `Добавить заявку` | `BRD: FR-023, FR-027, FR-030` | `POST /booking-requests` |
| `UC-REQ-02.2` | Заполнение и редактирование шапки заявки | Модальное окно `Новая заявка` | `BRD: FR-027, FR-NEW-11, FR-NEW-12, FR-NEW-13, FR-NEW-14, FR-NEW-48, FR-NEW-51` | `PATCH /booking-requests/{id}`, `GET /jde/work-orders`, `GET /jde/work-orders/{id}/steps` |
| `UC-REQ-02.3` | Поиск техники для добавления в заявку | Модалка `Добавить технику` | `BRD: FR-031, FR-040, FR-NEW-38, FR-NEW-39, FR-NEW-50, FR-NEW-68` | `GET /equipment/search`, reference APIs |
| `UC-REQ-02.4` | Добавление техники в draft-заявку | Модалка `Добавить технику` | `BRD: FR-031, FR-038, FR-NEW-04, FR-NEW-08, FR-NEW-15, FR-NEW-71` | `POST /booking-requests/{id}/items` |
| `UC-REQ-02.5` | Редактирование брони или замена техники в draft | Карточка booking item в draft | `BRD: FR-027, FR-031, FR-038, FR-040, FR-NEW-04, FR-NEW-08, FR-NEW-71` | `PATCH /booking-requests/{id}/items/{bookingId}`, `GET /equipment/search` |
| `UC-REQ-02.6` | Отправка draft-заявки | Модальное окно `Новая заявка` | `BRD: FR-023, FR-027, FR-NEW-08, FR-NEW-16, FR-NEW-17, FR-NEW-71` | `POST /booking-requests/{id}/submit` |

---

## Файлы

- `UC-REQ-01 - Просмотр моих заявок.md`
- `UC-REQ-02 - Создание новой заявки (Draft-first)/UC-REQ-02 - Создание новой заявки (Draft-first).md`
- `UC-REQ-02 - Создание новой заявки (Draft-first)/UC-REQ-02.1 - Создание пустого draft заявки.md`
- `UC-REQ-02 - Создание новой заявки (Draft-first)/UC-REQ-02.2 - Заполнение и редактирование шапки заявки.md`
- `UC-REQ-02 - Создание новой заявки (Draft-first)/UC-REQ-02.3 - Поиск техники для добавления в заявку.md`
- `UC-REQ-02 - Создание новой заявки (Draft-first)/UC-REQ-02.4 - Добавление техники в draft-заявку.md`
- `UC-REQ-02 - Создание новой заявки (Draft-first)/UC-REQ-02.5 - Редактирование брони или замена техники в draft.md`
- `UC-REQ-02 - Создание новой заявки (Draft-first)/UC-REQ-02.6 - Отправка draft-заявки.md`
