# GET /bookings/{id}/feedbacks

**Created:** 2026-06-09  
**Last updated:** 2026-06-09  
**Автор документов:** OpenCode

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить список feedback-записей по технике в рамках конкретной брони |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Common` |
| Endpoint URL | `/api/booking/v1/bookings/{id}/feedbacks` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод.

Используется как read-side продолжение feedback flow для получения списка отзывов по технике в контексте одной брони.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-022 | [`Requestor`](../../../requirements/Roles and Access Model.md) / SWP can submit feedback on equipment with confirmed booking | Confirmed | BRD v13 | Read-side continuation feedback flow |
| TCO Booking Tool | FR-025 | [`Requestor`](../../../requirements/Roles and Access Model.md) can view request details and status | Confirmed | BRD v13 | Feedback list is part of booking details transparency |
| TCO Booking Tool | FR-092 | [`FleetOwner`](../../../requirements/Roles and Access Model.md) can work with request / booking details in approvals context | Confirmed | BRD v13 | FO needs read access to feedback for own fleet bookings |

---

## 3. Описание логики работы метода

1. Проверить существование брони и права доступа текущего пользователя.
2. Выбрать записи из `EquipmentFeedbacks` по `bookingId`.
3. Подтянуть автора feedback из `Users`.
4. При необходимости подтянуть организационный контекст автора из `Departments`.
5. Отсортировать feedback-записи в предсказуемом порядке.
6. Вернуть одну запись на каждый сохраненный feedback.

Сущности:
- читаются: `Bookings`
- читаются: `EquipmentFeedbacks`
- читаются: `Users`
- читаются: `Departments` *(optional for author organization block)*

Правила метода:
- метод является booking-scoped, а не equipment-scoped;
- должны возвращаться только feedback-записи, где `EquipmentFeedbacks.bookingId = {id}`;
- feedback по другим броням той же техники не должен попадать в ответ;
- если feedback по броне нет, метод возвращает пустую коллекцию, а не ошибку.

Рекомендуемый порядок сортировки:
- `createdAt DESC`, затем `id DESC`.

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| [`Requestor`](../../../requirements/Roles and Access Model.md) | Может видеть feedback-list собственной брони |
| [`ServiceWorkProcessor`](../../../requirements/Roles and Access Model.md) | Может видеть feedback-list брони, относящейся к доступному Service Work Request |
| [`FleetOwner`](../../../requirements/Roles and Access Model.md) | Может видеть feedback-list брони своих fleet-ов / fleet-ов, где есть [`Fleet Management Access`](../../../requirements/Roles and Access Model.md#специальные-access-concepts) |
| [`Admin`](../../../requirements/Roles and Access Model.md) | Полный доступ |

---

## 5. Настройки системы, используемые в методе

| Наименование | Код | Тип значения | Описание | Значение по умолчанию |
|---|---|---|---|---|
| — | — | — | Не используются | — |

---

## 6. Ошибки, возвращаемые методом

| Код | Описание ошибки |
|---|---|
| `UNAUTHORIZED` | Пользователь не авторизован |
| `FORBIDDEN` | Нет доступа к списку feedback по этой брони |
| `NOT_FOUND` | Бронь не найдена |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор брони | `id` | `uuid` | `+` | Должен существовать | — | Path param | |

---

## 8. Пример запроса

```http
GET /api/booking/v1/bookings/8c4c8b6d-7bc0-41fb-9038-422cf55d1111/feedbacks
Authorization: Bearer <token>
Content-Type: application/json
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий [`result wrapper`](../../common/Result%20Wrapper.md).

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | value | array<object> | object[] | — | backend aggregation | Коллекция feedback-записей по одной брони |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend | |
| 3 | Ошибки | errors | array<object> | object[] | `[]` | backend | |

### Структура `value[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор записи | id | uuid | UUID v4 | — | EquipmentFeedbacks.id | |
| 2 | Идентификатор брони | bookingId | uuid | UUID v4 | — | EquipmentFeedbacks.bookingId | |
| 3 | Идентификатор техники | equipmentId | uuid | UUID v4 | — | EquipmentFeedbacks.equipmentId | |
| 4 | Текст обратной связи | feedback | string | string | — | EquipmentFeedbacks.feedback | |
| 5 | Дата и время создания | createdAt | datetime | ISO 8601 | — | EquipmentFeedbacks.createdAt | |
| 6 | Данные автора | createdBy | object | object | — | backend composition from EquipmentFeedbacks + Users | |

### Структура `value[].createdBy`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор автора | userId | uuid | UUID v4 | — | EquipmentFeedbacks.createdBy | |
| 2 | Отображаемое имя автора | displayName | string | string | — | Users.fullName / backend fallback | |
| 3 | Email автора | email | string | null | `null` | Users.email | Optional |
| 4 | Роль автора | role | string | null | `null` | backend composition | Например: `Requestor`, `ServiceWorkProcessor` |
| 5 | Подразделение автора | department | string | null | `null` | Users.departmentId -> Departments.name | Optional |

## 10. Пример ответа

```json
{
  "value": [
    {
      "id": "3e7e8f9a-a8e5-48d3-8ca2-a0cce5d40001",
      "bookingId": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
      "equipmentId": "c3b5af91-61f8-4bc0-bd88-d099d3e90001",
      "feedback": "Equipment was delivered on time and worked without issues.",
      "createdAt": "2026-05-22T17:30:00Z",
      "createdBy": {
        "userId": "8f83f79c-3d25-4f07-a17b-9dc4b9f25001",
        "displayName": "John Smith",
        "email": "john.smith@tco.example",
        "role": "Requestor",
        "department": "Maintenance Planning"
      }
    },
    {
      "id": "3e7e8f9a-a8e5-48d3-8ca2-a0cce5d40002",
      "bookingId": "8c4c8b6d-7bc0-41fb-9038-422cf55d1111",
      "equipmentId": "c3b5af91-61f8-4bc0-bd88-d099d3e90001",
      "feedback": "Tracker data matched actual equipment usage during the booked period.",
      "createdAt": "2026-05-22T18:10:00Z",
      "createdBy": {
        "userId": "9d92a3f1-5f8b-4da7-bf12-123456789001",
        "displayName": "Aruzhan Bekenova",
        "email": "aruzhan.bekenova@tco.example",
        "role": "ServiceWorkProcessor",
        "department": "Service Operations"
      }
    }
  ],
  "isSuccess": true,
  "errors": []
}
```

---

## 11. Замечания

1. Основные поля для UI: `createdBy.displayName`, `createdAt`, `feedback`.
2. Дополнительные поля UI может использовать опционально: `createdBy.role`, `createdBy.email`, `createdBy.department`.
3. Если feedback по броне отсутствует, UI должен показывать корректный empty state.
4. На текущем этапе рекомендуется возвращать полный список feedback по одной брони без пагинации; при необходимости later method can evolve to paginated result.
