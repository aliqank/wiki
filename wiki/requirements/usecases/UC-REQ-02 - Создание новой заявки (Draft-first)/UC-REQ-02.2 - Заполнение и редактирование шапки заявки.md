# UC-REQ-02.2 - Заполнение и редактирование шапки заявки

**Created:** 2026-05-15  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка use case

| Поле | Значение |
|---|---|
| Область действия | Модальное окно `Новая заявка` |
| Участник | `Requestor`, `ServiceWorkProcessor` |
| Покрываемые FR (BRD) | `FR-027`, `FR-NEW-11`, `FR-NEW-12`, `FR-NEW-13`, `FR-NEW-14`, `FR-NEW-48`, `FR-NEW-51` |
| Покрываемые FR (Equipment block list) | — |
| Триггер | Пользователь редактирует request-level поля draft-заявки |
| Ожидаемый результат | Шапка draft-заявки заполнена и сохранена через autosave |
| Используемые API | `PATCH /booking-requests/{id}`, `GET /jde/work-orders`, `GET /jde/work-orders/{id}/steps` |

---

## Основной сценарий

1. Пользователь редактирует request-level поля: `workOrderNumber`, `priority`, `location`, `workDescription`, `comments`.
2. Если нужен выбор WO из JDE, frontend вызывает `GET /jde/work-orders` и при необходимости `GET /jde/work-orders/{id}/steps`.
3. Если пользователь включает `Default Work Order`, frontend передаёт `isDefaultWorkOrder = true`, а `workOrderNumber = null`.
4. Frontend сохраняет изменения через debounced autosave: `PATCH /booking-requests/{id}`.
5. Backend обновляет header draft-заявки.

---

## Альтернативные сценарии

1. Пользователь не завершил заполнение.
   Draft остаётся частично заполненным.

2. Пользователь выключил `Default Work Order`.
   Поле `workOrderNumber` снова становится доступным для редактирования.
