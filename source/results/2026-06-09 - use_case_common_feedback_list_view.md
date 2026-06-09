**Created:** 2026-06-09 07:36  
**Last updated:** 2026-06-09 07:36  
**Author:** Telman Nurzhanov (SA)

---

# UC-COM-02 - Просмотр списка обратной связи по технике в рамках текущей брони

## 1. Назначение артефакта

Этот use case описывает общий read-only сценарий, в котором пользователь получает список feedback-записей по технике в рамках конкретной брони.

Артефакт подготовлен как аналитический результат в `source/results/` и не публиковался в `wiki/`.

Данный use case является общим для следующих ролей:
- `Requestor`;
- `ServiceWorkProcessor`;
- `FleetOwner`.

---

## 2. Confirmed Facts

Подтверждено по опубликованным артефактам:

1. Существует write API `POST /bookings/{id}/feedback` для создания feedback по технике в рамках конкретной брони.
2. Feedback хранится в сущности `EquipmentFeedbacks`.
3. `EquipmentFeedbacks` уже привязана и к `bookingId`, и к `equipmentId`.
4. Опубликованный API на получение списка feedback по конкретной брони в данный момент отсутствует.
5. В опубликованной модели common read flows уже есть booking-scoped read endpoints, например `GET /bookings/{id}/timeline`.
6. Различие между `Requestor` / `ServiceWorkProcessor` / `FleetOwner` в этом сценарии относится к access control, а не к business semantics списка.

---

## 3. Scope of This Use Case

Данный use case покрывает:
- просмотр списка feedback-записей по технике в рамках выбранной брони;
- доступ к списку из booking detail context;
- отображение автора, времени и текста feedback.

Данный use case не покрывает:
- создание feedback;
- редактирование feedback;
- удаление feedback;
- admin-wide monitoring list по всем feedback-записям системы.

---

## 4. Use Case Card

| Field | Value |
|---|---|
| Use case ID | `UC-COM-02` |
| Name | `Просмотр списка обратной связи по технике в рамках текущей брони` |
| Area | Страница `My Requests`, detail view заявки, detail view брони, Fleet Owner `Approvals` detail / action view |
| Actor | `Requestor`, `ServiceWorkProcessor`, `FleetOwner` |
| Covered FR (BRD) | `FR-022` as read-side continuation of feedback flow |
| Covered FR (Additional) | — |
| Precondition | Пользователь авторизован; пользователь уже находится в контексте конкретной брони; у пользователя есть право просмотра этой брони |
| Trigger | Открытие блока `Feedback`, вкладки `Отзывы` или ссылки `View feedback` в карточке брони |
| Expected result | Пользователь видит список feedback-записей, относящихся к текущей брони, без изменения данных |
| Proposed API | `GET /bookings/{id}/feedbacks` |

---

## 5. Main Scenario

1. Пользователь открывает экран, где уже доступен контекст конкретной брони.
2. Frontend уже имеет `bookingId` выбранной брони.
3. Пользователь открывает блок `Feedback` / `Отзывы`.
4. Frontend вызывает proposed common read API `GET /bookings/{id}/feedbacks`.
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

## 6. Role-Specific Access Variants

### 6.1 Requestor

- видит feedback-list собственной брони;
- не должен видеть feedback по брони другого пользователя без отдельного access context.

### 6.2 ServiceWorkProcessor

- видит feedback-list брони, относящейся к доступному `Service Work Request`;
- доступ определяется business context-ом JDE-sourced request.

### 6.3 FleetOwner

- видит feedback-list брони, относящейся к fleet-у, где у пользователя есть `Fleet Management Access`;
- не должен видеть feedback по броням вне зоны ответственности своих fleet-ов.

---

## 7. Returned Data Requirements

Минимально обязательные поля списка:

| Field | Purpose |
|---|---|
| `id` | Идентификатор feedback-записи |
| `bookingId` | Идентификатор текущей брони |
| `equipmentId` | Идентификатор техники |
| `feedback` | Текст обратной связи |
| `createdAt` | Дата и время создания |
| `createdBy.userId` | Идентификатор автора |
| `createdBy.displayName` | Отображаемое имя автора |

Рекомендуемые дополнительные поля:

| Field | Purpose |
|---|---|
| `createdBy.email` | Уточнение автора в enterprise-контексте |
| `createdBy.role` | Какая роль оставила feedback |
| `createdBy.department` | Организационный контекст при необходимости |

---

## 8. Alternative Scenarios

1. По текущей брони еще нет feedback-записей.
   Backend возвращает пустой список; frontend показывает empty state.

2. Пользователь открывает feedback-list из другого UI entry point того же booking context-а.
   Frontend использует тот же `bookingId` и тот же read API без изменения логики.

3. Пользователь не имеет доступа к брони.
   Backend возвращает `FORBIDDEN`.

4. Бронь не найдена.
   Backend возвращает `NOT_FOUND`.

---

## 9. Business Rules

1. Список feedback должен быть ограничен только текущей бронью.
2. Пользователь не должен видеть feedback по другим броням той же техники в рамках этого сценария.
3. Read flow должен быть booking-scoped, а не equipment-scoped.
4. Отображение feedback должно быть read-only.
5. UI должен показывать как минимум автора, дату/время и текст обратной связи.
6. Feedback-записи должны возвращаться в предсказуемом порядке; рекомендуется `createdAt DESC` для UI списка и `createdAt ASC` для timeline-like представления. Конкретный порядок нужно согласовать отдельно.

---

## 10. API Recommendation

Рекомендуемый единый common read endpoint:

`GET /bookings/{id}/feedbacks`

Почему common endpoint предпочтителен:

- feedback относится к конкретной брони;
- семантика ответа одинакова для `Requestor`, `ServiceWorkProcessor` и `FleetOwner`;
- различается только access control, а не data model;
- это соответствует уже существующему booking-scoped read pattern вроде `GET /bookings/{id}/timeline`.

---

## 11. Open Questions

1. Нужна ли пагинация, или feedback по одной брони всегда достаточно возвращать одним списком?
2. Какой порядок сортировки является основным для UI: новые сверху или старые сверху?
3. Нужно ли возвращать role/department автора прямо в payload или достаточно display name?
4. Должен ли `Requestor` видеть feedback, оставленный `ServiceWorkProcessor`, и наоборот, если они оба имеют доступ к той же брони?
5. Должен ли `FleetOwner` видеть feedback сразу после его создания, даже если бронь уже находится в terminal state?

---

## 12. Summary Recommendation

1. Использовать один общий read-only use case просмотра feedback-list по текущей брони.
2. Моделировать его как booking-scoped common read API.
3. Различия между ролями фиксировать в access variants, а не в отдельных use cases.
4. Обязательные поля ответа: данные автора, `createdAt`, `feedback`.
