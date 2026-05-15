**Created:** 2026-05-12  
**Last updated:** 2026-05-12  
**Автор документов:** Telman Nurzhanov (SA)

---

# POST /work-centers

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Создать новый work center в справочнике Admin Panel |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Admin Panel` |
| Endpoint URL | `/api/admin/v1/work-centers` |
| Метод запроса | `POST` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Обеспечивает создание записи work center через форму или inline-create на экране справочника Admin Panel.

---

## 2. Функциональные требования

Общий перечень требований: Требования к системе DMMS

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-013 | Admin создаёт equipment types / equipment-related master data | Confirmed | BRD v13 | Work centers являются связанным справочником для типов техники |
| TCO Booking Tool | FR-014 | Admin редактирует параметры техники и связанные справочные сущности | Confirmed | BRD v13 | Метод нужен для пополнения справочника work centers |
| TCO Booking Tool | FR-NEW-50 | Admin manages all reference/handbook values via Admin Panel | Confirmed | BRD v13 | Work centers управляются через Admin Panel |

---

## 3. Описание логики работы метода

1. Принять тело запроса; провалидировать обязательные поля (`code`, `name`).
2. Проверить уникальность `code` в таблице `WorkCenters` WHERE `isDeleted = false`. Если запись с таким кодом уже существует — вернуть `422 VALIDATION_ERROR`.
3. Проверить поле `name`: должен быть передан объект `{ En, Ru, Kz }`; `name.Ru` обязателен.
4. Проверить уникальность `name` в таблице `WorkCenters` WHERE `isDeleted = false`. Проверка выполняется по правилу локализованной уникальности для набора `name.En`, `name.Ru`, `name.Kz`. Если запись с таким локализованным названием уже существует — вернуть `422 VALIDATION_ERROR`.
5. Создать новую запись в `WorkCenters`; заполнить аудит-поля `createdAt` и `createdBy`.
6. Рассчитать `equipmentTypesCount` = `0`.
7. Вернуть созданный объект в формате общего `result wrapper` с HTTP 201.

Сущности, участвующие в методе:
- читаются: `WorkCenters` (валидация уникальности)
- изменяются: `WorkCenters` (INSERT)
- транзакционность: не требуется

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Admin` | Доступ к административной панели и справочнику work centers |

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
| `FORBIDDEN` | У пользователя нет роли `Admin` |
| `VALIDATION_ERROR` | Поле `code` пустое или уже существует в справочнике |
| `VALIDATION_ERROR` | Поле `name` не передано, `name.Ru` пустое или локализованное имя уже существует в справочнике |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Код work center | `code` | `string` | `+` | Непустая строка; уникальная среди `WorkCenters` WHERE `isDeleted = false` | — | Request body | |
| 2 | Наименование work center | `name` | `object` | `+` | Объект `{ En, Ru, Kz }`; `name.Ru` обязателен; локализованное имя уникально среди `WorkCenters` WHERE `isDeleted = false` | — | Request body | |

---

## 8. Пример запроса

```http
POST /api/admin/v1/work-centers
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "code": "WC-100",
  "name": {
    "En": "Drilling Operations",
    "Ru": "Буровые работы",
    "Kz": "Бұрғылау жұмыстары"
  }
}
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий `result wrapper` (использовать во всех API).  
В обёртке, в значении `value`, возвращается созданный объект типа `WorkCenter`.

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | `value` | `object` | `WorkCenter` | — | `WorkCenters` | Созданная запись |
| 2 | Признак успешности | `isSuccess` | `bool` | boolean | — | backend | |
| 3 | Ошибки | `errors` | `array<object>` | `ApiError[]` | `[]` | backend | При успешном ответе — пустой массив |

### Структура `WorkCenter`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор work center | `id` | `uuid` | UUID v4 | — | `WorkCenters.id` | Генерируется backend |
| 2 | Код work center | `code` | `string` | string | — | `WorkCenters.code` | |
| 3 | Наименование work center | `name` | `object` | `LocalizedName` | — | `WorkCenters.name` | `{ En, Ru, Kz }` |
| 4 | Количество типов техники | `equipmentTypesCount` | `int` | integer | `0` | backend | Для новой записи всегда `0` |

---

## 10. Пример ответа

```json
{
  "value": {
    "id": "12a8b1ce-3aaf-4f55-8ac8-f8cf5d86c222",
    "code": "WC-100",
    "name": {
      "En": "Drilling Operations",
      "Ru": "Буровые работы",
      "Kz": "Бұрғылау жұмыстары"
    },
    "equipmentTypesCount": 0
  },
  "isSuccess": true,
  "errors": []
}
```

---

## Замечания

1. Уникальность `code` и `name` проверяется только среди активных записей (`isDeleted = false`). Для `name` применяется правило локализованной уникальности.
2. После создания запись сразу доступна для выбора в `POST /equipment-types` и `PUT /equipment-types/:id`.
