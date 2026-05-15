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
| `UC-REQ-02` | Создание новой заявки (Draft-first) | Модальное окно `Новая заявка` | `BRD: FR-023, FR-027, FR-030, FR-031, FR-038, FR-040, FR-NEW-11, FR-NEW-12, FR-NEW-13, FR-NEW-14, FR-NEW-15, FR-NEW-16, FR-NEW-17, FR-NEW-38, FR-NEW-39, FR-NEW-48, FR-NEW-51, FR-NEW-68, FR-NEW-71; Equipment block: FR-010, FR-NEW-04, FR-NEW-08, FR-NEW-50, AFR-01, AFR-02, AFR-03, AFR-07` | `POST /booking-requests`, `PATCH /booking-requests/{id}`, `GET /equipment/search`, `POST /booking-requests/{id}/items`, `POST /booking-requests/{id}/submit` |

---

## Файлы

- `UC-REQ-01 - Просмотр моих заявок.md`
- `UC-REQ-02 - Создание новой заявки (Draft-first).md`
