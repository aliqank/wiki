# Use Cases

**Created:** 2026-05-15  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

Раздел содержит финальные use case-сценарии, подготовленные для разработки и детализации UI/API-потока.

## Список use cases

| ID | Наименование | Область действия | Основные API |
|---|---|---|---|
| `UC-REQ-01` | Просмотр моих заявок | Страница `Мои заявки` | `GET /booking-requests/my`, `GET /booking-requests/{id}` |
| `UC-REQ-02` | Создание новой заявки (Draft-first) | Модальное окно `Новая заявка` | `POST /booking-requests`, `PATCH /booking-requests/{id}`, `GET /equipment/search`, `POST /booking-requests/{id}/items`, `POST /booking-requests/{id}/submit` |

---

## Файлы

- `UC-REQ-01 - Просмотр моих заявок.md`
- `UC-REQ-02 - Создание новой заявки (Draft-first).md`
