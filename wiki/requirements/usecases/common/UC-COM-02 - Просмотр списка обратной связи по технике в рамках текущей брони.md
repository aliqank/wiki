# UC-COM-02 - Просмотр списка обратной связи по технике в рамках текущей брони

**Created:** 2026-06-09  
**Last updated:** 2026-06-09  
**Автор документов:** OpenCode

---

## Карточка use case

| Поле | Значение |
|---|---|
| Область действия | Страница `Мои заявки`, detail view заявки, detail view брони, [`Fleet Owner`](../../Roles%20and%20Access%20Model.md) `Approvals` detail / action view |
| Участник | Пользователь с ролью [`Requestor`](../../Roles%20and%20Access%20Model.md), [`ServiceWorkProcessor`](../../Roles%20and%20Access%20Model.md) или [`FleetOwner`](../../Roles%20and%20Access%20Model.md) |
| Покрываемые FR (BRD) | `FR-022` *(read-side continuation of feedback flow)* |
| Покрываемые FR (Additional list) | — |
| Предусловие | Пользователь авторизован; пользователь уже находится в контексте конкретной брони; у пользователя есть право просмотра этой брони |
| Триггер | Открытие блока `Feedback`, вкладки `Отзывы` или ссылки `View feedback` в карточке брони |
| Ожидаемый результат | Пользователь видит список feedback-записей, относящихся к текущей брони, без изменения данных |
| Используемые API | [`GET /bookings/{id}/feedbacks`](../../../api/booking/common/GET_bookings_id_feedbacks.md) |

---

## Основной сценарий

1. Пользователь открывает экран, где уже доступен контекст конкретной брони.
2. Frontend уже имеет `bookingId` выбранной брони.
3. Пользователь открывает блок `Feedback` / `Отзывы`.
4. Frontend вызывает [`GET /bookings/{id}/feedbacks`](../../../api/booking/common/GET_bookings_id_feedbacks.md).
5. Backend проверяет, что:
   - бронь существует;
   - текущий пользователь имеет право просмотра этой брони;
   - `bookingId` относится к допустимому business context-у пользователя.
6. Backend читает `EquipmentFeedbacks` по `bookingId`.
7. Backend подтягивает данные автора feedback из `Users`.
8. Backend сортирует feedback-записи по `createdAt`.
9. Backend возвращает список feedback-записей.
10. Frontend отображает список обратной связи внутри текущего экрана без обязательной навигации на отдельную страницу.

---

## Варианты доступа по ролям

### 1. [`Requestor`](../../Roles%20and%20Access%20Model.md)

- видит feedback-list собственной брони;
- не должен видеть feedback по брони другого пользователя без отдельного access context.

### 2. [`ServiceWorkProcessor`](../../Roles%20and%20Access%20Model.md)

- видит feedback-list брони, относящейся к доступному `Service Work Request`;
- доступ определяется business context-ом JDE-sourced request.

### 3. [`FleetOwner`](../../Roles%20and%20Access%20Model.md)

- видит feedback-list брони, относящейся к fleet-у, где у пользователя есть [`Fleet Management Access`](../../Roles%20and%20Access%20Model.md#%D1%81%D0%BF%D0%B5%D1%86%D0%B8%D0%B0%D0%BB%D1%8C%D0%BD%D1%8B%D0%B5-access-concepts);
- не должен видеть feedback по броням вне зоны ответственности своих fleet-ов.

---

## Альтернативные сценарии

1. По текущей брони еще нет feedback-записей.
   Backend возвращает пустой список; frontend показывает empty state.

2. Пользователь открывает feedback-list из другого UI entry point того же booking context-а.
   Frontend использует тот же `bookingId` и тот же read API без изменения логики.

3. Пользователь не имеет доступа к брони.
   Backend возвращает `FORBIDDEN`.

4. Бронь не найдена.
   Backend возвращает `NOT_FOUND`.

---

## Замечания

1. Список feedback должен быть ограничен только текущей бронью.
2. Пользователь не должен видеть feedback по другим броням той же техники в рамках этого сценария.
3. Read flow должен быть booking-scoped, а не equipment-scoped.
4. UI должен показывать как минимум:
   - данные того, кто оставил feedback;
   - дату и время;
   - текст обратной связи.
5. Один общий use case предпочтительнее отдельных role-specific use cases, потому что бизнес-семантика списка одинакова, а различаются только access rules.
