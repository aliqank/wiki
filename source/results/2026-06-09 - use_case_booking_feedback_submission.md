**Created:** 2026-06-09 07:10  
**Last updated:** 2026-06-09 07:10  
**Author:** Telman Nurzhanov (SA)

---

# UC-REQ-08 - Оставление отзыва по технике в рамках брони

## 1. Назначение артефакта

Этот use case описывает сценарий использования существующего API `POST /bookings/{id}/feedback`.

Артефакт подготовлен как аналитический результат в `source/results/` и не публиковался в `wiki/`.

Сценарий покрывает ситуацию, когда `Requestor` или `ServiceWorkProcessor` оставляет текстовый отзыв по технике, связанной с конкретной бронью:
- когда техника уже находится в работе;
- когда работа фактически завершена и пользователь хочет оставить отзыв рядом с close-сценарием.

---

## 2. Confirmed Facts

Подтверждено по существующим артефактам репозитория:

1. В BRD существует `FR-022`: `Requestor / SWP can submit feedback on equipment with confirmed booking`.
2. В historical BRD wording этот feedback был доступен starting from booking start datetime.
3. Существует опубликованный API-метод `POST /bookings/{id}/feedback`.
4. Логика метода уже зафиксирована:
   - проверить существование брони и доступ;
   - разрешить feedback только для брони, достигшей `Confirmed` и выше;
   - проверить наличие `equipmentId`;
   - создать запись в `EquipmentFeedbacks`.
5. Метод создает отдельную запись в `EquipmentFeedbacks` и не изменяет lifecycle status брони.
6. В опубликованном методе нет полей для close-операции и нет объединения с `POST /bookings/{id}/close`.

---

## 3. Scope of This Use Case

Данный use case покрывает:
- создание текстового feedback по технике в контексте конкретной брони;
- использование existing standalone API `POST /bookings/{id}/feedback`;
- сценарий во время фактического использования техники;
- сценарий после фактического завершения работ как отдельное действие рядом с close-flow.

Данный use case не покрывает:
- manual close брони;
- фиксацию `actualStartDateTime` и `actualEndDateTime`;
- lifecycle transition в `Closed`;
- approval comments;
- редактирование или удаление ранее созданного feedback.

---

## 4. Use Case Card

| Field | Value |
|---|---|
| Use case ID | `UC-REQ-08` |
| Name | `Оставление отзыва по технике в рамках брони` |
| Area | Страница `My Requests` / detail view заявки / detail view конкретной брони |
| Actor | `Requestor`, `ServiceWorkProcessor` |
| Covered FR (BRD) | `FR-022` |
| Covered FR (Additional) | — |
| Precondition | Пользователь авторизован; пользователь имеет доступ к брони; бронь достигла `Confirmed` и выше; у брони есть связанная техника (`equipmentId`) |
| Trigger | Пользователь нажимает кнопку `Leave feedback` / `Оставить отзыв` в карточке брони |
| Expected result | Система сохраняет feedback по технике, привязанный к booking и equipment, и возвращает созданную запись |
| Used API | [`POST /bookings/{id}/feedback`](../wiki/api/booking/requestor/POST_bookings_id_feedback.md) |

---

## 5. Main Scenario

1. Пользователь открывает страницу `My Requests` или detail view заявки.
2. Пользователь переходит в карточку нужной брони.
3. Frontend показывает контекст брони:
   - номер заявки;
   - технику;
   - текущий статус брони;
   - текущий период использования.
4. Если бронь находится в допустимом статусе, система показывает действие `Leave feedback`.
5. Пользователь инициирует это действие.
6. Frontend открывает форму ввода feedback.
7. Пользователь вводит текст отзыва по использованной технике.
8. Frontend вызывает [`POST /bookings/{id}/feedback`](../wiki/api/booking/requestor/POST_bookings_id_feedback.md).
9. Backend проверяет, что:
   - бронь существует;
   - пользователь имеет доступ к брони;
   - бронь достигла `Confirmed` и выше;
   - у брони есть `equipmentId`;
   - текст feedback непустой.
10. Backend создает запись в `EquipmentFeedbacks`.
11. Backend возвращает созданный feedback с `id`, `bookingId`, `equipmentId`, `feedback`, `createdAt`.
12. Frontend показывает пользователю успешный результат сохранения.

---

## 6. Alternative Scenario: Feedback During InProgress

