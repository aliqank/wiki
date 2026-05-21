# UC-REQ-02.5 - Редактирование брони или замена техники в draft

**Created:** 2026-05-15  
**Last updated:** 2026-05-19  
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
2. Если требуется изменить только период брони, `justification` или `workCenterId`, frontend вызывает `PATCH /booking-requests/{id}/items/{bookingId}`.
3. Если пользователь редактирует inline-поле `justification`, frontend сохраняет его автоматически через `PATCH /booking-requests/{id}/items/{bookingId}` без отдельной кнопки `Сохранить`.
4. Если требуется заменить технику, frontend повторно открывает окно выбора техники.
5. Frontend повторно вызывает `GET /equipment/search` с актуальным периодом и фильтрами.
6. Пользователь выбирает новую технику.
7. Frontend вызывает `PATCH /booking-requests/{id}/items/{bookingId}` и передаёт обновлённые `equipmentId`, `plannedStartDateTime`, `plannedEndDateTime`, `justification`, `workCenterId`.
8. Backend обновляет существующий booking item и пересчитывает, требуется ли для него `justification`.

---

## Альтернативные сценарии

1. При замене техники новая единица недоступна на выбранный период.
   Backend отклоняет обновление item ошибкой `EQUIPMENT_NOT_AVAILABLE`.

2. Пользователь меняет период брони.
   Backend повторно валидирует доступность техники в новом диапазоне по итоговым значениям item.

3. Для updated item требуется `justification`, но оно не заполнено.
   Item остаётся незавершённым до исправления.

4. Пользователь ввёл `justification`, но сразу не отправил заявку.
   Значение сохраняется автоматически через `PATCH /booking-requests/{id}/items/{bookingId}` и остаётся в draft.
