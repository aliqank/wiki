# UC-REQ-02.5 - Редактирование брони или замена техники в draft

**Created:** 2026-05-15  
**Last updated:** 2026-05-15  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка use case

| Поле | Значение |
|---|---|
| Область действия | Карточка booking item в draft-заявке |
| Участник | `Requestor`, `ServiceWorkProcessor` |
| Покрываемые FR (BRD) | `FR-027`, `FR-031`, `FR-038`, `FR-040`, `FR-NEW-04`, `FR-NEW-08`, `FR-NEW-71` |
| Покрываемые FR (Additional list) | — |
| Триггер | Пользователь редактирует период брони, justification или хочет заменить технику |
| Ожидаемый результат | Booking item обновлён без удаления всей draft-заявки |
| Используемые API | `PATCH /booking-requests/{id}/items/{bookingId}`, `GET /equipment/search` |

---

## Основной сценарий

1. Пользователь открывает booking item на редактирование.
2. Если требуется изменить только период брони или `justification`, frontend вызывает `PATCH /booking-requests/{id}/items/{bookingId}`.
3. Если требуется заменить технику, frontend повторно открывает модалку выбора техники.
4. Frontend повторно вызывает `GET /equipment/search` с актуальным периодом и фильтрами.
5. Пользователь выбирает новую технику.
6. Frontend вызывает `PATCH /booking-requests/{id}/items/{bookingId}` и передаёт обновлённые `equipmentId`, `startDt`, `endDt`, `justification`.
7. Backend обновляет существующий booking item.

---

## Альтернативные сценарии

1. При замене техники новая единица недоступна на выбранный период.
   Backend отклоняет обновление item.

2. Пользователь меняет период брони.
   Backend повторно валидирует доступность техники в новом диапазоне.

3. Для updated item требуется `justification`, но оно не заполнено.
   Item остаётся незавершённым до исправления.
