**Created:** 2026-05-12  
**Last updated:** 2026-05-12  
**Автор документов:** Telman Nurzhanov (SA)

---

# PUT /work-centers/:id

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Сохранить изменения work center в справочнике Admin Panel |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Admin Panel` |
| Endpoint URL | `/api/admin/v1/work-centers/:id` |
| Метод запроса | `PUT` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Обрабатывает редактирование строки work center в справочнике Admin Panel.

---

## 2. Функциональные требования

Общий перечень требований: Требования к системе DMMS

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-013 | Admin создаёт equipment types / equipment-related master data | Confirmed | BRD v13 | Work centers являются связанным справочником для типов техники |
| TCO Booking Tool | FR-014 | Admin редактирует параметры техники и связанные справочные сущности | Confirmed | BRD v13 | Метод нужен для редактирования справочника work centers |
| TCO Booking Tool | FR-NEW-50 | Admin manages all reference/handbook values via Admin Panel | Confirmed | BRD v13 | Work centers управляются через Admin Panel |

---

## 3. Описание логики работы метода

1. Найти запись в `WorkCenters` WHERE `id` = `:id` AND `isDeleted = false`. Если запись не найдена — вернуть `404 NOT_FOUND`.
2. Провалидировать поле `code`: непустая строка; уникальная в `WorkCenters` WHERE `id` ≠ `:id` AND `isDeleted = false`.
3. Провалидировать поле `name`: должен быть передан объект `{ En, Ru, Kz }`; `name.Ru` обязателен; локализованное имя уникально в `WorkCenters` WHERE `id` ≠ `:id` AND `isDeleted = false`.
4. Обновить запись в `WorkCenters`; заполнить аудит-поля `updatedAt` и `updatedBy`.
5. Рассчитать `equipmentTypesCount` = COUNT(`EquipmentTypes` WHERE `workCenterId` = `:id` AND `isDeleted = false`).
6. Вернуть обновлённый объект `WorkCenter` в формате общего `result wrapper` с HTTP 200.

Сущности, участвующие в методе:
- читаются: `WorkCenters`, `EquipmentTypes` (агрегат)
- изменяются: `WorkCenters` (UPDATE)
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
| `NOT_FOUND` | Work center с указанным `id` не найден или помечен как удалённый |
| `VALIDATION_ERROR` | Поле `code` пустое или уже существует в справочнике |
| `VALIDATION_ERROR` | Поле `name` не передано, `name.Ru` пустое или локализованное имя уже существует в справочнике |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор work center | `id` | `uuid` | `+` | Должен быть валидным UUID v4 | — | Path param | |
| 2 | Код work center | `code` | `string` | `+` | Непустая строка; уникальная среди `WorkCenters` WHERE `id` ≠ `:id` AND `isDeleted = false` | — | Request body | |
| 3 | Наименование work center | `name` | `object` | `+` | Объект `{ En, Ru, Kz }`; `name.Ru` обязателен; локализованное имя уникально среди `WorkCenters` WHERE `id` ≠ `:id` AND `isDeleted = false` | — | Request body | |

---

## 8. Пример запроса

```http
PUT /api/admin/v1/work-centers/12a8b1ce-3aaf-4f55-8ac8-f8cf5d86c222
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "code": "WC-100",
  "name": {
    "En": "Drilling and Wells",
    "Ru": "Бурение и скважины",
    "Kz": "Бұрғылау және ұңғымалар"
  }
}
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий `result wrapper` (использовать во всех API).  
В обёртке, в значении `value`, возвращается обновлённый объект типа `WorkCenter`.

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | `value` | `object` | `WorkCenter` | — | `WorkCenters` + агрегат | Обновлённая запись |
| 2 | Признак успешности | `isSuccess` | `bool` | boolean | — | backend | |
| 3 | Ошибки | `errors` | `array<object>` | `ApiError[]` | `[]` | backend | При успешном ответе — пустой массив |

### Структура `WorkCenter`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор work center | `id` | `uuid` | UUID v4 | — | `WorkCenters.id` | |
| 2 | Код work center | `code` | `string` | string | — | `WorkCenters.code` | |
| 3 | Наименование work center | `name` | `object` | `LocalizedName` | — | `WorkCenters.name` | `{ En, Ru, Kz }` |
| 4 | Количество типов техники | `equipmentTypesCount` | `int` | integer | `0` | COUNT(`EquipmentTypes`) | Нужен для guard удаления |

---

## 10. Пример ответа

```json
{
  "value": {
    "id": "12a8b1ce-3aaf-4f55-8ac8-f8cf5d86c222",
    "code": "WC-100",
    "name": {
      "En": "Drilling and Wells",
      "Ru": "Бурение и скважины",
      "Kz": "Бұрғылау және ұңғымалар"
    },
    "equipmentTypesCount": 6
  },
  "isSuccess": true,
  "errors": []
}
```

---

## Замечания

1. Уникальность `code` и `name` при обновлении проверяется с исключением самой редактируемой записи (`id ≠ :id`). Для `name` применяется правило локализованной уникальности.
2. Изменение `code` или `name` не требует обновления `EquipmentTypes`, так как тип техники хранит только `workCenterId`.