1. Бронь уже находится в статусе `InProgress`.
2. Пользователь открывает карточку активной брони во время реального использования техники.
3. Пользователь оставляет feedback через тот же action `Leave feedback`.
4. Backend принимает запрос, потому что бронь уже находится в разрешенном статусе `Confirmed` и выше.
5. Feedback сохраняется без изменения статуса брони.

Это и есть основной сценарий, который покрывает вашу уточненную потребность: feedback доступен не только после завершения, но и когда оборудование уже находится в работе.

---

## 7. Alternative Scenario: Feedback Near Close Flow

1. Пользователь завершил фактические работы и хочет закрыть бронь.
2. Пользователь оставляет feedback по технике как отдельное действие до или после close-flow.
3. Frontend вызывает `POST /bookings/{id}/feedback` отдельно от `POST /bookings/{id}/close`.
4. Backend сохраняет feedback независимо от close-операции.

Важное замечание:
- по текущей repository-documented API model feedback не является частью request body метода `POST /bookings/{id}/close`;
- поэтому feedback-at-completion нужно трактовать как отдельный соседний UI/action flow, а не как встроенное поле close-метода.

---

## 8. Alternative Scenarios and Errors

1. Пользователь не авторизован.
   Backend возвращает `UNAUTHORIZED`.

2. Пользователь не имеет доступа к брони.
   Backend возвращает `FORBIDDEN`.

3. Бронь не найдена.
   Backend возвращает `NOT_FOUND`.

4. Статус брони пока не позволяет оставить feedback.
   Backend возвращает `BOOKING_FEEDBACK_NOT_ALLOWED`.

5. У брони отсутствует связанная техника.
   Backend возвращает ошибку бизнес-валидации / conflict according to backend implementation.

6. Текст feedback пустой.
   Backend возвращает `VALIDATION_ERROR`.

---

## 9. Business Rules

1. Feedback относится к технике, но создается в контексте конкретной брони.
2. Один feedback должен быть связан и с `bookingId`, и с `equipmentId`.
3. Feedback не меняет booking lifecycle.
4. Feedback доступен только после достижения бронью статуса `Confirmed` и выше.
5. Сценарий должен быть доступен во время `InProgress`.
6. Feedback при завершении работ выполняется как отдельное действие, а не как часть close API transaction.
7. Пустой текст feedback запрещен.
8. Если бизнес позже захочет timeline-style execution notes, это уже другой сценарий, не равный equipment feedback.

---

## 10. Data Model Note

Сценарий опирается на уже существующую сущность `EquipmentFeedbacks`.

Минимально используемые поля по текущей API-модели:

| Field | Meaning |
|---|---|
| `id` | Идентификатор записи feedback |
| `bookingId` | Идентификатор брони |
| `equipmentId` | Идентификатор техники |
| `feedback` | Текст отзыва |
| `createdAt` | Дата и время создания |

Практический смысл модели:
- feedback хранится не как часть статуса брони;
- feedback хранится как отдельная запись, пригодная для аналитики и monitoring/reporting views.

---

## 11. UI / UX Implications

1. Кнопка `Leave feedback` должна отображаться только если бронь находится в допустимом состоянии.
2. UI должен clearly separate:
   - `Close booking`;
   - `Leave feedback`.
3. Если пользователь завершает работы и хочет сделать оба действия, UI может предложить их рядом, но как два отдельных backend calls.
4. Для активной брони `InProgress` feedback должен быть доступен без требования сначала закрыть бронь.

---

## 12. Open Questions

1. Разрешен ли feedback только один раз на бронь, или несколько записей на одну бронь допустимы?
2. Должен ли уже созданный feedback отображаться в detail view самой брони?
3. Разрешен ли feedback после перевода брони в `Closed`, или только до close?
4. Нужен ли отдельный `GET /bookings/{id}/feedbacks`, если UI должен показывать историю feedback-записей?
5. Нужны ли ограничения по длине текста feedback beyond non-empty validation?

---

## 13. Summary Recommendation

1. Для описанной потребности использовать существующий `POST /bookings/{id}/feedback`.
2. Считать этот flow сценарием equipment feedback, а не отдельным booking comment flow.
3. Явно фиксировать в use case, что feedback доступен во время `InProgress`.
4. Если feedback нужен при завершении работ, моделировать это как отдельный вызов рядом с close action, а не как часть `POST /bookings/{id}/close`.
